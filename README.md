# Slowly Changing Dimensions (SCD) ETL Pipeline using Python, S3, and MySQL

## Project Overview

This project implements an ETL pipeline in Python that processes order data stored as JSON files in Amazon S3. The pipeline extracts nested data, flattens it into fact and dimension tables, and loads the transformed data into a MySQL data warehouse.

The focus of the pipeline is to implement **Slowly Changing Dimensions (SCD)**:
- **SCD Type 1** for the `Customer` table (overwrite updates)
- **SCD Type 2** for the `Product` table (track history via versioning)

## Data Sources

- **Input Format**: JSON files containing customer orders
- **Source Location**: AWS S3 bucket structured as `etl_pipeline/YYYYMMDD/*.json`

## Sample JSON Structure:
Each order contains:
- `order_id`, `order_date`, `total_amount`
- Nested `customer` object: `customer_id`, `name`, `email`, `address`
- List of `products`: Each with `product_id`, `name`, `category`, `price`, `quantity`

## Pipeline Architecture

1. **Extract**:
   - Reads JSON order files from S3 using `boto3`
   - Parses and flattens into customer, product, and order records

2. **Transform**:
   - Normalizes JSON to separate `customers`, `products`, and `orders` DataFrames
   - Deduplicates based on latest `order_id` per entity
   - Prepares staging tables for SCD logic

3. **Load**:
   - **Customer Table (SCD Type 1)**:
     - Overwrites values with latest data (name, email, address)
   - **Product Table (SCD Type 2)**:
     - Compares new product attributes with existing records
     - If any change detected:
       - Closes current version with an `end_date`
       - Inserts new version with updated values
   - Inserts order data into the `Orders` fact table after joining with surrogate keys

## SCD Logic Summary

## SCD Type 1 — Customer Table
- Updates values in-place (no history maintained)
- Uses `UPDATE` + `INSERT` on `customer_id`

## SCD Type 2 — Product Table
- Closes current records with `end_date = CURDATE() - 1`
- Inserts new versioned row with `start_date = CURDATE()` and `end_date = '9999-12-31'`
- Compares `price`, `name`, and `category` for changes

## 🛠️ Technologies Used

- Python
    Pandas for transformation
    sqlalchemy and pymysql for MySQL interaction
    boto3 for S3 access
- Amazon S3
    Stores daily order JSON files
- MySQL
    Target data warehouse for dimensional modeling

## How to Run

1. Ensure AWS credentials are configured securely and upload the files (orders_ETL.json, orders_ETL_incremental.json) into a folder called **etl_pipeline** in a S3 bucket with a structure etl_pipeline --> yyyymmdd
2. Ensure MySQL schema and tables (customers1, products1, orders1) are created in a database called 'Python'
3. Run the script or notebook containing the main function: **scd_etl_pipeline.ipynb**

Verify:

-  Staging and dimension tables are updated
-  Surrogate keys are properly assigned in fact table
-  History is maintained in products1 with versioning

## Skills Demonstrated

-  Dimensional data modeling with SCD strategies
-  Real-world ETL design (Extract-Transform-Load)
-  JSON flattening and normalization
-  MySQL SQL scripting and query execution from Python
-  Data engineering best practices: modularity, staging, surrogate keys

This project simulates real-world ETL processes and demonstrates how SCD Type 1 and Type 2 can be implemented in a cloud-based data pipeline using open-source tools.
