## Module 1 Homework

## Docker & SQL

In this homework we'll prepare the environment 
and practice with Docker and SQL


## Question 1. Knowing docker tags

Run the command to get information on Docker 

```docker --help```

Now run the command to get help on the "docker build" command:

```docker build --help```

Do the same for "docker run".

Which tag has the following text? - *Automatically remove the container when it exits* 

Ans: `--rm`  
> docker run --help 


## Question 2. Understanding docker first run 

Run docker with the python:3.9 image in an interactive mode and the entrypoint of bash.
Now check the python modules that are installed ( use ```pip list``` ). 

What is version of the package *wheel* ?

Ans: - 0.42.0,
>docker run -it --entrypoint:bash python:3.9
>root@###:pip list

# Prepare Postgres

Run Postgres and load data as shown in the videos
We'll use the green taxi trips from September 2019:

```wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-09.csv.gz```

You will also need the dataset with zones:

```wget https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv```

Download this data and put it into Postgres (with jupyter notebooks or with a pipeline)


## Question 3. Count records 

How many taxi trips were totally made on September 18th 2019?

Tip: started and finished on 2019-09-18. 

Remember that `lpep_pickup_datetime` and `lpep_dropoff_datetime` columns are in the format timestamp (date and hour+min+sec) and not in date.

Ans: - 15612

WITH table_prepare AS (
SELECT *, lpep_pickup_datetime::DATE AS lpep_pickup_date,
	lpep_dropoff_datetime::DATE AS lpep_dropoff_date
FROM green_taxi_data
)
SELECT  count(*)
FROM table_prepare
WHERE lpep_pickup_date=lpep_dropoff_date and lpep_pickup_date='2019-09-18'
ORDER BY 1 DESC

## Question 4. Largest trip for each day

Which was the pick up day with the largest trip distance
Use the pick up time for your calculations.

Ans:- 2019-09-26

WITH table_prepare AS (
SELECT *, lpep_pickup_datetime::DATE AS lpep_pickup_date,
	lpep_dropoff_datetime::DATE AS lpep_dropoff_date
FROM green_taxi_data
)
SELECT  lpep_pickup_date,sum(Trip_distance)
FROM table_prepare
GROUP BY 1
ORDER BY 2 DESC

## Question 5. Three biggest pick up Boroughs

Consider lpep_pickup_datetime in '2019-09-18' and ignoring Borough has Unknown

Which were the 3 pick up Boroughs that had a sum of total_amount superior to 50000?
 
Ans: - "Brooklyn" "Manhattan" "Queens"
WITH prep_tab as (
SELECT g.*, 
	g.lpep_pickup_datetime::DATE AS lpep_pickup_date,
	g.lpep_dropoff_datetime::DATE AS lpep_dropoff_date,
	z."Borough" as PUBorough
FROM green_taxi_data AS g
LEFT JOIN zones AS z
ON g."PULocationID"=z."LocationID"
)

SELECT PUBorough,sum(Total_amount) AS suma
FROM prep_tab
WHERE lpep_pickup_date='2019-09-18'
GROUP BY 1
HAVING sum(Total_amount) > 50000;


## Question 6. Largest tip

For the passengers picked up in September 2019 in the zone name Astoria which was the drop off zone that had the largest tip?
We want the name of the zone, not the id.

Note: it's not a typo, it's `tip` , not `trip`

Ans: For me it was Astoria with 2135.3599999999965

But thinking only in the selection given it is Long Island City/ Queens Plaza

WITH prep_tab as (
SELECT g.*, 
	g.lpep_pickup_datetime::DATE AS lpep_pickup_date,
	g.lpep_dropoff_datetime::DATE AS lpep_dropoff_date,
	zpu."Borough" as PUBorough,
	zpu."Zone" as PUZone,
	zdo."Borough"  as DOBorough,
	zdo."Zone" as DOZone
FROM green_taxi_data AS g
LEFT JOIN zones AS zpu
ON g."PULocationID"=zpu."LocationID"
LEFT JOIN zones as zdo
ON g."DOLocationID"=zdo."LocationID")

SELECT DOZone,sum(Tip_amount)
FROM prep_tab
WHERE EXTRACT(YEAR FROM lpep_pickup_date) = 2019
AND EXTRACT(MONTH FROM lpep_pickup_date) = 09
AND PUZone = 'Astoria'
GROUP BY 1
ORDER BY 2 DESC;




## Terraform

In this section homework we'll prepare the environment by creating resources in GCP with Terraform.

In your VM on GCP/Laptop/GitHub Codespace install Terraform. 
Copy the files from the course repo
[here](https://github.com/DataTalksClub/data-engineering-zoomcamp/tree/main/01-docker-terraform/1_terraform_gcp/terraform) to your VM/Laptop/GitHub Codespace.

Modify the files as necessary to create a GCP Bucket and Big Query Dataset.


## Question 7. Creating Resources

After updating the main.tf and variable.tf files run:

```
terraform apply
```

Paste the output of this command into the homework submission form.

Ans: - 
(base) jafet@dezoomcampjaf:~/terrademo$ terraform apply

Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_bigquery_dataset.demo_dataset will be created
  + resource "google_bigquery_dataset" "demo_dataset" {
      + creation_time              = (known after apply)
      + dataset_id                 = "demo_dataset"
      + default_collation          = (known after apply)
      + delete_contents_on_destroy = false
      + effective_labels           = (known after apply)
      + etag                       = (known after apply)
      + id                         = (known after apply)
      + is_case_insensitive        = (known after apply)
      + last_modified_time         = (known after apply)
      + location                   = "US"
      + max_time_travel_hours      = (known after apply)
      + project                    = "astute-catcher-411918"
      + self_link                  = (known after apply)
      + storage_billing_model      = (known after apply)
      + terraform_labels           = (known after apply)
    }

  # google_storage_bucket.demo-bucket will be created
  + resource "google_storage_bucket" "demo-bucket" {
      + effective_labels            = (known after apply)
      + force_destroy               = true
      + id                          = (known after apply)
      + location                    = "US"
      + name                        = "demo-bucket-411918-terra-bucket"
      + project                     = (known after apply)
      + public_access_prevention    = (known after apply)
      + rpo                         = (known after apply)
      + self_link                   = (known after apply)
      + storage_class               = "STANDARD"
      + terraform_labels            = (known after apply)
      + uniform_bucket_level_access = (known after apply)
      + url                         = (known after apply)

      + lifecycle_rule {
          + action {
              + type = "AbortIncompleteMultipartUpload"
            }
          + condition {
              + age                   = 1
              + matches_prefix        = []
              + matches_storage_class = []
              + matches_suffix        = []
              + with_state            = (known after apply)
            }
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_bigquery_dataset.demo_dataset: Creating...
google_storage_bucket.demo-bucket: Creating...
google_bigquery_dataset.demo_dataset: Creation complete after 1s [id=projects/astute-catcher-411918/datasets/demo_dataset]
google_storage_bucket.demo-bucket: Creation complete after 1s [id=demo-bucket-411918-terra-bucket]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

## Submitting the solutions

* Form for submitting: https://courses.datatalks.club/de-zoomcamp-2024/homework/hw01
* You can submit your homework multiple times. In this case, only the last submission will be used. 

Deadline: 29 January, 23:00 CET