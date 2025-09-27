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


### 4. Seasonality in Orders

To understand seasonal purchasing patterns, we analyze product purchase variation with:
- **Hour of the day**
- **Day of the week**
- **Week of the year:** not directly available in this dataset. However, a proxy can be inferred from the `days since prior order` column to approximate yearly buying trends.


#### 4.1 Weekly and Daily Variations in Order Volume

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















#### 4.2 Weekly and Daily Variations in Basket Size

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



#### 4.3 Weekly and Daily Variations in Order Volume from Different Departments

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



{{< alert tip "Insight" "<svg class='alert-icon' xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'><path fill='#28a745' d='m17.989,4.341l-1.709-1.041L18.266.04l1.709,1.041-1.985,3.26Zm5.161.206l-3.331,1.504.822,1.822,3.331-1.504-.822-1.822Zm-5.54,1.75c1.541,1.517,2.39,3.542,2.39,5.703,0,2.295-.99,4.481-2.718,5.999-.814.717-1.282,1.833-1.282,3.064v2.937h-8v-3.07c0-1.155-.453-2.211-1.244-2.897-1.836-1.593-2.838-3.898-2.75-6.326.149-4.179,3.675-7.636,7.858-7.705,2.15-.042,4.205.779,5.746,2.296Zm-3.61,14.767c0-.362.036-.716.092-1.063h-4.169c.046.305.077.614.077.93v1.07h4v-.937Zm4-9.063c0-1.621-.637-3.141-1.793-4.277-1.155-1.138-2.691-1.751-4.31-1.722-3.138.052-5.781,2.644-5.894,5.777-.065,1.82.687,3.549,2.062,4.744.481.417.879.919,1.188,1.478h1.745v-4.184c-1.161-.414-2-1.514-2-2.816h2c0,.552.448,1,1,1s1-.448,1-1h2c0,1.302-.839,2.402-2,2.816v4.184h1.767c.312-.569.713-1.079,1.195-1.503,1.295-1.139,2.038-2.777,2.038-4.497ZM7.725,3.3L5.739.04l-1.709,1.041,1.985,3.26,1.709-1.041ZM.854,4.547L.032,6.369l3.33,1.504.822-1.822-3.33-1.504Z'/></svg>" >}} Looking at order volume by department across the week, the raw plots are hard to interpret because there are 21 departments. By normalizing the data and grouping departments, the following purchasing patterns can be seen
- Staple goods (`produce`, `dairy eggs`, `pantry`, `meat seafood`, `frozen`, `deli`, `canned goods`, `dry goods pasta`) are purchased the **most on Sundays**, with a **~43% midweek dip** in purchase around Thursdays.
- `Household`, `babies`, `household`, `personal care`, `pet` departments also see **purchase peak on Sundays**, but the **midweek drop** around Thursday is smaller **~25%**.
- `beverages`, `breakfast`, `snacks` departments show a purchase **peak on Monday** instead, with a secondary peak on Fridays.
- `Alcohol` purchase pattern is very different with purchases **peak on Fridays**, and dip on Sundays.

From hourly patterns within each department, we observe that across categories, most purchases happen between **9 AM and 4 PM. Within this window,** some more pattern can be seen:
- `Beverages`, `breakfast`, `snacks` department sees a **peak** in purchase in morning around **10 AM**.
- `Produce`, `dairy eggs` and `household` department sees a **smaller morning peak** in purchase.
- `Bakery`, `canned goods`, `deli`, `meat seafood`, `pantry` and `pets` shows relatively **flat demand across this time period**.
- `Dry goods pasta` & `frozen` department has a **minor peak** in purchase around **3 PM**.
- `Alcohol` department has a **bigger spike** in purchase at **3 PM**.

**In conclusion most departments see a purchase peak in Sunday, but, `beverages`, `breakfast`, `snacks`, and `alcohol` are an anomaly: Alcohol → strong Friday + 3 PM peak, and Beverages, breakfast, snacks → 10 AM + Monday and a secondary Friday peak.** {{< /alert >}}





#### 4.4 Variation in Reorder Ratio
The reorder ratio tells us how often customers repurchase products they’ve bought before. It is calculated by comparing the number of items reordered to the total number of items in a basket (or a set of baskets).

```python
# Querying the database to calculate reorder ratio by day of week and hour of day
query="""
SELECT 
    o.order_dow AS day_of_week, 
    o.order_hour_of_day AS hour_of_day,
    SUM(ot.reordered)*1.0 / COUNT(*) AS reorder_ratio
FROM order_products__all AS ot
JOIN orders AS o
    ON o.order_id = ot.order_id
GROUP BY o.order_hour_of_day, o.order_dow
"""
df = pd.read_sql(query, engine)
df['day_of_week']=df['day_of_week'].map(day_of_week_dict)

# Lineplot visualization
sns.lineplot(df, y='reorder_ratio', x='hour_of_day', hue='day_of_week', palette='viridis', linewidth=3)
```

{{< figure src="/images/Instacart173811.png" class="round"  width="50%"  >}}

{{< alert tip "Insight" "<svg class='alert-icon' xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'><path fill='#28a745' d='m17.989,4.341l-1.709-1.041L18.266.04l1.709,1.041-1.985,3.26Zm5.161.206l-3.331,1.504.822,1.822,3.331-1.504-.822-1.822Zm-5.54,1.75c1.541,1.517,2.39,3.542,2.39,5.703,0,2.295-.99,4.481-2.718,5.999-.814.717-1.282,1.833-1.282,3.064v2.937h-8v-3.07c0-1.155-.453-2.211-1.244-2.897-1.836-1.593-2.838-3.898-2.75-6.326.149-4.179,3.675-7.636,7.858-7.705,2.15-.042,4.205.779,5.746,2.296Zm-3.61,14.767c0-.362.036-.716.092-1.063h-4.169c.046.305.077.614.077.93v1.07h4v-.937Zm4-9.063c0-1.621-.637-3.141-1.793-4.277-1.155-1.138-2.691-1.751-4.31-1.722-3.138.052-5.781,2.644-5.894,5.777-.065,1.82.687,3.549,2.062,4.744.481.417.879.919,1.188,1.478h1.745v-4.184c-1.161-.414-2-1.514-2-2.816h2c0,.552.448,1,1,1s1-.448,1-1h2c0,1.302-.839,2.402-2,2.816v4.184h1.767c.312-.569.713-1.079,1.195-1.503,1.295-1.139,2.038-2.777,2.038-4.497ZM7.725,3.3L5.739.04l-1.709,1.041,1.985,3.26,1.709-1.041ZM.854,4.547L.032,6.369l3.33,1.504.822-1.822-3.33-1.504Z'/></svg>" >}}
Reorders are most **common around 9 AM**, and they **peak on Sundays**.{{< /alert >}}



#### Variation of order volume throughout the year
The dataset does not provide exact calendar dates, but it does record the number of days since each customer’s previous order. By cumulatively summing these values within each user’s history, we can approximate a “pseudo-date” for each order:
$\text{rolling\_days}_n = \sum_{i=1}^{n} \text{days\_since\_prior\_order}_i $

To perform this analysis:
- I have restricted to users with long purchase histories (≥ 359 days).
- I have discarded users with any `day_since_last_order` record of 30 days, as this variable has been capped in this dataset at the value of 30. 
- Then this was converted to to week numbers =rolling_days/7.

These rolling days or week numbers are  not tied to exact calendar months, it only provides a relative measure of long-term purchasing cycles.

```python
# Calculating "rolling days" for each user
query="""
WITH order_table_with_rolling_days AS (
    SELECT 
        order_id, user_id, order_number, days_since_prior_order,
        SUM(COALESCE(days_since_prior_order, 0)) OVER (
            PARTITION BY user_id 
            ORDER BY order_number
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS rolling_days
    FROM orders
),
""""
# Selecting only users with >359 day of history and no long gaps between orders
+ """"
decision_table AS (
    SELECT user_id
    FROM order_table_with_rolling_days
    GROUP BY user_id
    HAVING NAX(days_since_prior_order) <30
    AND MAX(rolling_days)>359),
"""
#
+""""
filtered_order_table_with_rolling_days AS (
    SELECT otrd.* 
    FROM order_table_with_rolling_days AS otrd
    INNER JOIN decision_table AS dt
    ON dt.user_id=otrd.user_id)

SELECT d.department, a.aisle, 
       Ceil(rolling_days/7) AS week_number, 
       COUNT(*) AS number_of_purchase
FROM filtered_order_table_with_rolling_days AS o
JOIN order_products__all AS ot
    ON ot.order_id=o.order_id
JOIN products AS p
    ON p.product_id=ot.product_id
JOIN departments AS d
    ON d.department_id=p.department_id
JOIN aisles AS a
    ON a.aisle_id=p.aisle_id
WHERE CEIL(rolling_days/7)<=51
GROUP BY d.department, a.aisle, CEIL(rolling_days/7)
"""
df = pd.read_sql(query, engine); 
df['week_number']=df['week_number'].astype('int8')

# Removed week_number=0, corresponding to rolling day = 0.
#    Because, all users start here, thus it can can give inflated spike
# Removed week_number >= 52 
#    Because beyond 359 days we have incomplete data for some users.
df=df[(df['week_number']<=51) & (df['week_number']>0)]

# Scaling:
# divided purchases of a week in one aisle, by the total number of purchases made in that week
# Divided number of purchase per week plot with the average value
df['scaled_number_of_purchase']=df.groupby('week_number')['number_of_purchase'].transform(lambda x: x/x.sum())
df['scaled_number_of_purchase']=df.groupby('aisle')['scaled_number_of_purchase'].transform(lambda x: x/x.mean())

# Visualization
g=sns.relplot(data=df, x='week_number', y='scaled_number_of_purchase',
              kind="line", col="aisle", facet_kws={"sharey": False})
```


{{< figure src="/images/Instacart034656.png" class="round" >}}



