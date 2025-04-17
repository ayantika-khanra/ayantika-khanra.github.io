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
    margin-top: 1.2rem;   /* space before heading */
    margin-bottom: 0.6rem; /* space after heading */
  }

  p {
    font-size: 1rem;
    line-height: 1.4;
    margin-top: 0.4rem;     /* space before paragraph (usually small or 0) */
    margin-bottom: 0.8rem;  /* space after paragraph */
  }

  ul, ol {
    margin-top: 0.5rem;
    margin-bottom: 0.8rem;
  }

  li {
    margin-bottom: 0.4rem;
  }
</style>

*This exploratory data analysis analysis focuses on how explicit themes in music has evolved over time, musical characteristics of explicit music, and the role of genres in these trends.*

#### Dataset:
- I have used the [spotify 2000 dataset from Kaggle](https://www.kaggle.com/datasets/paradisejoy/top-hits-spotify-from-20002019) (year 2000-2020 included) for this analysis. which contains songs from the years 2000 to 2020. The dataset has approximately an equal number of songs per year, with the exception of the years <2001 and 2020, which had fewer entries. I filtered out songs from these years in Tableau for consistency.
- The popularity metric in the dataset is normalized.


#### Driving question:


The first step in the analysis was to simply track how the share of explicit songs changed over time. By plotting the percentage of explicit tracks per year from 2000 to 2020, a distinct U-shaped trend emerged. There was a decline in explicit content leading up to 2015, followed by a steep rise in the years after. The increase post-2015 is particularly notable and more rapid than the previous decline.





- Each music in this dataset has 1-3 genre identifiers, as many songs have multiple genre influences. In this 






