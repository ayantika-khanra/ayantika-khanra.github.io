---
title: Market Basket Analysis with FP-Growth on Instacart Data
summary: 
date: 2025-08-29
authors:
  - admin
tags:
  - Data Visualization
  - Exploratory Data Analysis
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
    font-size: 0.2rem;   /* smaller font size */
    line-height: 0.2;    /* optional: adjust line spacing */
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
top_N_products=histogram_order.sort_values(ascending=False).iloc[0:1000].index.to_list()

# Selecting only rows containing atleast one of the top 1000 products
all_order_IDs = orders_pt['order_id'].unique()
selected_rows=orders_pt['product_id'].isin(top_N_products)
orders_pt=orders_pt.loc[selected_rows]
orders_pt.reset_index(drop=True, inplace=True)

orders_pt = orders_pt.merge(products, on='product_id', how='left')
orders_pt=orders_pt.drop(columns=['product_id','add_to_cart_order', 'reordered', 'aisle_id','department_id'])
orders_pt=orders_pt.groupby(['order_id','product_name'])['product_name'].count()
```
