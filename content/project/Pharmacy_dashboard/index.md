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


*This interactive dashboard provides a detailed view of pharmacy sales trends from a single pharmacy, using six years of data (2014–2019) covering sales of drugs classified into eight ATC categories.*

[Dataset Link](https://www.kaggle.com/datasets/milanzdravkovic/pharma-sales-data/data)

Note: No revenue information was provided, only the number of units sold.
Thus, the dashboard is focused on stocking and staffing needs rather than revenue analysis.

#### Dashboard Features:

- **Last 30 Days Snapshot:** Displays total units sold, units sold per drug category (to identify top-selling drugs), and the average number of units sold per day.

Sales Forecast:
Forecasts upcoming sales based on historical data for selected medicine categories, helping management better plan inventory and avoid stockouts.

Seasonal Sales Trend Analysis:
Sales trends across different months are visualized using a heatmap and bar chart.
This helps quickly identify seasonal patterns (e.g., increased cold medicine sales in winter) and allows pharmacies to stock up appropriately and plan pharmacist staffing based on expected demand.

Hourly Sales Trend:
Sales trends are broken down by hour and weekday.
This enables better scheduling of pharmacist shifts, lunch breaks, and closing hours based on real customer traffic patterns.

When the percentage of explicit tracks per year from 2000 to 2020 is plotted, we see the percentage of explicit music has a decreasing trend till 2014, however it has increased after that quiet rapidly. In 2018 the percentage of explicit track was close to 50%. This made me curious about this specific trend, and I wanted to dive deeper. 

<div class='tableauPlaceholder' id='viz1744856667295' style='position: relative; width: 600px; height: 400px; margin: auto;'>
  <noscript>
    <a href='#'>
      <img alt='Sheet 1' src='https://public.tableau.com/static/images/Bo/Book1_17448566116240/Sheet1/1_rss.png' style='border: none' />
    </a>
  </noscript>
  <object class='tableauViz' style='display:none;'>
    <param name='host_url' value='https%3A%2F%2Fpublic.tableau.com%2F' />
    <param name='embed_code_version' value='3' />
    <param name='site_root' value='' />
    <param name='name' value='Book1_17448566116240/Sheet1' />
    <param name='tabs' value='no' />
    <param name='toolbar' value='yes' />
    <param name='static_image' value='https://public.tableau.com/static/images/Bo/Book1_17448566116240/Sheet1/1.png' />
    <param name='animate_transition' value='yes' />
    <param name='display_static_image' value='yes' />
    <param name='display_spinner' value='yes' />
    <param name='display_overlay' value='yes' />
    <param name='display_count' value='yes' />
    <param name='language' value='en-US' />
  </object>
</div>

<script type='text/javascript'>
  var divElement = document.getElementById('viz1744856667295');
  var vizElement = divElement.getElementsByTagName('object')[0];
  vizElement.style.width = '600px';
  vizElement.style.height = '400px';
  var scriptElement = document.createElement('script');
  scriptElement.src = 'https://public.tableau.com/javascripts/api/viz_v1.js';
  vizElement.parentNode.insertBefore(scriptElement, vizElement);
</script>


#### Musical signature of explicit songs
Now I plotted different sonic qualities of explicit and non explicit music as a scatter plot, to understand how their music styles differ. I show only the sonic qualities where differences were seen.
- In the Energy vs speechiness scatter plot, we can see that explicit music has higher speechiness and lower energy, than non explicit music. 
- In the Danceability vs Tempo scatter plot, we can see that dancebility s higher for explicit songs, than non-explicit. The average tempo doesn't really change between explicit and non explicit songs. However in the scatter plot, two distinct cluster will be seen, one around 90 bpm, another around 100 bpm. As tempo data is clearly bimodal, it needs to be investigated differently than just averaging the tempo.


<div class='tableauPlaceholder' id='viz1744859182095' style='position: relative'><noscript><a href='#'><img alt='Energy vs Speechiness Scatter plot for explicit and non explicit songs ' src='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Bo&#47;Book1_17448566116240&#47;Sheet12&#47;1_rss.png' style='border: none' /></a></noscript><object class='tableauViz'  style='display:none;'><param name='host_url' value='https%3A%2F%2Fpublic.tableau.com%2F' /> <param name='embed_code_version' value='3' /> <param name='site_root' value='' /><param name='name' value='Book1_17448566116240&#47;Sheet12' /><param name='tabs' value='no' /><param name='toolbar' value='yes' /><param name='static_image' value='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Bo&#47;Book1_17448566116240&#47;Sheet12&#47;1.png' /> <param name='animate_transition' value='yes' /><param name='display_static_image' value='yes' /><param name='display_spinner' value='yes' /><param name='display_overlay' value='yes' /><param name='display_count' value='yes' /><param name='language' value='en-US' /><param name='filter' value='publish=yes' /></object></div>                <script type='text/javascript'>                    var divElement = document.getElementById('viz1744859182095');                    var vizElement = divElement.getElementsByTagName('object')[0];                    vizElement.style.width='100%';vizElement.style.height=(divElement.offsetWidth*0.75)+'px';                    var scriptElement = document.createElement('script');                    scriptElement.src = 'https://public.tableau.com/javascripts/api/viz_v1.js';                    vizElement.parentNode.insertBefore(scriptElement, vizElement);                </script>

<div class='tableauPlaceholder' id='viz1744862175239' style='position: relative'><noscript><a href='#'><img alt='Energy vs Speechiness Scatter plot for explicit and non explicit songs ' src='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Bo&#47;Book1_17448566116240&#47;Sheet13&#47;1_rss.png' style='border: none' /></a></noscript><object class='tableauViz'  style='display:none;'><param name='host_url' value='https%3A%2F%2Fpublic.tableau.com%2F' /> <param name='embed_code_version' value='3' /> <param name='site_root' value='' /><param name='name' value='Book1_17448566116240&#47;Sheet13' /><param name='tabs' value='no' /><param name='toolbar' value='yes' /><param name='static_image' value='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Bo&#47;Book1_17448566116240&#47;Sheet13&#47;1.png' /> <param name='animate_transition' value='yes' /><param name='display_static_image' value='yes' /><param name='display_spinner' value='yes' /><param name='display_overlay' value='yes' /><param name='display_count' value='yes' /><param name='language' value='en-US' /><param name='filter' value='publish=yes' /></object></div>                <script type='text/javascript'>                    var divElement = document.getElementById('viz1744862175239');                    var vizElement = divElement.getElementsByTagName('object')[0];                    vizElement.style.width='100%';vizElement.style.height=(divElement.offsetWidth*0.75)+'px';                    var scriptElement = document.createElement('script');                    scriptElement.src = 'https://public.tableau.com/javascripts/api/viz_v1.js';                    vizElement.parentNode.insertBefore(scriptElement, vizElement);                </script>

See the interactive plot below





- Each music in this dataset has 1-3 genre identifiers, as many songs have multiple genre influences. In this 






