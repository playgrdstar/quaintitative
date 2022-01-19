---
layout: default
title:  Setting Up a Data Lab Environment - Part 5
description: Setting Up a Data Lab Environment - Part 5
date:   2021-01-18 00:00:00 +0000
permalink: /docker_datalab_partfive/
category: Data Engineering
---
## Setting Up a Data Lab Environment - Part 5 - Databases

When we do stuff in Jupyter notebooks, we could save the output in a range of local files, from CSV, to JSON, to HDF5. But there might be instances where it might make sense to save things to a database, be it a SQL database like Postgres, or a NoSQL database like MongoDB.

Usually when we set-up a database on the server, we would have to have it running at some host and port, and then make a connection to the database.

So for MongoDB, we would usually first install and run MongoDB, then install a library like pymongo, and then make a connection to the host and port of the MongoDB instance.

With Docker, it’s similar, but slightly easier. Having a database setup within Docker is as easy as pulling an image. Continuing from where we left off on the _docker-compose.yml_ file in the previous [part][1] of this series, we just have to add a few lines right at the end to have access to MongoDB and Postgres databases within the Docker container.
```
version: '3'
services:
    jupyterone:
    build: docker/jupyter
    ports:
        - "8888:8888"
    volumes:
        - .:/home/jovyan/work
    env_file:
        - config/jupyter.env

# The lines below are the new ones!
    this_mongo:
    image: mongo
    volumes:
        - mongo_data:/data/db
    this_postgres:
    image: postgres
    volumes:
        - postgres_data:/var/lib/postgresql/data
volumes:
    postgres_data:
    mongo_data:
```

And connecting to it within the Docker container is super easy. Here’s one to make MongoDB play nice with Pandas. We just have to connect it to _this\_mongo_, which is what we named it in the _docker-compose.yml_ file.
```
from pymongo import MongoClient

def get_mongo_database(db_name, host='this_mongo'):
    conn = MongoClient('this_mongo')
    return conn[db_name]

def dataframe_to_mongo(df, db_name, collection):
    db = get_mongo_database(db_name)
    # 'records' means that it will be saved as an array of objects
    entry = df.to_dict('records')
    db[collection].insert(entry)

def mongo_to_dataframe(db_name, collection, query={}):
    db = get_mongo_database(db_name)
    cursor = db[collection].find(query)
    df =  pd.DataFrame(list(cursor))
    # Remove the mongo id
    if no_id: 
        del df['_id']

    return df
```
And here’s how to connect to the Postgres database.
```
import psycopg2 as pg2
import psycopg2.extras as pgex

conn = pg2.connect(host='this_postgres', 
                    user='postgres',
                    database='postgres')

cur = conn.cursor(cursor_factory=pgex.RealDictCursor)

cur.execute("""
BEGIN;
CREATE TABLE jupyter_test(
    _id INTEGER,
    name TEXT,
    list DOUBLE PRECISION[],
    vector BYTEA
);
COMMIT;
""")

cur.execute("""
BEGIN;
INSERT INTO jupyter_test VALUES(1, 'Test1', '{1,2,3,4,5}');
INSERT INTO jupyter_test VALUES(2, 'Test2', '{2,4,6,8,10}');
COMMIT;
""")

cur.execute("""
SELECT * FROM jupyter_test;
""")

result_raw = cur.fetchall()

print(result_raw[0]['name'])
```

Pretty simple right?

[1]:	https://medium.com/quaintitative/setting-up-a-data-lab-environment-part-4-dockerfile-cd1e3825530b
