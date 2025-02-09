# Module 3 Homework

## Inserting data manually:

1: Create a zoomcamp0208 bucket in Google Cloud Storage.

2: Upload parquet files to the bucket.

3: Create external table for the Yellow Taxi Trip Records for January 2024 - June 2024 

Run this query in Big Query:

```sql
CREATE OR REPLACE EXTERNAL TABLE kestra-sandbox-450114.nytaxi.external_yellow_tripdata
OPTIONS(
  FORMAT='PARQUET',
  URIS=['gs://zoomcamp0208/*']
)
```

4: Create native table 

```sql
CREATE OR REPLACE TABLE kestra-sandbox-450114.nytaxi.native_yellow_tripdata
AS(
  SELECT * FROM `kestra-sandbox-450114.nytaxi.external_yellow_tripdata`
);
```

## Question 1:

What is count of records for the 2024 Yellow Taxi Data?

```sql
SELECT count(1) FROM `kestra-sandbox-450114.nytaxi.external_yellow_tripdata`;
```

Answer: 20332093

## Question 2:

Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables.  
What is the **estimated amount** of data that will be read when this query is executed on the External Table and the Table?

Type this query:

```sql
 SELECT COUNT(DISTINCT PULocationID)
 FROM `kestra-sandbox-450114.nytaxi.external_yellow_tripdata`
```

You should see for the external table:

![](/Users/sitara/Desktop/截图/截屏2025-02-09%2015.58.42.png)

Type this query:

```sql
 SELECT COUNT(DISTINCT PULocationID)
 FROM `kestra-sandbox-450114.nytaxi.external_yellow_tripdata`
```

You should see for the native table:

 ![hw3](/Users/sitara/Desktop/截图/截屏2025-02-09%2016.10.44.png)

**Answer:**

0 MB for the External Table and 155.12 MB for the Materialized Table

## Question 3:

Write a query to retrieve the PULocationID from the table (not the external table) in BigQuery. Now write a query to retrieve the PULocationID and DOLocationID on the same table. Why are the estimated number of Bytes different?

Type  this query:

```sql
SELECT PULocationID
FROM `kestra-sandbox-450114.nytaxi.native_yellow_tripdata`;
```

You should see for the query:

![](/Users/sitara/Desktop/截图/截屏2025-02-09%2016.27.15.png)

Then type this query:

```sql
SELECT PULocationID,DOLocationID
FROM `kestra-sandbox-450114.nytaxi.native_yellow_tripdata`;
```

You should see for the query:

![](/Users/sitara/Desktop/截图/截屏2025-02-09%2016.28.57.png)

**Answer:**

BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.

## Question 4:

How many records have a fare_amount of 0?

Type this query:

```sql
SELECT COUNT(1)
FROM `kestra-sandbox-450114.nytaxi.external_yellow_tripdata`
WHERE fare_amount = 0;
```

**Answer:**

8333

## Question 5:

What is the best strategy to make an optimized table in Big Query if your query will always filter based on tpep_dropoff_datetime and order the results by VendorID (Create a new table with this strategy)

- Partition by tpep_dropoff_datetime and Cluster on VendorID
- Cluster on by tpep_dropoff_datetime and Cluster on VendorID
- Cluster on tpep_dropoff_datetime Partition by VendorID
- Partition by tpep_dropoff_datetime and Partition by VendorID

**Answer:**

When you see "filter," that's going to be where you can reduce the data you read. When you think of a "WHERE" clause, think of it as a way to read less data. What helps us read less data are partitions, where you divide the data into smaller chunks so that less has to be read. When you hear "order," that's more about clustering the data in the order in which it's stored. For this case, it makes sense to partition by the tpep_dropoff_datetime and cluster by the VendorID.

Creating the partitioned table:

```sql
CREATE OR REPLACE TABLE kestra-sandbox-450114.nytaxi.partitioned_yellow_tripdata
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID
AS(
  SELECT * FROM `kestra-sandbox-450114.nytaxi.native_yellow_tripdata`
)
```

## Question 6:

Write a query to retrieve the distinct VendorIDs between tpep_dropoff_datetime 2024-03-01 and 2024-03-15 (inclusive)

Use the materialized table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 5 and note the estimated bytes processed. What are these values?

Type the query:

```sql
SELECT DISTINCT VendorID
FROM `kestra-sandbox-450114.nytaxi.native_yellow_tripdata`
WHERE tpep_dropoff_datetime >= '2024-03-01'
AND tpep_dropoff_datetime <= '2024-03-15'
```

You should see for the native table:

![hw4](/Users/sitara/Desktop/Q6-1.png)

 Type the query:

```sql
SELECT DISTINCT VendorID
FROM `kestra-sandbox-450114.nytaxi.partitioned_yellow_tripdata`
WHERE tpep_dropoff_datetime >= '2024-03-01'
AND tpep_dropoff_datetime <= '2024-03-15'
```

You should see for the partitioned table:

![](/Users/sitara/Desktop/截图/Q6-2.png)

**Answer:**

310.24 MB for non-partitioned table and 26.84 MB for the partitioned table

## Question 7:

Where is the data stored in the External Table you created?

- Big Query
- Container Registry
- GCP Bucket
- Big Table

**Answer:**

GCP Bucket

If you created an external table in BigQuery, for example, the data would be stored in a specified location, such as a **Google Cloud Storage bucket**, and BigQuery would query this data when needed.

## Question 8:

It is best practice in Big Query to always cluster your data:

- True
- False

**Answer:**

False.

Clustering is less effective for small datasets (e.g., under 1 GB), where partitioning or clustering adds metadata overhead and costs. So the answer to question seven is false.

## (Bonus: Not worth points)

No Points: Write a `SELECT count(*)` query FROM the materialized table you created. How many bytes does it estimate will be read? Why?

```sql
SELECT count(*)
FROM `kestra-sandbox-450114.nytaxi.native_yellow_tripdata`;
```

You should see:

![hw6](/Users/sitara/Desktop/截图/bonus.png)

it's not going to read any bytes when it counts the number of rows. If you go down here to the native table and just open that up, it makes sense. If you scroll down here, the number of rows is known because it's stored in BigQuery how many rows it has. So if you just select count star, it's not really going to read the data, it's going to look at its metadata.

 
