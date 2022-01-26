---
layout: default
title:  Monte Carlo Simulation of Value at Risk in Python
description: Monte Carlo Simulation of Value at Risk in Python
date:   2021-01-26 00:00:02 +0000
permalink: /monte_carlo_var/
category: Finance
---
## Monte Carlo Simulation of Value at Risk in Python

If you recall the basics of the [notebook][1] where we provided an introduction on market risk measures and VAR, you will recall that parametric VAR simply assumes a distribution and uses the first two moments (mean and standard deviation) to compute the VAR; whereas for historical VAR, you use the actual historical data and use the specific datapoint (or interpolated values between 2 datapoints) for the confidence level.

VAR can also be computed via simulation. Which is a good way to provide a quick introduction to Monte Carlo simulation.

Simulated VAR at its core is quite simple. You basically take the moments (say mean and standard deviation if you assume a normal distribution), generate a simulated set of data with Monte Carlo simulation, and then get the required percentile. What this means is that we could also assume a non-normal distribution, say a t-distribution, and use that for simulation and to compute VAR.

First, let's get market prices of AAPL from Quandl again, and compute the returns.
```
end = datetime.datetime.now()
start = end - datetime.timedelta(365)
AAPL = quandl.get('EOD/AAPL', start_date=start, end_date=end)

rets_1 = (AAPL['Close']/AAPL['Close'].shift(1))-1
```

We shall compute the mean and standard deviation of the AAPL returns first as we will use this later to perform Monte Carlo simulation.
```
mean = np.mean(rets_1)
std = np.std(rets_1)
Z_99 = stats.norm.ppf(1-0.99)
price = AAPL.iloc[-1]['Close']
print(mean, std, Z_99, price)

Out:
0.0016208298475378427 0.013753943856014762 -2.32634787404 220.79
```

Now, let's compute the parametric and historical VAR numbers so we have a basis for comparison.
```
ParamVAR = price*Z_99*std
HistVAR = price*np.percentile(rets_1.dropna(), 1)

print('Parametric VAR is {0:.3f} and Historical VAR is {1:.3f}'
        .format(ParamVAR, HistVAR))

Out:
Parametric VAR is -7.064 and Historical VAR is -6.166
```

For Monte Carlo simulation, we simply apply a simulation using the assumptions of normality, and the mean and std computed above.
```
np.random.seed(42)
n_sims = 1000000
sim_returns = np.random.normal(mean, std, n_sims)
SimVAR = price*np.percentile(sim_returns, 1)
print('Simulated VAR is ', SimVAR)

Out:
Simulated VAR is  -6.7185294884
```

And that’s it! 
The full code can be found at the notebook [here][2].

[1]:	https://github.com/playgrdstar/measure_marketrisk_VAR/blob/master/Introduction%20-%20Measuring%20Market%20Risk%20in%20Python.ipynb
[2]:	https://github.com/playgrdstar/monte_carlo_sim_VAR/blob/master/Simulating%20VAR.ipynb