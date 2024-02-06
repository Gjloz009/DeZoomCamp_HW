## Week 2 Homework

ATTENTION: At the end of the submission form, you will be required to include a link to your GitHub repository or other public code-hosting site. This repository should contain your code for solving the homework. If your solution includes code that is not in file format, please include these directly in the README file of your repository.

> In case you don't get one option exactly, select the closest one 

For the homework, we'll be working with the _green_ taxi dataset located here:

`https://github.com/DataTalksClub/nyc-tlc-data/releases/tag/green/download`

You may need to reference the link below to download via Python in Mage:

`https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/`

### Assignment

The goal will be to construct an ETL pipeline that loads the data, performs some transformations, and writes the data to a database (and Google Cloud!).

- Create a new pipeline, call it `green_taxi_etl`
- Add a data loader block and use Pandas to read data for the final quarter of 2020 (months `10`, `11`, `12`).
  - You can use the same datatypes and date parsing methods shown in the course.
  - `BONUS`: load the final three months using a for loop and `pd.concat`
  
  Ans: 
import io
import pandas as pd
import requests
import os
if 'data_loader' not in globals():
    from mage_ai.data_preparation.decorators import data_loader
if 'test' not in globals():
    from mage_ai.data_preparation.decorators import test


@data_loader
def load_data_from_api(*args, **kwargs):
    """
    Template for loading data from API
    """
    taxi_dtypes = {
                    'VendorID': pd.Int64Dtype(),
                    'passenger_count': pd.Int64Dtype(),
                    'trip_distance': float,
                    'RatecodeID':pd.Int64Dtype(),
                    'store_and_fwd_flag':str,
                    'PULocationID':pd.Int64Dtype(),
                    'DOLocationID':pd.Int64Dtype(),
                    'payment_type': pd.Int64Dtype(),
                    'fare_amount': float,
                    'extra':float,
                    'mta_tax':float,
                    'tip_amount':float,
                    'tolls_amount':float,
                    'improvement_surcharge':float,
                    'total_amount':float,
                    'congestion_surcharge':float
                }
    parse_dates = ['lpep_pickup_datetime','lpep_dropoff_datetime']
    
    lista = []
    
    for day in [10,11,12]:
        lista.append(pd.read_csv(f'https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2020-{day}.csv.gz', sep= ",", compression="gzip", dtype=taxi_dtypes, parse_dates=parse_dates))

    return pd.concat(lista)

@test
def test_output(output, *args) -> None:
    """
    Template code for testing the output of the block.
    """
    assert output is not None, 'The output is undefined'

- Add a transformer block and perform the following:
  - Remove rows where the passenger count is equal to 0 _or_ the trip distance is equal to zero.
  - Create a new column `lpep_pickup_date` by converting `lpep_pickup_datetime` to a date.
  - Rename columns in Camel Case to Snake Case, e.g. `VendorID` to `vendor_id`.
  - Add three assertions:
    - `vendor_id` is one of the existing values in the column (currently)
    - `passenger_count` is greater than 0
    - `trip_distance` is greater than 0
	
Ans:
if 'transformer' not in globals():
    from mage_ai.data_preparation.decorators import transformer
if 'test' not in globals():
    from mage_ai.data_preparation.decorators import test

@transformer
def transform(data, *args, **kwargs):
    """
    Template code for a transformer block.

    Add more parameters to this function if this block has multiple parent blocks.
    There should be one parameter for each output variable from each parent block.

    Args:
        data: The output from the upstream parent block
        args: The output from any additional upstream blocks (if applicable)

    Returns:
        Anything (e.g. data frame, dictionary, array, int, str, etc.)
    """
    # Specify your transformation logic here
    data['lpep_pickup_date'] = data['lpep_pickup_datetime'].dt.date

    return data[(data['passenger_count'] > 0) & (data['trip_distance'] > 0.0)].rename(columns={'VendorID': 'vendor_id',
                        'RatecodeID': 'ratecode_id',
                        'PULocationID':'pu_Location_id',
                        'DOLocationID':'do_Location_id',})


@test
def test_output(data) -> None:
    """
    Template code for testing the output of the block.
    """
    #assert output is not None, 'The output is undefined'
    assert 'vendor_id' in data.columns, 'si existe'
    

@test
def test_output(data) -> None:
    """
    Template code for testing the output of the block.
    """
    #assert output is not None, 'The output is undefined'
    assert data['passenger_count'].isin([0]).sum() == 0,'no es'

@test
def test_output(data) -> None:
    """
    Template code for testing the output of the block.
    """
    #assert output is not None, 'The output is undefined'
    assert data['trip_distance'].isin([0]).sum() == 0,'no es'
    


- Using a Postgres data exporter (SQL or Python), write the dataset to a table called `green_taxi` in a schema `mage`. Replace the table if it already exists.
Ans: 
PostgressSQL dev mage green_taxi
-- Docs: https://docs.mage.ai/guides/sql-blocks
SELECT * FROM {{df_1}}

- Write your data as Parquet files to a bucket in GCP, partioned by `lpep_pickup_date`. Use the `pyarrow` library!
Ans:

import pyarrow as pa
import pyarrow.parquet as pq
import os



if 'data_exporter' not in globals():
    from mage_ai.data_preparation.decorators import data_exporter

os.environ['GOOGLE_APPLICATION_CREDENTIALS']="/home/src/my-k.json"

bucket_name = 'mage-zoomcamp-jafet-flores-1'
project_id = 'astute-catcher-411918'

table_name = 'nyc_taxi_data_green'

root_path = f'{bucket_name}/{table_name}'

@data_exporter
def export_data(data, *args, **kwargs):
    """
    Exports data to some source.

    Args:
        data: The output from the upstream parent block
        args: The output from any additional upstream blocks (if applicable)

    Output (optional):
        Optionally return any object and it'll be logged and
        displayed when inspecting the block run.
    """
    # Specify your data exporting logic here
    data['lpep_pickup_date'] = data['lpep_pickup_datetime'].dt.date

    table = pa.Table.from_pandas(data)

    gcs = pa.fs.GcsFileSystem()

    pq.write_to_dataset(
        table,
        root_path=root_path,
        partition_cols=['lpep_pickup_date'],
        filesystem=gcs
    )
- Schedule your pipeline to run daily at 5AM UTC.

### Questions

## Question 1. Data Loading

Once the dataset is loaded, what's the shape of the data?

* 266,855 rows x 20 columns

## Question 2. Data Transformation

Upon filtering the dataset where the passenger count is greater than 0 _and_ the trip distance is greater than zero, how many rows are left?

* 139,370 rows


## Question 3. Data Transformation

Which of the following creates a new column `lpep_pickup_date` by converting `lpep_pickup_datetime` to a date?

* `data['lpep_pickup_date'] = data['lpep_pickup_datetime'].dt.date`


## Question 4. Data Transformation

What are the existing values of `VendorID` in the dataset?

* 1 or 2


## Question 5. Data Transformation

How many columns need to be renamed to snake case?

* 4

## Question 6. Data Exporting

Once exported, how many partitions (folders) are present in Google Cloud?

* 96


## Submitting the solutions

* Form for submitting: https://courses.datatalks.club/de-zoomcamp-2024/homework/hw2
* Check the link above to see the due date
  
## Solution

Will be added after the due date