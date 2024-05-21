# Bike-Store- Sales Analysis Dashboard

## Table of contents
- [Project Overview](#project-overview) 
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Data Cleaning and Preparation](#data-cleaning-and-preparation)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Data Analysis](#data-analysis)
- [Limitation](#limitation)
- [References](#references)

### Project Overview 
This data analysis project aims to provide insights into the sales performance of a Bike store company over the years 2016 to 2018. By analyzing various aspects of the sales data, we seek to identify the different trends, make data-driven recommendations, and gain a deeper understanding of the company's performance to help management make informed decisions. 

### Data Sources 
Sales Data: The primary dataset used for this analysis is the "Bikestore.xlsx" file, containing detailed information about each sale made by the company.

### Tools
- Microsoft SQL Server (SQL) - Data Analysis 
- Azure Data Studio - used to host MSSQL
- Microsoft Excel - Data cleaning, Data Analysis 
- Tableau - Create Dashboard Report

### Data Cleaning and Preparation
In the initial data preparation phase, we performed the following tasks: 

1.	Connect to the company’s regional database by writing an SQL script to generate a detailed data set that will provide all the data and information needed.
2.	Load and inspect data.
3.	Handled missing values.
4.	Data cleaning and formatting.
5.	Import the SQL-generated dataset into Excel and then use pivot tables to generate a dynamic dashboard.
6.	Connect the Excel datasheet  that contains the SQL-generated dataset to Tableau and use the information to generate a visually pleasing and dynamic dashboard for management.

### Exploratory Data Analysis

Explored the sales data to answer key questions, such as:

- Who are the top customers?
- What was the yearly and monthly revenue from 2016-2018?
- What was the  revenue per state?
- What was each store's revenue?
- What was the revenue per category?
- What was the revenue per sales rep?

### Data Analysis 
  Code/features worked with
  
```SQL

CREATE DATABASE Bikestore;
go 

Use Bikestore;
go

-- create schemas
CREATE SCHEMA production;
go

CREATE SCHEMA sales;
go

-- create tables
CREATE TABLE production.categories (
	category_id INT IDENTITY (1, 1) PRIMARY KEY,
	category_name VARCHAR (255) NOT NULL
);

CREATE TABLE production.brands (
	brand_id INT IDENTITY (1, 1) PRIMARY KEY,
	brand_name VARCHAR (255) NOT NULL
);

CREATE TABLE production.products (
	product_id INT IDENTITY (1, 1) PRIMARY KEY,
	product_name VARCHAR (255) NOT NULL,
	brand_id INT NOT NULL,
	category_id INT NOT NULL,
	model_year SMALLINT NOT NULL,
	list_price DECIMAL (10, 2) NOT NULL,
	FOREIGN KEY (category_id) REFERENCES production.categories (category_id) ON DELETE CASCADE ON UPDATE CASCADE,
	FOREIGN KEY (brand_id) REFERENCES production.brands (brand_id) ON DELETE CASCADE ON UPDATE CASCADE
);

CREATE TABLE sales.customers (
	customer_id INT IDENTITY (1, 1) PRIMARY KEY,
	first_name VARCHAR (255) NOT NULL,
	last_name VARCHAR (255) NOT NULL,
	phone VARCHAR (25),
	email VARCHAR (255) NOT NULL,
	street VARCHAR (255),
	city VARCHAR (50),
	state VARCHAR (25),
	zip_code VARCHAR (5)
);

CREATE TABLE sales.stores (
	store_id INT IDENTITY (1, 1) PRIMARY KEY,
	store_name VARCHAR (255) NOT NULL,
	phone VARCHAR (25),
	email VARCHAR (255),
	street VARCHAR (255),
	city VARCHAR (255),
	state VARCHAR (10),
	zip_code VARCHAR (5)
);

CREATE TABLE sales.staffs (
	staff_id INT IDENTITY (1, 1) PRIMARY KEY,
	first_name VARCHAR (50) NOT NULL,
	last_name VARCHAR (50) NOT NULL,
	email VARCHAR (255) NOT NULL UNIQUE,
	phone VARCHAR (25),
	active tinyint NOT NULL,
	store_id INT NOT NULL,
	manager_id INT,
	FOREIGN KEY (store_id) REFERENCES sales.stores (store_id) ON DELETE CASCADE ON UPDATE CASCADE,
	FOREIGN KEY (manager_id) REFERENCES sales.staffs (staff_id) ON DELETE NO ACTION ON UPDATE NO ACTION
);

CREATE TABLE sales.orders (
	order_id INT IDENTITY (1, 1) PRIMARY KEY,
	customer_id INT,
	order_status tinyint NOT NULL,
	-- Order status: 1 = Pending; 2 = Processing; 3 = Rejected; 4 = Completed
	order_date DATE NOT NULL,
	required_date DATE NOT NULL,
	shipped_date DATE,
	store_id INT NOT NULL,
	staff_id INT NOT NULL,
	FOREIGN KEY (customer_id) REFERENCES sales.customers (customer_id) ON DELETE CASCADE ON UPDATE CASCADE,
	FOREIGN KEY (store_id) REFERENCES sales.stores (store_id) ON DELETE CASCADE ON UPDATE CASCADE,
	FOREIGN KEY (staff_id) REFERENCES sales.staffs (staff_id) ON DELETE NO ACTION ON UPDATE NO ACTION
);

CREATE TABLE sales.order_items (
	order_id INT,
	item_id INT,
	product_id INT NOT NULL,
	quantity INT NOT NULL,
	list_price DECIMAL (10, 2) NOT NULL,
	discount DECIMAL (4, 2) NOT NULL DEFAULT 0,
	PRIMARY KEY (order_id, item_id),
	FOREIGN KEY (order_id) REFERENCES sales.orders (order_id) ON DELETE CASCADE ON UPDATE CASCADE,
	FOREIGN KEY (product_id) REFERENCES production.products (product_id) ON DELETE CASCADE ON UPDATE CASCADE
);

CREATE TABLE production.stocks (
	store_id INT,
	product_id INT,
	quantity INT,
	PRIMARY KEY (store_id, product_id),
	FOREIGN KEY (store_id) REFERENCES sales.stores (store_id) ON DELETE CASCADE ON UPDATE CASCADE,
	FOREIGN KEY (product_id) REFERENCES production.products (product_id) ON DELETE CASCADE ON UPDATE CASCADE
);


SELECT
 ord. order_id ,
 CONCAT(cus.first_name, ' ', cus.last_name) AS 'customers',
 cus.city,
 cus.state,
 ord.order_date,
 SUM(ite.quantity) AS 'total_units',
 SUM(ite.quantity * ite.list_price) AS 'revenue',
 pro.product_name,
 cat.category_name,
 sto.store_name,
 CONCAT(sta.first_name,' ',sta.last_name) AS 'sales_rep'
FROM sales.orders ord
 JOIN sales.customers cus 
 ON ord.customer_id = cus.customer_id
 JOIN  sales.order_items ite 
 ON ord.order_id = ite.order_id 
 JOIN production.products pro
 ON ite.product_id = pro.product_id
 JOIN production.categories cat 
 ON pro.category_id = cat.category_id
 JOIN sales.stores sto
 ON  ord.store_id = sto.store_id
 JOIN sales.staffs sta
 ON ord.staff_id = sta.staff_id
 GROUP BY 
ord. order_id ,
 CONCAT(cus.first_name, ' ', cus.last_name),
 cus.city,
 cus.state,
 ord.order_date,
 pro.product_name,
 cat.category_name,
 sto.store_name ,
 CONCAT(sta.first_name,' ',sta.last_name) 
```
### Results and Finding

The analysis results are summarized as follows:
1. The year 2018 was the best in revenue.
2. The bike sales increased on average from February to April.
3. The company sells more bikes in New York compared to Texas or California (lowest sales ).
4. The Baldwin Bike store makes 67.91% of the total bike sales of the company.
5. The mountain Bikes are the most popular bikes bought by customers.
6. The top customer of the company is Pamelia Newman who spent $37,802.
7. The best sales rep is Marcelene Boyer totaling $2,938,889 in sales revenue in her actif. 


### Recommendation

Based on the analysis, we recommend the following actions:

- Invest in marketing and promotions during peak seasons (February to April) to maximize revenue.
- Focus on expanding more on the Mountain Bikes and road bikes product category.
- Implement a customer segmentation to target customers effectively.

### Limitation 
  None

### References

1. Dataset [https://docs.google.com/spreadsheets/d/1ESMiCguVJjUzjVNxLffngDrHsQcMFHrt/edit#gid=1194135803]
  








