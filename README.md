# walmart_mock_training
# Walmart Data Analysis: End-to-End SQL + Python Project P-9

## Project Overview

![Project Pipeline](https://github.com/najirh/Walmart_SQL_Python/blob/main/walmart_project-piplelines.png)


This project is an end-to-end data analysis solution designed to extract critical business insights from Walmart sales data. We utilize Python for data processing and analysis, SQL for advanced querying, and structured problem-solving techniques to solve key business questions. The project is ideal for data analysts looking to develop skills in data manipulation, SQL querying, and data pipeline creation.

---

## Project Steps

### 1. Set Up the Environment
   - **Tools Used**: Visual Studio Code (VS Code), Python, SQL (MySQL and PostgreSQL)
   - **Goal**: Create a structured workspace within VS Code and organize project folders for smooth development and data handling.

### 2. Set Up Kaggle API
   - **API Setup**: Obtain your Kaggle API token from [Kaggle](https://www.kaggle.com/) by navigating to your profile settings and downloading the JSON file.
   - **Configure Kaggle**: 
      - Place the downloaded `kaggle.json` file in your local `.kaggle` folder.
      - Use the command `kaggle datasets download -d <dataset-path>` to pull datasets directly into your project.

### 3. Download Walmart Sales Data
   - **Data Source**: Use the Kaggle API to download the Walmart sales datasets from Kaggle.
   - **Dataset Link**: [Walmart Sales Dataset](https://www.kaggle.com/najir0123/walmart-10k-sales-datasets)
   - **Storage**: Save the data in the `data/` folder for easy reference and access.

### 4. Install Required Libraries and Load Data
   - **Libraries**: Install necessary Python libraries using:
     ```bash
     pip install pandas numpy sqlalchemy mysql-connector-python psycopg2
     ```
   - **Loading Data**: Read the data into a Pandas DataFrame for initial analysis and transformations.

### 5. Explore the Data
   - **Goal**: Conduct an initial data exploration to understand data distribution, check column names, types, and identify potential issues.
   - **Analysis**: Use functions like `.info()`, `.describe()`, and `.head()` to get a quick overview of the data structure and statistics.

### 6. Data Cleaning
   - **Remove Duplicates**: Identify and remove duplicate entries to avoid skewed results.
   - **Handle Missing Values**: Drop rows or columns with missing values if they are insignificant; fill values where essential.
   - **Fix Data Types**: Ensure all columns have consistent data types (e.g., dates as `datetime`, prices as `float`).
   - **Currency Formatting**: Use `.replace()` to handle and format currency values for analysis.
   - **Validation**: Check for any remaining inconsistencies and verify the cleaned data.

### 7. Feature Engineering
   - **Create New Columns**: Calculate the `Total Amount` for each transaction by multiplying `unit_price` by `quantity` and adding this as a new column.
   - **Enhance Dataset**: Adding this calculated field will streamline further SQL analysis and aggregation tasks.

### 8. Load Data into MySQL and PostgreSQL
   - **Set Up Connections**: Connect to MySQL and PostgreSQL using `sqlalchemy` and load the cleaned data into each database.
   - **Table Creation**: Set up tables in both MySQL and PostgreSQL using Python SQLAlchemy to automate table creation and data insertion.
   - **Verification**: Run initial SQL queries to confirm that the data has been loaded accurately.

### 9. SQL Analysis: Complex Queries and Business Problem Solving
   - **Business Problem-Solving**: Write and execute complex SQL queries to answer critical business questions, such as:
     - Revenue trends across branches and categories.
     - Identifying best-selling product categories.
     - Sales performance by time, city, and payment method.
     - Analyzing peak sales periods and customer buying patterns.
     - Profit margin analysis by branch and category.
   - **Documentation**: Keep clear notes of each query's objective, approach, and results.
```sql
   ##1. Find the different payment methods, number of transactions, and  quantity sold. 
    SELECT
      	payment_method,
      	count(*)as no_payments,
      	sum(quantity)
    from
    	walmart
    GROUP
    	BY
    	payment_method;
```

```sql
   ## 2. Identify the highest-rated category in each branch, displaying the branch, category, AVG RATING.
   select *
   from
   (	select 
   		branch,
   		category,
   		avg(rating) as AVG_RATING,
   		rank() over(partition by branch order by avg(rating) desc) as rank
   	from
   		walmart
   	group
   		by
   		1, 2
   )
   where rank = 1
```
```sql
   ## Identify the busiest day for each branch based on the number of transactions
   select *
   from
   	(select 
   		branch,
   		to_char(to_date(date, 'DD/MM/YY'), 'DAY') as day_name,
   		count(*) as busy_days,
   		rank() over(partition by branch order by count(*) desc) as rank
   	from 
   		walmart
   	group
   		by
   		1,2
   	)
   where rank = 1;
```
```sql
   ##Determine the average, minimum, and maximum rating of products for each city.
   ##List the city, average_rating,  and min_rating 
   select
   	city,
   	category,
   	min(rating) as min_rating,
   	max(rating) as max_rating,
   	avg(rating) as avg_rating
   from
   	walmart
   group
   	by
   	1,2
```
```sql
   ##Calculate the total profit for each category by considering total_profit as
   ## (Unit price * quantity * profit_margin)
   ## List category and total_profit, ordered from highest to lowest profit.
   select
   	category,
   	sum(total) as total_revenue,
   	sum(unit_price * quantity * profit_margin) as total_profit
   from
   	walmart
   group
   	by
   	1
   	order
   		by
   		total_profit desc;
```
```sql
   -- 7.
   -- Determin the most common payment method for each branch --
   -- Display branch and the preferred_payment_method. --
   with cte
   as
   (select 
   	branch,
   	payment_method,
   	count(*) as preferred_payment_method,
   	rank() over(partition by branch order by count(*) desc) as rank
   from
   	walmart
   group
   	by
   	1,2)
   select * from cte
   where rank = 1
```
```sql
   ##8.
   ## Categorize sales into 3 groups: MORNING, AFTERNOON, EVENING
   ## FIND OUT each of the shifts and number of invoices
   select * from walmart;
   select
   	branch,
   	case 
   	when extract (hour from(time::time)) < 12 then 'Morning'
   	when extract (hour from(time::time)) between 12 and 17 then 'Afternoon'
   	else
   		'Evening'
   	end shifts,
   	count(*)
   from 
   	walmart
   group 
   	by
   	1,2
   order
   	by 1,3 desc;
```
```sql
## 9)
## Identify 5 branches with the highest decreased ratio in
## revenue compare to last year(current year 2023 and last year 2022)

with revenue_2022
as
(	
	select
		branch,
		sum(total) as revenue
	from
		walmart
	where extract(year from to_date(date, 'DD/MM/YY')) = 2022
	group 
		by 1
),

revenue_2023
as

(
	select
		branch,
		sum(total) as revenue
	from
		walmart
	where extract(year from to_date(date, 'DD/MM/YY')) = 2023
	group 
		by 1
)
select 
	ls.branch,
	ls.revenue as last_year_revenue,
	cs.revenue as cr_year_revenue,
	round(
	(ls.revenue - cs.revenue)::numeric/
	ls.revenue::numeric * 100,
	2)	as revenue_decrease_ratio
from revenue_2022 as ls
join
revenue_2023 as cs
on ls.branch = cs.branch
where
	ls.revenue > cs.revenue
order by 4 desc
limit 5;
```
```pythpn
	# Step 1 Data Exploration & Leading
	# import dependencies
	import pandas as pd
	
	# mysql tool kit
	import pymysql  # this will work as an adapter
	from sqlalchemy import create_engine
	
	# psql
	import psycopg2
	
	print(pd.__version__)
	pd.set_option("display.max_columns", None)  # Display all columns
	# pd.set_option("display.width", None)  # Optional: Prevents line wrapping
	# pd.set_option("display.max_colwidth", None)
	df = pd.read_csv("walmart_clean_data.csv", encoding_errors="ignore")
	df.shape
	df.head()
	df.describe()
	df.info()
	# all duplicates
	df.duplicated().sum()
	df.isnull().sum()
	df.drop_duplicates(inplace=True)
	df.duplicated().sum()
	df.shape
	# dropping all rows with missing records
	df.dropna(inplace=True)
	# verify if nulls have been dropped.
	df.isnull().sum()
	df.shape
	df.dtypes
	df["unit_price"].astype(float)
	df["unit_price"] = df["unit_price"].str.replace("$", "").astype(float)
	df.head()
	df.info()
	df.columns
	df["total"] = df["unit_price"] * df["quantity"]
	df.head()
	df.shape
	
	
	
	df.to_csv("walmart_clean_data.csv", index=False)
	df.to_sql
	df.columns = df.columns.str.lower()
	df.columns
	engine_psql = create_engine(
	    "postgresql+psycopg2://postgres:password@localhost:5432/walmart_clean_data"
	)
	try:
	    engine_psql
	    print("Connection successful to psql")
	except:
	    print("Unsuccessful database connection.")
	
	df.to_sql(name="walmart_clean_data", con=engine_psql, if_exists="append", index=False)
	df.columns
	df.columns
```
### 10. Project Publishing and Documentation
   - **Documentation**: Maintain well-structured documentation of the entire process in Markdown or a Jupyter Notebook.
   - **Project Publishing**: Publish the completed project on GitHub or any other version control platform, including:
     - The `README.md` file (this document).
     - Jupyter Notebooks (if applicable).
     - SQL query scripts.
     - Data files (if possible) or steps to access them.

---

## Requirements

- **Python 3.8+**
- **SQL Databases**: MySQL, PostgreSQL
- **Python Libraries**:
  - `pandas`, `numpy`, `sqlalchemy`, `mysql-connector-python`, `psycopg2`
- **Kaggle API Key** (for data downloading)

## Getting Started

1. Clone the repository:
   ```bash
   git clone <repo-url>
   ```
2. Install Python libraries:
   ```bash
   pip install -r requirements.txt
   ```
3. Set up your Kaggle API, download the data, and follow the steps to load and analyze.

---

## Project Structure

```plaintext
|-- data/                     # Raw data and transformed data
|-- sql_queries/              # SQL scripts for analysis and queries
|-- notebooks/                # Jupyter notebooks for Python analysis
|-- README.md                 # Project documentation
|-- requirements.txt          # List of required Python libraries
|-- main.py                   # Main script for loading, cleaning, and processing data
```
---

## Results and Insights

This section will include your analysis findings:
- **Sales Insights**: Key categories, branches with highest sales, and preferred payment methods.
- **Profitability**: Insights into the most profitable product categories and locations.
- **Customer Behavior**: Trends in ratings, payment preferences, and peak shopping hours.

## Future Enhancements

Possible extensions to this project:
- Integration with a dashboard tool (e.g., Power BI or Tableau) for interactive visualization.
- Additional data sources to enhance analysis depth.
- Automation of the data pipeline for real-time data ingestion and analysis.

---



