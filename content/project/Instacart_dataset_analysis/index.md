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
</style>

<div class='tableauPlaceholder' id='viz1745585415094' style='position: relative'><noscript><a href='#'><img alt='Dashboard 2 ' src='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Bo&#47;Book2_17454320271010&#47;Dashboard2&#47;1_rss.png' style='border: none' /></a></noscript><object class='tableauViz'  style='display:none;'><param name='host_url' value='https%3A%2F%2Fpublic.tableau.com%2F' /> <param name='embed_code_version' value='3' /> <param name='site_root' value='' /><param name='name' value='Book2_17454320271010&#47;Dashboard2' /><param name='tabs' value='no' /><param name='toolbar' value='yes' /><param name='static_image' value='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Bo&#47;Book2_17454320271010&#47;Dashboard2&#47;1.png' /> <param name='animate_transition' value='yes' /><param name='display_static_image' value='yes' /><param name='display_spinner' value='yes' /><param name='display_overlay' value='yes' /><param name='display_count' value='yes' /><param name='language' value='en-US' /></object></div>                <script type='text/javascript'>                    var divElement = document.getElementById('viz1745585415094');                    var vizElement = divElement.getElementsByTagName('object')[0];                    if ( divElement.offsetWidth > 800 ) { vizElement.style.width='920px';vizElement.style.minHeight='713px';vizElement.style.maxHeight='887px';vizElement.style.height=(divElement.offsetWidth*1)+'px';} else if ( divElement.offsetWidth > 500 ) { vizElement.style.width='920px';vizElement.style.minHeight='713px';vizElement.style.maxHeight='887px';vizElement.style.height=(divElement.offsetWidth*1)+'px';} else { vizElement.style.width='100%';vizElement.style.height='1877px';}                     var scriptElement = document.createElement('script');                    scriptElement.src = 'https://public.tableau.com/javascripts/api/viz_v1.js';                    vizElement.parentNode.insertBefore(scriptElement, vizElement);                </script>

Understanding customer purchasing behavior is important for retail businesses. Market Basket Analysis (MBA) is performed to uncover patterns in what products are frequently bought together. Here, I will discuss how I applied FP-Growth algorithm for mining frequent itemsets on a retail dataset to generate insights into customer purchase pattern.

## Data Preparation

To focus on the most popular products, I only selected the top 1000 most frequently purchased products for this analysis, and I have used `order_products__train.csv` for this analysis. Each order was then transformed into a basket format, where rows represent individual orders IDs and columns represent product names. A value of 1 indicates that the specific product was purchased in that order ID.

