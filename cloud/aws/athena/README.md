# AWS Athena

## What is it ? 

[Amazon Athena](https://aws.amazon.com/athena/) is an interactive query service that makes it easy to analyze data in Amazon S3 using standard SQL. Athena is serverless, so there is no infrastructure to manage, and you pay only for the queries that you run.

Athena is easy to use. Simply point to your data in Amazon S3, define the schema, and start querying using standard SQL. Most results are delivered within seconds. With Athena, there’s no need for complex ETL jobs to prepare your data for analysis. This makes it easy for anyone with SQL skills to quickly analyze large-scale datasets.

---

## Get Millions of domains in seconds

We are going to query the [Common Crawl](https://commoncrawl.org/) S3 bucket to get the list of all the domains it has crawled

---
 Open the [Athena query editor](https://console.aws.amazon.com/athena/home?region=us-east-1#query).

---
 Select us-east-1 as your location

---
Run the query 
```SQL
CREATE DATABASE ccindex
```
This will create a database

---
Create a new table with the following query 
```SQL
CREATE EXTERNAL TABLE IF NOT EXISTS ccindex (
  url_surtkey                   STRING,
  url                           STRING,
  url_host_name                 STRING,
  url_host_tld                  STRING,
  url_host_2nd_last_part        STRING,
  url_host_3rd_last_part        STRING,
  url_host_4th_last_part        STRING,
  url_host_5th_last_part        STRING,
  url_host_registry_suffix      STRING,
  url_host_registered_domain    STRING,
  url_host_private_suffix       STRING,
  url_host_private_domain       STRING,
  url_protocol                  STRING,
  url_port                      INT,
  url_path                      STRING,
  url_query                     STRING,
  fetch_time                    TIMESTAMP,
  fetch_status                  SMALLINT,
  content_digest                STRING,
  content_mime_type             STRING,
  content_mime_detected         STRING,
  content_charset               STRING,
  content_languages             STRING,
  warc_filename                 STRING,
  warc_record_offset            INT,
  warc_record_length            INT,
  warc_segment                  STRING)
PARTITIONED BY (
  crawl                         STRING,
  subset                        STRING)
STORED AS parquet
LOCATION 's3://commoncrawl/cc-index/table/cc-main/warc/';
```

This will map the table ccindex that your created woth the location of the S3 bucket where the data is store

---
Run 
```SQL 
MSCK REPAIR TABLE ccindex
```

Note that you need to rerun this command every month as new data is added by the commo crawl foundation.

---
Run
```SQL
SELECT DISTINCT(url_host_name)
FROM "ccindex"."ccindex"
WHERE crawl = 'CC-MAIN-2018-05'
  AND subset = 'warc'
  AND url_host_tld = 'no'
```

This will return all the hostname from Norway that were crawled in May 2018

![](images/AWS-ATHENA-NORWAY-EXAMPLE.png
)

```SQL
SELECT COUNT(*) AS count,
       url_host_registered_domain
FROM "ccindex"."ccindex"
WHERE crawl = 'CC-MAIN-2018-05'
  AND subset = 'warc'
  AND url_host_tld = 'no'
GROUP BY  url_host_registered_domain
HAVING (COUNT(*) >= 100)
ORDER BY  count DESC
```
This Query will return all the url_host_registered_domain count from Norway
