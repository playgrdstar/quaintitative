---
layout: default
title:  Measuring Market Risk in Python
description: Measuring Market Risk in Python
date:   2021-01-19 00:00:00 +0000
permalink: /market_risk/
category: Finance
---
## Measuring Market Risk in Python

VAR is a common term that one would usually come across in finance when it comes to the measurement of market risks.

VAR, or Value At Risk, is basically a measure of the potential losses that one could face, at a specific level of confidence - e.g. 99%. 

But before we get into VAR, we first need to discuss what value we are assessing risk against. What we want to measure would be the change in market prices over a time period (e.g. day to day). So what VAR would then tell us then would be how much we could lose (or gain) due to the change in prices. It's quite common to use lognormal instead of normal returns when computing the change in prices.

Useful links which provide more information on the difference between the two -

- https://quantivity.wordpress.com/2011/02/21/why-log-returns/
- http://www.insight-things.com/log-normal-distribution-mistaken

Essentially, a few points to note -
- We assume prices are lognormal, then the log returns are normally distributed.
- When returns are small, it is hard to tell the difference between a lognormal and normal distribution
- Lognormal allows us, when compounding, to simply add returns (rather than multiplying). The sum of log returns then simply becomes the difference in the log of final and initial price.

Computing relative or lognormal returns is fairly straight forward with the shift function in pandas. 

**Relative returns**
```
FX_Returns = (FX_DF/FX_DF.shift(1))-1
```

**Lognormal returns**
```
FX_DF_LogReturns = np.log(FX_DF/FX_DF.shift(1))
```

Now for **Value at Risk**.

There's nothing very complicated about Value at Risk (VAR). To put it simply, it's simply a single metric that shows the potential losses of a portfolio etc (at different confidence levels). There are two main methods to compute VAR -

- Parametric
- Historical

Very often, the parametric VAR is based on a normal distribution. Plotting a normal distribution and the VAR on a chart will give us a good overview of how this works.
```
# We use z to define how many standard deviations away from the mean
# Here we use z = -2.33 to get to a 99% confidence interval. Why 99% will be obvious once we plot out the distribution
z = -2.33

plt.figure(figsize=(12,8))
# plotting the normal distribution, using the scipy stats norm function
plt.ylim(0, 0.5)
x = np.arange(-5,5,0.1)
y1 = stats.norm.pdf(x)
plt.plot(x, y1)

x2 = np.arange(-5, z, 0.01) # we use this range from the -ve end to -2.33 to compute the area
sum = 0
# s = np.arange(-10,z,0.01)
for i in x2:
    sum+=stats.norm.pdf(i)*0.01 # computing area under graph from -5 to -2.33 in steps of 0.01

plt.annotate('Area is ' + str(round(sum,3)), xy = (z-1.3, 0.05), fontsize=12)
plt.annotate('z=' + str(z), xy=(z, 0.01), fontsize=12)
plt.fill_between(x2, stats.norm.pdf(x2))
plt.show()
```

Computing parametric VAR is fairly straightforward.
```
confidence = 0.99
Z = stats.norm.ppf(1-confidence)
mean = np.mean(EQ_Returns)
stddev = np.std(EQ_Returns)

Param_VAR = latest_price*Z*stddev
```

And so is computing historical VAR.
```
Hist_VAR = latest_price*np.percentile(EQ_Returns, 1))
```

I've simplified some of the code in this post for ease of understanding, but the full code can be found at the notebook [here][1].

[1]:	https://github.com/playgrdstar/measure_marketrisk_VAR/blob/master/Introduction%20-%20Measuring%20Market%20Risk%20in%20Python.ipynb