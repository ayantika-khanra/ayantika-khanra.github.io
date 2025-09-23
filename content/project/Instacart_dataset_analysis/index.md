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

Customers buying Organic Strawberries and Organic Hass Avocado together also buy Bag of Organic Bananas with 46% confidence and a lift of 3.91, and buying Organic Raspberries predicts a purchase of Organic Strawberries with 30% confidence and lift 3.63. Many rules have Bananas as a common consequent. Most of the association rules were initially dominated by produce items due to their high purchase frequency.

**Insights like these can guide targeted marketing, promotions, product placement, and inventory planning.**


#### Interesting relationships Other than these high support items, by reducing the support threshold reducing

When I performed FP growth on non-produce items, the high confidence and high lift relationships were between different flavors or variations of the same product. For example, 

Similar high lift relationships can be found between similar or different flavour of the same products like 
- `Lime Sparkling Water → Grapefruit Sparkling Water` (Conf: 0.25, Lift:  10)
- `(Passionfruit Sparkling Water, Lime Sparkling Water) → Grapefruit Sparkling Water` (Conf: 0.72, Lift: 28)
- `Total 2% Lowfat Greek Strained Yogurt With Blueberry → Total 2% with Strawberry Lowfat Greek Strained Yogurt` (Conf: 0.36, Lift: 49)
- `Chocolate Sea Salt → Coconut Chocolate Bar` (Conf: 0.32, Lift: 226)
- `Apple Pie Fruit & Nut Food Bar → Cherry Pie Fruit & Nut Bar` (Conf: 0.31, Lift: 212)
- `Organic Pinto Beans →  Organic Black Beans` (Conf: 0.24, Lift: 20)
- `Broccoli & Apple Stage 2 Baby Food → Blueberry Pear & Purple Carrot Stage 2 Baby Food` (Conf: 0.31, Lift: 131)



Other than these, notable relationships are found between not completely related products such as 85% Lean Ground Beef'})	frozenset({'Boneless Skinless Chicken Breasts'})	0.004892957	0.015913542516138374	0.000785007	0.16043613707165108	10.081736163330588

frozenset({'Sea Salt Pita Chips'})	frozenset({'Original Hummus'})	0.00538835	0.021782042390384806	0.0009603	0.1782178217821782	8.181869201615752

While these rules are practically useful, grouping items by aisle can help understand broader patterns of which product categories drive purchases in other categories.




Organic 90% Ground Beef	Organic Yellow Onion	0.211679	6.474165	0.000221
Tonic Water	Limes	0.278351	6.053720	0.000412
226108	Mild Diced Green Chiles	Limes	0.273764	5.953976	0.000549
Bag of Large Lemons	Hass Avocados	0.225434	12.996005	0.000297
Tortillas, Corn, Organic	Organic Hass Avocado	0.287805	5.177923	0.000450
Organic Large Brown Eggs	Organic Avocado	0.231018	4.091188	0.001090
Organic Tahini	Large Lemon	0.228261	3.681608	
Organic Egg Whites	Organic Baby Spinach	0.238609	3.199884	0.001517




#### Finding Connections at the Aisle Level Using Network Visualization

Instead of focusing on individual products, we now aggregate at the aisle level. Each `basket` dataframe now has `aisle` as columns and `order_ID` as rows. Values indicate whether that aisle was part of the order (0 → False, 1 → True).

```python
import pandas as pd

# Merge necessary dataframes
orders_pt = (pd.read_csv("order_products__train.csv").
             merge(pd.read_csv("products.csv"), on='product_id', how='left')
             .merge(pd.read_csv("aisles.csv"), on='aisle_id', how='left'))

# Drop columns that are not needed
orders_pt = orders_pt.drop(columns=['product_id','add_to_cart_order',
                                    'reordered', 'product_name', 
                                    'aisle_id','department_id'])

# Creating basket dataframe: each row is an order, each column is an aisle
orders_pt = orders_pt.groupby(['order_id','aisle'])['aisle'].count()
orders_pt = orders_pt.apply(lambda x: 1 if x>1 else x)
basket = orders_pt.unstack().fillna(0).astype('int8')

# Association rule mining
from mlxtend.frequent_patterns import association_rules, fpgrowth
frequent_itemsets = fpgrowth(basket, min_support=0.075, use_colnames=True)
rules = association_rules(frequent_itemsets, metric="lift", min_threshold=1)
rules = rules[(rules['confidence'] > 0.3) & (rules['lift'] > 1)]
```
Association rule mining can generate item-to-itemset or itemset-to-item relationships, but here we only focus on one aisle to one aisle relationships. This helps us create a clean network plot showing the strength of the relationships between aisles, without dealing with more complex multi-aisle combinations.
```python
# Each antecedent and consequent in the rules is a frozenset
# and we only keep rules where both the antecedent and consequent
# consist of a single item, representing a one-to-one aisle relationship.
rules_1to1 = rules[
    (rules['antecedents'].apply(lambda x: len(x)) == 1) &
    (rules['consequents'].apply(lambda x: len(x)) == 1)]
```
In the network plot, each antecedent and consequent is represented as a circular node. We want the node size to reflect the support of that aisle (frequency of the aisle in orders). So, we create `aisle_support_dict` to map each aisle to a node size based on its support.

```python
all_aisle_names=pd.concat([rules_1to1['antecedents'],rules_1to1['consequents']]).apply(lambda x: list(x)[0]).to_list()
node_size=pd.concat([rules_1to1['antecedent support'],rules_1to1['consequent support']])
node_size=node_size-node_size.min()*0.6; # normalization
node_size=(node_size/node_size.max()*0.5).to_list() # normalization
aisle_support_dict=dict(zip(all_aisle_names,node_size))
```
In the network plot, the nodes are aisles, and edges represent aisle-to-aisle relationships. Node are shown as circles, with its size representing aisle support. The edges/relationships are shown as arrows, with its width and color encoding the strength of the relationship: lift and confidence.
```python
import networkx as nx
G = nx.DiGraph()

for _, row in rules_1to1.iterrows():
    a = list(row['antecedents'])[0]
    c = list(row['consequents'])[0]
    conf = row['confidence']
    lift = row['lift']
    supp = row['support']

    # We Use lift as a weight, higher lift pulls nodes closer in spring layout
    G.add_edge(a, c, weight=lift, confidence=conf, support=supp, lift=lift)


plt.figure(figsize=(12, 10))
pos = nx.spring_layout(G, k=1.5, seed=42)

edges = G.edges(data=True)
labels = {node: node.replace(" ", "\n") for node in G.nodes()}

# Node sizes scaled according to support
node_sizes = [aisle_support_dict[node] * 2000 for node in G.nodes()]

# Draw nodes and labels
nx.draw_networkx_nodes(G, pos, node_color="lightblue", edgecolors="gray", node_size=node_sizes)
nx.draw_networkx_labels(G, pos, labels=labels, font_size=10)

# Draw edges: line width proportional to lift, color proportional to confidence
nx.draw_networkx_edges(
    G, pos, edgelist=edges,
    width=[(d['lift']-1)*10 for (_,_,d) in edges],
    edge_color=[d['confidence'] for (_,_,d) in edges],
    edge_cmap=plt.cm.Greens, alpha=0.7,
    edge_vmin=0, edge_vmax=rules_1to1['confidence'].max(),
    connectionstyle="arc3,rad=0.05", arrows=True, arrowstyle='-|>', arrowsize=15
)

plt.axis("off")
plt.show()
```
![instacart](/images/instacart032914.png)

