---
title: AdventureWorks Sales Analytics Dashboard
summary: 
date:
authors:
  - admin
sidebar:
  left: true
  widgets:
    - type: tag_cloud 
tags:
  - Excel
  - Pivot Table and Pivot Charts
  - PowerQuery
  - PowerPivot 
  - M language
  - DAX
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
    font-size: 0.8rem;   /* smaller font size */
    line-height: 1.1;    /* optional: adjust line spacing */
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


<video controls autoplay muted loop playsinline width="100%">
  <source src="/videos/dashboard_excel2.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>




<a href="/uploads/AdventureWorks_dashboard_2015-2017.xlsx" download style="display:inline-flex;align-items:center;gap:8px;padding:10px 18px;background:#007acc;color:white;border-radius:6px;text-decoration:none;font-weight:500;">
Dashboard File <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 5v14"/><path d="M5 12l7 7 7-7"/><path d="M5 19h14"/></svg>
</a>


<a href="/videos/dashboard_excel2.mp4" download style="display:inline-flex;align-items:center;gap:8px;padding:10px 18px;background:#007acc;color:white;border-radius:6px;text-decoration:none;font-weight:500;">
Demo Video <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 5v14"/><path d="M5 12l7 7 7-7"/><path d="M5 19h14"/></svg>
</a>


I built a Excel-based dashboard using the [AdventureWorks dataset from Kaggle](https://www.kaggle.com/datasets/ukveteran/adventure-works). The dataset came as several CSV files: Calendar, Customers, Product Categories, Product Subcategories, Products, Territories, and three years of Sales (2015–2017). My goal was to clean and model the data with Power Query and Power Pivot, then design an interactive dashboard highlighting sales trends and customer demographics.

### Data Preparation in Power Query

All CSV files were imported through Data → Get Data → From File (except sales files).

● **Territory Table**: I brought all source files into Power Query and applied a series of transformations. Most of the auto-generated steps (header promotion, simple type changes) weren't reused in the write-up, but I kept the parts where I performed non-trivial logic.
``` m
Merged_region_country_name =  
       Table.AddColumn(Previous_step, "Clean_region_name", 
                       each if [Region] = [Country] then [Region] 
                              else [Country]&" ("&[Region]&")" 
                       ),
Short_region_country_name = 
       Table.TransformColumns(Merged_region_country_name,
                              {{"Clean_region_name",
                              each
                                     let
                                     str  = _,
                                     rep1 = Text.Replace(str, "United Kingdom", "U.K."),
                                     rep2 = Text.Replace(rep1, "United States", "U.S.")
                                     in
                                     rep2,
                              type text
                              }}
                              )
```
● **Calendar table**: I parsed all date columns, including the calender table with US locale handling. I also added a year column in Calender table.
``` m
ParseUSDate = 
       Table.TransformColumns(Previous_Step, 
                              {{"Date", each Date.From(DateTimeZone.From(_, "en_US")), 
                              type date}})
```
``` m
AddYearColumn = 
       Table.AddColumn(ParseUSDate, "Year", each Date.Year([Date]), Int64.Type)
```
● **Customer Table**: Here, I formatted the Annual Income column, calculated customer age from birthday, and created age groups.
``` m
CleanIncome =  
       Table.TransformColumns(Prev, {{"AnnualIncome($)", 
                              each Number.FromText(Text.Remove(_, "$")), 
                              type number}}
                             ),
IncomeBand =
       Table.AddColumn(CleanIncome, "Annual_Income",
                       each "$ " & Number.ToText(Number.IntegerDivide([AnnualIncome($)], 1000)) & " K",
                       type text)
AgeCalc = 
       Table.AddColumn(IncomeBand, "Age",
                       each Duration.Days(#date(2017, 6, 30) - Date.From([BirthDate])) / 365,
                       type number),
RoundAge = Table.TransformColumnTypes(AgeCalc, {{"Age", Int64.Type}}),
AgeGroup =
       Table.AddColumn(RoundAge, "Age_Group",
                       each if [Age] < 40 then "<40"
                              else if [Age] < 50 then "40-49"
                              else if [Age] < 60 then "50-59"
                              else if [Age] < 70 then "60-69"
                              else if [Age] < 80 then "70-79"
                              else ">=80",
                       type text)
```
● **Sales Files (2015–2017)**: The Sales files were brought in differently because they needed to be merged. I used Data → Get Data → From Folder to load the entire folder. A text filter kept only files whose names contained "Sales". After that, I combined the files into a single fact table called fSales.

### Data Modeling in Power Pivot

After loading all queries, I modeled the data in powerpivot:
- **Fact Table**: fSales
- **Dimension tables**: dCalendar, dCustomers, dProducts, dProduct_Subcategories, dProduct_Categories, dTerritories

Then relationships were set up as follows:

{{< figure src="/images/excel/schema.png" class="round" >}}

DAX:

### "Revenue, Profit & Volume" page of the Dashboard

The first page focused on business metrics. 

**KPI cards:**
- **Total Revenue (YTD)**
- **Total Profit (YTD)**
- **Profit Margin (YTD)**
- **Units Sold (YTD)**

Each KPI displayed the year-over-year growth (2017 vs 2016) using the same YTD window (Jan–Jun) for consistency. Conditional formatting was addded to color increase in green and decrease in red.

**Visualizations:**
- **Monthly Revenue Trend:** area charts for 2015 and 2016. A thin line for 2017 with a highlighted endpoint + data label. 
- **Revenue by Category, Subcategory, and Products:** Bar charts showing top performers.
- **Revenue by Country:** A filled map chart, with data labels.

**Slicers:**
- **Category, Subcategory and Region slicers**
- **Metric Switching:** Users may want to switch between revenue, profit, and volume, since the highest-revenue item isn’t always the most profitable. I built duplicate sheets using different metrics and linked them with hyperlinked buttons that look like slicers, to keep the dashboard consistent.

### "Customer Demographics" page of the Dashboard

**KPI cards:**
- **Number of Customers (YTD)**
- **Average Order Value (YTD)**
- **Revenue per Customer (YTD)**
- **Orders per Customer (YTD)**
Each one includes a YoY comparison.

**Visualizations:**
- **Revenue by Income Band** (stacked bar chart)
- **Revenue Contribution by Gender** (donut chart)
- **Revenue by Marital Status & Home Ownership** (sunburst chart)
- **Revenue by Number of Children** (grouped bar chart)
- **Revenue by Occupation** (horizontal bar chart)
- **Revenue by Education Level** (horizontal bar chart)
- **Revenue by Age Group** (horizontal bar chart)

**Slicers:**
- **Category, Subcategory and Region slicers**


add the dax

add the insights

add downlaod video button