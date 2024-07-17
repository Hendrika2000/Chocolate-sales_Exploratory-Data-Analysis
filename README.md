
```sql
SELECT * FROM sales;
```
Find the first and latest order date
```sql
SELECT
	MAX(saledate), 
	MIN(saledate)
FROM sales;
```
Find the details of sales where amounts are more than 2000 and boxes are < 100
```sql
SELECT 
* 
FROM sales 
WHERE Amount > 2000 AND Boxes <100;
```
Find the total amount of each country from the highest to the lowest
```sql
SELECT
*
FROM (
	SELECT
	g.Geo,
	SUM(s.amount) OVER(PARTITION BY g.geo) TotalAmount
	FROM sales s
JOIN geo g ON g.GeoID = s.GeoID
)t ORDER BY TotalAmount DESC;
```
How many shipments (sales) each of the sales persons had in the month of January 2022?
```sql
SELECT 
	p.Salesperson, 
	COUNT(*) AS 'Shipment Count'
FROM sales s
JOIN people p ON p.spid = s.spid
WHERE SaleDate BETWEEN '2022-01-01' AND '2022-01-31'
GROUP BY p.Salesperson;
```
Which shipments had under 100 customers and under 100 boxes? Did any of them occur on Wednesday?
```sql
SELECT
*
FROM sales
WHERE Customers < 100 AND Boxes < 100;
SELECT *,
CASE WHEN WEEKDAY(saledate)=2 then 'Wednesday Shipment' 
ELSE "" 
END 'W Shipment'
FROM sales
WHERE customers <100 AND boxes<100;
```
Find the total number of Orders and the total number of Orders for each products
```sql
SELECT
	s.saledate,
	pr.product, 
	COUNT(*) OVER() TotalOrders,
	COUNT(*) OVER(PARTITION BY pr.product) OrdersByProduct
FROM sales s
JOIN products pr ON pr.pid = s.pid;
```
Find the total amount across all orders and the total customer for each cost per boxes
```sql
SELECT * 
FROM (
SELECT
	s.customers,
	pr.product,
	pr.cost_per_box,
	SUM(s.customers) OVER() TotalCustomers,
	SUM(s.customers) OVER(PARTITION BY pr.cost_per_box ) CustomersByCostperbox
FROM sales s
JOIN products pr ON pr.pid = s.pid
) AS TotalCustomersByCostperbox
ORDER BY CustomersByCostperbox;
```
India or Australia? Who buys more chocolate boxes a monthly basis?
```sql
SELECT 
	YEAR(saledate) Year,
	MONTH(saledate) Month,
	SUM(CASE WHEN g.geo='India' = 1 THEN boxes ELSE 0 END) 'India Boxes',
	SUM(CASE WHEN g.geo='Australia' = 1 THEN boxes ELSE 0 END) 'Australia Boxes'
FROM sales s
JOIN geo g ON g.geoid = s.geoid
GROUP BY YEAR(saledate), MONTH(saledate)
ORDER BY YEAR(saledate), MONTH(saledate);
```
Did we ship at least one box of After Nines to New Zealand on all the months
```sql
SET @product_name = 'After Nines';
SET @country_name = 'New Zealand';
SELECT YEAR(saledate) Year, 
	MONTH(saledate) Month,
	IF(SUM(boxes)>1, 'Yes','No') Status
FROM sales s
JOIN products pr ON pr.pid = s.pid
JOIN geo g ON g.geoid = s.geoid
WHERE pr.product = @product_name AND g.geo = @country_name
GROUP BY YEAR(saledate), MONTH(saledate)
ORDER BY YEAR(saledate), MONTH(saledate);
```
Find the Percentage contribution of each orders to the total amount
```sql
SELECT
	s.SaleDate,
	p.Salesperson,
	pr.product,
	s.amount,
	SUM(s.amount) OVER() TotalAmount,
	ROUND(s.amount / SUM(s.amount) OVER() * 100,3) PercentageOfTotal
	FROM sales s
JOIN products pr ON pr.pid = s.pid
JOIN people p ON p.spid = s.spid;
```
Find the average amount across all orders and find the average amount for each product
```sql
SELECT
	s.saledate,
	pr.product,
	s.amount,
	AVG(s.amount) OVER() AvgAmount,
	AVG(s.amount) OVER(PARTITION BY pr.product) AvgAmountByProducts
FROM sales s
JOIN products pr ON pr.pid = s.pid;
```
Find all orders where amount are higher than the average amount across all orders
```sql
SELECT
*
FROM (
SELECT
	pr.product,
	s.amount,
	AVG(s.amount) OVER() AvgAmount
FROM sales s
JOIN products pr ON pr.pid = s.pid
)t 
WHERE amount > AvgAmount
ORDER BY amount DESC;
```
Find the highest and lowest amount and boxes all orders
Find the highest and lowest amount and boxes for each product 
```sql
SELECT
	s.spid,
	s.saledate,
	pr.product,
	s.boxes,
	MAX(amount) OVER() HighestAmount,
	MIN(amount) OVER() LowestAmount,
	MAX(boxes) OVER() HighestBoxes,
	MIN(boxes) OVER() LowestBoxes,
	MAX(amount) OVER(PARTITION BY pr.product) HighestAmountByProduct,
	MIN(amount) OVER(PARTITION BY pr.product) LowesttAmountByProduct,
	MAX(Boxes) OVER(PARTITION BY pr.product) HighestBoxesByProduct,
	MIN(Boxes) OVER(PARTITION BY pr.product) LowesttBoxesByProduct
FROM sales s
JOIN products pr ON pr.pid = s.pid;
```
Show the Sales Person who have the highest amount sales
```sql
SELECT
* 
FROM (
SELECT
	p.Salesperson,
	s.amount,
	MAX(s.amount) OVER() HighestAmount
FROM sales s
JOIN people p ON p.spid = s.spid
)t WHERE amount = HighestAmount;
```
Find the deviation of each amount sales from minimum and maximum amount 
```sql
SELECT
	amount,
	saledate,
	MAX(amount) OVER() HighestAmount,
	MIN(amount) OVER() LowestAmount,
	amount - MIN(amount) OVER() DeviationFromMin,
	MAX(amount) OVER() - Amount DeviationFromMax
FROM sales ;
```
Calculate moving average of amount for each product over time
Calculate moving average of amount for each product over time, including only the next order
```sql
SELECT
	pr.product,
	s.amount,
	AVG(s.amount) OVER(PARTITION BY pr.product) AvgByProduct,
	AVG(s.amount) OVER(PARTITION BY pr.product ORDER BY saledate) MovingAvg,
	AVG(s.amount) OVER(PARTITION BY pr.product ORDER BY saledate ROWS BETWEEN CURRENT ROW AND 1 FOLLOWING) RollingAvg
FROM sales s
JOIN products pr ON pr.pid = s.pid;
```
Rank the Orders based on their amount sales from highest to lowest
```sql
SELECT
	pr.product,
	s.saledate,
	s.amount,
	ROW_NUMBER() OVER(ORDER BY s.amount DESC) AmountRank_Row,
    RANK() OVER(ORDER BY s.amount DESC) AmountRank_Rank,
	DENSE_RANK() OVER(ORDER BY s.amount DESC) AmountRank_Dense
FROM sales s
JOIN products pr ON pr.pid = s.pid;
```
Find the top highest amount for each product
```sql
SELECT
*
FROM (
SELECT
	pr.product,
	s.amount,
    s.boxes,
    s.customers,
    pr.Cost_per_box,
	ROW_NUMBER() OVER(PARTITION BY pr.product ORDER BY s.amount DESC) RankByProduct
FROM sales s
JOIN products pr ON pr.pid = s.pid
)t WHERE RankByProduct =1;
```
Find the 2 lowest of Sales Person on their total Boxes
```sql
SELECT
*
FROM (
SELECT
	p.salesperson,
	SUM(s.boxes) TotalBoxes,
	ROW_NUMBER() OVER(ORDER BY SUM(s.boxes)) RankSalesPerson
FROM sales s
JOIN people p ON p.spid = s.spid
GROUP BY p.salesperson
)t WHERE RankSalesPerson <= 2;
```
Segment all amount sales into 5 categories : Very high, high, medium, low, very low Boxes
```sql
SELECT
*,
CASE WHEN Buckets = 1 THEN 'Very high'
     WHEN Buckets = 2 THEN 'High'
     WHEN Buckets = 3 THEN 'Medium'
     WHEN Buckets = 4 THEN 'Low'
     WHEN Buckets = 1 THEN 'Very low'
END BoxesSegmentations
FROM (
	SELECT
		pr.product,
		boxes,
		NTILE(5) OVER(ORDER BY amount DESC) Buckets
	FROM sales s
    JOIN products pr ON pr.pid = s.pid
)t;
```
Find the products that fall within the highest 40% of the amount
```sql
SELECT
*
FROM (
SELECT
	pr.product,
	s.amount,
	ROUND(CUME_DIST() OVER(PARTITION BY pr.product ORDER BY s.amount DESC),4) DistRank
FROM sales s
JOIN products pr ON pr.pid = s.pid
)t WHERE DistRank <= 0.4;
```
Analyze the 2021 month-over-month performance by finding the percentage change
in sales between the current and previous months
```sql
SELECT
*,
CurrentMonthSales - PreviousMonthSales AS MoM_Change,
ROUND((CurrentMonthSales - PreviousMonthSales)/PreviousMonthSales*100, 2) AS MoM_Percentage
FROM (
SELECT
	MONTH(saledate) OrderMonth,
	SUM(amount) CurrentMonthSales,
    LAG(SUM(amount)) OVER (ORDER BY MONTH(saledate)) PreviousMonthSales
FROM sales
 WHERE YEAR(saledate) = '2021'
GROUP BY
	MONTH(saledate)
)t ;
```
Analyze the year-over-year performance by finding the percentage change
in sales between the current and previous year
```sql
SELECT
*,
CurrentYearSales - PreviousYearSales AS YoY_Change,
ROUND((CurrentYearSales - PreviousYearSales)/PreviousYearSales*100, 2) AS YoY_Percentage
FROM (
SELECT
	YEAR(saledate) OrderYear,
	SUM(amount) CurrentYearSales,
    LAG(SUM(amount)) OVER (ORDER BY YEAR(saledate)) PreviousYearSales
FROM sales
GROUP BY
	YEAR(saledate)
    )t;
```
