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

## Applying FP-Growth Algorithm for Association Rule Mining

**Support** measures how often an item or combination of items appears in the same order, divided by the total number of orders, indicating which items or item combinations are frequent enough to matter.

\\(x^2\\)

# Applying FP-Growth algorithm for associative rule mining
$a+b$

**Support:** Here, the support measures how often an item or combination of items together appears in the same order_ID in the dataset, divided by the total number of order_IDs

**FP growth algorithm:** This algorithm scans the dataset once to count the frequency (support) of each item. Items below the minimum support are discarded. Then it constructs a compact tree structure called an FP-Tree, where shared paths represent shared purchase patterns across orders. Then the tree is traversed from the bottom up, and frequent itemsets are generated. FP-Growth algorithm is a faster alternative to the Apriori algorithm. 

In the association rules (X->Y) calculated we find two other quantities other than support (frequency of XUY in orders/number of order), confidence and lift
confidence: definition and math equation please insert
Lift: 

I have set a minimum support of 0.5%. I have set conidence to be 0.03 so that only strong wenough rules are shown. I have set lift>1 to only find relationships that are unlikely to be random.
```python
# FP tree creation
from mlxtend.frequent_patterns import fpgrowth, association_rules
frequent_itemsets = fpgrowth(basket, min_support=0.005, use_colnames=True)

# association rule mining
rules = association_rules(frequent_itemsets, metric="lift", min_threshold=1)
rules = rules[rules['confidence'] > 0.3]
rules = rules.sort_values(by='lift', ascending=False)
display(rules[['antecedents', 'consequents', 'support',	'confidence',	'lift']	])
```
Output:
![instacart](/images/instacart232338.png)
