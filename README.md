# tera-data-
Tera data to hadoop -etl 
# teradata-to-hadoop-etl/
# Project structure with README and example scripts

# ===================== README.md =====================
# A guide for setting up the data migration and ETL pipeline from Teradata to Hadoop

"""
# Customer Analytics: Teradata to Hadoop ETL Pipeline

This project demonstrates how to migrate data from a Teradata database to the Hadoop ecosystem and perform ETL (Extract, Transform, Load) operations using Sqoop, Hive, and Spark.

## Tools Used
- Teradata (source database)
- Sqoop (data ingestion)
- HDFS (storage)
- Hive (SQL transformation)
- PySpark (ETL pipeline)

## Project Structure
```
teradata-to-hadoop-etl/
├── sqoop/
│   ├── import_customers.sh
│   ├── import_sales.sh
│   └── import_products.sh
├── hive/
│   ├── create_tables.hql
│   └── transform_etl.hql
├── spark/
│   └── transform_etl.py
└── README.md
```

## Getting Started
### Prerequisites
- Hadoop and HDFS set up
- Hive and Spark installed
- Teradata JDBC driver and credentials

### 1. Import Data with Sqoop
Go to the `sqoop/` directory and run:
```bash
bash import_customers.sh
bash import_sales.sh
bash import_products.sh
```

### 2. Run Hive Transformation
```bash
hive -f hive/create_tables.hql
hive -f hive/transform_etl.hql
```

### 3. Run PySpark Transformation
```bash
spark-submit spark/transform_etl.py
```

"""

# ===================== sqoop/import_customers.sh =====================

"""
#!/bin/bash
sqoop import \
--connect jdbc:teradata://<TERADATA_HOST>/DATABASE=mydb \
--username <USER> --password <PASS> \
--table customers \
--target-dir /user/hadoop/customers \
--as-parquetfile \
--num-mappers 1
"""

# ===================== sqoop/import_sales.sh =====================

"""
#!/bin/bash
sqoop import \
--connect jdbc:teradata://<TERADATA_HOST>/DATABASE=mydb \
--username <USER> --password <PASS> \
--table sales \
--target-dir /user/hadoop/sales \
--as-parquetfile \
--num-mappers 1
"""

# ===================== sqoop/import_products.sh =====================

"""
#!/bin/bash
sqoop import \
--connect jdbc:teradata://<TERADATA_HOST>/DATABASE=mydb \
--username <USER> --password <PASS> \
--table products \
--target-dir /user/hadoop/products \
--as-parquetfile \
--num-mappers 1
"""

# ===================== hive/create_tables.hql =====================

"""
CREATE EXTERNAL TABLE customers (
  id INT,
  name STRING,
  email STRING
)
STORED AS PARQUET
LOCATION '/user/hadoop/customers';

CREATE EXTERNAL TABLE sales (
  id INT,
  customer_id INT,
  product_id INT,
  amount DOUBLE
)
STORED AS PARQUET
LOCATION '/user/hadoop/sales';

CREATE EXTERNAL TABLE products (
  id INT,
  product_name STRING
)
STORED AS PARQUET
LOCATION '/user/hadoop/products';
"""

# ===================== hive/transform_etl.hql =====================

"""
CREATE TABLE cleaned_sales AS
SELECT s.id as sale_id, c.name as customer_name, p.product_name, s.amount
FROM sales s
JOIN customers c ON s.customer_id = c.id
JOIN products p ON s.product_id = p.id
WHERE s.amount > 0;
"""

# ===================== spark/transform_etl.py =====================

"""
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("ETL_Teradata_to_Hadoop").enableHiveSupport().getOrCreate()

customers = spark.read.parquet("/user/hadoop/customers")
sales = spark.read.parquet("/user/hadoop/sales")
products = spark.read.parquet("/user/hadoop/products")

cleaned_sales = sales.join(customers, sales.customer_id == customers.id) \
                   .join(products, sales.product_id == products.id) \
                   .filter(sales.amount > 0) \
                   .select(sales.id.alias("sale_id"),
                           customers.name.alias("customer_name"),
                           products.product_name,
                           sales.amount)

cleaned_sales.write.mode("overwrite").saveAsTable("cleaned_sales")
"""
