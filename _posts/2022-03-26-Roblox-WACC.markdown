---
layout: posts
title:  "WACC for Roblox"
date:   2022-05-10 09:00:00 +0100
categories: python
author: Olayinka Ola
---
The weighted average cost of capital is a useful tool in investment appraisal. For an investment to be worth while the expected return must be greater that the cost of capital.

Publicly traded companies are funded by a combination of equity and debt. Both equity and debt have a cost which is expressed as an annual percentage of market value.

- The weighted average cost of capital (WACC) is the average percentage cost of all sources of capital.
- WACC requires us to add up the weighted cost of each source of capital for the company to ensure the calculation is a true reflection of the company's capital structure.
- The calculation of the WACC usually uses the market values for debt & equity components rather than their book values because market values are the price levels the market is ready to pay

## How to do it

To calculate the weighted average cost of capital I will follow the three-step process laid out by Professor **Aswath Damodaran, in his [spring 2021 valuation lecture series on Youtube](https://www.youtube.com/watch?v=oi6M5KBWydg&list=PLUkh9m2BorqlJsEfix7R9jtSXClFZhGvC).**

1. First I infer the cost of each kind of capital that the company uses
2. Then I calculate the proportion of capital structure that debt and equity capital contributions to the entire enterprise, using the market values of total debt and equity
3. Finally, I weigh the cost of each kind of capital by the proportion that each component (debt & equity) contributes to the entire capital structure

[Github Repo][Github Repo]

## Roblox

For demonstration purposes, I will calculate the WACC for Roblox. According to Finbox Roblox is an entertainment platform and game creation system developed by Roblox Corporation. It allows users to build, publish, and operate experiences and other content.

```python
company = 'Roblox'
```

## Cost of capital Ingredients

The formula I will use.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/WACC_formula.png" alt="full model">

## **Step 1: Calculate the cost of each kind of capital that the enterprise uses.**


### Market Value of Equity.

The market value of equity is the market estimate of what the equity in the firm is worth.

For now, I will use Roblox Market Cap - calculated by multiplying the number of shares outstanding by their share price.

*My market value numbers ignore any options issued by Roblox through warrants, convertible bonds or even management options.*

```python
mkt_value_of_equity = roblox_mc[0]['marketCap']
print("Market value of equity for %s is %.02f billion" % (company, mkt_value_of_equity))
```

```
**## OUTPUT**
Market value of equity for ROBLOX is 27048669184.00 billion
```

### Total Debt

To enable the calculation of the market value of debt I first need to calculate Roblox’s total debt based on the balance sheet data, current portion of long term debt, long term Debt and notes payable.

```python
def total_book_debt(current_portion_LT_debt, long_term_Debt, notes_payable):
    """
    Calculate the total debt based off balance sheet data, current_portion_LT_debt, long_term_Debt and notes_payable.
    """
    return current_portion_LT_debt + long_term_Debt + notes_payable

# ROBLOX EXAMPLE
cltd = 0
ld = roblox_bs[0]['longTermDebt']
np = 0

total_book_debt = total_book_debt(cltd, ld, np)
print("Total debt for %s is %.02f billion" % (company, total_book_debt))
```

```
**## OUTPUT**
Total debt for ROBLOX is 1182339000.00 billion
```

### Roblox Market Value of Debt

The market value of debt (MVD) is the price investors are willing to pay for a company's debt. To convert book value debt into market value debt I treat the entire debt on the books as a bond using the bond pricing formula.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/BondPriceFormula_lax.png" alt="full model">

Adapted from - [https://corporatefinanceinstitute.com/resources/knowledge/finance/market-value-of-debt/](https://corporatefinanceinstitute.com/resources/knowledge/finance/market-value-of-debt/)

where

- C = The interest expenses
- F = Total Debt
- r = Current cost of debt
- n = Weighted average maturity (in years)

```python
def mv_debt_by_average_maturity(average_maturity, cost_of_debt, total_book_debt, interest_expense):
    """
    Calculate the market value of debt based off financial statement data, cost of debt, and average maturity.
    """
    coupon_rate = interest_expense / total_book_debt
    principal = total_book_debt
    coupon_payment = coupon_rate * principal

    return coupon_payment * ((1 - (1 + cost_of_debt)**(-average_maturity))/cost_of_debt) + principal/(1 + cost_of_debt)**average_maturity

# ROBLOX EXAMPLE
interest_expense = roblox_is[0]['interestExpense']
cost_of_debt = 0.159
average_maturity = 5

mkt_value_of_debt = mv_debt_by_average_maturity(average_maturity, cost_of_debt, total_book_debt, interest_expense)
print("Market value of debt for %s is %.02f million" % (company, mkt_value_of_debt))
```

```
**## OUTPUT**
Market value of debt for ROBLOX is 588326743.72 million
```

With over 1 billion in debt, 7m in interest expenses, maturity of 5 years and a cost of debt at 16%, my estimated market value of debt for Roblox is 588 million.

## ****Step 2 - Calculate the proportion that debt and equity capital contributions to the entire enterprise, using the market values of total debt and equity****

```python
def capital_weights(total_debt, common_stock):
    """
    Summary: Given a firm's capital structure, calculate the weights of each group.

    PARA total_capital: The company's total capital.
    PARA type: float

    PARA common_stock: The company's common stock outstanding.
    PARA type: float

    PARA total_debt: The company's total debt.
    PARA type: float

    RTYP weights_dict: A dictionary of all the weights.
    RTYP weights_dict: dictionary

    """      
    # initalize the dictionary
    weights_dict = {}

    # calculate the total capital
    total_capital = common_stock + total_debt

    # calculate each weight and store it in the dictionary
    weights_dict['common_stock'] = common_stock / total_capital
    weights_dict['total_debt'] = total_debt / total_capital

    return weights_dict

weights = capital_weights(mkt_value_of_debt, mkt_value_of_equity)
```

### Capital Structure Weights

For Roblox I now have my cost of equity and debt weights. 98% of Roblox is capitalised through equity and 2% through debt.

```python
{'common_stock': 0.9787123482864574, 'total_debt': 0.021287651713542563}
```

## **Step 3 - Finally I weigh the cost of each kind of capital by the proportion that each contributes to the entire capital structure.**

Two key components of the cost of capital calculation are the cost of equity and the cost of debt.

The **cost of equity** is the rate of return that stockholders in Roblox expect to make when they buy the stock. [See how I calculated my cost of equity here](https://hightowerr.github.io/fun-and-games/r/2022/03/19/Roblox-Cost-of-equity-in-three-stages.html)

```python
cost_equity = 0.064
```

**Cost of debt** is the rate at which Roblox can borrow long term today. [See how I calculated cost of debt for Roblox here.](https://hightowerr.github.io/fun-and-games/python/2022/04/05/Roblox-Cost-of-debt.html)

```python
cost_of_debt = 0.159
```

The **marginal tax** rate is the rate on the last or next dollar/pound of earnings.

```python
marginal_tax = 0.27 # The tax rate for the last dollar of income
```

Roblox's weighted average cost of capital is thus

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/WACC_formula.png" alt="full model">

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Wacc_form.png" alt="full model">

```python
def weighted_average_cost_of_capital(cost_of_common, cost_of_debt, marginal_tax, weights_dict):
    """
    Summary: Calculate a firm's WACC.

    PARA cost_of_common: The firm's cost of common equity.
    PARA type: float

    PARA cost_of_debt: The firm's cost of debt.
    PARA type: float

    PARA marginal_tax: The firm's tax rate on an additional dollar of income.
    PARA type: float

    PARA weights_dict: The capital weights for each capital structure.
    PARA type: dictionary

    """    

    weight_debt = weights_dict['total_debt']
    weight_common = weights_dict['common_stock']
    tax = 1 - marginal_tax
    adj_cost_debt = cost_of_debt * tax

    return (weight_common * cost_of_common) + (adj_cost_debt * weight_debt)

# ROBLOX EXAMPLE
cost_equity = 0.064
marginal_tax = 0.27 # The tax rate for the last dollar of income

wacc = weighted_average_cost_of_capital(cost_equity, cost_of_debt, marginal_tax, weights)
print("Weighted Cost of Capital for %s is %.04f" % (company, wacc))
```

```
**## OUTPUT**
Weighted Cost of Capital for ROBLOX is 0.0651
```

WACC represents the expense of raising money so the higher it is the lower a company’s net profit. A WACC of 6.5% means that a business will have to pay investors on average £0.10 in return for every pound of funding.

### Summary

The weighted average cost of capital is a useful tool in investment appraisal. For an investment to be worth while the expected return must be greater that the cost of capital.

I used the following steps to calculate the weighted average cost of capitaI for Roblox.

1. I calculated the capital structure for Roblox.
2. I worked out the proportion of capital structure that debt and equity capital contribute to the entire enterprise, using the market values of total debt and equity
3. Finally I weighted the cost of each kind of capital by the proportion
that each component (debt & equity) contributes to the entire capital structure

### Inputs

- [Calculate the Cost of Equity](https://hightowerr.github.io/fun-and-games/r/2022/03/19/Roblox-Cost-of-equity-in-three-stages.html)
- [Calculate the Cost of Debt](http://127.0.0.1:4000/fun-and-games/python/2022/04/05/Roblox-Cost-of-debt.html)

### Extensions

This section lists some ideas for extending the tutorial.

- Hybrids and Preferred Stock
- Capitalising Lease Commitments

### Useful Links

- [https://academy.treasurers.org/resources/how-to-calculate-the-cost-of-capital](https://academy.treasurers.org/resources/how-to-calculate-the-cost-of-capital)
- [https://courses.lumenlearning.com/boundless-finance/chapter/capital-structure-considerations/](https://courses.lumenlearning.com/boundless-finance/chapter/capital-structure-considerations/)

[Github Repo]: https://github.com/hightowerr/Investing/tree/main/WACC
