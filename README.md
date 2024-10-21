# DVM-Car-Dataset

The car market in the UK is the target of analysis for this project, focusing on the most popular car brands, models and units sold between 2001 and 2020.

# Dataset description

"DVM-CAR A Large-Scale Dataset for Automotive Applications " is the dataset, publicly available, used for this analysis.  [View website to download](https://deepvisualmarketing.github.io/) Although the dataset also offers a wide range of cars images, for the purposes of this project our interest is in the following tables:

+ Basic Table: contains basic information about brand, model and model ID;
+ Price Table: Price of each model over the years;
+ Sales Table: Units sold per year for each model;
+ Trim Table: Trim attributes, such as fuel type, engine size, etc.;
+ Ad Table: Trim information, as well as sales, of the used car market.

# Tools

+ Microsoft SQL Server Management Studios for data analysis - View [SQL Scripts](https://github.com/GleiAzevedo/DVM-Car-Dataset/blob/main/DVM%20Car%20Dataset%20queries%20sql)
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
![1-1 DVM CAR](https://github.com/user-attachments/assets/88ef2a82-e9e5-439c-befc-404adf3c64da)  ![1-2 DVM CAR](https://github.com/user-attachments/assets/b7aacf86-5832-4cfa-965a-5164a5e3608d)
![1-3 DVM CAR](https://github.com/user-attachments/assets/26c8ff2e-d036-4094-b674-05918c407e42)

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

3 - Average car price per year:

```
SELECT
	[Year],
	AVG(Entry_Price) AS [Average_car_price]
FROM Price_tbl
GROUP BY [Year]
ORDER BY [Year];
```
![3-1 DVM CAR](https://github.com/user-attachments/assets/b03b755b-f8c1-44b7-9a40-e410d6de3a9b)

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

![4-1 DVM CAR](https://github.com/user-attachments/assets/beb91630-87b8-4239-b32c-dddfc20834f4)  ![4-2 DVM CAR](https://github.com/user-attachments/assets/3cb2766e-0cfb-4699-83c6-3a1abc251208)
 ![4-3 DVM CAR](https://github.com/user-attachments/assets/f91368ab-adc6-4289-9bf2-50ae076ca45d)

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
![5-1 DVM CAR](https://github.com/user-attachments/assets/533db024-d85a-4b14-a4a6-7d0538201a31)  ![5-2 DVM CAR](https://github.com/user-attachments/assets/87a7b2a6-53de-468d-8a69-af527c2834bc)

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

![6-1 DVM CAR](https://github.com/user-attachments/assets/4f5e7dce-1ca7-49a7-8287-a72c6df9a328)

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

![7-1 DVM CAR](https://github.com/user-attachments/assets/7113f64f-5b3f-495e-aec6-a157781204f4) ![7-3 DVM CAR](https://github.com/user-attachments/assets/54b50d4f-3081-4414-bf61-7f047c5113e0)
![7-2 DVM CAR](https://github.com/user-attachments/assets/c5c262de-c70a-4bec-8a93-9b72ec0c4240)

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

![8-1 DVM CAR](https://github.com/user-attachments/assets/3bc51b94-a2ab-4864-864e-d09ca80c911b)

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
![8-2 DVM CAR](https://github.com/user-attachments/assets/8568d892-61f6-413d-8630-3234b0eb8149)

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

![8-3 DVM CAR](https://github.com/user-attachments/assets/864038ad-e9fa-4b0f-a9c8-544911c88ec1)

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

![9-1 DVM CAR](https://github.com/user-attachments/assets/4470ee4f-5af9-4305-8a41-748fc5fc61ae) ![9-2 DVM CAR](https://github.com/user-attachments/assets/17093900-3071-4824-a6ce-32979754a61b)
![9-3 DVM CAR](https://github.com/user-attachments/assets/790372c5-654a-43dd-9e90-c30471afd6b0)

![9-4 DVM CAR](https://github.com/user-attachments/assets/29c6ba82-c663-4db6-ba29-02e719042148) ![9-5 DVM CAR](https://github.com/user-attachments/assets/0c9efa05-47d2-41ea-8ab1-07d2329220cc)
![9-6 DVM CAR](https://github.com/user-attachments/assets/de628039-34d1-4a11-a61c-68b9fc7ef9ad)

Most popular colour:

```
SELECT
	Color,
	COUNT(Color) AS [Quantity]
FROM Ad_tbl
GROUP BY Color
ORDER BY [Quantity] DESC;
```

![10 DVM CAR](https://github.com/user-attachments/assets/ff34623c-e86b-4e05-9ea1-42812b188c54)

Most popular body type:

```
SELECT
	Bodytype,
	COUNT(Bodytype) AS [Quantity]
FROM Ad_tbl
GROUP BY Bodytype
ORDER BY [Quantity] DESC;
```

![11 DVM CAR](https://github.com/user-attachments/assets/d5e65500-a910-4af4-8dd8-b775aae56f20)

Average Runned Miles:

```
SELECT AVG(CAST(Runned_Miles AS BIGINT)) AS [Average_runned_miles]
FROM Ad_tbl
WHERE Runned_Miles IS NOT NULL;
```

![12 DVM CAR](https://github.com/user-attachments/assets/bd95b772-eab9-4d2d-91f4-c0ae236b89df)

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

![13 DVM CAR](https://github.com/user-attachments/assets/9c8ee803-1324-4f24-8f69-a5dd688cd0e3)

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

![14 DVM CAR](https://github.com/user-attachments/assets/26b3a0b9-083b-4c66-8962-63a5b19059b3)


PICTURE 14
