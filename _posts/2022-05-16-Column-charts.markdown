---
pagetitle: Order Payment Type
layout: posts
title:  "Order Payment Type"
date:   2022-05-10 09:00:00 +0100
categories: python
author: Olayinka Ola
output:
  html_document:
    highlight: zenburn
    theme: cosmo
    df_print: paged
    toc: yes
    code_folding: hide
    code_download: true
    keep_md: true
---
### **A Basic Chart: Column Charts**

Charts are very important because a good chart can convey much more information than a table of numbers. The next section starts with this topic: charting in Python. The first example, uses a simple aggregation query, the number of orders for each payment type.

The query that pulls the data is:

```sql
SELECT PaymentType, COUNT(*) as cnt
FROM Orders o
GROUP BY PaymentType
ORDER BY cnt
```

```python
df_1 = pd.read_csv('Orders_byType.csv')

```

![NumberOrdersPaymentType.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/558dea38-ff6d-4bb0-9412-e87e4a9ecf03/NumberOrdersPaymentType.png)

```python
fig = px.bar(df_1, x='paymenttype', y='cnt',
             labels={
                     "cnt": "Number of Orders",
                     "paymenttype": "Payment Type"
                 },
             title='Order by Payment type - Column Chart')
fig.show()
```

![Order by Payment type - Column Chart.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dd59242a-5254-4154-850a-d05244f40f09/Order_by_Payment_type_-_Column_Chart.png)

### **Useful Variations on the Column Chart**

The query divides the orders into four groups, based on the size of the orders. This dataset will be used to demonstrate different ways to compare values using column charts.

The below provides more information about the payment types, information such as:

- Number of orders with each code
- Number of orders whose price is in the range $0–$10, $10–$100, $100–$1,000,

    and over $1,000

- Total revenue for each code

```sql
SELECT paymenttype,
  SUM(CASE WHEN 0 <= TotalPrice AND TotalPrice < 10 THEN 1 ELSE 0 END) AS cnt_0_10,
  SUM(CASE WHEN 10 <= TotalPrice AND TotalPrice < 100 THEN 1 ELSE 0 END) AS cnt_0_10_100,
  SUM(CASE WHEN 100 <= TotalPrice AND TotalPrice < 1000 THEN 1 ELSE 0 END) AS cnt_0_100_1000,
  SUM(CASE WHEN TotalPrice >= 1000 THEN 1 ELSE 0 END) AS cnt_0_1000,
  COUNT(*) as cnt, SUM(TotalPrice) as revenue
FROM Orders
GROUP BY PaymentType
ORDER BY PaymentType;
```

```python
import plotly.express as px
import pandas as pd
import plotly.graph_objs as go
```

```python
df =  pd.read_csv('F02-06.csv')
df = df[['paymenttype','cnt_0_10','cnt_0_10_100','cnt_0_100_1000', 'cnt_0_1000']]
df_clust = df.groupby(['paymenttype']).sum().reset_index(); df_clust
```

![df_grouping.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4f0e52e1-7647-4ac8-9711-8b63949c4839/df_grouping.png)

### ****Side-by-Side Columns****

This chart shows the actual value of the number of orders for different groups

```python
anchos = [0.2] * 6
fig = go.Figure()
fig.add_trace(go.Bar(x = df_clust['paymenttype'],
                     y = df_clust['cnt_0_10'],
                     width = anchos, name = 'cnt_0_10'))
fig.add_trace(go.Bar(x = df_clust['paymenttype'],
                     y = df_clust['cnt_0_10_100'],
                     width = anchos, name = 'cnt_0_10_100'))
fig.add_trace(go.Bar(x = df_clust['paymenttype'],
                     y = df_clust['cnt_0_100_1000'],
                     width = anchos, name = 'cnt_0_100_1000'))
fig.add_trace(go.Bar(x = df_clust['paymenttype'],
                     y = df_clust['cnt_0_1000'],
                     width = anchos, name = 'cnt_0_1000'))
fig.update_layout(title =  "Order by Payment type and order Size - Side by Side",
                  barmode = 'group',title_font_size = 30,
                  width = 854, height = 480)
fig.update_layout(legend=go.layout.Legend(
            x=1,y=1,
            traceorder= "normal",
            font=dict(family="Verdana",size= 10, color = "black")))
fig.update_xaxes(title_text = 'paymenttype',
           title_font=dict(size=10,family='Verdana',color='black'),
           tickfont=dict(family='Calibri', color='darkred',size=10))
fig.update_yaxes(title_text = "Num Orders",
           title_font=dict(size=10,family='Verdana',color='black'),
           tickfont=dict(family='Calibri', color='darkred',size=10))         
#fig.write_image(path + "figclust1.png")
fig.show()
```

![Order by Payment type and order Size - Side by Side.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4b353a88-7c2f-4622-a5fa-0df9773179a4/Order_by_Payment_type_and_order_Size_-_Side_by_Side.png)

### **Stacked Columns**

This communicates the total number of orders for each payment type, making it possible to find out, for instance, where the most popular payment mechanisms are.

```python
anchos = 0.9
fig = go.Figure()
fig.add_trace(go.Bar(x = df_clust['paymenttype'],
                     y = df_clust['cnt_0_10'],
                     width = anchos, name = 'cnt_0_10'))
fig.add_trace(go.Bar(x = df_clust['paymenttype'],
                     y = df_clust['cnt_0_10_100'],
                     width = anchos, name = 'cnt_0_10_100'))
fig.add_trace(go.Bar(x = df_clust['paymenttype'],
                     y = df_clust['cnt_0_100_1000'],
                     width = anchos, name = 'cnt_0_100_1000'))
fig.add_trace(go.Bar(x = df_clust['paymenttype'],
                     y = df_clust['cnt_0_1000'],
                     width = anchos, name = 'cnt_0_1000'))
fig.update_layout(title =  "Order by Payment type and order Size - Stacked",
                  barmode = 'overlay',title_font_size = 20,
                  width = 854, height = 480)
fig.update_layout(legend=go.layout.Legend(
            x=1,y=1,traceorder= "normal",
            font=dict(family="Verdana",size= 8, color = "black")))
fig.update_xaxes(title_text = 'paymenttype',
           title_font=dict(size=10,family='Verdana',color='black'),
           tickfont=dict(family='Calibri', color='darkred',size=10))
fig.update_yaxes(title_text = "Num Orders",
           title_font=dict(size=10,family='Verdana',color='black'),
           tickfont=dict(family='Calibri', color='darkred',size=10))         
#fig.write_image(path + "figclust1.png")
fig.show()
```

![Order by Payment type and order Size - Stacked.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/28a53390-d266-4d4c-8734-d0a96f5f5f3a/Order_by_Payment_type_and_order_Size_-_Stacked.png)

### **Stacked and Normalised Columns**

Stacked and normalised columns provide the ability to see proportions across different groups. Their drawback is that small numbers—in this case, very rare payment types—have as much weight visually as the more common ones. These outliers can dominate the chart.

```python
df_total = df["cnt_0_10"] + df["cnt_0_10_100"] + df["cnt_0_100_1000"] + df["cnt_0_1000"]
df_rel = df[df.columns[1:]].div(df_total, 0)*100
df_rel['paymenttype'] = df['paymenttype']
data = df_rel.copy(); data
```

![data_stacked.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/932fb2f4-3de5-46b9-aecd-81c35457bc95/data_stacked.png)

```python
# Code for Stacked chart
fig = go.Figure()
fig.add_trace(go.Bar(y=data["cnt_0_10"],
                     x=data.paymenttype,
                     name="cnt_0_10",
    marker=dict(
        color='rgba(99,110,250)',
        line=dict(color='rgba(0,128,0, 0.5)', width=0.05)
    )
))
fig.add_trace(go.Bar(y=data["cnt_0_10_100"],
                     x=data.paymenttype,
                     name="cnt_0_10_100",
    marker=dict(
        color='rgba(239,85,60)',
        line=dict(color='rgba(0,0,255, 0.5)', width=0.05)
    )
))
fig.add_trace(go.Bar(y=data["cnt_0_100_1000"],
                     x=data.paymenttype,
                     name="cnt_0_100_1000",
    marker=dict(
        color='rgba(1,204,150)',
        line=dict(color='rgba(0,0,255, 0.6)', width=0.05)
    )
))
fig.add_trace(go.Bar(y=data["cnt_0_1000"],
                     x=data.paymenttype,
                     name="cnt_0_1000",
    marker=dict(
        color='rgba(171,99,251)',
        line=dict(color='rgba(0,0,255, 0.9)', width=0.05)
    )
))
fig.update_layout(yaxis=dict(title_text="Num Orders",
                             ticktext=["0%", "20%", "40%", "60%","80%","100%"],
                             tickvals=[0, 20, 40, 60, 80, 100],
                             tickmode="array",
                             titlefont=dict(size=15),
    ),
    autosize=False,
    width=854,
    height=480,
    paper_bgcolor='rgba(0,0,0,0)',
    plot_bgcolor='rgba(0,0,0,0)',
    title={
        'text': "Order by Payment type and order Size - Stacked Normalized",
        'y':0.96,
        'x':0.5,
        'xanchor': 'center',
        'yanchor': 'top'},
    barmode='stack')
fig.show()
```

![Order by Payment type and order Size - Stacked Normalized.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a913e4d8-3a8d-43c7-9e96-9d1da6402d3e/Order_by_Payment_type_and_order_Size_-_Stacked_Normalized.png)

### **Two Axes overlapping**

This variation is where one column has the number of orders, and the other has the total revenue.

The trick is to plot the two series using different scales, this chart has the number of orders on the left and the total revenue on the right. Using a second axis for column charts creates overlapping columns. To get around this, the columns for the number of orders are wide and those for the revenue are narrow.

```python
df_clust =  pd.read_csv('F02-06.csv'); df_clust

#Create a figure and add three traces to it
fig = go.Figure()

fig.add_trace(                               #Add a bar graph to the figure
        go.Bar(
        x=df_clust['paymenttype'],
        y=df_clust['cnt'],
        width = 0.5,
        name="Num Orders",
        marker_color='#636efa',              #Specify the color of the bars    
        hoverinfo='none',                    #Hide the hoverinfo
        ),
        )                                    

fig.add_trace(                               #Add a bar graph to the figure
        go.Bar(
        x=df_clust['paymenttype'],
        y=df_clust['revenue'],
        width = 0.2,
        name="Revenue",
        marker_color='#ef553c',              #Specify the color of the bars    
        hoverinfo='none',                    #Hide the hoverinfo
        yaxis="y2"),
        )  

fig.update_layout(title =  "Num Orders and Revenue - Overlapping on Two Axes",
      xaxis=dict(
          domain=[0.25, 1]                 #Sets the domain of this axis (in plot fraction). Play with the numbers to see how it affects the plot
      ),
    yaxis=dict(                         
        title="Num Orders",               #Add an axis title for the primary axis
        titlefont=dict(
            color="#d99b16"               #Make the color of the y-axis the same as the bar color
        ),
        tickfont=dict(
            color="#d99b16"
        )
    ),
    yaxis2=dict(
        title="Revenue",   #Add axis title for the second y-axis
        titlefont=dict(
            color="#0000ff"              #Make the color of the third y-axis the same as its corresponding line color
        ),
        tickfont=dict(
            color="#0000ff"
        ),
        anchor="x",                      #If set to "x", this axis is bound to the corresponding opposite-letter axis - in this case, y-axis. And the 'side' parameter specifies which side this y-axis is placed
        overlaying="y",
        side="right"
    ),
)

fig.show()
```

![Num Orders and Revenue - Overlapping on Two Axes.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e117c759-7907-496e-a21b-6519d6a94f6f/Num_Orders_and_Revenue_-_Overlapping_on_Two_Axes.png)

## ****Other Types of Charts****

****Line Charts****

```python
fig = px.line(df_clust, x='paymenttype', y=['cnt_0_10','cnt_0_10_100','cnt_0_100_1000','cnt_0_1000'], title='Order by Payment type and order Size - Line Chart')
fig.show()
```

![Order by Payment type and order Size - Line Chart.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/78443145-dbeb-4a32-bf65-f05ed8cad674/Order_by_Payment_type_and_order_Size_-_Line_Chart.png)

This chart emphasises that the three main payment types are responsible for most orders and most revenue. Notice, that AE and MC have about the same number of orders, but AE has much more revenue. This means that the average revenue for customers who pay by American Express is larger than the average revenue for customers who pay by MasterCard.

```python
#Create a figure and add three traces to it
fig = go.Figure()

fig.add_trace(                               #Add a bar graph to the figure
        go.Bar(
        x=df_clust['paymenttype'],
        y=df_clust['cnt'],
        name="Num Orders",
        marker_color='#636efa',              #Specify the color of the bars    
        hoverinfo='none',                    #Hide the hoverinfo
        ),
        )                                    

fig.add_trace(                               #Add the second trace (line graph) to the figure
    go.Scatter(                          
    x=df_clust['paymenttype'],
    y=df_clust['revenue'],
    name="Revenue",
    mode='lines',                            #Use go.Scatter() and specify the mode to 'lines' to add the line graph
    text=df_clust['revenue'],               
    hoverinfo='text',                        #Pass the 'text' column to the hoverinfo parameter to customize the tooltip
    line = dict(color='#ef553c', width=3),   #Specify the color of the line
    yaxis="y2" ),                            #By specifying yaxis='y2' we tell plotly that this line chart uses a secondary y-axis
    )                   
# Create axis objects
fig.update_layout(title =  "Num Orders and Revenue - Overlapping on Two Axes",
      xaxis=dict(
          domain=[0.25, 1]                 #Sets the domain of this axis (in plot fraction). Play with the numbers to see how it affects the plot
      ),
    yaxis=dict(                         
        title="Num Orders",               #Add an axis title for the primary axis
        titlefont=dict(
            color="#636efa"               #Make the color of the y-axis the same as the bar color
        ),
        tickfont=dict(
            color="#636efa"
        )
    ),
    yaxis2=dict(
        title="Revenue",   #Add axis title for the second y-axis
        titlefont=dict(
            color="#ef553c"              #Make the color of the third y-axis the same as its corresponding line color
        ),
        tickfont=dict(
            color="#ef553c"
        ),
        anchor="x",                      #If set to "x", this axis is bound to the corresponding opposite-letter axis - in this case, y-axis. And the 'side' parameter specifies which side this y-axis is placed
        overlaying="y",
        side="right"
    ),
)

fig.show()
```

![Num Orders and Revenue - Overlapping on Two Axes with Line.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6635acd3-47c7-4e61-a8a9-3c0e7869fa54/Num_Orders_and_Revenue_-_Overlapping_on_Two_Axes_with_Line.png)

### **X-Y Charts (Scatter Plots)**

Scatter plots are very powerful and are used for many examples. This example shows an obvious relationship between the two variables payment types with more orders have more revenue. Hover over the trend line, to see each additional order brings in about $75 additional revenue

```python
fig = px.scatter(df_clust, x="cnt", y="revenue", trendline="ols")
fig.show()
```

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Revenue vs Num Order for different Payment Types.png" alt="full model">
