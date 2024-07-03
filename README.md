<p>

# Orders Dataset Analysis and SQL Integration

This project entails cleaning and analyzing an orders dataset, followed by loading the cleaned data into a MySQL database and performing various SQL queries to extract meaningful insights. The analysis and integration steps are outlined below.

---

## Dataset Overview

The dataset includes various attributes related to orders, such as:

- **order_id**: Unique identifier for each order
- **order_date**: Date when the order was placed
- **ship_mode**: Mode of shipment
- **customer_id**: Unique identifier for each customer
- **city**: City of the customer
- **region**: Region of the customer
- **product_id**: Unique identifier for each product
- **list_price**: Listed price of the product
- **cost_price**: Cost price of the product
- **discount_percent**: Discount percentage applied to the product
- **sale_price**: Sale price of the product after discount

## Data Cleaning and Preprocessing

Several steps were taken to clean and preprocess the data:

1. **Loading the Data**: Imported the dataset and handled missing values.
   ```python
   df = pd.read_csv('C:\\Users\\Lab-02-06\\Downloads\\orders.csv', na_values=['Not Available', 'unknown'])
   ```

2. **Checking for Missing Values**: Identified columns with missing values.
   ```python
   df.isnull().sum()
   ```

3. **Renaming Columns**: Standardized column names to lowercase and replaced spaces with underscores.
   ```python
   df.rename(columns={'Order Id':'order_id', 'City':'city'})
   df.columns = df.columns.str.lower()
   df.columns = df.columns.str.replace(' ', '_')
   ```

4. **Calculating Additional Columns**: Computed discount, sale price, and profit.
   ```python
   df['discount'] = df['list_price'] * df['discount_percent'] * 0.01  
   df['sale_price'] = df['list_price'] - df['discount_percent']
   df['profit'] = df['sale_price'] - df['cost_price']
   ```

5. **Converting Date Columns**: Converted the `order_date` column to datetime format.
   ```python
   df['order_date'] = pd.to_datetime(df['order_date'], format="%Y-%m-%d")
   ```

6. **Dropping Unnecessary Columns**: Removed columns that are no longer needed.
   ```python
   df.drop(columns=['list_price', 'cost_price', 'discount_percent'], inplace=True)
   ```

## SQL Integration

The cleaned data was then loaded into a MySQL database and several SQL queries were performed:

1. **Establishing Database Connection**: Connected to the MySQL database using SQLAlchemy and PyMySQL.
   ```python
   from sqlalchemy import create_engine
   import pymysql 

   username = 'root'
   password = 'root'
   server = 'localhost'
   port = '3306'
   dbname = 'sakila'

   connection_url = f'mysql+pymysql://{username}:{password}@{server}:{port}/{dbname}'
   engine = create_engine(connection_url)

   try:
       connection = engine.connect()
       print("Connection successful!")
       connection.close()
   except Exception as e:
       print(f"Connection failed: {e}")
   ```

2. **Loading Data into MySQL**: Loaded the cleaned data into a table named `df_orders`.
   ```python
   df.to_sql('df_orders', con=engine, index=False, if_exists='append')
   ```

3. **SQL Queries**:
   - **Top 10 Highest Revenue Generating Products**:
     ```sql
     SELECT product_id, SUM(sale_price) AS sales
     FROM df_orders
     GROUP BY product_id
     ORDER BY sales DESC
     LIMIT 10;
     ```

   - **Top 5 Highest Selling Products in Each Region**:
     ```sql
     WITH cte AS (
         SELECT region, product_id, SUM(sale_price) AS sales
         FROM df_orders
         GROUP BY region, product_id
     )
     SELECT * FROM (
         SELECT *, ROW_NUMBER() OVER(PARTITION BY region ORDER BY sales DESC) AS rn
         FROM cte
     ) A
     WHERE rn <= 5;
     ```

   - **Month-over-Month Growth Comparison for 2022 and 2023 Sales**:
     ```sql
     WITH cte AS (
         SELECT YEAR(order_date) AS order_year, MONTH(order_date) AS order_month, SUM(sale_price) AS sales
         FROM df_orders
         GROUP BY YEAR(order_date), MONTH(order_date)
     )
     SELECT order_month,
            SUM(CASE WHEN order_year = 2022 THEN sales ELSE 0 END) AS sales_2022,
            SUM(CASE WHEN order_year = 2023 THEN sales ELSE 0 END) AS sales_2023
     FROM cte
     GROUP BY order_month
     ORDER BY order_month;
     ```

## Conclusion

This project showcases the complete process of data cleaning, preprocessing, and integration with a MySQL database. The SQL queries performed provide valuable insights into the sales performance and trends, aiding in data-driven decision-making.

## Requirements

- Python 3.x
- pandas
- numpy
- matplotlib
- seaborn
- sqlalchemy
- pymysql

## Usage

To run this analysis, clone the repository and execute the Jupyter notebook or Python script provided. Make sure to install the required libraries using:
```bash
pip install -r requirements.txt
```

## License

This project is licensed under the MIT License.

---
  
</p>
