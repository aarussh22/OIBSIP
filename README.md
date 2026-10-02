# EDA on Retail Sales Data

OIBSIP DATA ANALYSTICS INTERSHIP 

## Objective 1

Exploratory Data Analysis on a retail sales dataset to uncover sales trends, customer behavior patterns, and actionable business insights using an online data set.

## Dataset (Kaggle)

Retail Sales Dataset (Kaggle) which contains about 1,000 transactions, 9 columns (Transaction ID, Date, Customer ID, Gender, Age, Product Category, Quantity, Price per Unit, Total Amount).

## Tools which I have Used

Python, pandas, matplotlib, seaborn, Jupyter Notebook

## Analysis Performed

- Data inspection (shape, dtypes, nulls)
- Descriptive statistics on numeric columns
- Monthly sales trend (time series)
- Customer age group distribution
- Gender distribution
- Revenue and units sold by product category
- Correlation heatmap
- Average spend by gender per category

## Key Insights

- The Average transaction value achieved is $456, but the median is only $135 which means spending is right-skewed, driven by a smaller number of high-value purchases rather than typical transactions.
- Monthly sales peaked in the month of May 2023 where amount was $53,150 and was lowest in September 2023 with only $23,620. But the small $1,530 figure for January 2024 is not a real drop as it reflects that the data is incomplete at the end of the dataset, not an actual sales decline.
- Customer base is broadly in age (18–64, average ~41) with no single dominant age group.
- Gender split is nearly even: 51% Female, 49% Male.
- Price per unit varies widely ($25–$500), indicating a mix of low-cost and premium products in the catalog.
- Revenue is nearly split evenly across categories like Electronics leads narrowly at $156,905, followed closely by Clothing at $155,580, with Beauty trailing at $143,515. None of the category is a dominent.

## Business Recommendations

1. Work on the September Sales with targeted promotions or seasonal campaigns like Back-to-school or stuff like that to pump up sales in fall.
2. Since spending is right-skewed driven, I would reccommend considering building loyalty (Strong customer base) or bundling offers aimed at increasing average transaction value for the more frequent, lower-spend customers.
3. Since Electronics, Clothing, and Beauty are closely evenly distributed, instead of spending in one category's marketing, try and test combining two different category promotions together (for instance get this shirt and get 25% off on lip balm) in that case we can also work on lifting Beauty's sale relatively.

## Author

Aarush Gupta
Oasis Infobyte Summer Internship Program (Data Analytics Track)