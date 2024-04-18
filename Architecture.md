# Cloud Data Platform Architecture

```mermaid

  flowchart LR;

  subgraph Extract & Load

    data_lake_s3(S3 Data Lake)

    src_api(REST API) <-->
      |Call API| python_script

    src_db(Database) -->
      |Copy Snapshot| data_lake_s3

    src_stream(Kafka) <-->
      |Read Topic| spark_streaming

    python_script(Python Script) -->
      |Load| data_lake_s3

    spark_streaming(Spark Streaming) -->
      |Load| data_lake_s3

  end

  subgraph Transform

    data_lake_s3 --> spark(Spark)

    spark -->
      |Transformation| redshift(Redshift)

  end

  redshift -->
    |Read-only| bi_tool(BI Tool)

```

## Tools

Assuming AWS as a cloud provider

- Python script for REST APIs
  - Pair with Lambda, ECS, AWS Batch
- Apache Spark for ingesting stream data and transforming raw data
- S3 for storing raw data in data lake
- Apache Iceberg to query across S3 data lake
- Redshift for storing dimensions, facts, aggregated analytics
- SQL is used for Trino, Iceberg, and Redshift
- Power BI or Tableau for analysts, but may be analysts tool of choice

## Extract

```mermaid

  flowchart LR;

  data_lake_s3(S3 Data Lake)

  src_api(REST API) <-->
    |Call API| python_script

  src_db(Database) -->
    |Copy Snapshot| data_lake_s3

  src_stream(Kafka) <-->
    |Read Topic| spark_streaming

  python_script(Python Script) -->
    |Load| data_lake_s3

  spark_streaming(Spark Streaming) -->
    |Load| data_lake_s3
```  

- AWS RDS can export daily snapshots to S3
  - AWS uses compressed parquet, preferred file format for data engineers
  - Snapshots will likely exist anyway for disaster recovery purposes
- Python script is used to read data from REST APIs
  - Python is the preferred language of the majority of data engineers
  - The script is versatile and platform agnostic (Lambda, ECS Task, AWS Batch)
  - Fine control over logic, testable, but must maintain
- Apache Spark can read events over a window, and store in S3
- S3 is used as a Data Lake to store raw data
- Apache Iceberg to maintain raw data metadata
- Trino to query across raw data with SQL


## Data Warehouse

## Analytics

- Use a Python script to interact with REST APIs
  - Use API provided SDK or requests library
  - Schedule the task when next day of data available (Airflow or EventBridge)
  - Script allows platform versatility (Lambda, ECS task, AWS Batch)
  - Unit test script
- Export daily snapshots from RDS to S3
  - Snapshots should be available as part of disaster recovery anyway
  - AWS exports in compressed parquet format
- Spark Streaming read from Kafka
  - Batch events with Spark and write to S3 in relatively raw format
