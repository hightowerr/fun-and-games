---
layout: posts
title:  "Cleaning data (sales data example)"
categories: cleaning
author: Olayinka Ola
---

 I have an example of a dataset which is not clean. The issue is that data hasn’t been loaded correctly into into the columns so my job here is to clean this data so it’s usable for analysis  & modelling.

 I have a sales.csv file which has sales data from different customer purchases in the past few years.  I have identified the following issues that need to be rectified.

- Column 1: Product and column 2: line should be added together to create a variable  labelled **product line**.
- Column 3: Product.1 and column 4: type should be added together to create a variable  labelled **product type**.
- Column 5: **Product.2**, column 6: **Order** and Column 7: **method** should be added together to create a variable labelled **product.**
- Column 9: **Retailer**, column 10: **Country** should be added together to create a variable labelled retailer country**.**
- Rename column 8

### Import of the packages is needed
{% highlight ruby %}
import pandas as pd # Import pandas module
import resumetable as rt
{% endhighlight %}

### Load Data
Now I load the sales document and printed the data frame.

{% highlight ruby %}
sales =  pd.read_csv('sales.csv'); sales # Load the data
{% endhighlight %}

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/sales.png" alt="Import of the packages is needed">

### Explore data
I called the function of a locally stored copy of the resume table. I called the function of a locally stored copy of the resume table.

This python script gives me a snapshot of the columns specifically how many values are missing and how many unique values are stored in the column. The below visual shows 91 missing values in column 10 Country.

{% highlight ruby %}
rt.resumetable(sales)
{% endhighlight %}

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/sales_miss.png" alt="Summary table">

### Implement changes

Now I implement the changes covered in my introduction.. I create four new columns and rename a fifth.

{% highlight ruby %}
sales['Product Line'] = sales['Product']+' '+sales['line']
sales['Product Type'] = sales['Product.1']+' '+sales['type']
sales['Product'] = sales['Product.2']+' '+sales['Order']+' '+sales['method']
sales['Retailer country'] = sales['Retailer']+' '+sales['country']
sales = sales.rename(columns={sales.columns[8]: 'Order Method'}) # Rename column by index in Pandas
{% endhighlight %}

**Problem**
Following the execution of the previous code snippet I can see that my new column retail country still has an issue with missing values.Following the execution of the previous code snippet I can see that my new column retail country still has an issue with missing values.

Before adding together the retailer and country columns I first need to replace all NaNs with an empty string.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/sales_high.png" alt="problem missing numbers">

**Solution**
When I run the same code again the issue was resolved.

{% highlight ruby %}
sales=sales.fillna('') # replace nan values with blank in pandas
sales['Retailer country'] = sales['Retailer']+' '+sales['country']; sales
{% endhighlight %}

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/sales_fix.png" alt="fix missing numbers">

To complete my work I can now remove all unnecessary columns from the sales data frame.

{% highlight ruby %}
sales = sales.drop(['Product', 'line', 'Product.1', 'type', 'Product.2',
                    'Order', 'method', 'type', 'country', 'Retailer'], axis=1); sales
{% endhighlight %}

With all my changes implemented I can now check the data frame to confirm there are no further issues to fix.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/sales_final.png" alt="final sales table">

### Summary

This simple example shows how to correct some common problems in files. After identifying the problems with the data file, solutions are implemented and the final output is tidied up ready for exploration.
