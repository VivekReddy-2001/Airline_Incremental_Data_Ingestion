# Incremental Airline Data Processing Pipeline on AWS

## Introduction

This project demonstrates how to build a robust, event-driven, and scalable end-to-end data pipeline on Amazon Web Services (AWS). The pipeline is designed to incrementally ingest, process, and analyze airline flight data. It handles daily data feeds, processes them efficiently using serverless services, and loads the transformed data into a data warehouse for analytics and reporting. The entire workflow is automated and orchestrated to ensure reliability and fault tolerance.

The architecture leverages a suite of AWS services to create a decoupled and resilient system. Data is ingested into S3, cataloged by AWS Glue, transformed using a Glue ETL job that tracks incremental changes, and orchestrated by AWS Step Functions. This setup is a common pattern for handling periodic data ingestion in a cloud environment.

## Tech Stack

The following technologies and AWS services are used to build this data pipeline:

* **Data Storage:** AWS S3 (Simple Storage Service)
* **Data Catalog:** AWS Glue Data Catalog, AWS Glue Crawler
* **Data Processing:** AWS Glue (ETL Jobs with Job Bookmarking)
* **Data Warehouse:** AWS Redshift
* **Orchestration:** AWS Step Functions
* **Event-Driven Trigger:** AWS EventBridge
* **Notifications:** AWS SNS (Simple Notification Service)
* **Language:** Python

## Architecture Overview

The data pipeline is orchestrated as follows, triggered by the arrival of new data:

1.  **Data Ingestion**:
    * Dimension data, such as `airport_codes.csv`, is pre-loaded into an S3 bucket (`dim-data`).
    * Transactional data, representing daily flights (`YYYY-MM-DD.csv`), is uploaded by a client to a separate S3 bucket (`flight-raw`).

2.  **Event Trigger**:
    * The `PutObject` event in the `flight-raw` S3 bucket triggers an **AWS EventBridge** rule.
    * This rule, in turn, invokes the **AWS Step Function** state machine to start the processing workflow.

3.  **Data Cataloging**:
    * An **AWS Glue Crawler** is configured to scan the `dim-data` and `flight-raw` buckets. It infers the schema and registers the tables in the **AWS Glue Data Catalog**, making the data queryable.
    * A separate crawler is also set up for the target tables in **AWS Redshift**.

4.  **Workflow Orchestration (AWS Step Functions)**:
    * The Step Function begins its execution.
    * **Step 1: Start Crawler**: It first initiates the Glue Crawler to catalog any new daily flight data.
    * **Step 2: Wait Loop**: It waits until the crawler successfully completes its run.
    * **Step 3: Trigger Glue Job**: Once the catalog is updated, it triggers the **AWS Glue ETL job**.
    * **Step 4: Send Notification**: Based on the success or failure of the Glue job, it publishes a message to an **AWS SNS** topic.

5.  **Data Processing (AWS Glue Job)**:
    * The Glue ETL job, written in Python, reads the daily flight data from the `flight-raw` catalog table.
    * Crucially, it uses **Glue Job Bookmarking** to process only the new, unprocessed data since the last run, ensuring incremental loads.
    * The job joins the flight data with the airport codes dimension data.
    * The transformed, enriched data is then written to the final `flight_processed` table in **AWS Redshift**.

6.  **Notifications (AWS SNS)**:
    * Subscribers to the SNS topic (e.g., developers, operations team) receive an email or text message notification indicating the outcome of the data processing job.

This architecture provides a fully automated and serverless solution for handling incremental data loads in a scalable and maintainable way.
