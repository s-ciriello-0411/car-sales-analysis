# car-sales-analysis
performing some data analysis on a set of data to answer questions.

## Tools
- **Programming language**: SQL (BigQuery)

## Data
The data set is on car sales over a period of several months in the United States. available from the Kaggle website [Car Sales Dataset](https://www.kaggle.com/datasets/syedanwarafridi/vehicle-sales-data)

## Cleaning Data 
I created a table to eliminate invalid data and divided the information regarding the time of sale 
```sql
CREATE TABLE `provaprova-431308.car_sales.car_sales_valid`  AS 
SELECT 
year,
make,
`model`,
`trim`,
body,
transmission,
vin,
state,
condition,
odometer,
color,
interior,
seller,
mmr,
sellingprice,
saledate,
  SUBSTR(saledate, 12, 4) AS sale_year,
  SUBSTR(saledate, 5, 3) AS sale_month_text,
  SUBSTR(saledate, 9, 2) AS sale_day,
  CAST(CASE SUBSTR(saledate, 5, 3)
    WHEN "Jan" THEN "1"
    WHEN "Feb" THEN "2"
    WHEN "Mar" THEN "3"
    WHEN "Apr" THEN "4"
    WHEN "May" THEN "5"
    WHEN "Jun" THEN "6"
    WHEN "Jul" THEN "7"
    WHEN "Aug" THEN "8"
    WHEN "Sep" THEN "9"
    WHEN "Oct" THEN "10"
    WHEN "Nov" THEN "11"
    WHEN "Dec" THEN "12"
    ELSE NULL 
  END AS INT64) AS sale_month
FROM `provaprova-431308.car_sales.car_sales` 
WHERE body!="Navitgation"
AND saledate IS NOT NULL
```
**Which models have the most number of sales?**

```sql
SELECT 
make, 
model,
COUNT (*)
FROM `provaprova-431308.car_sales.car_sales_valid`
GROUP BY make,model 
ORDER BY COUNT (*) DESC;
```
**Results:**

<img width="585" alt="Schermata 2024-12-11 alle 17 06 45" src="https://github.com/user-attachments/assets/015f3411-80b8-4371-a91d-85cd6a09abd2" />

**What is the average sale price in each state?**

```sql
SELECT 
state,
AVG(sellingprice) AS avg_selling_price 
FROM `provaprova-431308.car_sales.car_sales_valid`
GROUP BY state
ORDER BY avg_selling_price ASC;
```
**Results:**

<img width="389" alt="Schermata 2024-12-11 alle 17 10 55" src="https://github.com/user-attachments/assets/f042e41f-d21a-4dde-adc4-2f39e31b97f3" />

**What is the average sale price for each year and month?**

```sql
SELECT 
sale_year,
sale_month,
AVG(sellingprice) AS avg_selling_price
FROM `provaprova-431308.car_sales.car_sales_valid`
GROUP BY sale_year, sale_month
ORDER BY sale_year, sale_month;
```
**Results:**

<img width="514" alt="Schermata 2024-12-11 alle 17 13 08" src="https://github.com/user-attachments/assets/58e9a66f-ec6d-4103-b7d4-8ccbf990cae6" />

**Which month of the year has the most sales?**

```sql
SELECT
sale_month,
COUNT(*)
FROM `provaprova-431308.car_sales.car_sales_valid`
GROUP BY sale_month
ORDER BY sale_month ASC;
```
**Results:**

<img width="514" alt="Schermata 2024-12-11 alle 17 13 08" src="https://github.com/user-attachments/assets/804f1166-8509-4376-bb29-ae5ef07639ab" />

**What are the top 5 selling vehicles for each body type?**

```sql
SELECT
make,
model,
body,
num_sales,
body_rank
FROM (
  SELECT
  make,
  model,
  body,
  COUNT(*) AS num_sales,
  RANK() OVER(PARTITION BY body ORDER BY COUNT(*)DESC) AS body_rank
  FROM `provaprova-431308.car_sales.car_sales_valid`
  GROUP BY make,model,body
)
WHERE body_rank <= 5
ORDER BY body ASC,num_sales DESC
```

**Results:**

<img width="909" alt="Schermata 2024-12-11 alle 17 21 55" src="https://github.com/user-attachments/assets/07e3da86-7ee6-443c-b328-123402d6a170" />

**Which sales are higher than the average for that car's model?**

```sql
SELECT
make,
model,
vin,
sale_year,
sale_month,
sale_day,
sellingprice,
avg_model,
  FROM(
  SELECT
    make,
    model,
    vin,
    sale_year,
    sale_month,
    sale_day,
    sellingprice,
    AVG(sellingprice) OVER (PARTITION BY make, model) AS avg_model
    FROM `provaprova-431308.car_sales.car_sales_valid`
)
    WHERE sellingprice > avg_model;
```
**Results:**

<img width="992" alt="Schermata 2024-12-11 alle 17 34 25" src="https://github.com/user-attachments/assets/4e543a37-1fac-437b-9dc2-e08225839fde" />


**How does the odometer value impact the sale price?**

```sql
SELECT
  CASE
    WHEN odometer < 100000 THEN '0 - 99999'
    WHEN odometer < 200000 THEN '100000 - 199999'
    WHEN odometer < 300000 THEN '200000 - 299999'
    WHEN odometer < 400000 THEN '300000 - 399999'
    WHEN odometer < 500000 THEN '400000 - 499999'
    WHEN odometer < 600000 THEN '500000 - 599999'
    WHEN odometer < 700000 THEN '600000 - 699999'
    WHEN odometer < 800000 THEN '700000 - 799999'
    WHEN odometer < 900000 THEN '800000 - 899999'
    WHEN odometer < 1000000 THEN '900000 - 999999'
  END AS odometer_bucket,
  COUNT(*) AS num_sales,
  AVG(sellingprice) AS avg_selling_price
FROM `provaprova-431308.car_sales.car_sales_valid` 
GROUP BY odometer_bucket
ORDER BY odometer_bucket ASC;
```
**Results:**

<img width="510" alt="Schermata 2024-12-11 alle 17 28 08" src="https://github.com/user-attachments/assets/2fe2bfda-2254-4193-b321-b6c264be509f" />

**What is the highest, lowest, and average selling price for each brand, and how many different models
are sold?**

```sql
SELECT
make,
COUNT(DISTINCT model) AS num_models,
COUNT(*) AS num_sales,
MIN(sellingprice) AS min_price,
MAX(sellingprice) AS max_price,
AVG(sellingprice) AS avg_price,
FROM `provaprova-431308.car_sales.car_sales_valid` 
GROUP BY make
ORDER BY avg_price DESC;
```
**Results:**

<img width="887" alt="Schermata 2024-12-11 alle 17 31 15" src="https://github.com/user-attachments/assets/eb580701-ac02-4a3a-aa0a-cfb9b9376bba" />

**Are there any cars that are sold more than once? If so, what are the details of each sale?**

```sql
SELECT *
FROM (
  SELECT *,
  COUNT (*) OVER(PARTITION BY vin) AS vin_sales
FROM `provaprova-431308.car_sales.car_sales_valid` 
)
WHERE vin_sales > 1;
```
**Results:**

<img width="984" alt="Schermata 2024-12-11 alle 17 33 23" src="https://github.com/user-attachments/assets/7ca3de02-b7be-4f43-9665-b6ed365b7183" />







