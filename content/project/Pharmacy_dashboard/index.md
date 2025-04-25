---
title: Dashboard for sales in a Pharmacy
summary: 
date: 2025-04-17
authors:
  - admin
tags:
  - Data visualization
  - Tableau
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

*This interactive dashboard provides a detailed view of pharmacy sales trends from a single pharmacy, using six years of data (2014–2019) covering sales of drugs classified into eight ATC categories.*

[Dataset Link](https://www.kaggle.com/datasets/milanzdravkovic/pharma-sales-data/data)

Note: No revenue information was provided, only the number of units sold.
Thus, the dashboard is focused on stocking and staffing needs rather than revenue analysis.

#### Dashboard Features:

- **Last 30 Days Snapshot:** Displays total units sold, units sold per drug category (to identify top-selling drugs), and the average number of units sold per day.

- **Sales Forecast:** Forecasts upcoming sales based on historical data for selected medicine categories, helping management better plan inventory.

- **Seasonal Sales Trend Analysis:** Sales trends across different months visualized using a heatmap. This helps (1) plan pharmacist staffing based on expected total demand, and (2) quickly identify seasonal patterns in medicine sales category-wise and allows pharmacies to stock up appropriately and .

- **Hourly Sales Trend:** Sales trends are broken down by hour, can be filtered for specific weekdays, months and years. Helps better scheduling of pharmacist shifts, lunch breaks, and closing hours based on customer traffic patterns.

Interactivity:Users can filter by weekday, year, month, and medicine category to customize their analysis.




