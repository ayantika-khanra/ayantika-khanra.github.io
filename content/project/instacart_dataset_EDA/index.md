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


#### Weekly and Daily Variations in Order Volume

To understand how order volume varies across different times, I queried the database and created visualizations by day of week and hour of day, as follows:

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

{{< figure src="/images/instacart230131.png" class="round" >}}


{{< alert tip "Insight" "<svg class='alert-icon' xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'><path fill='#28a745' d='m17.989,4.341l-1.709-1.041L18.266.04l1.709,1.041-1.985,3.26Zm5.161.206l-3.331,1.504.822,1.822,3.331-1.504-.822-1.822Zm-5.54,1.75c1.541,1.517,2.39,3.542,2.39,5.703,0,2.295-.99,4.481-2.718,5.999-.814.717-1.282,1.833-1.282,3.064v2.937h-8v-3.07c0-1.155-.453-2.211-1.244-2.897-1.836-1.593-2.838-3.898-2.75-6.326.149-4.179,3.675-7.636,7.858-7.705,2.15-.042,4.205.779,5.746,2.296Zm-3.61,14.767c0-.362.036-.716.092-1.063h-4.169c.046.305.077.614.077.93v1.07h4v-.937Zm4-9.063c0-1.621-.637-3.141-1.793-4.277-1.155-1.138-2.691-1.751-4.31-1.722-3.138.052-5.781,2.644-5.894,5.777-.065,1.82.687,3.549,2.062,4.744.481.417.879.919,1.188,1.478h1.745v-4.184c-1.161-.414-2-1.514-2-2.816h2c0,.552.448,1,1,1s1-.448,1-1h2c0,1.302-.839,2.402-2,2.816v4.184h1.767c.312-.569.713-1.079,1.195-1.503,1.295-1.139,2.038-2.777,2.038-4.497ZM7.725,3.3L5.739.04l-1.709,1.041,1.985,3.26,1.709-1.041ZM.854,4.547L.032,6.369l3.33,1.504.822-1.822-3.33-1.504Z'/></svg>" >}}
Orders peak on **Sundays and Mondays**, especially **Sunday evenings** and **Monday mornings**. Across all days, most purchases occur between **9 AM – 4 PM.**
{{< /alert >}}















#### Weekly and Daily Variations in Basket Size

Here I queried the database to calculate the average number of products bought per order across different days of the week and hours of the day.

```python
# CTE to calculate number of products per order
query_cte="""
WITH cte_product_count AS (
   SELECT COUNT(ot.product_id) AS num_of_product_bought, 
          o.order_dow AS order_dow
   FROM order_products__all AS ot
   LEFT JOIN orders AS o
          ON ot.order_id=o.order_id
   GROUP BY o.order_id
)
"""

# Querying average basket size by day of the week
query=query_cte+"""
SELECT order_dow AS day_of_the_week, 
       AVG(num_of_product_bought) AS avg_num_of_product_bought
FROM cte_product_count
GROUP BY order_dow
ORDER BY order_dow ASC
"""
df = pd.read_sql(query, engine)

# Barplot visualization
fig, axes = plt.subplots(1, 2, figsize=(14, 6))  
sns.barplot(data=df , x='day_of_the_week', y='avg_num_of_product_bought',              
            hue='day_of_the_week', palette='viridis', ax=axes[0])

# Querying average basket size by day of the week and hour of day
query =query_cte+"""
SELECT order_dow AS day_of_the_week, 
       order_hour_of_day, 
       avg(num_of_product_bought) AS avg_num_of_product_bought
FROM cte_product_count
GROUP BY order_dow, order_hour_of_day
ORDER BY order_dow ASC"""
df = pd.read_sql(query, engine)

# Scale hourly basket sizes vs hour of days plots for easier comparison
df['scaled_avg_num_of_product_bought']=(df.groupby('day_of_the_week')['avg_num_of_product_bought']
                                        .transform(lambda x: x/x.max()))

# line plot visualization
df['day_of_the_week']=df['day_of_the_week'].map(day_of_week_dict)
sns.lineplot(data=df, x="order_hour_of_day", y="scaled_avg_num_of_product_bought",
             hue='day_of_the_week', palette='viridis', linewidth=3, , ax=axes[1])
```

{{< figure src="/images/Instacart040511.png" class="round" >}}

{{< alert tip "Insight" "<svg class='alert-icon' xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'><path fill='#28a745' d='m17.989,4.341l-1.709-1.041L18.266.04l1.709,1.041-1.985,3.26Zm5.161.206l-3.331,1.504.822,1.822,3.331-1.504-.822-1.822Zm-5.54,1.75c1.541,1.517,2.39,3.542,2.39,5.703,0,2.295-.99,4.481-2.718,5.999-.814.717-1.282,1.833-1.282,3.064v2.937h-8v-3.07c0-1.155-.453-2.211-1.244-2.897-1.836-1.593-2.838-3.898-2.75-6.326.149-4.179,3.675-7.636,7.858-7.705,2.15-.042,4.205.779,5.746,2.296Zm-3.61,14.767c0-.362.036-.716.092-1.063h-4.169c.046.305.077.614.077.93v1.07h4v-.937Zm4-9.063c0-1.621-.637-3.141-1.793-4.277-1.155-1.138-2.691-1.751-4.31-1.722-3.138.052-5.781,2.644-5.894,5.777-.065,1.82.687,3.549,2.062,4.744.481.417.879.919,1.188,1.478h1.745v-4.184c-1.161-.414-2-1.514-2-2.816h2c0,.552.448,1,1,1s1-.448,1-1h2c0,1.302-.839,2.402-2,2.816v4.184h1.767c.312-.569.713-1.079,1.195-1.503,1.295-1.139,2.038-2.777,2.038-4.497ZM7.725,3.3L5.739.04l-1.709,1.041,1.985,3.26,1.709-1.041ZM.854,4.547L.032,6.369l3.33,1.504.822-1.822-3.33-1.504Z'/></svg>" >}}
- The average basket size ranges between **9–11 products**.
- **Friday through Monday** customers purchase with higher-than-average basket sizes, with **Sunday with the largest average basket sizes**.
- On all days, customers purchase with larger basket sizes during **late-night hours (10 PM – 1 AM)**. On Friday to Monday, a second peak in basket sizes appears in **morning around 9 AM**.{{< /alert >}}



#### Weekly and Daily Variations in Order Volume from Different Departments

To understand customer purchasing patterns across product categories, we analyze how the number of purchases varies by day of the week and by department. We normalize the purchase counts for comparison across departments 

```python
# Query the count of purchases by department, aisle, and day of week
query="""
SELECT COUNT(p.product_id) AS number_of_purchase, 
       a.aisle AS aisle, 
       d.department AS department,
       o.order_dow AS day_of_week
FROM order_products__all AS ot
LEFT JOIN orders AS o
       ON ot.order_id=o.order_id
LEFT JOIN products AS p
       ON p.product_id=ot.product_id
LEFT JOIN aisles AS a
       ON p.aisle_id = a.aisle_id
LEFT JOIN departments AS d
       ON d.department_id = p.department_id
GROUP BY d.department, a.aisle, o.order_dow
"""
df = pd.read_sql(query, engine)

# Normalize purchase count plots
df['norm_number_of_purchase']=(df.groupby('aisle')['number_of_purchase']
                               .transform(lambda x: x/x.mean()))

# Grouping departments in 4 categories and 
#creating visualizations with departmentwise grouping
department_groups=[['produce','deli','dairy eggs','canned goods','meat seafood','missing','dry goods pasta','pantry','frozen'],
                   ['other', 'pets','household','babies','personal care'],
                   ['beverages','breakfast','snacks'],
                   ['alcohol']];
fig, axes = plt.subplots(1, len(department_groups), figsize=(14, 6)) 
for i in range(len(department_groups)):
    sns.lineplot(data=df[(df['department'].isin(department_groups[i]))], 
                 x="day_of_week", y="norm_number_of_purchase", hue="department", ax=axes[i])
```
{{< figure src="/images/Instacart040647.png" class="round" >}}
   
By replacing `order_dow` with `order_hour_of_day`, the same code can be used to analyze order volume across hours of the day.

{{< figure src="/images/Instacart040718.png" class="round" >}}

insight:
just observing plot of day of week vs order volume for different departments is hard to interpret as there are 21 departmentas. however after careful observation, when plots are normalized and grouped together four kind of main patterns can be seen
- peroduce daity egs pantry meat seafood frozen deli canned goods dry good pasta etc are bought on weekend times, especially a lot on sunday and we see a 43% drop at Thursday
- babies hourhold personal care pet related product show a very similar pattern with dsunday peak however the midweek dip near thursday is lesser, 25%. Thus day to day purchase pattern is more unifrorm here
- beverage breakfast snack related items have a  peak on Monday nstead and another small peak on Friday
- alcohol has a very different pattern all of these, they get stocked ob Friday with a strong peak with sunday being the dip for alcohol purchase

From similar grouping for the day of week vs order volume plots we find
- In each department the allmost sme purchase is done between 9-10 am to 3-4 pm. however in that region some peaks can be seen
- beverage breakfast snacks have a peak at morning time around 10 pm
- produce dairy eggs household has a shorter peak at morning time.
- bakery canned goods deli meat seafpod panry and pets department has uniform purchase in this range
- dry goods pasta and frozen department has a small peak at 3 pm 
- alcohol has a even larger peak around 3pm

ocerall in conclusion we would say beverage breakfast snacks and alcohol are department groups that diverge strongly in purchase behaviour from other departments: alchold having a 3pm and Friday peaks, and beverage breakfast snacks has morning peaks with Monday peaks and shorter Friday peaks




# Variation in Reorder Ratio


{{< figure src="/images/Instacart173811.png" class="round" >}}




# Fake date
{{< figure src="/images/Instacart034656.png" class="round" >}}


