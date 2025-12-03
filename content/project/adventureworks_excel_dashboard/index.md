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





I built a Excel-based dashboard using the AdventureWorks dataset from [Kaggle](https://www.kaggle.com/datasets/ukveteran/adventure-works). The dataset came as several CSV files: Calendar, Customers, Product Categories, Product Subcategories, Products, Territories, and three years of Sales (2015–2017). My goal was to clean and model the data with Power Query and Power Pivot, then design an interactive dashboard highlighting sales trends and customer demographics.

### Data Preparation in Power Query

All CSV files were imported through Data → Get Data → From File (except sales files).
- **Territory Table**: I brought all source files into Power Query and applied a series of transformations. Most of the auto-generated steps (header promotion, simple type changes) weren't reused in the write-up, but I kept the parts where I performed non-trivial logic.
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
- **Calendar table**: I parsed all date columns, including the calender table with US locale handling. I also added a year column in Calender table.
``` m
ParseUSDate = Table.TransformColumns(Previous_Step, {{"Date", each Date.From(DateTimeZone.From(_, "en_US")), type date}})
```
``` m
AddYearColumn = Table.AddColumn(ParseUSDate, "Year", each Date.Year([Date]), Int64.Type)
```

- **Customer Table**: Here, I formatted the Annual Income column, calculated customer age from birthday, and created age groups.
``` m
CleanIncome =  Table.TransformColumns(Prev, {{"AnnualIncome($)", 
                                              each Number.FromText(Text.Remove(_, "$")), 
                                              type number}}
                                     ),
IncomeBand =Table.AddColumn(CleanIncome, "Annual_Income",
                            each "$ " & Number.ToText(Number.IntegerDivide([AnnualIncome($)], 1000)) & " K",
                            type text)

AgeCalc = Table.AddColumn(IncomeBand, "Age",
                          each Duration.Days(#date(2017, 6, 30) - Date.From([BirthDate])) / 365,
                          type number),
RoundAge = Table.TransformColumnTypes(AgeCalc, {{"Age", Int64.Type}}),
AgeGroup =Table.AddColumn(RoundAge, "Age_Group",
                         each if [Age] < 40 then "<40"
                              else if [Age] < 50 then "40-49"
                              else if [Age] < 60 then "50-59"
                              else if [Age] < 70 then "60-69"
                              else if [Age] < 80 then "70-79"
                              else ">=80",
                         type text)
```

- **Sales Files (2015–2017)**: The Sales files were brought in differently because they needed to be merged. I used Data → Get Data → From Folder to load the entire folder. A text filter kept only files whose names contained "Sales". After that, I combined the files into a single fact table called fSales.


### Data Modeling in Power Pivot

After loading all queries, I modeled the data in powerpivot:
- **Fact Table**: fSales
- **Dimension tables**: dCalendar, dCustomers, dProducts, dProduct_Subcategories, dProduct_Categories, dTerritories

Then relationships were set up as follows:

{{< figure src="/images/excel/schema.png" class="round" >}}






3. Revenue, Profit & Volume Dashboard

The first page focused on business metrics. I designed it around four KPI cards:

Total Revenue (YTD)

Total Profit (YTD)

Profit Margin (YTD)

Units Sold (YTD)

Each KPI displayed the year-over-year growth using the same YTD window (Jan–Jun) for consistency with the 2017 incomplete data.

Below the KPIs, I built several visuals:

Monthly Revenue Trend:
I intentionally used area charts for 2015 and 2016, and a thin line for 2017 with a highlighted endpoint. The idea was to emphasize that 2017 is still “in progress” and to visually separate it without overwhelming the layout.

Revenue by Category, Subcategory, and Products:
Bar charts showing top performers. I limited products and subcategories to top 10 to avoid clutter.

Revenue by Country:
A filled map, with labels only where the region size allowed. The labels were generated from the cleaned region names.

Metric Switching Mechanism:
Excel doesn’t allow native metric switching in pivot charts, so I simulated the behavior with a set of buttons styled to look like slicers. Each button hyperlinks to a sheet containing identical visuals but using a different measure (Revenue, Profit, Margin, Volume). It’s not the most elegant workaround, but it gives the user a feeling of “switching the metric” without breaking the visual flow.