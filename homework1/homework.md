## Week 1 Homework

In this homework we'll prepare the environment and practice Docker and SQL
This repository should contain the code for solving the homework.
When your solution has SQL or shell commands and not code (e.g. python files) file format, include them directly in the README file of your repository.

## Question 1. Understanding docker first run

Run docker with the python:3.12.8 image in an interactive mode, use the entrypoint bash.

What's the version of pip in the image?

> Answer:

```
24.3.1
```

> How to get it?
> 
> After installing the docker, excute the code below in the terminal

```shell
docker run -it --entrypoint=bash  python:3.12.8
#After the docker runs
pip list
```

## Question 2.  Understanding Docker networking and docker-compose

Given the following `docker-compose.yaml`, what is the `hostname` and `port` that **pgadmin** should use to connect to the postgres database?

```docker
services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - '5433:5432'
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    ports:
      - "8080:80"
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin  

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data
```

- postgres:5433
- localhost:5432
- db:5433
- postgres:5432
- db:5432

If there are more than one answers, select only one of them

> Answer:

   db:5432

> Why？

1.Hostname: In Docker Compose, services communicate with each other using service names. The service name for the Postgres database is db, so pgadmin should use db as the hostname.

2.Port: pgadmin and postgres are on the same Docker network, so pgadmin can directly access the internal port 5432 of the postgres container without needing to go through the host-mapped port 5433.

## Prepare Postgres

Run Postgres and load data as shown in the videos

> Command:

```bash
# from the working directory 
docker run -it \
-e POSTGRES_USER="root" \
-e POSTGRES_PASSWORD="root" \
-e POSTGRES_DB="taxi" \
-v $pwd/taxi_postgress_data:/var/lib/postgressql/data \
-p 5432:5432 \
postgres:13
```

We'll use the yellow taxi trips from January 2021:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz
```

You will also need the dataset with zones:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv
```

Download  data and then I use upload-data.ipynb to put it to Postgres, See the file for more code.

## Question 3.  Trip Segmentation Count

During the period of October 1st 2019 (inclusive) and November 1st 2019 (exclusive), how many trips, **respectively**, happened:

1. Up to 1 mile
2. In between 1 (exclusive) and 3 miles (inclusive),
3. In between 3 (exclusive) and 7 miles (inclusive),
4. In between 7 (exclusive) and 10 miles (inclusive),
5. Over 10 miles

> Proposed command for solution:

```sql
SELECT
    SUM(CASE WHEN trip_distance <= 1 THEN 1 ELSE 0 END) AS "Up to 1 mile",
    SUM(CASE WHEN trip_distance > 1 AND trip_distance <= 3 THEN 1 ELSE 0 END) AS "1 to 3 miles",
    SUM(CASE WHEN trip_distance > 3 AND trip_distance <= 7 THEN 1 ELSE 0 END) AS "3 to 7 miles",
    SUM(CASE WHEN trip_distance > 7 AND trip_distance <= 10 THEN 1 ELSE 0 END) AS "7 to 10 miles",
    SUM(CASE WHEN trip_distance > 10 THEN 1 ELSE 0 END) AS "Over 10 miles"
FROM
    green_taxi_data
WHERE
    lpep_pickup_datetime >= '2019-10-01 00:00:00' AND lpep_dropoff_datetime < '2019-11-01 00:00:00';
```

> Answer :

104,802; 198,924; 109,603; 27,678; 35,189

## Question 4. Longest trip for each day

Which was the pick up day with the longest trip distance? Use the pick up time for your calculations.

Tip: For every day, we only care about one single trip with the longest distance.

- 2019-10-11
- 2019-10-24
- 2019-10-26
- 2019-10-31

> Proposed command for solution:

```sql
SELECT
    lpep_pickup_datetime AS pickup_date,
    MAX(trip_distance) AS max_trip_distance
FROM
    green_taxi_data
GROUP BY
    lpep_pickup_datetime
ORDER BY
    max_trip_distance DESC
LIMIT 1; 
```

> Answer (correct):

```
2019-10-31
The tip was 515.89
```

## Question 5. Three biggest pickup zones

Which were the top pickup locations with over 13,000 in `total_amount` (across all trips) for 2019-10-18?

Consider only `lpep_pickup_datetime` when filtering by date.

- East Harlem North, East Harlem South, Morningside Heights
- East Harlem North, Morningside Heights
- Morningside Heights, Astoria Park, East Harlem South
- Bedford, East Harlem North, Astoria Park

> Proposed command for solution:

```sql
WITH A AS
(SELECT
    "PULocationID",                              -- 取货地点 ID
    COUNT(*) AS pickup_count,                    -- 取货次数
    SUM(total_amount) AS total_amount_sum        -- 总金额
FROM
    green_taxi_data
WHERE
    lpep_pickup_datetime >= '2019-10-18 00:00:00' AND lpep_pickup_datetime < '2019-10-19 00:00:00'   -- 筛选 2019 年 10 月 18 日
GROUP BY
    "PULocationID"                               -- 按取货地点分组
HAVING
    SUM(total_amount) > 13000                             -- 取货次数超过 13,000 次
ORDER BY
    pickup_count DESC)
SELECT a.*,b."Zone"
FROM A a
join zones b
on a."PULocationID"= b."LocationID";   
```

> Answer :

```
    PULocationID    pickup_count    total_amount_sum    Zone
0    74    1236    18686.68    East Harlem North
1    75    1101    16797.26    East Harlem South
2    166    764    13029.79    Morningside Heights
```

## Question 6. Largest tip

For the passengers picked up in October 2019 in the zone name "East Harlem North" which was the drop off zone that had the largest tip?

Note: it's `tip` , not `trip`

We need the name of the zone, not the ID.

- Yorkville West
- JFK Airport
- East Harlem North
- East Harlem South

> Proposed command for solution:

```sql
WITH pickup_zone AS (
    -- 获取“East Harlem North”的区域 ID
    SELECT
        "LocationID" AS pulocation_id
    FROM
        zones
    WHERE
        "Zone" = 'East Harlem North'  -- 筛选上车区域为“East Harlem North”
)
SELECT
    t."DOLocationID",                     -- 下车区域 ID
    z."Zone" AS dropoff_zone,             -- 下车区域名称
    SUM(t.tip_amount) AS total_tip_amount -- 小费总和
FROM
    green_taxi_data t
JOIN
    zones z ON t."DOLocationID" = z."LocationID"  -- 关联下车区域名称
JOIN
    pickup_zone p ON t."PULocationID" = p.pulocation_id  -- 筛选上车区域
WHERE
    DATE_TRUNC('month', t.lpep_pickup_datetime) = '2019-10-01'  -- 筛选 2019 年 10 月
GROUP BY
    t."DOLocationID", z."Zone"            -- 按下车区域分组
ORDER BY
    total_tip_amount DESC                 -- 按小费总和降序排序
LIMIT 5;  
```

> Answer :

```
East Harlem South
trip distance is 4076.44
```

## Question 7. Terraform Workflow

Which of the following sequences, **respectively**, describes the workflow for:

1. Downloading the provider plugins and setting up backend,
2. Generating proposed changes and auto-executing the plan
3. Remove all resources managed by terraform

> Answer :

terraform init、terraform apply -auto-approve、terraform destroy
