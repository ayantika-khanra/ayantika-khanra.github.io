---
title: Trends of Explicit Themes  in Music over Two Decades using Spotify’s Top 2000
summary: 
date: 2025-04-17
authors:
  - admin
tags:
  - Data visualization
  - Exploratory data analysis
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
</style>


*This exploratory data analysis analysis focuses on how explicit themes in music has evolved over time, musical characteristics of explicit music, and the role of genres in these trends.*

#### Dataset:
I have used the [spotify 2000 dataset from Kaggle](https://www.kaggle.com/datasets/paradisejoy/top-hits-spotify-from-20002019) (year 2000-2020 included) for this analysis. which contains songs from the years 2000 to 2020. The dataset has approximately an equal number of songs per year, with the exception of the years <2001 and 2020, which had fewer entries. I filtered out songs from these years in Tableau for consistency. The popularity metric in the dataset is normalized.


#### Driving question:

When the percentage of explicit tracks per year from 2000 to 2020 is plotted, we see the percentage of explicit music has decreased till 2015, however it has increased after that quiet rapidly. In 2019 the percentage of explicit track was close to 50%. This made me curious about this specific trend, and I wanted to dive deeper. 

<div class='tableauPlaceholder' id='viz1744856667295' style='position: relative; width: 400px; height: 300px; margin: auto;'>
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
  vizElement.style.width = '400px';
  vizElement.style.height = '300px';
  var scriptElement = document.createElement('script');
  scriptElement.src = 'https://public.tableau.com/javascripts/api/viz_v1.js';
  vizElement.parentNode.insertBefore(scriptElement, vizElement);
</script>





- Each music in this dataset has 1-3 genre identifiers, as many songs have multiple genre influences. In this 






