## Advanced Customer and Product Report Analysis:

##                     Project Overview
This is an advanced sql analytics project.
This project analytics covers different levels of project
analysis such as "change over-time trend analysis"
which gives us the overview of total sales in each 
month and year respectively. it gives  a longterm view
over data and change in the aspect if total customers 
gotten, total quantity sold and total sales amount.

In this project we also calculate the "moving average"
and the "running-total", and as well dealing with the advanced
aspect of SQL project such as "Performance Analysis, Part-to-whole 
Analysis, Data Segmentation Analysis". All thess steps will enable us
aggregate customer and product metrics, calculates valuable KPIs to
build our " Customer and Product Report".

## Project Objective:
         - To analyze the change over time trends in month or year
         - To have an overview of total customers, total sales, total quantity sold
                  over a period of time.
         - To ascertain the performance of the products
         - To derive total revenue performance based on customer age group.
         - To ascertain the spending power of customers based on total spending over time
         - To the running total of sales on monthly and yearly bases.
         - To know the total number of orders placed in a month or year
         - To group customers on their spending power.
         - To calculate the price moving average.
         - To ascertain product performance based on category
         - To present a customer and product report.
## Key Problems to be solved:
         - Total revenue generated monthly and yearly
         - Total quantity of products puchased over time
         - Total customers 
         - Total orders placed
         - Average order revenue
         - Customer spending behaviour
         - Customer age group
         - Customer lifespan
         - average monthly revenue per customer
         - product performance based on category
         
## Tool used:
         - Microsoft sql_server

         

## FIRST STAGE ANALYSIS --> ANALYSIS OF CHANGE-OVER-TIME TRENDS: 
#### Here we view the order date and the sales amount.
```sql
select 
	order_date,	
	sales_amount
from fact_sales
where order_date is not null
order by order_date;
```
#### Here we have an overview of the order year and total sales in each year.
```sql
select 
	year(order_date) as order_year,	
	sum(sales_amount) as total_sales
from fact_sales
where order_date is not null
group by year(order_date)
order by year(order_date);
```
#### This gives us longterm view over data, and change over time in the aspect of total customers gotten,  total quantity sold,  and total sales amount.

```sql
select 
	year(order_date) as order_year,	
	sum(sales_amount) as total_sales,
	count(distinct customer_key) as total_customer,     
	sum(quantity) as total_quantity
from fact_sales
where order_date is not null
group by year(order_date)
order by year(order_date);
```
#### Here it gives us an oversight of order_year, order_month, total_sales in each month, total customers in each month of the year and total quantity sold according to the month and year.
```sql
select 
	year(order_date) as order_year,
	month(order_date) as order_month,	
	sum(sales_amount) as total_sales,
	count(distinct customer_key) as total_customer,     
	sum(quantity) as total_quantity
from fact_sales
where order_date is not null
group by year(order_date), MONTH(order_date)
order by year(order_date), MONTH(order_date);
```

#### Here we as well analysed the table based on the year where we have the order year, toatal sales in each year, total customers in each year and as well total quantity sold in each year.
```sql
select 
	datetrunc(year, order_date) as order_year,
	sum(sales_amount) as total_sales,
	count(distinct customer_key) as total_customer,     
	sum(quantity) as total_quantity
from fact_sales
where order_date is not null
group by datetrunc(year, order_date)
order by datetrunc(year, order_date);
```

#### Here we as well analysed the table based on the month where we have the order month, toatal sales in each month, total customers in each month and as well total quantity sold in each month.
```sql
select 
	datetrunc(month, order_date) as order_month,
	sum(sales_amount) as total_sales,
	count(distinct customer_key) as total_customer,     
	sum(quantity) as total_quantity
from fact_sales
where order_date is not null
group by datetrunc(month, order_date)
order by datetrunc(month, order_date);
```


#### Here we organized the order date into year and month abbreviation. but the best option in this project to use still remains 'datetrunc' as we used and demostrated above.
```sql
select 
	format(order_date, 'yyyy-MMM') as order_date,
	sum(sales_amount) as total_sales,
	count(distinct customer_key) as total_customer,     
	sum(quantity) as total_quantity
from fact_sales
where order_date is not null
group by format(order_date, 'yyyy-MMM')
order by format(order_date, 'yyyy-MMM');
```

									
## SECOND STAGE ANALYSIS ---> CUMULATIVE ANALYSIS :

         --- Aggregate the data progressively overtime.
         --- this helps us to understand whether our business is growing or declining.
         --- in this type of analysis, we will use the aggregate window functions:

             --- calculate the total sales per month and year
	    --- and the running total of sales over-time

#### Here we calculates the runing total sales on monthly bases.
```sql
select
	order_date,
	total_sales,
	SUM(total_sales) over (order by order_date) as running_total_sales  ---> window function.
	from
	(
select 
	datetrunc(month, order_date) as order_date,
	sum(sales_amount) as total_sales
from fact_sales
where order_date is not null
group by datetrunc(month, order_date)
	)t;
```



#### We partition and order by order_date the running total sales on monthly individual bases.
```sql
select
	order_date,
	total_sales,
	SUM(total_sales) over (partition by order_date order by order_date) as running_total_sales  
	from																						
	(
select 
	datetrunc(month, order_date) as order_date,
	sum(sales_amount) as total_sales
from fact_sales
where order_date is not null
group by datetrunc(month, order_date)
	)t;
```

#### Here we calculate the running_total sales on yearly bases.
```sql
select
	order_date,
	total_sales,
	SUM(total_sales) over (order by order_date) as running_total_sales  ---> window function.
	from																						
	(
select 
	datetrunc(year, order_date) as order_date,
	sum(sales_amount) as total_sales
from fact_sales
where order_date is not null
group by datetrunc(year, order_date)
	) as t;
```


#### Calculates the moving average of price.
```sql
select
	order_date,
	total_sales,
	SUM(total_sales) over (order by order_date) as running_total_sales, ---> window function.
	avg(avg_price) over (order by order_date) as moving_average_price
	from												(
select 
	datetrunc(year, order_date) as order_date,
	sum(sales_amount) as total_sales,
	avg(price) as avg_price
from fact_sales
where order_date is not null
group by datetrunc(year, order_date)
	) as t;
```

## THIRD  STAGE ANALYSIS ---> PERFORMANCE ANALYSIS:

#### Here we compare the current value to a target value.
####       It helps us to measure success and compare performance in the sense that we subtract the current[measure] - target[measure]
#### for example: current sales - average sales
#### current year sales - previous year sales.  ie --> yoy Analysis. 
#### current sales - lowest sales



#### Task:  analyze the yearly performance of the products by comparing each product's sales to both its average sales performance and the prevous year's sales.
#### ---> we use CTE and window function (common table expression ) 
```sql
with yearly_product_sales as (			
select
	year(f.order_date) as order_year,
	p.product_name,
	sum(f.sales_amount) as current_sales
from fact_sales as f
join dim_products as p
on f.product_key = p.product_key
where order_date is not null
group by year(f.order_date), p.product_name
)
select
	order_year,
	product_name,
	current_sales,
	AVG(current_sales) over (partition by product_name) as avg_sales,
	current_sales - AVG(current_sales) over (partition by product_name) as diff_avg,
	case when current_sales - AVG(current_sales) over (partition by product_name) > 0 then 'above avg'
		 when current_sales - AVG(current_sales) over (partition by product_name) < 0 then 'below avg'
		 else 'average'
		 end avg_change
	from yearly_product_sales
	order by product_name, order_year;
```



#### Here we use 'lag function' to access the previous values of current salesand to the difference between the previous year sales and current year sales and also to know their changes. This type of analysis is called 'year over analysis and year over analysis is good for longterm analysis.
```sql
with yearly_product_sales as (				
select
	year(f.order_date) as order_year,
	p.product_name,
	sum(f.sales_amount) as current_sales
from fact_sales as f
join dim_products as p
on f.product_key = p.product_key
where order_date is not null
group by year(f.order_date), p.product_name
)
select
	order_year,
	product_name,
	current_sales,
	AVG(current_sales) over (partition by product_name) as avg_sales,
	current_sales - AVG(current_sales) over (partition by product_name) as diff_avg,
	case when current_sales - AVG(current_sales) over (partition by product_name) > 0 then 'above avg'
		 when current_sales - AVG(current_sales) over (partition by product_name) < 0 then 'below avg'
		 else 'average'
		 end avg_change,
	  LAG(current_sales) over (partition by product_name order by order_year asc) as previous_year_sales,
	  current_sales - LAG(current_sales) over (partition by product_name order by order_year asc) as diff_previous_year_sales,
	  case when current_sales - LAG(current_sales) over (partition by product_name order by order_year asc) > 0 then 'increase'
	       when current_sales - LAG(current_sales) over (partition by product_name order by order_year asc) < 0 then 'decrease'
		   else 'no change'
		   end previous_year_sales
	from yearly_product_sales
	order by product_name, order_year;
```



#### Month over month analysis. The month over month analysis is for the short term analysis unlike the 'year over year' analysis that focuses on the long term foecast.
```sql
with monthly_product_sales as (				
select
	month(f.order_date) as order_month,
	p.product_name,
	sum(f.sales_amount) as current_sales
from fact_sales as f
join dim_products as p
on f.product_key = p.product_key
where order_date is not null
group by month(f.order_date), p.product_name
)
select
	order_month,
	product_name,
	current_sales,
	AVG(current_sales) over (partition by product_name) as avg_sales,
	current_sales - AVG(current_sales) over (partition by product_name) as diff_avg,
	case when current_sales - AVG(current_sales) over (partition by product_name) > 0 then 'above avg'
		 when current_sales - AVG(current_sales) over (partition by product_name) < 0 then 'below avg'
		 else 'average'
		 end avg_change,
	  LAG(current_sales) over (partition by product_name order by order_month asc) as previous_month_sales,
	  current_sales - LAG(current_sales) over (partition by product_name order by order_month asc) as diff_previous_month_sales,
	  case when current_sales - LAG(current_sales) over (partition by product_name order by order_month asc) > 0 then 'increase'
	       when current_sales - LAG(current_sales) over (partition by product_name order by order_month asc) < 0 then 'decrease'
		   else 'no change'
		   end previous_month_sales
	from monthly_product_sales
	order by product_name, order_month;
```

									---- Advanced Analytics project:
## FOURTH STAGE ANALYSIS	---> PART TO WHOLE ANALYSIS :

#### here we analysze how an individual part is performing compare to the overal, allowing us to understand which category has the greatest impact on the business. Formulae --> ([measure]/ total[measure])*100 by [dimenssion] ie. (sales/total sales)*100 by category or (quantity/total quantity)* 100 by country


#### Task:  which categories contribute the most to the overall sales.
```sql
with category_sales as (
select
	category,
	sum(sales_amount) as total_sales
from fact_sales as f
join dim_products as p
on f.product_key = p.product_key
group by category
)
select
	 category,
	 total_sales,
	 sum(total_sales) over () as overal_sales,
	 concat(round((cast(total_sales as float) / sum(total_sales) over ()) * 100, 2), '%') as percentage_of_total
from category_sales
order by total_sales desc;
```


							--- advanced analytics project.
## FIFTH STAGE ANALYSIS ---> DATA SEGMENTATION  ANALYSIS--- 

#### Here we group the data based on specific range. It helps us to understand the correlation between two measures. ---> example: [measure] by [measure], ie. total products by sales range or total customers by age.

#### Task: segment products into cost ranges and count how many products fall into segment.

```sql
with product_segments as (
select 
	product_key,
	product_name,
	cost,
	case when cost < 100 then 'below 100'
		 when cost between 100 and 500 then '100-500'
		 when cost between 500 and 1000 then '500-1000'
		 else 'above 1000'
		 end cost_range
from dim_products
)
select
	 cost_range,
	 COUNT(product_key) as total_products
from product_segments
group by cost_range
order by total_products desc; 
```


#### TASK: group customers into three segments based on their spending behaviour: 
			-- vip:		at least months of history and spending more than £5,000.
			-- Regular:       at least 12 months of history but spending £5,000 or less.
			-- New:		lifespan less than 12 months.

                                    -- find the total number of custmers by each group
```sql                                    
with customer_spending as (
select
	c.customer_key,
	sum(f.sales_amount) as total_spending ,
	MIN(order_date) as first_order,
	MAX(order_date) as last_order,
	DATEDIFF (MONTH, MIN(order_date), max(order_date)) as lifespan
from fact_sales as f
left join dim_customers as c
on f.customer_key = c.customer_key
group by c.customer_key 
)
	select
		customer_segment,
		COUNT(customer_key) as total_customers
		from (
				select
					customer_key,
					total_spending,
					lifespan,
				case when lifespan >= 12 and total_spending > 5000 then 'vip'
					 when lifespan >= 12 and total_spending <= 5000 then 'regular'
					 else 'new_customer'
				end customer_segment
				from customer_spending) as t
				group by customer_segment
				order by total_customers desc;
```



		
## LAST STAGE ---> building customer report:
#### customer report:
####     purporse:
		- this report consolidates fields such as names, ages, and transaction details.
####     highlights:
		1. Gathers essential fields such as names, ages and transaction details.
		2. Segments customers into categories (vip, regular, new_customer) and age groups.
		3. Aggregates customer-level metrics:
				- total orders
				- total sales
				- total quantity purchased
				- total products 
				- lifespan (in months)
		4. Calculates valuable kpis:
				- recency (months since last order )
				- average order value 
				- average monthly spend.

#### 1. Base Query: Retrieves core columns from tables.

## Customer report: 
```sql
with Base_query as (
select
	f.order_number,
	f.product_key,
	f.sales_amount,
	f.order_date,
	f.quantity,
	c.customer_key,
	c.customer_number,
	concat(c.first_name, '  ', c.last_name) as customer_name,
	datediff(year, c.birthdate, getdate()) as Age
from fact_sales as f
left join dim_customers as c
on f.customer_key = c.customer_key
where f.order_date is not null
),
customer_aggregation as (
---- 2. customer Aggregatons: summerizes key metrics at the customer level.
select
	customer_key,
	customer_number,
	customer_name,
	Age,
	count(distinct order_number) as total_orders,
	sum(sales_amount) as total_sales,
	SUM(quantity) as total_quantity,
	COUNT(distinct product_key) as total_products,
	MAX(order_date) as last_order_date,
	MIN(order_date) as first_order_date,
	DATEDIFF(MONTH, MIN(order_date), max(order_date)) as lifespan
from Base_query
group by
	customer_key,
	customer_number,
	customer_name,
	Age
) 
select
	customer_key,
	customer_number,
	customer_name,
	Age,
case when Age < 20 then 'under 20'
	 when Age between 20 and 29 then '20-29'
	 when Age between 30 and 39 then '30-39'
	 when Age between 40 and 49 then '40-49'
	 else '50 and avove'
end as age_group,
case when lifespan >= 12 and total_sales > 5000 then 'vip'
	 when lifespan >= 12 and total_sales <= 5000 then 'regular'
	 else 'new_customer'
	 end as customer_segment,
	total_orders,
	total_sales,
	total_quantity,
	total_products,
	last_order_date,
DATEDIFF(MONTH, last_order_date, getdate()) as recency,
	first_order_date,
DATEDIFF(MONTH, first_order_date, GETDATE()) as latency,
	lifespan,
--- compute average order value (AVO)
	case when total_sales = 0 then 0
	else total_sales / total_orders
	end  as avg_order_value,
--- average monthly spend
	case when lifespan = 0 then total_sales
	else total_sales / lifespan
	end as average_monthly_spend
from customer_aggregation;
```


## PRODUCT REPORT
#### PURPOSE:
	- This report consolidates key product metrics and behaviours.

#### Highligths: 
	1. Gathers essential fielsds such as product name, category, subcategory, and cost.
	2. Segments products by revenue to identity high-performers, mid-performers or low-performers.
	3. Aggregates product-level metrics:
			- total orders 
			- total sales 
			- total quantity
			- total customers (unique)
			- lifespan (in months)
	4. Calculates valuable KPIs:
			- recency (months since last sale)
			- average order revenue (AOR)
			- average monthly revenue


```sql
with Base_query as (
select	 
	 f.order_number,
	 f.order_date,
	 sales_amount,
	 f.quantity,
	 f.customer_key,
	 p.product_name,
	 p.category,
	 p.subcategory,
	 p.product_key,
	 p.cost
from fact_sales as f
left join dim_products as p
on f.product_key = p.product_key
where order_date is not null
),
product_aggregation as (
--- product_aggregation:summerizes key metrics at the product level.
select
	 product_name,
	 category,
	 subcategory,
	 count(distinct product_key) as total_products,
	 count(distinct customer_key) as total_customers,
	 MAX(order_date) as last_sales_date,
	 datediff(month, min(order_date), max(order_date)) as lifespan,
	 sum(cost) as total_cost,	
	 count(distinct order_number)  as total_order_number,
	 sum(quantity) as total_quantity,
	 sum(sales_amount) as total_revenue
from Base_query
group by
	product_name,
	 category,
	 subcategory
)
--- calculations of valuable KPIs:
select
	product_name,
	category,
	subcategory,
	total_products,
	total_customers,
	last_sales_date,
	lifespan,
	total_cost,
	total_order_number,
	total_quantity,
	total_revenue,
	case when total_revenue > 1000000 then 'high performer'
		 when total_revenue between 400000 and 1000000 then 'mid performer'
		 else 'low performer'
		 end as 'product performance',
	datediff(month,last_sales_date, getdate()) as recency,
	case when total_revenue = 0 then 0
	else total_revenue / total_order_number
	end as Average_order_revenue,
case when lifespan = 0 then total_revenue
	 else total_revenue / lifespan
	 end as 'avg_monthly_revenue'
from product_aggregation;
```

	

      
