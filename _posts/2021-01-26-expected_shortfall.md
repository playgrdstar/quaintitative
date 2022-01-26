---
layout: default
title:  Measuring Expected Shortfall in Python
description: Measuring Expected Shortfall in Python
date:   2021-01-26 00:00:01 +0000
permalink: /expected_shortfall/
category: Finance
---
## Expected Shortfall in Python

Google VAR and you will find lots of criticisms on VAR as a measure of market risk. And you will inevitably see Expected Shortfall (ES) being put forward as an alternative. 

What is the difference between the two?

> Say we are trying to assess our VAR (or to put it simply, potential losses) at a confidence level of 99%, that means we will have a range of loss outcomes (or scenarios) in the 1% tail, and -

- VAR answers this question - What is the minimum loss over the whole range of outcomes in the 1% tail?
- ES answers this question - What is the average loss over the whole range of outcomes in the 1% tail?

Let’s try to compute the two measures in Python to see the difference.
First, VAR.
```
h = 10. # horizon of 10 days
mu_h = 0.1 # this is the mean of % returns over 10 days - 10%
sig = 0.3 # this is the vol of returns over a year - 30%
sig_h = 0.3 * np.sqrt(h/252) # this is the vol over the horizon
alpha = 0.01

VaR_n = norm.ppf(1-alpha)*sig_h - mu_h 

print("99% VaR is", round(VaR_n*100,2)) 

Out:
99% VaR is 3.9
```

And next for ES.
```
# with the same parameters as above
CVaR_n = alpha**-1 * norm.pdf(norm.ppf(alpha))*sig_h - mu_h

print("99% CVaR/ES is", round(CVaR_n*100,2))


Out:
99% CVaR/ES is 5.93
```

We don’t have to assume a normal distribution. We can also assume a t-distribution. 
```
from scipy.stats import t
nu = 5 # degree of freedom, the larger, the closer to normal distribution
xanu = t.ppf(alpha, nu)

VaR_t = np.sqrt(h/252 * (nu-2)/nu) * t.ppf(1-alpha, nu)*sig - mu_h

print("99% VaR (Student-t with v=5) is", round(VaR_t*100,2))

Out:
99% VaR (Student-t with v=5) is 5.58

CVaR_t = -1/alpha * (1-nu)**(-1) * (nu-2+xanu**2) * t.pdf(xanu, nu)*sig_h - mu_h
print("99% CVaR (Student-t with v=5) is", round(CVaR_t*100,2))

Out:
99% CVaR (Student-t with v=5) is 13.35
```

The larger the degree of freedom, the closer to a normal distribution.
```
# to verify that the normal and Student-t VAR will be the same for big v
nu = 10000000 # degree of freedom, the larger, the closer to normal distribution
xanu = t.ppf(alpha, nu)

VaR_t = np.sqrt(h/252 * (nu-2)/nu) * t.ppf(1-alpha, nu)*sig - mu_h
print("99% VaR (Student-t with with v->infinity) is", round(VaR_t*100,2))

Out:
99% VaR (Student-t with with v->infinity) is 3.9

CVaR_t = -1/alpha * (1-nu)**(-1) * (nu-2+xanu**2) * t.pdf(xanu, nu)*sig_h - mu_h
print("99% CVaR (Student-t with with v->infinity) is", round(CVaR_t*100,2))

Out:
99% CVaR (Student-t with with v->infinity) is 5.93
```

We can compute something similar with actual market data. First we fit the data  to normal and t-distributions.
```
mu_norm, sig_norm = norm.fit(returns) 

nu, mu_t, sig_t = t.fit(returns)
```

And the respective VAR and ES can be computed quite easily.
```
h = 1
alpha = 0.01
xanu = t.ppf(alpha, nu)

CVaR_n = alpha**-1 * norm.pdf(norm.ppf(alpha))*sig_norm - mu_norm
VaR_n = norm.ppf(1-alpha)*sig_norm - mu_norm
    
VaR_t = np.sqrt((nu-2)/nu) * t.ppf(1-alpha, nu)*sig_norm  - h*mu_norm
CVaR_t = -1/alpha * (1-nu)**(-1) * (nu-2+xanu**2) * t.pdf(xanu, nu)*sig_norm  - h*mu_norm
```

The full code can be found at the notebook [here][1].

[1]:	https://github.com/playgrdstar/expected_shortfall/blob/master/Expected%20Shortfall.ipynb