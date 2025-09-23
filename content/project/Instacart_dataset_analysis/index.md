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


Understanding customer purchasing behavior is important for retail businesses. Market Basket Analysis (MBA) is performed to uncover patterns in what products are frequently bought together. Here, I will discuss how I applied FP-Growth algorithm for mining frequent itemsets on a retail dataset to generate insights into customer purchase pattern.

#### Data Preparation

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

#### Applying FP-Growth Algorithm for Association Rule Mining

**Support** measures how often an item or combination of items appears in the same order, divided by the total number of orders, indicating which items or item combinations are frequent enough to matter.

**FP growth algorithm:** The frequent pattern growth algorithm is an efficient method for finding frequent itemsets. Here,  the dataset once to count the frequency, and thus support of each item. Items having support values below a chosen threshold are discarded. Then, an FP-Tree is bulit, which is a compact branched structure where each branch represent purchase patterns across orders. Then the tree is traversed bottom-up to find all frequent itemsets.

**Association Rules:** After frequent itemsets are found, we can generate association rules of the form $X\rightarrow Y$ (if X is bought, Y is likely bought as well). Here item/itemset X is called the antecedant, and Y the consequent. Other than $Support(X→Y)$ = # of orders containing $X\cap Y$/ Total # of orders, two other quantites are used to queantify these association rules:

**Confidence:** A measure of how often Y is purchased when X is purchased:
$$\scriptstyle Confidence(X \rightarrow Y)= \frac{Support(X \cap Y)}{Support(X)}$$

**Lift:** Measures how much more likely X and Y occur together than if they were independent:

$$\scriptstyle Lift(X \rightarrow Y)= \frac{Support(X \cap Y)}{Support(X)\times Support (Y)}$$

High confidence indicates a strong predictive relationship, and a lift > 1 indicates that a positive association exists that is stronger than a random chance.

For this analysis, I have set:
- Minimum support = 0.5% 
- Minimum confidence = 0.3, to ignore weak rules
- Lift > 1, to view only non-random associations

```python
# Find frequent itemsets using FP-Growth
from mlxtend.frequent_patterns import fpgrowth, association_rules
frequent_itemsets = fpgrowth(basket, min_support=0.005, use_colnames=True)

# Find association rules from frequent itemsets
rules = association_rules(frequent_itemsets, metric="lift", min_threshold=1)
rules = rules[rules['confidence'] > 0.3]
rules = rules.sort_values(by='lift', ascending=False)
display(rules[['antecedents', 'consequents', 'support',	'confidence',	'lift']	])
```
Output:
![instacart](/images/instacart232338.png)

Customers buying Organic Strawberries and Organic Hass Avocado together also buy Bag of Organic Bananas with 46% confidence and a lift of 3.91, and buying Organic Raspberries predicts a purchase of Organic Strawberries with 30% confidence and lift 3.63. Many rules have Bananas as a common consequent. 

**Insights like these can guide targeted marketing, promotions, product placement, and inventory planning.**

Most of the association rules were initially dominated by produce items due to their high purchase frequency. When I performed FP growth on non-produce items, the high confidence and high lift relationships were between different flavors or variations of the same product. For example, `Lime Sparkling Water → Grapefruit Sparkling Water` purchase pattern has a confidence of 0.25 and a lift of 10, while `(Passionfruit Sparkling Water, Lime Sparkling Water) → Grapefruit Sparkling Water`  purchase pattern has a much higher confidence of 0.72 and lift of 28. Similar high lift relationships can be found between products like `Chocolate Sea Salt → Coconut Chocolate Bar` (Conf: 0.32, Lift: 226), `Organic Pinto Beans →  Organic Black Beans` (Conf: 0.24, Lift: 20), `Apple Pie Fruit & Nut Food Bar → Cherry Pie Fruit & Nut Bar` (Conf: 0.31, Lift: 212), `Broccoli & Apple Stage 2 Baby Food → Blueberry Pear & Purple Carrot Stage 2 Baby Food` *Conf: 0.31, Lift: 131)

-----

frozenset({'85% Lean Ground Beef'})	frozenset({'Boneless Skinless Chicken Breasts'})	0.004892957	0.015913542516138374	0.000785007	0.16043613707165108	10.081736163330588

frozenset({'Zero Calorie Cola'})	frozenset({'Soda'})	0.002644636	0.011485492611025157	0.001036514	0.3919308357348703	34.12399006366065

frozenset({'Sea Salt Pita Chips'})	frozenset({'Original Hummus'})	0.00538835	0.021782042390384806	0.0009603	0.1782178217821782	8.181869201615752

While these rules are practically useful, grouping items by aisle can help understand broader patterns of which product categories drive purchases in other categories.

# FP growth Algorithm Applied at Aisle Level Grouping  

Due to the customers buying produce with a high frequency most of th associative rules mined were produce based. removing produce items from the dataset helped us mine rules from rest of the departments, however what we noticed is customers oftem buy different falvour of the same product with high lift and high confidence,


 thus once again dominating the mined results. for ecample  antecedents	consequents	antecedent support	consequent support	support	confidence	lift	representativity	leverage	conviction	zhangs_metric	jaccard	certainty	kulczynski
760	(Cherry Pie Fruit & Nut Bar, Lemon Fruit & Nut...	(Apple Pie Fruit & Nut Food Bar)	0.000335	0.001921	0.000236	0.704545	366.836129

(Baby Food Stage 2 Pumpkin Banana, Spinach Pea...	(Baby Food Stage 2 Blueberry Pear & Purple Car...



(Total 0% Blueberry Acai Greek Yogurt, Fat Fre...	(Total 0% Raspberry Yogurt)

while these rules remain important practically, we coul use the aisle groupijngs to draw larger scale conclusions from our own understanding, to realize what product group causes users to buy the second product group.
