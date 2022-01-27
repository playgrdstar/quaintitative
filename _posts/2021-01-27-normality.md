---
layout: default
title:  How normal are you? Checking distributional assumptions.
description: How normal are you? Checking distributional assumptions.
date:   2021-01-27 00:00:00 +0000
permalink: /normality_distribution_test/
category: Finance
---
## How normal are you? Checking distributional assumptions.

The need to understand the underlying distribution of data is critical in most parts of quantitative finance. Statistical tests can be applied for this purpose. 

The assumption of a normal distribution is commonplace, so let's start with simple tests to check whether the data we are analysing follows a normal distribution.
```
import numpy as np
from scipy import stats

# Create a normal distribution
np.random.seed(42)
mean = 1.2
std = 0.3
n = 10000

returns = np.random.normal(mean, std, n)

# And check it!

# Shapiro-Wilk test for normality
# See here for more details https://docs.scipy.org/doc/scipy-0.19.1/reference/generated/scipy.stats.shapiro.html

print('Test stat: ', stats.shapiro(returns)[0], 
		'p value: ', stats.shapiro(returns)[1])

Out:
Test stat:  0.9999347925186157 
p value:  0.9987024664878845
```

The confidence level for the Shapiro-Wilk test is 95% and the null hypothesis is that the returns data follow a normal distribution. Since the p-value \>\> 0.05 (i.e. 5%), we accept the hypothesis that the returns follow a normal distribution.

Now, let’s look at an exponential distribution.
```
returns_exp = np.random.exponential(0.2, n)
```

And contrast the results of the Shapiro test.
```
print('Test stat: ', stats.shapiro(returns_exp)[0], 
	      'p value: ', stats.shapiro(returns_exp)[1])

Out:
Test stat:  0.8156617879867554 
p value:  0.0
```

In this case, the p-value is \< 0.05, and so there is enough evidence to reject the null hypothesis of the distribution being normal.

Now let’s do some tests with real market data.
```
import quandl
import datetime
quandl.ApiConfig.api_key = ""

end = datetime.datetime.now()
start = end - datetime.timedelta(365*5)

AAPL = quandl.get('EOD/AAPL', start_date=start, end_date=end)

AAPLreturns = (AAPL['Close']/AAPL['Close'].shift(1))-1

AAPL_array = np.array(AAPL['Close'])
print('AAPL Prices\nTest stat: ', stats.shapiro(AAPL_array)[0], 
		'\np value: ', stats.shapiro(AAPL_array)[1])

Out:
AAPL Prices
Test stat:  0.6068973541259766 
p value:  0.0

AAPLreturns_array = np.array(AAPLreturns.dropna())
print('AAPL Returns\nTest stat: ', stats.shapiro(AAPLreturns_array)[0], 
		'\np-value: ', stats.shapiro(AAPLreturns_array)[1])

Out:
AAPL Returns
Test stat:  0.3324963450431824 
p value:  0.0
```

Looks like neither are normal since the p-value is \< 5%

The Anderson Darling test is another way to check whether a distribution is normal.
```
print('Anderson Darling test')
print('Stats for normal distribution\n', stats.anderson(returns))
print('\n')
print('Stats for exp distribution\n', stats.anderson(returns_exp))
print('\n')
print('Stats for AAPL Returns\'s distribution\n', stats.anderson(AAPLreturns_array))

Out:
Anderson Darling test
Stats for normal distribution
	AndersonResult(statistic=0.093276307066844311, critical_values=array([ 0.576,  0.656,  0.787,  0.918,  1.092]), significance_level=array([ 15. ,  10. ,   5. ,   2.5,   1. ]))


Stats for exp distribution
	AndersonResult(statistic=461.96932223869953, critical_values=array([ 0.576,  0.656,  0.787,  0.918,  1.092]), significance_level=array([ 15. ,  10. ,   5. ,   2.5,   1. ]))


Stats for AAPL Returns's distribution
	AndersonResult(statistic=127.40971068941758, critical_values=array([ 0.574,  0.654,  0.785,  0.915,  1.089]), significance_level=array([ 15. ,  10. ,   5. ,   2.5,   1. ]))
```

Here, we have three sets of values: the Anderson-Darling test statistic, a set of critical values, and a set of corresponding confidence levels, such as 15 percent, 10 percent, 5 percent, 2.5 percent, and 1 percent, as shown in the previous output.

For the normal distribution, if we choose a 1 percent confidence level—the last value of the critical values, we see 1.088. The test statistic of 0.34 is \< 1.088, hence the null hypothesis of a normal distribution cannot be rejected.

The converse is true for the exponential distribution and the distribution of AAPL returns, which mean the null hypothesis of normality can be rejected.

The stats.anderson package can test for a whole range of other distributions. Take a look at the documentation with the help function.
```
help(stats.anderson)
```

Computing the 4 moments of a distribution (formulas are in the notebook link at the end of this post) is another way to understand more about the underlying shape of the distribution.

The 4 moments are easily computed with numpy functions.
```
print('\n Mean: ', np.mean(returns), '\n',
		'Standard Deviation: ', np.std(returns), '\n',
		'Skew: ', stats.skew(returns), '\n',
		'Kurtosis', stats.kurtosis(returns))

Out:
Mean:  1.19935920499 
Standard Deviation:  0.301023661839 
Skew:  0.0019636977663557873 
Kurtosis 0.026479272360443673
```

For a normal distribution, skewness is around 0.03, and kurtosis around 3. In this case, the statistic being computed is excess kurtosis, so the value is close to 0 instead.
```
print ('Moments for AAPL Returns')
print('\n Mean: ', np.mean(AAPLreturns_array), '\n',
		'Standard Deviation: ', np.std(AAPLreturns_array), '\n',
		'Skew: ', stats.skew(AAPLreturns_array), '\n',
		'Kurtosis', stats.kurtosis(AAPLreturns_array))

Out:
Moments for AAPL Returns

Mean:  0.000328091673061 
Standard Deviation:  0.0279855695958 
Skew:  -22.675005004193835 
Kurtosis 691.1559815602365

```

Obviously not normal!

We can also use t-tests.
```
# T tests - test statistic follows a student's t distribution if the null hypothesis is supported
# Recall that the mean of our rets was 1.2
# Let's apply t and p tests on different means

print('\n Mean of 0.2\n', stats.ttest_1samp(returns, 0.2))

Out:
Mean of 0.2
Ttest_1sampResult(statistic=331.9703274071677, pvalue=0.0)
```

We reject the null hypothesis since the T-value is huge and the P-value is 0.
```
print('\n Mean of 1.205\n', stats.ttest_1samp(returns, 1.205))

Out:
Mean of 1.205
Ttest_1sampResult(statistic=-1.8737772736094489, pvalue=0.060990271728498281)
```

More tests (checking if the variances of two distributions are statistically similar; testing for seasonality) can be found at the notebook at this [link][1].

[1]:	https://github.com/playgrdstar/distributional_tests/blob/master/Testing%20Distributions.ipynb