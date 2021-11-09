---
layout: posts
title:  "What is a response variable"
date:   2021-11-08 13:00:00 +0100
categories: Regression
author: Olayinka Ola
---

The response variable, **'y'**,  is the outcome variable or the predicted variable in regression analysis. Response variables are often related to independent variables, sometimes denoted as the predictor variables, x.

- These two variables are often referred to by different names:
    - Independent Variable (X) and Dependent Variable (y)
    - Feature Variable (X) and Target Variable (y)
    - Input Variable (X) and Output Variable (y)

The following is an example scenario involving explanatory and response variables.

Example: A marketer would like to understand the relationship between sales of a product and advertising spend (measured in 000's)

- In this example, we have:
    - **Explanatory Variable:** Advertising Spend. This is the variable chosen to observe its effect on product sales
    - **Response Variable:** Product sales. This is the variable we wish to predict as a result of the advertising spend put behind it.

In the following [worked example][response-variable] I will use a Simple Linear Regression to predict product sales given advertising spend.


Linear regression allows you to determine the connection between two variables in the data set. In a simple linear regression problem you need to use a single explanatory numerical variable to predict a numerical response variable.

One of the simplest forms of Linear Regression is the Ordinary Least Squares Method (OLS). OLS minimises the total squared differences between the actual values of the response variable and the predicted values for the response variable.

Check out my simple linear regression example [here][here].

[response-variable]: https://github.com/hightowerr/notebooks
[here]: https://github.com/hightowerr/notebooks
