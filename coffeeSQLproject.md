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
This query says we have 636 small markets and 426 major markets in this dataset.
```SQL
SELECT
    SUM(CASE WHEN market_size = 'Small Market' THEN 1 ELSE 0 END) AS num_small_markets,
    SUM(CASE WHEN market_size = 'Major Market' THEN 1 ELSE 0 END) AS num_major_markets
FROM CoffeeChainSales
-- returns: num_small_market = 636, num_major_markets = 426
```
### How do sales vary among different market sizes?
Using this query we discover that major markets in this dataset make up the majority of sales, but small market's sales aren't far behind. This might mean I would need to explore how each market performed throughout each year.
```SQL
SELECT
    SUM(CASE WHEN market_size = 'Small Market' THEN sales ELSE 0 END) AS total_sales_small_market,
    SUM(CASE WHEN market_size = 'Small Market' THEN profit ELSE 0 END) AS total_profit_small_market,
    SUM(CASE WHEN market_size = 'Major Market' THEN sales ELSE 0 END) AS total_sales_major_market,
    SUM(CASE WHEN market_size = 'Major Market' THEN profit ELSE 0 END) AS total_profit_major_market
FROM CoffeeChainSales;
```
<img width="674" alt="image" src="https://github.com/ToneChaser/ChaseLeBlancPortfolio/assets/145052217/93ae75ea-ef99-4c20-ba68-c0a6df7b74f7">

### Are there any specific markets that consistently outperform others?
Using this query we discover that in 2012 and 2013 the company performed best in smaller markets. However, when 2014 hit there was a massive surge in sales from larger markets. This did not last though as 2015 saw the return of small market sales rising.
```SQL
SELECT
    SUM(CASE WHEN market_size = 'Small Market' AND YEAR(date) = 2012 THEN sales ELSE 0 END) AS sales_small_market_2012,
    SUM(CASE WHEN market_size = 'Major Market' AND YEAR(date) = 2012 THEN sales ELSE 0 END) AS sales_major_market_2012,
    SUM(CASE WHEN market_size = 'Small Market' AND YEAR(date) = 2013 THEN sales ELSE 0 END) AS sales_small_market_2013,
    SUM(CASE WHEN market_size = 'Major Market' AND YEAR(date) = 2013 THEN sales ELSE 0 END) AS sales_major_market_2013,
    SUM(CASE WHEN market_size = 'Small Market' AND YEAR(date) = 2014 THEN sales ELSE 0 END) AS sales_small_market_2014,
    SUM(CASE WHEN market_size = 'Major Market' AND YEAR(date) = 2014 THEN sales ELSE 0 END) AS sales_major_market_2014,
    SUM(CASE WHEN market_size = 'Small Market' AND YEAR(date) = 2015 THEN sales ELSE 0 END) AS sales_small_market_2015,
    SUM(CASE WHEN market_size = 'Major Market' AND YEAR(date) = 2015 THEN sales ELSE 0 END) AS sales_major_market_2015  
FROM CoffeeChainSales;
```
<img width="1289" alt="image" src="https://github.com/ToneChaser/ChaseLeBlancPortfolio/assets/145052217/24ca3a06-5226-4108-87f6-394b940d8a92">
However, profits tell a very different story. Here we can see that in 2012 large markets had the most profit, but 2013 saw a return in small market profits. In the previous query we saw massive sales in large markets and that is represented here, but what is concerning is the large market sales dropping off in 2015. Definitely some later analysis to answer that question.

```SQL
SELECT
    SUM(CASE WHEN market_size = 'Small Market' AND YEAR(date) = 2012 THEN profit ELSE 0 END) AS profit_small_market_2012,
    SUM(CASE WHEN market_size = 'Major Market' AND YEAR(date) = 2012 THEN sales ELSE 0 END) AS profit_major_market_2012,
    SUM(CASE WHEN market_size = 'Small Market' AND YEAR(date) = 2013 THEN profit ELSE 0 END) AS profit_small_market_2013,
    SUM(CASE WHEN market_size = 'Major Market' AND YEAR(date) = 2013 THEN profit ELSE 0 END) AS profit_major_market_2013,
    SUM(CASE WHEN market_size = 'Small Market' AND YEAR(date) = 2014 THEN profit ELSE 0 END) AS profit_small_market_2014,
    SUM(CASE WHEN market_size = 'Major Market' AND YEAR(date) = 2014 THEN profit ELSE 0 END) AS profit_major_market_2014,
    SUM(CASE WHEN market_size = 'Small Market' AND YEAR(date) = 2015 THEN profit ELSE 0 END) AS profit_small_market_2015,
    SUM(CASE WHEN market_size = 'Major Market' AND YEAR(date) = 2015 THEN profit ELSE 0 END) AS profit_major_market_2015
FROM CoffeeChainSales;
```
<img width="1320" alt="image" src="https://github.com/ToneChaser/ChaseLeBlancPortfolio/assets/145052217/5121d92c-fd03-4a2b-8973-677b00cd9c88">

## Product Analysis:
### What are the most popular products in terms of sales?
Using this query we discover that the most popular products in terms of sales for this entire table are Colombian Coffee, Lemon Tea, and Caffe Mocha's.
```SQL
SELECT
	SUM(sales) AS product_total_sales,
	product_line,
	product_type,
	product
FROM
	CoffeeChainSales
GROUP BY
	product_line,
	product_type,
	product
ORDER BY
	product_total_sales DESC
```
<img width="480" alt="image" src="https://github.com/ToneChaser/ChaseLeBlancPortfolio/assets/145052217/7273ca51-a6b2-4f19-b0e6-4d1b2205333e">

### How do sales vary by product type or product line?
This query is a simplified version of the previous question. According to these queries beans have more sales than leaves.
```SQL
SELECT
	SUM(sales) AS product_total_sales,
	product_line
FROM
	CoffeeChainSales
GROUP BY
	product_line
ORDER BY
	product_total_sales DESC
```
<img width="323" alt="image" src="https://github.com/ToneChaser/ChaseLeBlancPortfolio/assets/145052217/cc4468ec-b4a8-411a-90ef-956781a8dc3f">

```SQL
SELECT
	SUM(sales) AS product_total_sales,
	product_type
FROM
	CoffeeChainSales
GROUP BY
	product_type
ORDER BY
	product_total_sales DESC
 ```
<img width="259" alt="image" src="https://github.com/ToneChaser/ChaseLeBlancPortfolio/assets/145052217/b235f692-28ea-44eb-8cd9-5bec6689078d">

### Are there any products that are consistently popular across markets?
Colombian Coffee and Lemon Tea tend to be favorites across each region. However, it is worth noting that not every product is offered across each region. This raises the question as to why some products have not yet been offerred to certain regions as having uniformity across regions would have more conclusive results.
```SQL
SELECT
	market,
	product,
	SUM(sales) AS total_sales
FROM
	CoffeeChainSales
WHERE
	market = 'Central' -- ran this query with South, East, and West as well.
GROUP BY
	market,
	product
ORDER BY
	market,
	total_sales DESC
```
<img width="312" alt="image" src="https://github.com/ToneChaser/ChaseLeBlancPortfolio/assets/145052217/a54d09fa-51a1-49ea-8191-205513f5c595"> <img width="311" alt="image" src="https://github.com/ToneChaser/ChaseLeBlancPortfolio/assets/145052217/d31b1c38-4e98-4c75-a416-ba7e26feb4fe"> <img width="308" alt="image" src="https://github.com/ToneChaser/ChaseLeBlancPortfolio/assets/145052217/5b042701-38d2-478c-8e37-f7c2de2dac9f"> <img width="311" alt="image" src="https://github.com/ToneChaser/ChaseLeBlancPortfolio/assets/145052217/df2a53e5-a1e3-4bcb-afe1-dd2233fdbb54">

## Geographic Analysis:
### How are sales distributed across different states?
This query ranks the states sales in this dataset from most sales to least sales. California, followed by New York, followed by Illinois are the top three in sales. Missouri, New Mexico, and New Hampshire take the lowest of sales.
```SQL
SELECT
	SUM(sales) as total_state_sales,
	state,
	market,
	market_size
FROM 
	CoffeeChainSales
GROUP BY
	state,
	market,
	market_size
ORDER BY
	total_state_sales DESC
```
<img width="410" alt="image" src="https://github.com/ToneChaser/ChaseLeBlancPortfolio/assets/145052217/56fbe7e7-3bc5-457f-b50f-f075b2b98352">

### Do certain states or regions have unique preferences or behaviors?
Since there are 20 states and 13 unique products across the dataset, I have limited the query to return the top selling product in each state. Here are some unique preferences:
* Iowa's most popular product is Earl Grey Tea.
* Nevada's most popular drink is Darjeeling.
* New Yorker's like the hard stuff: Regular Esspresso.
* New Hampshire's most popular drink is Amaretto.
```SQL
WITH RankedProducts AS (
    SELECT
        SUM(sales) AS total_state_sales,
        product,
        state,
        ROW_NUMBER() OVER (PARTITION BY state ORDER BY SUM(sales) DESC) AS product_rank
    FROM 
        CoffeeChainSales
    GROUP BY
        state,
        product
)
SELECT
    total_state_sales,
    product,
    state
FROM RankedProducts
WHERE product_rank = 1
ORDER BY state;
```
<img width="391" alt="image" src="https://github.com/ToneChaser/ChaseLeBlancPortfolio/assets/145052217/d560b7f5-31af-43d6-a43c-9469e48737c9">

## Type Analysis:
### How do sales of decaffeinated and regular products compare?
This query reveals that caffeinated beverages sell more than decaf, however, I found it surprising how well decaf actually sold in comparision to caffeinated.
```SQL
SELECT
    SUM(CASE WHEN type = 'Decaf' THEN sales ELSE 0 END) AS total_sales_decaf,
    SUM(CASE WHEN type = 'Regular' THEN sales ELSE 0 END) AS total_sales_regular
FROM
	CoffeeChainSales;
```
<img width="280" alt="image" src="https://github.com/ToneChaser/ChaseLeBlancPortfolio/assets/145052217/b3f90e2c-4f78-4820-9cdf-e45ec6ad1498">

### Are there any differences in sales trends for different types of products?
Throughout the 35 months in this dataset you can see that decaf products have more predictable sales growth than decaf, I hypothesize this could be that as the business obtained more customers, decaf drinkers didn't tend to switch to caffeinated drinks. However, there are a few months in the chart where decaf outsold regular beverages. I would propose more time to analyze the cause of those low caffeinated drink sales.
```SQL
SELECT
	CONCAT(YEAR(date), '-', LPAD(MONTH(date), 2, '0')) AS ym_date,
	SUM(CASE WHEN TYPE = 'Decaf' THEN sales ELSE 0 END) AS decaf_sales,
	SUM(CASE WHEN TYPE = 'Regular' THEN sales ELSE 0 END) AS regular_sales
FROM
	CoffeeChainSales
GROUP BY
	ym_date
ORDER BY
	ym_date
```
<img width="1143" alt="image" src="https://github.com/ToneChaser/ChaseLeBlancPortfolio/assets/145052217/30091bee-9840-41b4-af48-846819c9335c"> <img width="847" alt="image" src="https://github.com/ToneChaser/ChaseLeBlancPortfolio/assets/145052217/634b8d5b-d591-4311-9258-3cfb81c3fb93">

### Do certain markets have a preference for decaffeinated or regular products?
In this query we find that in the east market caffeine is a more popular beverage type than decaf. In the south, however, the difference in sales between decaf and caffeinated is much smaller than the other markets. West and Central markets hold caffeinated beverages as the popular choice, but the difference in sales between decaf and cafffeinated beverages are similar.
```SQL
SELECT
    CONCAT(market, ' ', type) as market_with_type,
    SUM(CASE WHEN type = 'Decaf' THEN sales ELSE 0 END) AS decaf_sales,
    SUM(CASE WHEN type = 'Regular' THEN sales ELSE 0 END) AS regular_sales
FROM CoffeeChainSales
GROUP BY market, type
ORDER BY market_with_type
```
<img width="1213" alt="image" src="https://github.com/ToneChaser/ChaseLeBlancPortfolio/assets/145052217/9b1263d4-b25c-4bff-b832-f9e00fac383e">

## Closing Thoughts:
In this project, it was intriguing to uncover notable trends within the dataset. However, it is evident that there is more to explore, with certain aspects such as margins and target profits remaining unexplored. Each overarching question has given rise to specific inquiries, such as understanding the abrupt decline in sales. Furthermore, our analysis revealed differences in sales patterns over time, particularly between caffeinated and decaffeinated beverages, with the latter absolved of blame.

In summary, this project has been an engaging exercise, and I would recommend a concentrated examination of the substantial sales decline observed in 2015. Additionally, investigating the specific states that underperform relative to others could provide insights into how to maximize their potential. The identification of patterns within the coffee company's data is paramount in recommending the subsequent actions for its stakeholders.
