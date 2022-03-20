---
layout: posts
title:  "Three stage model Cost of Equity"
date:   2022-03-19 05:00:00 +0100
categories: r, data science,
author: Olayinka Ola
---

This post is a look cost of equity. The cost of equity is a key ingredient of every discounted cash flow model.

Link to my [Github Repo][Github Repo]

## Purpose

The cost of equity is the rate of return an investor requires from stock before looking at alternative investment opportunities. Represented as a percentage Cost of equity is the threshold that must be surpassed for an investor to proceed further with an investment. Generally speaking the higher the risk of investment the higher the cost of equity.

## Inputs

The cost of equity has three components.

1. Risk-Free Rate (rf) - The risk-free rate is the return that an investor would expect an investment with zero risk (no default and reinvestment risk), over a defined period.
2. Implied Equity risk premium (IERP) - A forward-looking version of the historic equity risk premium.
3. Leveraged Beta (LV) - The beta of a firm inclusive of the effects of the capital structure.

## Outputs

A percentage that encapsulates the price of risk in the market

## How to do it

### Import the Data

```python
import pandas as pd
import numpy as np
from datetime import date
from sympy.solvers import solve
from sympy import Symbol
from datetime import date
import pandas_datareader.data as web
import requests
from pandas_datareader.data import DataReader
```

### Risk-Free Rate (rf)

The risk-free rate is the return that an investor would expect an investment with zero risk (no default and reinvestment risk), over a defined period.

In the code posted below, I calculate the rolling 5-year historical average yield in the 5-year US note from 2017 - to 2022.

Roblox is a US company. The US government is considered safe so I will use straightforward risk-free in my calculation. If I were valuing a Russian company I would subtract the stable market (US) CDS spread from the Russian CDS spread. I would then subtract this difference spread from the stable market risk-free rate.

```python
start = 2017
end = 2022

periods_per_year = 1
# Load historical 5-year Treasury rate:
treasury_5yr_monthly = (web.DataReader('DGS5', 'fred', start, end)
                         .resample('M')
                         .last()
                         .div(periods_per_year)
                         .squeeze())

riskfree_rate = treasury_5yr_monthly.mean()
print("5-year historical average US Treasury rate is %.03f" % (riskfree_rate)) # 1.576% on 06.03.mar
```

```
5-year historical average US Treasury rate is 1.576
```

## Implied Equity risk premium (IERP)

To calculate Implied equity risk premium (IERP) I take the S&P 500 as a proxy for the broad stock market of Morocco. To estimate the implied equity risk premium, I will use a three-stage model.

Using data from the following source - [https://pages.stern.nyu.edu/~adamodar/New_Home_Page/datafile/spearn.htm](https://pages.stern.nyu.edu/~adamodar/New_Home_Page/datafile/spearn.htm)
I look at dividends as a percentage of the index for 2011 to 2021 to get normalised dividends.

```python
# Create dictionary from a bunch of Series/data
Year = pd.Series(data = ["2021", "2020", "2019", "2018", "2017", "2016", "2015", "2014", "2013", "2012", "2011"])
index = pd.Series(data = ["4766.18", "3756.07", "3230.78", "2506.85", "2673.61", "2238.83", "2043.94", "2058.90", "1848.36", "1426.19", "1257.60"])
Dividend_Yield = pd.Series(data = ["0.0124", "0.0151", "0.0183", "0.0214", "0.0183", "0.02034","0.0212", "0.0192", "0.0189", "0.0219", "0.0211"])

# Create a dictionary with the data given above
a_dict = {'Year':Year,'index':index,'Dividend Yield':Dividend_Yield}

# Use the dictionary to create a Pandas DataFrame
sp500 = pd.DataFrame(a_dict)
sp500['Dividend Yield'] = sp500['Dividend Yield'].astype(float)
sp500['Dividend Yield'] = sp500['Dividend Yield'].astype(float)
sp500['index'] = sp500['index'].astype(float)
sp500 = sp500.set_index('Year')
```

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/sp500.png" alt="full model">

```python
div_avg = sp500['Dividend Yield'].mean()
print("The dividends averaged %.05f" % (div_avg))
```

```
The dividends averaged 0.01892
```

```python
current_index = 4204.31 # The current market value of the index as of right now
print("The current market value of the index as of right now is %.05f" % (current_index))
```

```
The current market value of the index as of 13/03/2021 is 4204.31000
```

```python
norm_dividends = sp500['Dividend Yield'].mean() * current_index; norm_dividends
print("Applying the %.05f yield to the current market value of the index %.05f results in normalized dividends of %.02f index points:" % (div_avg, current_index, norm_dividends))
```

```
Applying the 0.01892 yield to the current market value of the index 4204.31000 results in normalized dividends of 79.55 index points:
```

As we can see in the table above, the dividends averaged 1.89% of the index each year. Applying the 1.89% yield to the current market value of the index (4,204.31 as of 13/03/2021) results in normalized dividends of 79.55 index points.

I will assume that dividends will grow at 2% the expected dividends over the next 5 years can be seen below.

```python
rfrate = riskfree_rate / 100 # get decimal percentage
growth_rt = 0.02 # assumption of how much dividends grow

cashflow = [norm_dividends]
for i in range(5): # Estimate expected cash flows for the next five years
    cashflow.append(cashflow[i]* (1+ growth_rt))

cashflow
```

```
[79.55318940000001,
 81.14425318800001,
 82.76713825176,
 84.4224810167952,
 86.1109306371311,
 87.83314924987373]
```

Based on the expected dividends and the actual price level of the index, we will solve (using the sympy package) for the implied cost of equity. The implied ERP is calculated netting out the risk-free rate from the implied cost of equity.

```python
# Solve for an internal rate of return and subtract out the risk rate
r = Symbol('r')
result = solve(81.144/(1+r) +
               82.767/(1+r)**2 +
               84.422/(1+r)**3 +
               86.110/(1+r)**4 +
               87.833/(1+r)**5 +
               87.833*(1 + rfrate)/((r-rfrate)*(1+r)**5)-current_index, r)
print("all results:")
print(result)
print("final result")
for rs in result:
    try:
        if rs > 0:
            print(rs)
            exp_ret = float(rs)
    except TypeError:
        # To capture non-real error
        pass

irp = exp_ret - rfrate; irp # Implied Equity Risk Premium
```

```
all results:
[0.0353724842017012, -1.0674011010265 - 0.0636057271478609*I, -1.0674011010265 + 0.0636057271478609*I, -0.932753377245433 - 0.0725077134569027*I, -0.932753377245433 + 0.0725077134569027*I]

final result
0.0353724842017012

0.019609150868367817
```

### Bottom up Beta

The weight average Beta of the business or business a firm is in. There are 5 steps in in estimating bottom up betas.

### Step 1: Find as many publicly businesses that your firm operates in.

Roblox is in the entertainment industry. I pulled the below CSV from the website finbox

```python
data = pd.read_csv('Roblox - Entertainment - Bottom Beta.csv')
df1 = data.dropna()#; rblx_1.info() # Drop all rows with NaN
df = df1.copy() # Copy the dataframe
df['debt2Equity'] = df['Total Debt'] / df['Total Equity']; df.head() # calculate the debt ratio
```

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Roblox_BB.png" alt="full model">

### Step 2: Find publicly traded firms in each of these businesses and obtain their regression betas.

This step computes the simple average across all betas to arrive at an average beta for the listed publicly traded firms.

```python
rblx = df.groupby('Industry').mean(); rblx
```

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Roblox_industry.png" alt="full model">

### Step 3: Estimate how much value your firm derives from each of the different businesses it is in.

Calculating the revenue and EV/Sales ratio gives me the weighted value of each business the company operates in. Roblox is 100% entertainment so this step is not essential.

```python
rblx['EV/Sales'] = rblx['Enterprise Value (EV)'] / rblx['Revenue']
rblx['Value of Business'] = rblx['Revenue'] * rblx['EV/Sales']; rblx
rblx['percent'] = (rblx['Value of Business'] / rblx['Value of Business'].sum()); rblx
```

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Roblox_industryv2.png" alt="full model">

### Step 4: Compute a weighted average of the unlevered betas of the different businesses (from step 2) using the weights from step 3.

Roblox operates solely in the entertainment industry so I only need to calculate the unlevered beta for the Entertainment industry. If this business mix changes over time this allocation can evolve.

### Unlevered Beta for Entertainment industry

```python
avg_beta_ent = rblx['Beta (5 Year)'][0]
avg_taxrate = 0.27 # https://home.kpmg/xx/en/home/services/tax/tax-tools-and-resources/tax-rates-online/corporate-tax-rates-table.html
avg_debt_equity_ent = rblx['debt2Equity'][0]; avg_debt_equity_ent

unlevered_beta_ent = avg_beta_ent / (1 + (1 - avg_taxrate)*(avg_debt_equity_ent)) # One for every industry the firm operates in
unlevered_beta = unlevered_beta_ent * rblx['percent'][0]; unlevered_beta # This is the weighted average unlevered beta for the industries the company operates in
```

```
0.7991200824345993
```

### Step 5: Compute a levered beta (equity beta) for your firm, using the market debt to equity ratio for your firm.

```python
total_equity = 593 # https://finbox.com/NYSE:RBLX as of 19.03.22
total_debt = 1234 # https://finbox.com/NYSE:RBLX as of 19.03.22
rblx_de_ratio = total_debt / total_equity # This is Roblox specific
effective_taxrate = 0.0 # zero due to Roblox making a loss
```

### Levered Beta for Roblox

```python
levered_beta = unlevered_beta*(1 + (1 - effective_taxrate)*(rblx_de_ratio))
```

```
2.4620445035548277
```

### **Cost of equity**

With the below formula I can calculate the cost of equity for Roblox. The cost of equity for Roblox is 6.4

```python
cost_equity = rfrate + (levered_beta * irp); cost_equity
```

```
0.0640419354481757
```

The files for this post can be found in the below Repo.
[Github Repo]: https://github.com/hightowerr/Investing/tree/main/WACC

### **Next steps**

- Calculate the cost of debt
