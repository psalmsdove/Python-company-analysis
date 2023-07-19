# Python company analysis

In this repo, we'll use **Python, Pandas and Plotly** libraries to analyze some data from a dataset I've taken from a website. We are using VSCode Interactive Window to run codes. [Here's](https://stackoverflow.com/questions/64730660/how-do-i-find-excute-python-interactive-mode-in-visual-studio-code) how you can use Interactive Window.

First we need to import all libraries, read the data from .csv file and print it. Dataset is taken from statso.io

```
import pandas as pd
import plotly.express as px
import plotly.io as pio
import plotly.graph_objects as go
pio.templates.default = "plotly_white"

data = pd.read_csv("supply_chain_data.csv")
print(data.head())
```

Now we imported the libraries and read the file, we can start analyzing and visualizing it.

```
sales_data = data.groupby('Product type')['Number of products sold'].sum().reset_index()

pie_chart = px.pie(sales_data, values='Number of products sold', names='Product type', 
                   title='Sales by Product Type', 
                   hover_data=['Number of products sold'],
                   hole=0.5,
                   color_discrete_sequence=px.colors.qualitative.Pastel)
                   
pie_chart.update_traces(textposition='inside', textinfo='percent+label')
pie_chart.show()
```
### Code explanation

``data.groupby('Product type'):`` This line groups the ``DataFrame`` data by the column **'Product type'**. It means that the data will be grouped based on the unique values in the '**Product type'** column.

``['Number of products sold'].sum():`` After grouping, this line calculates the sum of the **'Number of products sold'** for each group. It adds up the values of the **'Number of products sold'** column within each group.

``.reset_index():`` This resets the index of the resulting ``DataFrame`` after the groupby and sum operations. As a result, the **'Product type'** column becomes part of the ``DataFrame`` again, and a new ``DataFrame`` called ``sales_data`` is created with two columns: **'Product type' and 'Number of products sold'.**

![Ekran görüntüsü 2023-07-19 090304](https://github.com/psalmsdove/Python-supply-chain-analysis/assets/48603868/e776f47b-a2eb-4845-a05c-540a9e94d483)

Above, we can see the product types and how much sale does it have. Next, we'll check the revenue by each carrier.

```
total_revenue = data.groupby('Shipping carriers')['Revenue generated'].sum().reset_index()
fig = go.Figure()
fig.add_trace(go.Bar(x=total_revenue['Shipping carriers'], 
                     y=total_revenue['Revenue generated']))
fig.update_layout(title = 'Total revenue by each shipping carrier', xaxis_title = 'Shipping carrier', yaxis_title = 'Revenue')
fig.show()
```

![2](https://github.com/psalmsdove/Python-supply-chain-analysis/assets/48603868/a2d86605-1d78-449b-ab57-a6258a1c463c)

We can see that **Carrier B** makes the most revenue from all the carriers.

Now let's check average lead time and average manufacturing costs that our company have.

```
avg_lead_time = data.groupby('Product type')['Lead time'].mean().reset_index()
avg_manufacturing_costs = data.groupby('Product type')['Manufacturing costs'].mean().reset_index()
result = pd.merge(avg_lead_time, avg_manufacturing_costs, on='Product type')
result.rename(columns={'Lead time': 'Average Lead Time', 'Manufacturing costs': 'Average Manufacturing Costs'}, inplace=True)
print(result)
```

### Code explanation

``data.groupby('Product type')``: Similar to the previous code, this line groups the ``DataFrame`` data by the column **'Product type'.**

``['Lead time'].mean()``: After grouping, this line calculates the **mean** (average) of the **'Lead time'** for each product type. It computes the average value of the **'Lead time'** column within each group.

``pd.merge:`` This function is used to merge the two ``DataFrames`` ``avg_lead_time`` and ``avg_manufacturing_costs`` based on the common column **'Product type**'. It performs an inner join by default, meaning only the product types that exist in both ``DataFrames`` will be included in the result.

After running this code, we'll get this output:

|   | Produt Type | Average Lead Time | Average Manufacturing Costs |
|---|-------------|-------------------|-----------------------------|
| 0 | cosmetics   | 13.538462         | 43.052740                   |
| 1 | haircare    | 18.705882         | 48.457993                   |
| 2 | skincare    | 18.000000         | 48.993157                   |

We can make this analysis from this table:

- Cosmetics have the shortest average lead time (13.54 units) among the three product types, indicating a relatively quick production process.

- Haircare has the longest average lead time (18.71 units) among the three product types, suggesting a longer time to produce compared to cosmetics and skincare.

- Cosmetics have the lowest average manufacturing costs (43.05 units) among the three product types.

- Skincare has the highest average manufacturing costs (48.99 units) among the three product types, indicating relatively higher expenses in production compared to cosmetics and haircare.


In the columns, you probably saw a column named **'SKU'**. SKU stands for "Stock Keeping Unit." It is a unique identifier or code assigned to a specific product or item in a company's inventory or stock. The SKU is used to track and manage individual products separately, making it easier for businesses to monitor their inventory, sales, and restocking processes. Now let's check our company's revenue generated by each SKU.


```
revenue_chart = px.line(data, x = 'SKU', y = 'Revenue generated', title = 'Revenue generated by each SKU')
revenue_chart.show()
```

![3](https://github.com/psalmsdove/Python-supply-chain-analysis/assets/48603868/4a4438a2-366b-426b-9585-22254d0698c6)

Our stock levels by SKU:

```
stock_chart = px.line(data, x='SKU', 
                      y='Stock levels', 
                      title='Stock Levels by SKU')
stock_chart.show()
```

![4](https://github.com/psalmsdove/Python-supply-chain-analysis/assets/48603868/01d94681-efa3-465a-850a-43c62e02f437)

Everything looks good for now but our company has costs for sure. Let's see our shipping costs and see if we can reduce them or how to reduce them.

```
cost_chart = px.bar(data, x = 'Shipping carriers', y = 'Costs', title = 'Shipping costs')
cost_chart.show()
```

![5](https://github.com/psalmsdove/Python-supply-chain-analysis/assets/48603868/cdb61215-73db-489b-bda2-c0e6e945bb89)

So, Carrier B **costs most** from all. But also makes profit the most. It might be a challenging decision to make. If we had more data, maybe we could tell something from them.

Let's see what percentage of our transports are made with which transport vehicle.

```
transportation_chart = px.pie(data, values = 'Costs', names = 'Transportation modes', title = 'Costs by each transportation type', hole = 0.5, color_discrete_sequence=px.colors.sequential.RdBu)
transportation_chart.show()
```

![6](https://github.com/psalmsdove/Python-supply-chain-analysis/assets/48603868/a317e089-818f-479d-b9b9-02ca8b051d20)

And finally, we may check and analyze the cost by routes for each product type. We can use **scatter plot** for this. Here's how to implement scatter plot:

```
fig = px.scatter(data, x='Product type', y='Costs', color='Routes', title='Cost - Route and Product Types')
fig.show()
```

This is the chart:

![7](https://github.com/psalmsdove/Python-company-analysis/assets/48603868/9c9ca81e-ee3c-4d5f-bbd0-9a1e6bc6e277)
