---
layout: posts
title:  "Cost of debt for Roblox"
date:   2022-04-05 09:00:00 +0100
categories: python, data science,
author: Olayinka Ola
---

The pre-tax cost of debt is the rate at which a firm can borrow long term today. It reflects the premium of default in recovery. ([Link](http://pages.stern.nyu.edu/~adamodar/New_Home_Page/definitions.html))

There are multiple approaches to estimating the cost of debt:

1. First, we have the historical Cost of debt. Here we look at what the company paid last time. This approach is not the best as it is backwards-looking, not forward-looking and so does not reflect the current market.
2. The next option is to use the current yield on debt. This option lets me see what investors are currently trading the company's debt at and see what the yield to maturity is. This only works for large companies as it requires current market prices for debt. Additionally, some companies may have numerous bonds so which one do you use?
3. Finally, we can compute the rating adjusted yield. Looking at the moody’s credit rating, for the firm, and just adding that basis point premium on top of the risk-free rate gives me a robust cost of debt. This is the best option because other firms can be used as comparables. In situations where there is no rating or they are inaccessible for a firm then estimate a synthetic rating can be used to calculate the cost of debt. [For this situation, Aswath Damodaran’s interest coverage ratio system will be used.](http://pages.stern.nyu.edu/~adamodar/New_Home_Page/datafile/ratings.htm)

For demonstration purposes, I will calculate the cost of debt for Tesla. Tesla, Inc. designs, develops, manufactures, leases, and sells electric vehicles, and energy generation and storage systems in the United States, China, and internationally.

Moody’s company rating is behind a paywall so in this example I will use the Synthetic rating developed by Aswath Damodaran.

So I will use the following cost of debt formula:

$Cost of Debt = Risk Free rate + Credit Spread$

Link to my [Github Repo][Github Repo]

## Risk Free Rate

The risk-free rate is the return that an investor would expect an investment with zero risk (no default and reinvestment risk), over a defined period. However in reality there is no such thing as zero risk. Theoretically, the safest place for my money is in a stable economy bond like the 5y or 10y US treasury bond. Even investing your money in economies like the US comes with a tiny risk of default. I will use the 5-year historical average US Treasury rate as a proxy for my risk-less asset. My rationale being the US dollar is the world's reserve currency so it is likely nearly if not all countries have a few dead presidents tucked away for emergencies.

```python
start = 2017
end = 2022

periods_per_year = 1
# Load historical 5-year Treasury rate:
treasury_5yr_monthly = (web.DataReader('DGS5', 'fred', start, end)
                         .resample('M')
                         .last()
                         .div(periods_per_year)
                         .div(100)
                         .squeeze())

riskfree_rate = treasury_5yr_monthly.mean()
print("5-year historical average US Treasury rate is %.03f" % (riskfree_rate))
```

```
5-year historical average US Treasury rate is 0.016
```

- This sets the floor on my expected rate of return
- Any risky asset has to earn me some premium over the risk-free rate
- The US treasury is the benchmark

## Credit Default Spread

My biggest concern when investing in a company is the risk of that company not paying me back my money. To understand how likely Tesla is to default on its debt I will estimate a synthetic rating using the [Damodaran site rating system](http://pages.stern.nyu.edu/~adamodar/New_Home_Page/datafile/ratings.htm). The key input to calculate the synthetic rating is the interest coverage ratio.

$Interest Coverage Ratio = \frac{EBIT}{Interest Expense}$

The	rating for a firm can be	estimated using the financial characteristics of the firm. In its simplest form, the rating can be estimated from the interest coverage ratio.

```python
def interest_coverage(symbol):
  IS= requests.get(f'https://financialmodelingprep.com/api/v3/income-statement/{symbol}?apikey={api}').json()
  EBIT= IS[0]['ebitda'] - IS[0]['depreciationAndAmortization']
  interest_expense = IS[0]['interestExpense']
  interest_coverage_ratio = EBIT / interest_expense
  return interest_coverage_ratio

int_cov_ratio = interest_coverage(ticker); int_cov_ratio
```

```
-69.30165761646185
```

With my risk-free rate and interest coverage ratio, I can calculate the cost of debt. I have updated the below cost of debt function found at [coding and fun](https://codingandfun.com/calculating-weighted-average-cost-of-capital-wacc-with-python/). The updated ratings replicate the [credit rating system defined by Aswath Damodaran](https://pages.stern.nyu.edu/~adamodar/New_Home_Page/datafile/ratings.html)

### Cost of Debt Function

```python
def cost_of_debt(company, RF,interest_coverage_ratio):
  if interest_coverage_ratio > 12.5:
    #Rating is AAA
    credit_spread = 0.0067
  if (interest_coverage_ratio > 9.5) & (interest_coverage_ratio <= 12.499999):
    #Rating is AA
    credit_spread = 0.0082
  if (interest_coverage_ratio > 7.5) & (interest_coverage_ratio <=  9.499999):
    #Rating is A+
    credit_spread = 0.0103
  if (interest_coverage_ratio > 6) & (interest_coverage_ratio <=  7.499999):
    #Rating is A
    credit_spread = 0.0114
  if (interest_coverage_ratio > 4.5) & (interest_coverage_ratio <=  5.999999):
    #Rating is A-
    credit_spread = 0.0129
  if (interest_coverage_ratio > 4) & (interest_coverage_ratio <=  4.499999):
    #Rating is BBB
    credit_spread = 0.0159
  if (interest_coverage_ratio > 3.5) & (interest_coverage_ratio <=  3.999999):
    #Rating is BB+
    credit_spread = 0.0193
  if (interest_coverage_ratio > 3) & (interest_coverage_ratio <=  3.499999):
    #Rating is BB
    credit_spread = 0.0215
  if (interest_coverage_ratio > 2.5) & (interest_coverage_ratio <=  2.999999):
    #Rating is B+
    credit_spread = 0.0315
  if (interest_coverage_ratio > 2) & (interest_coverage_ratio <=  2.499999):
    #Rating is B
    credit_spread = 0.0378
  if (interest_coverage_ratio > 1.5) & (interest_coverage_ratio <=  1.999999):
    #Rating is B-
    credit_spread = 0.0462
  if (interest_coverage_ratio > 1.25) & (interest_coverage_ratio <=  1.499999):
    #Rating is CCC
    credit_spread = 0.0778
  if (interest_coverage_ratio > 0.8) & (interest_coverage_ratio <=  1.249999):
    #Rating is CC
    credit_spread = 0.0880
  if (interest_coverage_ratio > 0.5) & (interest_coverage_ratio <=  0.799999):
    #Rating is C
    credit_spread = 0.1076
  if interest_coverage_ratio <=  0.499999:
    #Rating is D
    credit_spread = 0.1434

  cost_of_debt = RF + credit_spread
  #print(cost_of_debt)
  return cost_of_debt
```

With the above function, I can calculate the cost of Debt for Roblox as 15.9%

```python
cost_debt = cost_of_debt(company,riskfree_rate,int_cov_ratio); cost_debt
print("Pre-tax cost of debt (borrowing) for %s is %.03f" % (company, cost_debt))
```

```
Pre-tax cost of debt (borrowing) for ROBLOX is 0.159
```

### Extensions

- Company default risk as a result of production facilities

### Next steps

- Calculate Weighted Average Cost of Capital

### Reference

Board of Governors of the Federal Reserve System (US), Market Yield on U.S. Treasury Securities at 5-Year Constant Maturity [DGS5], retrieved from FRED, Federal Reserve Bank of St. Louis; [https://fred.stlouisfed.org/series/DGS5](https://fred.stlouisfed.org/series/DGS5), January 5, 2022.

[Github Repo]: https://github.com/hightowerr/Investing/tree/main/WACC
