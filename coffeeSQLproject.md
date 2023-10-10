## This project uses the Coffee Chain Sales Analysis dataset from Kaggle. 
#### https://www.kaggle.com/datasets/amruthayenikonda/coffee-chain-sales-dataset?select=Coffee_Chain_Sales+.csv.
### I will be using MySQL Community Server within TablePlus to run queries and get some quick visualizations.

### Here are the column headers:
###### area_code, cogs, dif_actual_v_target_profit, date, inventory_margin, margin, market_size, market, marketing, product_line, product_type, product, profit, sales, state, target_cogs, target_margin, target_profit, target_sales, total_expenses, type

 1. area_code: A unique identifier for different geographical areas or regions where the coffee chain operates.
 2. cogs (Cost of Goods Sold): The total cost incurred by the coffee chain in producing or purchasing the products it sells.
 3. dif_actual_v_target_profit: This attribute indicates how well the company performed in terms of profit compared to its target. It reflects the financial performance against predefined goals.
 4. date: The date of sales transactions, which allows for time-based analysis of sales trends and patterns.
 5. inventory_margin: The difference between the cost of maintaining inventory and the revenue generated from selling those inventory items.
 6. margin: The profit margin, which is the percentage of profit earned from sales. It's a critical financial metric.
 7. market_size: Information about the size of the market in each area, helping to understand the potential customer base and market dynamics.
 8. profit: financial gain achieved by the company after deducting the cost of goods sold (COGS) and other expenses from the revenue generated through sales.
 9. sales: represent the revenue generated from the coffee chain's products, reflecting its financial performance and customer demand.

#### According to the dataset the date is our primary key. Using this query I discovered we have 1061 dates to work with.
```SQL
SELECT
	COUNT(DISTINCT(date)) AS PK_COUNT
FROM
	CoffeeChainSales
```
PK_COUNT = 1061

#### We can also see the range of dates we have
```SQL
SELECT 
	MIN(date) AS min_date,
	MAX(date) AS max_date
FROM
	CoffeeChainSales
```
min_date = 2012-10-01, max_date = 2015-08-27

#### Next let's find all the distint nominal data.
```SQL
SELECT DISTINCT market_size
FROM CoffeeChainSales;
-- returns: Major Market, Small Market

SELECT DISTINCT market
FROM CoffeeChainSales;
-- returns: Central, South, East, West

SELECT DISTINCT product
FROM CoffeeChainSales;
-- returns: Lemon, Mint, Darjeeling, Green Tea, Decaf Espresso, Decaf Irish Cream, Amaretto, Colombian, Caffe Mocha, Caffe Latte, Chamomile, Earl Grey, Regular Espresso

SELECT DISTINCT product_type
FROM CoffeeChainSales;
-- returns: Herbal Tea, Tea, Espresso, Coffee

SELECT DISTINCT product_line
FROM CoffeeChainSales;
-- returns: Leaves, Beans

SELECT DISTINCT state
FROM CoffeeChainSales;
-- returns: Colorado, Texas, Florida, California, Iowa, Connecticut, Oklahoma, Nevada, Utah, New Hampshire, Louisiana, Oregon, Missouri, Wisconsin, Washington, Massachusetts, Illinois, New Mexico, Ohio, New York

SELECT DISTINCT type
FROM CoffeeChainSales;
-- returns: Decaf, Regular
```

## Now that we have date ranges and nominal data identified we can answer a few questions about the data.
## Date Analysis:
### How are sales distributed across the different years and months?
After running this query, TablePlus allows you to create a chart witht the data gathered. Here you can see that 2014 was a pivotal year in growing sales and the sales peaked around August-September that year. The previous year showed not too impressive sales results around that time period. This demonstrates that there might be more to explore as to why this occured.
```SQL
SELECT 
    CONCAT(YEAR(date), '-', LPAD(MONTH(date), 2, '0')) AS ym_date,
    SUM(sales) AS total_sales
FROM CoffeeChainSales
GROUP BY CONCAT(YEAR(date), '-', LPAD(MONTH(date), 2, '0'))
ORDER BY CONCAT(YEAR(date), '-', LPAD(MONTH(date), 2, '0'))
```
<img width="1428" alt="image" src="https://github.com/ToneChaser/ChaseLeBlancPortfolio/assets/145052217/60ff7153-faa1-4ea7-9b84-2db8e068d25d">

## Market Analysis:
### What is the distribution of market sizes in the dataset?
### How do sales vary among different market sizes?
### Are there any specific markets that consistently outperform others?
## Product Analysis:
### What are the most popular products in terms of sales?
### How do sales vary by product type or product line?
### Are there any products that are consistently popular across markets?
## Geographic Analysis:
### How are sales distributed across different states?
### Are there any geographic trends or variations in sales?
### Do certain states or regions have unique preferences or behaviors?
## Type Analysis:
### How do sales of decaffeinated and regular products compare?
### Are there any differences in sales trends for different types of products?
### Do certain regions or markets have a preference for decaffeinated or regular products?
## Combining Factors:
### Are there interesting relationships or interactions between factors such as market size, product type, and state that influence sales?
