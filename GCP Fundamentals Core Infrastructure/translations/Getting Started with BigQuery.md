# Google Cloud Fundamentals: Getting Started with BigQuery

## Objectives

- In this lab, We learn how to perform the following tasks:
  - Load data from Cloud Storage into BigQuery.
  - Perform a query on the data in BigQuery.

## Load data from Cloud Storage into BigQuery

1. Create a new dataset within your project for Dataset ID, type logdata and location = US

```
bq mk --project_id=qwiklabs-gcp-02-7ab18f5b3609 --location=us logdata
```

2. Create a new table in the logdata to store the data from the CSV file.

The following command loads data from gs://cloud-training/gcpfci/access_log.csv into a table named accesslog in mydataset. The schema auto-detection

--autodetect: When specified, enable schema auto-detection for CSV and JSON data.

```
bq load --autodetect=true --source_format=CSV logdata.accesslog gs://cloud-training/gcpfci/access_log.csv
```

3. Perform a queries on the data using the bq command

```
bq query "select int64_field_6 as hour, count(*) as hitcount from logdata.accesslog group by hour order by hour"
```

- Result should be like this:

```
Waiting on bqjob_r69f205e15ed29363_0000017462b152a8_1 ... (0s) Current status: DONE

| hour | hitcount |
| ---- | -------- |
| 0    | 26983    |
| 1    | 12287    |
| 2    | 8824     |
| 3    | 6607     |
| 4    | 10519    |
| 5    | 14581    |
| 6    | 26634    |
| 7    | 73708    |
| 8    | 218842   |
| 9    | 219769   |
| 10   | 115119   |
| 11   | 58151    |
| 12   | 55623    |
| 13   | 55625    |
| 14   | 56057    |
| 15   | 55675    |
| 16   | 55573    |
| 17   | 55740    |
| 18   | 55800    |
| 19   | 55935    |
| 20   | 55996    |
| 21   | 55797    |
| 22   | 55778    |
| 23   | 55755    |
```

```
bq query "select string_field_10 as request, count(*) as requestcount from logdata.accesslog group by request order by requestcount desc"
```

- Result should be like this:

```
Waiting on bqjob_rbd6d62e916c4c2c_0000017462afef3b_1 ... (0s) Current status: DONE

| request                                | requestcount |
| -------------------------------------- | ------------ |
| GET /store HTTP/1.0                    | 337293       |
| GET /index.html HTTP/1.0               | 336193       |
| GET /products HTTP/1.0                 | 280937       |
| GET /services HTTP/1.0                 | 169090       |
| GET /products/desserttoppings HTTP/1.0 | 56580        |
| GET /products/floorwaxes HTTP/1.0      | 56451        |
| GET /careers HTTP/1.0                  | 56412        |
| GET /services/turnipwinding HTTP/1.0   | 56401        |
| GET /services/spacetravel HTTP/1.0     | 56176        |
| GET /favicon.ico HTTP/1.0              | 55845        |

```
