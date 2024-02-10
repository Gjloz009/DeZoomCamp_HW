## Week 3 Homework
ATTENTION: At the end of the submission form, you will be required to include a link to your GitHub repository or other public code-hosting site. This repository should contain your code for solving the homework. If your solution includes code that is not in file format (such as SQL queries or shell commands), please include these directly in the README file of your repository.

<b><u>Important Note:</b></u> <p> For this homework we will be using the 2022 Green Taxi Trip Record Parquet Files from the New York
City Taxi Data found here: </br> https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page </br>
If you are using orchestration such as Mage, Airflow or Prefect do not load the data into Big Query using the orchestrator.</br> 
Stop with loading the files into a bucket. </br></br>
<u>NOTE:</u> You will need to use the PARQUET option files when creating an External Table</br>

<b>SETUP:</b></br>
Create an external table using the Green Taxi Trip Records Data for 2022. </br>
Create a table in BQ using the Green Taxi Trip Records for 2022 (do not partition or cluster this table). </br>
</p>

## Question 1:
Question 1: What is count of records for the 2022 Green Taxi Data??
- 840,402


## Question 2:
Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables.</br> 
What is the estimated amount of data that will be read when this query is executed on the External Table and the Table?

- 0 MB for the External Table and 6.41MB for the Materialized Table



## Question 3:
How many records have a fare_amount of 0?

- 1,622

## Question 4:
What is the best strategy to make an optimized table in Big Query if your query will always order the results by PUlocationID and filter based on lpep_pickup_datetime? (Create a new table with this strategy)

- Partition by lpep_pickup_datetime  Cluster on PUlocationID


## Question 5:
Write a query to retrieve the distinct PULocationID between lpep_pickup_datetime
06/01/2022 and 06/30/2022 (inclusive)</br>

Use the materialized table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 4 and note the estimated bytes processed. What are these values? </br>

Choose the answer which most closely matches.</br> 

- 12.82 MB for non-partitioned table and 1.12 MB for the partitioned table



## Question 6: 
Where is the data stored in the External Table you created?

- GCP Bucket


## Question 7:
It is best practice in Big Query to always cluster your data:

- False


## (Bonus: Not worth points) Question 8:
No Points: Write a `SELECT count(*)` query FROM the materialized table you created. How many bytes does it estimate will be read? Why?
- Maybee it was proccessed before and doesnt have to precces again all 
 

CREATE OR REPLACE EXTERNAL TABLE `astute-catcher-411918.nytaxi.external_green_data_2022`
OPTIONS (
  format = 'CSV',
  uris = ['gs://mage-zoomcamp-jafet-flores-1/green_taxi_2022.csv'] 
);

CREATE TABLE astute-catcher-411918.nytaxi.internal_green_data_2022
AS SELECT * FROM astute-catcher-411918.nytaxi.external_green_data_2022;

--840402
select count(*) from astute-catcher-411918.nytaxi.external_green_data_2022;

--6.41
select distinct(PULocationID) from  astute-catcher-411918.nytaxi.internal_green_data_2022;

--1622
select count(fare_amount) from astute-catcher-411918.nytaxi.external_green_data_2022 where fare_amount=0;


CREATE OR REPLACE TABLE astute-catcher-411918.nytaxi.internal_green_data_2022_partitoned
PARTITION BY
DATE(lpep_pickup_datetime) AS
SELECT * FROM astute-catcher-411918.nytaxi.external_green_data_2022;

--1.12
SELECT DISTINCT(PULocationID)
FROM astute-catcher-411918.nytaxi.internal_green_data_2022_partitoned
WHERE DATE(lpep_pickup_datetime) BETWEEN '2022-06-01' AND '2022-06-30';

--12.82
SELECT DISTINCT(PULocationID)
FROM astute-catcher-411918.nytaxi.internal_green_data_2022
WHERE DATE(lpep_pickup_datetime) BETWEEN '2022-06-01' AND '2022-06-30';

--0
SELECT COUNT(*) FROM astute-catcher-411918.nytaxi.internal_green_data_2022_partitoned;


