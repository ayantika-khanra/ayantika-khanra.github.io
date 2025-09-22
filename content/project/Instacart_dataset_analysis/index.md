---
title: Market Basket Analysis with FP-Growth on Instacart Data
summary: 
date: 2025-08-29
authors:
  - admin
tags:
  - Data Visualization
  - Market Basket Analysis
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
    font-size: 1.0rem;   /* smaller font size */
    line-height: 1.4;    /* optional: adjust line spacing */
  }
</style>


Understanding customer purchasing behavior is important for retail businesses. Market Basket Analysis (MBA) is performed to uncover patterns in what products are frequently bought together. Here, I will discuss how I applied FP-Growth algorithm for mining frequent itemsets on a retail dataset to generate insights into customer purchase pattern.

## Data Preparation

To focus on the most popular products, I only selected the top 1000 most frequently purchased products for this analysis, and I have used `order_products__train.csv` for this analysis. Each order was then transformed into a basket format, where rows represent individual orders IDs and columns represent product names. A value of 1 indicates that the specific product was purchased in that order ID.

```python
import pandas as pd

# Load relevant datasets
orders_pt = pd.read_csv("order_products__train.csv")
products = pd.read_csv("products.csv")

# Finding the Top 1000 products purchased
histogram_order=orders_pt.groupby(['product_id'])['product_id'].count()
top_N_products=(histogram_order.sort_values(ascending=False).
                iloc[0:1000].index.to_list())

# Saving all order IDs before modifying the dataframe
all_order_IDs = orders_pt['order_id'].unique() 

# Selecting only rows containing atleast one of the top 1000 products
selected_rows=orders_pt['product_id'].isin(top_N_products)
orders_pt=orders_pt.loc[selected_rows]
orders_pt.reset_index(drop=True, inplace=True)

# Joining the orders_pt and products dataframe on product_id column,
# and dropping the unnecessary columns
orders_pt = orders_pt.merge(products, on='product_id', how='left')
orders_pt=orders_pt.drop(columns=['product_id','add_to_cart_order', 
                                  'reordered', 'aisle_id','department_id'])

# Creating a multi-index Pandas Series with order_id and product_name 
# as index. Values show purchase count in that order 
# (converted to 1 for any purchase > 1)
orders_pt=orders_pt.groupby(['order_id','product_name'])['product_name'].count()
orders_pt=orders_pt.apply(lambda x: 1 if x>1 else x)

# Unstacking the grouped orders_pt to create a basket dataframe
# Rows = orders, columns = products, values = 0/1 indicating 
# if product was purchased
basket=orders_pt.unstack().fillna(0).astype('int8')
basket = basket.reindex(all_order_IDs, fill_value=0)
```
Now, the `basket` dataframe is ready to be used in FP growth algorithm from mlxtend library.

Applying FP-Growth

FP-Growth algorithm is a faster alternative to the Apriori algorithm. Unlike Apriori, it doesn’t generate candidate sets, which makes it much faster for large datasets. I set a minimum support of 0.1% to capture patterns that appear frequently enough.
![instacart](instacart232338.png)
