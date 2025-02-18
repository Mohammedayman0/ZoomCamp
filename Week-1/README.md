# README

## Run Python in Docker
To start an interactive shell using Python 3.12.8 Docker image:
```bash
docker run -it --entrypoint bash python:3.12.8
```
Check the installed pip version:
```bash
pip --version
```

## Run PostgreSQL in Docker
To run a PostgreSQL Docker container with customized environment variables:
```bash
winpty docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:13
```

## SQL Queries

### 1. Trip Distance Distribution
This query calculates the distribution of trips based on their distance:
```sql
SELECT
    COUNT(CASE WHEN trip_distance <= 1 THEN 1 END) AS up_to_1_mile,
    COUNT(CASE WHEN trip_distance > 1 AND trip_distance <= 3 THEN 1 END) AS between_1_and_3_miles,
    COUNT(CASE WHEN trip_distance > 3 AND trip_distance <= 7 THEN 1 END) AS between_3_and_7_miles,
    COUNT(CASE WHEN trip_distance > 7 AND trip_distance <= 10 THEN 1 END) AS between_7_and_10_miles,
    COUNT(CASE WHEN trip_distance > 10 THEN 1 END) AS over_10_miles
FROM green_trips
WHERE lpep_pickup_datetime >= '2019-10-01' AND lpep_pickup_datetime < '2019-11-01';
```

### 2. Longest Trip Per Day
Find the longest trip for each day:
```sql
SELECT
    DATE(lpep_pickup_datetime) AS pickup_day,
    MAX(trip_distance) AS longest_trip
FROM green_trips
GROUP BY pickup_day
ORDER BY longest_trip DESC
LIMIT 1;
```

### 3. High Revenue Pickup Zones
Identify the top 3 pickup zones with total revenue over $13,000 on a specific day:
```sql
SELECT
    z."Zone" AS pickup_zone,
    SUM(t.total_amount) AS total_amount
FROM green_trips t
JOIN zones z ON t."PULocationID" = z."LocationID"
WHERE DATE(t.lpep_pickup_datetime) = '2019-10-18'
GROUP BY z."Zone"
HAVING SUM(t.total_amount) > 13000
ORDER BY total_amount DESC
LIMIT 3;
```

### 4. Largest Tip by Dropoff Zone
Retrieve the dropoff zone with the largest tip for trips starting from 'East Harlem North':
```sql
SELECT
    z2."Zone" AS dropoff_zone,
    MAX(t.tip_amount) AS largest_tip
FROM green_trips t
JOIN zones z1 ON t."PULocationID" = z1."LocationID"
JOIN zones z2 ON t."DOLocationID" = z2."LocationID"
WHERE z1."Zone" = 'East Harlem North'
  AND DATE(t.lpep_pickup_datetime) >= '2019-10-01'
  AND DATE(t.lpep_pickup_datetime) < '2019-11-01'
GROUP BY z2."Zone"
ORDER BY largest_tip DESC
LIMIT 1;


