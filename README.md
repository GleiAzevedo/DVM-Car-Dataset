# DVM-Car-Dataset

The car market in the UK is the objective of analysis of this project, focusing on the most popular car brands, models and units sold between 2001 and 2020.

# Dataset description

"DVM-CAR A Large-Scale Dataset for Automotive Applications " is the dataset, publicly available, used for this analysis.  [View website to download](https://deepvisualmarketing.github.io/) Although the dataset also offers a wide range of cars images, for the purposes of this project our interest is in the following tables:

+ Basic Table: contains basic information about brand, model and model ID;
+ Price Table: Price of each model over the years;
+ Sales Table: Units sold per year for each model;
+ Trim Table: Trim attributes, such as fuel type, engine size, etc.;
+ Ad Table: Trim information, as well as sales, of the used car market.

# Tools

+ Microsoft SQL Server Management Studios for data analysis - View [SQL Scripts]()
+ Power BI for data visualization.

# Key Questions Explored

1 - Quantity of brands and how many models per brand:

```
SELECT COUNT(DISTINCT Automaker_ID) AS [Brands]
FROM Basic_tbl;
SELECT
	'Total' AS [Automaker],
	COUNT(Genmodel) AS [Number_of_models]
FROM Basic_tbl
UNION ALL
SELECT
	Automaker,
	COUNT(Genmodel) AS [Number_of_models]
FROM Basic_tbl
GROUP BY Automaker
ORDER BY Number_of_models DESC;
```
PICTURE 1

2 -  Average car price, most expensive and cheapest car per brand and year:

```
SELECT
	Maker,
	AVG(Entry_price) AS [Avg_Price],
	MIN(Entry_price) AS [Min_Price],
	MAX(Entry_price) AS [Max_Price],
	[Year]
FROM Price_tbl
GROUP BY
	Maker,
	[Year]
ORDER BY Maker;
```

PICTURE 2

3 - Average car price per year:

```
SELECT
	[Year],
	AVG(Entry_Price) AS [Average_car_price]
FROM Price_tbl
GROUP BY [Year]
ORDER BY [Year];
```
PICTURE 3

4 - Automakers from the most sold to the least sold brand (Sales table, new units) (2001 - 2020):

```
CREATE VIEW UnpivotedSales AS
SELECT Maker, Genmodel, Genmodel_ID, [Year], Units
FROM 
   (SELECT Maker, [Genmodel],[Genmodel_ID],[Year_2020],[Year_2019],[Year_2018],[Year_2017]
      ,[Year_2016],[Year_2015],[Year_2014],[Year_2013],[Year_2012],[Year_2011],[Year_2010]
      ,[Year_2009],[Year_2008],[Year_2007],[Year_2006],[Year_2005],[Year_2004],[Year_2003]
      ,[Year_2002],[Year_2001]
    FROM Sales_tbl) AS p
UNPIVOT
   (Units FOR [Year] IN ([Year_2020],[Year_2019],[Year_2018],[Year_2017]
      ,[Year_2016],[Year_2015],[Year_2014],[Year_2013],[Year_2012],[Year_2011],[Year_2010]
      ,[Year_2009],[Year_2008],[Year_2007],[Year_2006],[Year_2005],[Year_2004],[Year_2003]
      ,[Year_2002],[Year_2001])) AS unpvt;
GO;

SELECT
	Maker,
	SUM(Units) AS [Units_sold]
FROM UnpivotedSales
GROUP BY Maker
ORDER BY Units_sold DESC;
```

PICTURE 4

5 - 3 most sold brands per year:

```
SELECT
	Maker,
	Units_sold,
	[Year],
	rn
FROM (
	SELECT
		Maker,
		SUM(Units) AS [Units_sold],
		[Year],
		ROW_NUMBER() OVER (PARTITION BY [Year] ORDER BY SUM(Units) DESC) AS rn
	FROM UnpivotedSales
	GROUP BY 
		Maker,
		[Year]
) AS RankedBrands
WHERE rn IN (1,2,3)
ORDER BY [Year];
```

PICTURE 5

6 -  Top 10 sold models in general (2001 - 2020):

```
SELECT TOP 10
	Maker,
	Genmodel,
	SUM(Units) AS [Units_sold]
FROM UnpivotedSales
GROUP BY
	Maker,
	Genmodel
ORDER BY [Units_sold] DESC;
```

PICTURE 6

7 - Most sold model per brand (2001 - 2020):

```
WITH Maxsold_cte AS (
	SELECT
		Maker,
		Genmodel,
		RANK() OVER (PARTITION BY Maker ORDER BY SUM(Units) DESC) AS rk,
		SUM(Units) AS [Units_sold]
	FROM UnpivotedSales
	GROUP BY
		Maker,
		Genmodel
)
SELECT
	Maker,
	Genmodel,
	[Units_sold]
FROM Maxsold_cte
WHERE rk = 1;
```

PICTURE 7

8 - Fuel type (1998 - 2021):

```
WITH Fuel_cte  AS (
	SELECT
		Fuel_Type,
		COUNT(Fuel_type) AS [Quantity]
	FROM Trim_tbl
	GROUP BY Fuel_type
)
SELECT
	Fuel_cte.*,
	ROUND(100 * [Quantity] / SUM(Quantity) OVER(),2) AS [Percentage]
FROM Fuel_cte
GROUP BY
	Fuel_type,
	Quantity;
```

Electric cars on the 'Other' category, based on 0 gas emission. How many are they, cheapest and most expensive?

```
SELECT
	Fuel_Type,
	COUNT(Fuel_type) AS [Quantity],
	MIN(Price) AS [Cheapest],
	MAX(Price) AS [Most expensive]
FROM Trim_tbl
WHERE Gas_emission = 0
	AND Fuel_type = 'Other'
GROUP BY Fuel_type;
```

The hybrid cars number below is only the minimum number as they may have different trim details, abbreviations, gas emission etc., not included on the parameters searched.

```
WITH Hybrid_cte AS (
	SELECT
		'Hybrid' AS [Fuel_type],
		Fuel_type AS [Category_found],
		COUNT(Fuel_type) AS [Quantity]
	FROM Trim_tbl
	WHERE Fuel_type = 'Other'
		AND [Trim] LIKE '%hybrid%'
	GROUP BY Fuel_type
	UNION ALL
	SELECT
		'Hybrid' AS [Fuel_type],
		Fuel_type,
		COUNT(Fuel_type) AS [Quantity]
	FROM Trim_tbl
	WHERE Fuel_type <> 'Other'
		AND [Trim] LIKE '%hybrid%'
	GROUP BY Fuel_type
	UNION ALL
	SELECT
		'Hybrid' AS [Fuel_type],
		Fuel_type,
		COUNT(Fuel_type) AS [Quantity]
	FROM Trim_tbl
	WHERE [Trim] NOT LIKE '%hybrid%'
		AND [Trim] LIKE '%hev%'
	GROUP BY Fuel_type
)
SELECT 
	'Hybrid' AS [Fuel_type],
	'Total' AS [Total],
	SUM([Quantity]) AS [Quantity]
FROM Hybrid_cte;
```

PICTURE 8

9 - Most popular brand and model on used cars advertisement (2012-2021):

```
SELECT
	Maker,
	COUNT(Maker) AS [Quantity]
FROM Ad_tbl
GROUP BY Maker
ORDER BY [Quantity] DESC;

WITH Brand_cte AS (
SELECT
	Maker,
	Genmodel,
	RANK() OVER (PARTITION BY Maker ORDER BY COUNT(Genmodel) DESC) AS rk,
	COUNT(Genmodel) AS [Quantity]
FROM Ad_tbl
GROUP BY
	Maker,
	Genmodel
)
SELECT
	Maker,
	Genmodel,
	[Quantity]
FROM Brand_cte
WHERE rk = 1
ORDER BY Maker;
```

PICTURE 9

Most popular colour:

```
SELECT
	Color,
	COUNT(Color) AS [Quantity]
FROM Ad_tbl
GROUP BY Color
ORDER BY [Quantity] DESC;
```

PICTURE 10

Most popular body type:

```
SELECT
	Bodytype,
	COUNT(Bodytype) AS [Quantity]
FROM Ad_tbl
GROUP BY Bodytype
ORDER BY [Quantity] DESC;
```

PICTURE 11

Average Runned Miles:

```
SELECT AVG(CAST(Runned_Miles AS BIGINT)) AS [Average_runned_miles]
FROM Ad_tbl
WHERE Runned_Miles IS NOT NULL;
```

PICTURE 12

Average age by brand:

```
SELECT TOP 10
	Maker,
	AVG((Adv_year - Reg_year)) AS [Average_age_of_vehicle],
	COUNT(Genmodel) AS [Quantity_of_vehicles]
FROM Ad_tbl
GROUP BY Maker
ORDER BY [Quantity_of_vehicles] DESC;
```

PICTURE 13

Most popular register and selling year:

```
SELECT TOP 1 Reg_year
FROM Ad_tbl
GROUP BY Reg_year
ORDER BY COUNT(Reg_year) DESC;

SELECT TOP 1 Adv_year
FROM Ad_tbl
GROUP BY Adv_year
ORDER BY COUNT(Adv_year) DESC;
```

PICTURE 14