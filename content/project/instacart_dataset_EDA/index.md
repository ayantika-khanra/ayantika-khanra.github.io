---
title: Exploratory data analysis of the Instacart Dataset
summary: 
date:
authors:
  - admin
sidebar:
  left: true
  widgets:
    - type: tag_cloud 
tags:
  - SQL
  - Postgres database
  - ERD diagrams
  - Exploratory Data Analysis
  - Data Visualization
  - Python (Pandas, seaborn, SQLAlchemy)

markup:
  goldmark:
    parser:
      attribute:
        block: true
---


<style>
  body {
    font-size: 1rem;
    line-height: 1.4;
  }

  h1, h2, h3, h4 {
    font-size: 1.2rem;
    line-height: 1.2;
  }

  p {
    font-size: 1rem;
    line-height: 1.4;
    margin-bottom: 0.8rem;
  }
  ul, ol {
    font-size: 1rem;
    line-height: 1.4;
    margin-left: 1.5rem;
  }

  li {
    margin-bottom: 0.4rem;
  }

  .highlight pre,
  .chroma pre,
  pre code {
    font-size: 0.8rem;   /* smaller font size */
    line-height: 1.1;    /* optional: adjust line spacing */
  }

img {
    display: block;   /* ensures it behaves like a block element */
    margin-top: 0 !important;
    margin-bottom: 0 !important;
}

pre + img,
p + img,
div + img {
    margin-top: 0 !important;
    margin-bottom: 0 !important;
}

</style>


### Seasonality in Orders

To understand seasonal purchasing patterns, we analyze product purchase variation with:
- **Hour of the day**
- **Day of the week**
- **Week of the year:** not directly available in this dataset. However, a proxy can be inferred from the `days since prior order` column to approximate yearly buying trends.


#### Number of Orders by Day of the Week and Hour of the Day

```python
# Query: total orders per day of week & hour of day
query ="""
SELECT order_dow AS day_of_the_week, 
       order_hour_of_day, 
       COUNT(order_id) AS number_of_orders_placed 
FROM orders
GROUP BY order_dow, order_hour_of_day
"""
df = pd.read_sql(query, engine)

# Map numeric days to day names
day_of_week_dict={0: 'Sun', 1: 'Mon', 2: 'Tue', 3: 'Wed', 4: 'Thu', 5: 'Fri', 6: 'Sat'}
df['day_of_the_week']=df['day_of_the_week'].map(day_of_week_dict)

# Visualization 1. Orders across hours, split by weekday
grouped_data_dow=df.groupby('day_of_the_week')['number_of_orders_placed'].sum().reset_index()
sns.barplot(data=grouped_data_dow , x='day_of_the_week', y='number_of_orders_placed', 
            hue='day_of_the_week', palette='viridis', ax=axes[0])

# Visualization 2. Orders by weekday
fig, axes = plt.subplots(1, 2, figsize=(14, 6)) 
sns.lineplot(data=df, x="order_hour_of_day", y="number_of_orders_placed", 
             hue="day_of_the_week", palette='viridis', linewidth=3,  ax=axes[1])
```
<p style="color: gray; font-size: 0.6em;">
For readability, portions of the visualization code (axis labeling, legends, etc.) has been omitted.
</p>

![](images/instacart230131.png)

> [!tip] Insight
> Number of orders peak on **Sundays and Mondays**, especially **Sunday evenings** and **Monday mornings**.  Across all days, most purchases occur between **9 AM – 4 PM**.



> [!note]
> This is a note to highlight important information.






















#### avg number of distinct products in an order depending on day of week and hour of day

```python
query_cte="""
WITH cte_product_count AS (
   SELECT COUNT(ot.product_id) AS num_of_product_bought, 
          o.order_dow AS order_dow
   FROM order_products__all AS ot
   LEFT JOIN orders AS o
          ON ot.order_id=o.order_id
   GROUP BY o.order_id)
"""

query=query_cte+"""
SELECT order_dow AS day_of_the_week, 
       AVG(num_of_product_bought) AS avg_num_of_product_bought
FROM cte_product_count
GROUP BY order_dow
ORDER BY order_dow ASC
"""
df = pd.read_sql(query, engine)

fig, axes = plt.subplots(1, 2, figsize=(14, 6))  
sns.barplot(data=df , x='day_of_the_week', y='avg_num_of_product_bought',              
            hue='day_of_the_week', palette='viridis', ax=axes[0])

query =query_cte+"""
SELECT order_dow as day_of_the_week, 
       order_hour_of_day, 
       avg(num_of_product_bought) as avg_num_of_product_bought
FROM cte_product_count
group by order_dow, order_hour_of_day
order by order_dow asc"""
df = pd.read_sql(query, engine)

df['scaled_avg_num_of_product_bought']=df.groupby('day_of_the_week')['avg_num_of_product_bought'].transform(lambda x: x/x.max())
df['day_of_the_week']=df['day_of_the_week'].map(day_of_week_dict)
sns.lineplot(data=df, x="order_hour_of_day", y="scaled_avg_num_of_product_bought",
             hue='day_of_the_week', palette='viridis', linewidth=3, , ax=axes[1])
```

insight: number of products per order on average hovers around 9-11. it is typically high in Fri-mon, sunday being the highest. On all days higher product containing orders happen between 10 pm and 1 am. in fri to mon however another peak emerges around 9 am where people place orders with more products in it.
