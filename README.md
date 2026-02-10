# Monday Coffee Expansion SQL Project

![Company Logo](https://github.com/najirh/Monday-Coffee-Expansion-Project-P8/blob/main/1.png)

## Objective
The goal of this project is to analyze the sales data of Monday Coffee, a company that has been selling its products online since January 2023, and to recommend the top three major cities in India for opening new coffee shop locations based on consumer demand and sales performance.

## Project Structure

### 1. Database Setup

- **Database Creation**: The project starts by creating a database named `Monday_coffee`.
- **Table Creation**:
  The following tables are created to organize and manage the project data effectively:

`sales`: Stores all sales transaction data.

`city`: Stores information related to different cities.

`customer`: Stores customer details and profiles.

`product`: Stores product-related information.
```sql


create database Monday_coffee;
use Monday_coffee;

DROP TABLE IF EXISTS sales;
DROP TABLE IF EXISTS customers;
DROP TABLE IF EXISTS products;
DROP TABLE IF EXISTS city;

-- Monday Coffee SCHEMAS

DROP TABLE IF EXISTS sales;
DROP TABLE IF EXISTS customers;
DROP TABLE IF EXISTS products;
DROP TABLE IF EXISTS city;

-- Import Rules
-- 1st import to city
-- 2nd import to products
-- 3rd import to customers
-- 4th import to sales


CREATE TABLE city
(
	city_id	INT PRIMARY KEY,
	city_name VARCHAR(15),	
	population	BIGINT,
	estimated_rent	FLOAT,
	city_rank INT
);

CREATE TABLE customers
(
	customer_id INT PRIMARY KEY,	
	customer_name VARCHAR(25),	
	city_id INT,
	CONSTRAINT fk_city FOREIGN KEY (city_id) REFERENCES city(city_id)
);


CREATE TABLE products
(
	product_id	INT PRIMARY KEY,
	product_name VARCHAR(35),	
	Price float
);


CREATE TABLE sales
(
	sale_id	INT PRIMARY KEY,
	sale_date	date,
	product_id	INT,
	customer_id	INT,
	total FLOAT,
	rating INT,
	CONSTRAINT fk_products FOREIGN KEY (product_id) REFERENCES products(product_id),
	CONSTRAINT fk_customers FOREIGN KEY (customer_id) REFERENCES customers(customer_id) 
);

-- END of SCHEMAS
```
### 2. Data Analysis & Findings

The following SQL queries were developed to answer specific business questions:

**Q1. Coffee Consumers Count
-- How many people in each city are estimated to consume coffee, given that 25% of the population does?**

```sql
select city_name,
round(population * 0.25/1000000,2) as Consume_coffee_Millions,
city_rank from city
order by 2 desc; 

```
-- Q2. Total Revenue from Coffee Sales
-- What is the total revenue generated from coffee sales across all cities in the last quarter of 2023?
```sql

select ci.city_name,sum(s.total) as total_revenue
from sales as s
join customers as c
on s.customer_id=c.customer_id
join city as ci
on ci.city_id=c.city_id
where year(s.sale_date)=2023 AND quarter(s.sale_date)=4
group by ci.city_name
order by 2 desc;

```
-- Q3. Sales Count for Each Product
-- How many units of each coffee product have been sold?
```sql

select p.product_name,count(s.product_id) as Unit_sold
from products as p 
left join sales as s
on p.product_id=s.product_id
group by  p.product_id,p.product_name
order by 2 desc;

```
-- Q4. Average Sales Amount per City
-- What is the average sales amount per customer in each city?
```sql

select ci.city_name,round(sum(s.total),2)  as Total_sales,
count(distinct s.customer_id) as No_customer, 
round(sum(s.total)/count(distinct s.customer_id),2) as Average_per_customer
from sales as s
join customers as c
on s.customer_id=c.customer_id
join city as ci
on c.city_id=ci.city_id
group by 1
order by 2 desc;

```
-- Q5. City Population and Coffee Consumers
-- Provide a list of cities along with their populations and estimated coffee consumers.
```sql

with cte_city as(
select  city_name,
round(population * 0.25/1000000,2) as coffee_consumer
from city),cte_cus as(
select ci.city_name,count(distinct s.customer_id) as No_customer
from sales as s
join customers as c
on s.customer_id=c.customer_id
join city as ci
on c.city_id=ci.city_id
group by 1)
select cte_city.city_name,cte_city.coffee_consumer,cte_cus.No_customer
from cte_city 
join cte_cus
on cte_city.city_name = cte_cus.city_name;

```
-- Q6. Top Selling Products by City
-- What are the top 3 selling products in each city based on sales volume?
```sql

with cte as(
select ct.city_name, p.product_name,count(s.product_id)  as Total_buy
, dense_rank() over(partition by ct.city_name order by count(s.product_id) desc) as rnk
from products as p
join sales as s
on p.product_id=s.product_id
join customers as c
on c.customer_id=s.customer_id
join city as ct
on c.city_id=ct.city_id
group by 1,2)
select * from cte
where rnk<4;

```
-- 7. customer Segmentation by City
-- How many unique customers are there in each city who have purchased coffee products?
```sql

select ct.city_name,count(distinct s.customer_id)
from  city as ct
left join customers as c
on ct.city_id=c.city_id
join sales as s 
on s.customer_id=c.customer_id
where s.product_id in (1,2,3,4,5,6,7,8,9,10,11,12,13,14)
group by 1;

```
-- 8) Average Sale vs Rent
-- Find each city and their average sale per customer and avg rent per customer
```sql

with city_table as(
select ci.city_name,count(distinct s.customer_id)  as disnt_customer, 
round(sum(s.total)/count(distinct s.customer_id),2) as Average_per_customer
from sales as s
join customers as c
on s.customer_id=c.customer_id
join city as ci
on c.city_id=ci.city_id
group by 1
order by 2 desc) ,
city_rent as(
select city_name,estimated_rent from city)
select ct.city_name, cr.estimated_rent,ct.disnt_customer,ct.Average_per_customer,round(cr.estimated_rent/ct.disnt_customer,2) as Average_rent_customer
from city_table as ct
join city_rent as cr
on ct.city_name= cr.city_name
order by 4 desc;

```
-- Q9. Monthly Sales Growth
-- Sales growth rate: Calculate the percentage growth (or decline) in sales over different time periods (monthly).
-- by each city
```sql
with Monthly_sales as(
select  ct.city_name,
month(s.sale_date) as Mnth ,
Year(s.sale_date) as Yr, sum(s.total) as Total_sales
from sales as s
join customers as c
on s.customer_id=c.customer_id
join city as ct 
on ct.city_id=c.city_id
group by 1,2,3
order by 1,3,2), growth_rate as(
select city_name,Mnth,Yr,Total_sales as cr_month_sales,
lag(Total_sales,1) over(partition by city_name) as last_month_sales from Monthly_sales)
select city_name,
Mnth,Yr
,cr_month_sales
,last_month_sales,
round((cr_month_sales-last_month_sales)/last_month_sales*100,2) as Growth_ratio
from growth_rate
where last_month_sales is not null;

```
-- Q10. Market Potential Analysis
-- Identify top 3 city based on highest sales, return city name, total sale, total rent, total customers, estimated coffee consumer
```sql

with city_table as(
select ci.city_name,sum(s.total) as total_sales ,count(distinct s.customer_id)  as disnt_customer, 
round(sum(s.total)/count(distinct s.customer_id),2) as Average_per_customer
from sales as s
join customers as c
on s.customer_id=c.customer_id
join city as ci
on c.city_id=ci.city_id
group by 1
order by 2 desc) ,
city_rent as(
select city_name,
(population * 0.25)/1000000 as estimated_coffee_consumer,
estimated_rent from city)
select ct.city_name,
ct.total_sales, 
cr.estimated_rent,
ct.disnt_customer,cr.estimated_coffee_consumer as estimated_coffee_consumer_million,
Average_per_customer,round(cr.estimated_rent/ct.disnt_customer,2) as Average_Rent_per_customer
from city_table as ct
join city_rent as cr
on ct.city_name= cr.city_name
order by 2 desc;

    
```
## Recommendations
After analyzing the data, the recommended top three cities for new store openings are:

**City 1: Pune**  
1. Average rent per customer is very low.  
2. Highest total revenue.  
3. Average sales per customer is also high.

**City 2: Delhi**  
1. Highest estimated coffee consumers at 7.7 million.  
2. Highest total number of customers, which is 68.  
3. Average rent per customer is 330 (still under 500).

**City 3: Jaipur**  
1. Highest number of customers, which is 69.  
2. Average rent per customer is very low at 156.  
3. Average sales per customer is better at 11.6k.

```
