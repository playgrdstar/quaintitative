---
layout: default
title:  Extracting and Processing GDELT GKG datasets from BigQuery
description: Extracting and Processing GDELT GKG datasets from BigQuery
date:   2022-01-24 00:00:00 +0000
permalink: /extract_gdelt_gkg_bigquery/
category: Data Engineering
---
## Extracting and Processing GDELT GKG datasets from BigQuery

Pre-requisites:
- This article assumes that you know how to set up a BigQuery account, and use the cloud console to add and query public datasets. If not, you can read more on how to do so [here][1]. 
- This article also assumes that you know to get the json key file so that you can access the API. If not, you can read more on how to do so [here][2]. 

### Extract GDELT GKG data from BigQuery ###
There are many ways to access the rich data available in the [GDELT Project][3]. One of the fastest ways is to leverage on the data that they have uploaded and made available on BigQuery. In this article, I will share two notebooks that you can use to download GDELT Global Knowledge Graph (GKG) data from BigQuery. 

Other than the usual libraries, you will also need to install the following library.

```
pip install --upgrade google-cloud-bigquery
```


The first notebook is [here][4]. 

We first set the locations from which we access the Google Cloud json key file, and save the data downloaded.

```
home_dir = Path(os.getcwd())
key_dir = #<location of your Google Cloud json key>
data_dir = home_dir/'data'
```

We then use the credentials in the key file.
```
os.environ['GOOGLE_APPLICATION_CREDENTIALS'] = str(key_dir/'gdelt-key.json')
```


We then generate the list of dates to download data from.
```
start_ = '2017-12-03'
end_ = '2020-12-31'
final_start_time = datetime.strptime(start_, '%Y-%m-%d')
final_end_time = datetime.strptime(end_, '%Y-%m-%d') 
num_days = (final_end_time - final_start_time).days
time_list = []
start_time_ = datetime.strptime(start_, '%Y-%m-%d')
for d in range(0,num_days,1):
    end_time_ = start_time_ + rd.relativedelta(days=+1)
    time_list.append((start_time_.date(), end_time_.date()))
    start_time_ = start_time_ + rd.relativedelta(days=+1)
```

Next, set up the connection. The authentication is handled automatically as you already have the credentials in the environment.
```
bqclient = bigquery.Client()
```

And then we just need to craft the SQL query and send it on its merry way to Google Cloud, and then save the data that is returned.
```
for d in range(len(time_list)):
    start_date, end_date = time_list[d]
    print(f'Downloading for {start_date}...')
    query_string = f"""
        SELECT DATE, SourceCollectionIdentifier, SourceCommonName, DocumentIdentifier, V2Themes, V2Locations, V2Persons, V2Organizations, V2Tone, SharingImage, RelatedImages, Quotations, AllNames, Amounts
        FROM `gdelt-bq.gdeltv2.gkg_partitioned` 
        WHERE DATE(_PARTITIONTIME) >= "{start_date}" and DATE(_PARTITIONTIME) < "{end_date}"
        LIMIT 5000000
    """
    # print(start_date, end_date)
    # # Note that each day gives around 300-400k rows

    dataframe = (
        bqclient.query(query_string)
        .result()
        .to_dataframe(
            create_bqstorage_client=False,
        )
    )

    dataframe['date'] = pd.to_datetime(dataframe.DATE, format='%Y%m%d%H%M%S')
    print(f'On {start_date}, there are {len(dataframe)} rows from {dataframe["date"].min()} to {dataframe["date"].max()}')

    filename = str(start_date) + '.csv'
    dataframe.to_csv(data_dir/filename, index=False)
```

A few points to note:
- For the fields to download, i.e. `SELECT DATE, SourceCollectionIdentifier, SourceCommonName, DocumentIdentifier, V2Themes, V2Locations, V2Persons, V2Organizations, V2Tone, SharingImage, RelatedImages, Quotations, AllNames, Amounts`, the easiest way to check what you need is to go to the Google CLoud console and examine the fields in the specific table you need.
- We use the table `gdelt-bq.gdeltv2.gkg_partitioned` in our query. There are other tables but we use the partitioned version as they allow for cheaper queries to be run. Google charges you based on the size of the query. So by choosing a partitioned table (based on date here), we can minimize the size of our query and hence our costs.


Refer to the `sample.csv` file in [here][5] to see what can be retrieved.

### Process GDELT GKG data ###

Now that we have our dataset, what can we do with it. Remember that these are just rows and rows of attributes of articles, such as locations, persons, organizations that appeared in an article on that day, themes of the articles, links to the articles, tone of the articles etc. So we could use these as tabular data for say sentiment analysis. 

One thing we may need to do first is to tag each of these rows though. Say for example, we may need to tag it by the name of a company. But there are many ways in which the name of a company may appear. One easy and quick way is to try to do a fuzzy matching of the company names.

We first install the swifter library (to help with multi-processing), and the rapidfuzz library (for fuzzy matching with Levenstein distance).
```
pip install swifter
pip install rapidfuzz
```

The second notebook is [here][6]. 

I have provided a master list of companies with their ticker symbols and common names, that can be used to create two lists.
```
master_ticker_pairs = pd.read_csv('master_ticker_name.csv')
master_ticker_pairs['COMNAM'] = master_ticker_pairs['COMNAM'].str.lower()
master_ticker_pairs['combined'] = master_ticker_pairs['COMNAM'].str.replace(' ', '')
master_ticker_pairs = master_ticker_pairs.fillna('')
master_ticker_pairs['combined'] = master_ticker_pairs['combined'].apply(replace_short)

namelist = list(master_ticker_pairs.combined)
tickerlist = list(master_ticker_pairs.TICKER)
```

Next, we define a function to process the dataset by 'fuzzy' matching names in the main `V2Organizations` field against the `namelist` and `tickerlist`. The threshold refers to the confidence level, and can be found by trial and error.

```
threshold = 75
def map_to_comnam(row):
    mapped = []
    for name in row:
        ticker_result, ticker_score, _ = process.extractOne(name, tickerlist, scorer=fuzz.QRatio)      
        if ticker_score > threshold:
            found_ticker = master_ticker_pairs[master_ticker_pairs.TICKER == ticker_result].TICKER.values[0]
            if (found_ticker not in mapped):
                mapped.append(found_ticker)

        name_result, name_score, _ = process.extractOne(name, namelist, scorer=fuzz.QRatio)
        if name_score > threshold:
            found_ticker = master_ticker_pairs[master_ticker_pairs.combined == name_result].TICKER.values[0]
            if (found_ticker not in mapped):
                mapped.append(found_ticker)
    if len(mapped) == 0:
        return None
    else:
        return mapped
```

Now, we use a sample of the data downloaded from BigQuery (based on the first part of this article), and then apply this function.
```
gkg_df = pd.read_csv('sample.csv')
gkg_df_ = gkg_df[~gkg_df.V2Organizations.isnull()].reset_index(drop=True)\

print(f'Length before removing nulls: {len(gkg_df)}, after removing nulls: {len(gkg_df_)}')

gkg_df_['orgs'] = gkg_df_.V2Organizations.apply(extract_orgs)
gkg_df_['tags'] = gkg_df_.loc[:,'orgs'].swifter.progress_bar(False).apply(map_to_comnam)

filtered_gkg = gkg_df_[~gkg_df_.tags.isnull()]
print(f'Number of articles with tags: {len(filtered_gkg)}')
```

The code can be found in this [repository][7].

And that's it. Happy data explorations!


[1]:	https://cloud.google.com/bigquery/docs/quickstarts/quickstart-cloud-console
[2]:	https://cloud.google.com/bigquery/docs/quickstarts/quickstart-client-libraries 
[3]:    https://www.gdeltproject.org/
[4]:    https://github.com/playgrdstar/gdelt_gkg/blob/main/get_gkg.ipynb
[5]:    https://github.com/playgrdstar/gdelt_gkg/blob/main/sample.csv
[6]:    https://github.com/playgrdstar/gdelt_gkg/blob/main/process_gkg.ipynb
[7]:    https://github.com/playgrdstar/gdelt_gkg
