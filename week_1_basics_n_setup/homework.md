# Week - 1 Question answer

## Question 1: gloud version
`Install Google Cloud SDK. What's the version you have? (To get the version, run gcloud --version)`

```text
Google Cloud SDK 369.0.0
```

## Question 2: terraform apply *
```text
var.project
  Your GCP Project ID

  Enter a value: secure-petal-339116


Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  + create

Terraform will perform the following actions:

  /# google_bigquery_dataset.dataset will be created
  + resource "google_bigquery_dataset" "dataset" {
      + creation_time              = (known after apply)
      + dataset_id                 = "trips_data_all"
      + delete_contents_on_destroy = false
      + etag                       = (known after apply)
      + id                         = (known after apply)
      + last_modified_time         = (known after apply)
      + location                   = "europe-west6"
      + project                    = "secure-petal-339116"
      + self_link                  = (known after apply)

      + access {
          + domain         = (known after apply)
          + group_by_email = (known after apply)
          + role           = (known after apply)
          + special_group  = (known after apply)
          + user_by_email  = (known after apply)

          + view {
              + dataset_id = (known after apply)
              + project_id = (known after apply)
              + table_id   = (known after apply)
            }
        }
    }

  # google_storage_bucket.data-lake-bucket will be created
  + resource "google_storage_bucket" "data-lake-bucket" {
      + force_destroy               = true
      + id                          = (known after apply)
      + location                    = "EUROPE-WEST6"
      + name                        = "dtc_data_lake_secure-petal-339116"
      + project                     = (known after apply)
      + self_link                   = (known after apply)
      + storage_class               = "STANDARD"
      + uniform_bucket_level_access = true
      + url                         = (known after apply)

      + lifecycle_rule {
          + action {
              + type = "Delete"
            }

          + condition {
              + age                   = 30
              + matches_storage_class = []
              + with_state            = (known after apply)
            }
        }

      + versioning {
          + enabled = true
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_bigquery_dataset.dataset: Creating...
google_storage_bucket.data-lake-bucket: Creating...
google_storage_bucket.data-lake-bucket: Creation complete after 1s [id=dtc_data_lake_secure-petal-339116]
google_bigquery_dataset.dataset: Creation complete after 2s [id=projects/secure-petal-339116/datasets/trips_data_all]
```

## Question 3: Count records

`How many taxi trips were there on January 15?`

```sql
select count(1) 
  from yellow_taxi_data yt 
 where tpep_pickup_datetime::date = '2021-01-15'
 ```
 ```text
 Output: 53024
 ```

## Question 4: Largest tip for each day

`On which day it was the largest tip in January? (note: it's not a typo, it's "tip", not "trip")`

```sql
SELECT 
	CAST(tpep_pickup_datetime as date) as "Date",
	MAX(tip_amount) as max_tip
FROM
	yellow_taxi_trips
GROUP BY
	"Date"
ORDER BY
	max_tip DESC
LIMIT 1;
```
```text
output: 2021-01-20
```

## Question 5: Most popular destination

`What was the most popular destination for passengers picked up in central park on January 14? Enter the zone name (not id). If the zone name is unknown (missing), write "Unknown"`

```sql
SELECT 
	dp."Zone", COUNT(1) num_dropoff
FROM (
	SELECT
		t."DOLocationID", z."Zone"
	FROM
		yellow_taxi_data t JOIN zones z
		ON t."DOLocationID" = z."LocationID"
		JOIN zones zpu
		ON t."PULocationID" = zpu."LocationID"
		WHERE zpu."Zone" = 'Central Park' AND
		CAST(tpep_pickup_datetime AS date)='2021-01-14'
	) AS dp
GROUP BY dp."Zone"
ORDER BY num_dropoff DESC
LIMIT 1;
```	

```text
output: Upper East Side South
```
## Question 6: Most expensive route

`What's the pickup-dropoff pair with the largest average price for a ride (calculated based on total_amount)? Enter two zone names separated by a slashFor example:"Jamaica Bay / Clinton East" If any of the zone names are unknown (missing), write "Unknown". For example, "Unknown / Clinton East".`

```sql
select avg(total_amount) avgg,
       t1."Zone" || '/' || t2."Zone"
  from yellow_tripdata yt
  join taxi t1
    on pulocationid = t1.locationid
  join taxi t2
    on dolocationid = t2.locationid 
 group by dolocationid, t1."Zone", t2."Zone"
 order by 1 desc 
 limit 1
```
```text
output: Alphabet City/Unknown
```