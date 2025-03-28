#  AWS Batch ETL Pipeline  
A scalable **ETL pipeline** using AWS Glue, Kinesis, Kafka, and a Vector Database for **batch and real-time data processing**.

## 📌 Table of Contents  
- [📌 Project Overview](#project-overview)  
- [🎯 Architecture](#architecture)  
- [🛠 Setup & Execution](#setup--execution)  
  - [1️⃣ Batch Processing Pipeline](#1️⃣-batch-processing-pipeline)  
  - [2️⃣ Vector Database Setup](#2️⃣-vector-database-setup)  
  - [3️⃣ Connecting ML Model to Vector DB](#3️⃣-connecting-ml-model-to-vector-db)  
  - [4️⃣ Streaming Pipeline Implementation](#4️⃣-streaming-pipeline-implementation)  
- [✅ Final Validation](#✅-final-validation)  
- [📌 Resources & References](#resources--references)  
- [📌 Contributors](#contributors)  

---

## 📌 Project Overview  
This project builds an **end-to-end data pipeline** for processing user interactions and generating **real-time product recommendations**.

**Tech Stack:**  
✅ AWS Glue  
✅ AWS Lambda  
✅ Amazon RDS  
✅ Amazon Kinesis  
✅ Terraform  
✅ Kafka  
✅ PostgreSQL (pgvector)  

---

## 🎯 Architecture  
This pipeline consists of:  
- **Batch Processing** with AWS Glue  
- **Real-Time Streaming** via Kafka & Kinesis  
- **Vector Database for ML-based Recommendations**  
- **AWS Lambda-based API for Predictions**  

### 🖼 Architecture Diagram  
![High-Level Architecture](diagrams/high_level_architecture.png)  

---

## 🛠 Setup & Execution  

### 1️⃣ Batch Processing Pipeline  
This pipeline extracts data from **Amazon RDS (MySQL)**, transforms it with **AWS Glue**, and stores it in **Amazon S3**.

#### 🔹 **Step 1: Connect to RDS**  
```sh
aws rds describe-db-instances --db-instance-identifier de-c1w4-rds --query "DBInstances[].Endpoint.Address"
```
```sh
mysql --host=<MySQL_ENDPOINT> --user=admin --password=adminpwrd --port=3306
```
```sh
USE classicmodels;
SHOW TABLES;
SELECT * FROM ratings LIMIT 20;
EXIT;
```
#### 🔹 **Step 2: Run AWS Glue Job**
```sh
aws glue start-job-run --job-name de-c1w4-etl-job | jq -r '.JobRunId'
```
```sh
aws glue get-job-run --job-name de-c1w4-etl-job --run-id <JobRunID> --query "JobRun.JobRunState"
```
### 2️⃣ Vector Database Setup
We use PostgreSQL with pgvector to store embeddings.

####🔹 **Step 1: Deploy Vector DB using Terraform**
```sh
cd terraform
terraform apply
```
####🔹 **Step 2: Retrieve Credentials**
```sh
terraform output vector_db_master_username
terraform output vector_db_master_password
terraform output vector_db_host
```
####🔹 **Step 3: Connect to PostgreSQL**
```sh
psql --host=<VectorDB_Host> --username=postgres --password --port=5432
```
####🔹 **Step 4: Run SQL script to create tables**
```sh
\i '../sql/embeddings.sql';
```
### 3️⃣ Connecting ML Model to Vector DB

AWS Lambda loads the trained ML model from S3.

Lambda queries Vector DB for similar item embeddings.

Recommender system provides real-time suggestions.

####🔹 **Step 1: Deploy Lambda & API Gateway**
```sh
cd terraform
terraform apply
```
####🔹 **Step 2: Invoke Lambda function**
```sh
aws lambda invoke --function-name de-c1w4-recommender --payload '{}' response.json
```
### 4️⃣ Streaming Pipeline Implementation
Kafka & Kinesis process real-time user interactions.

The ML Model recommends products based on recent activity.

Updated recommendations are pushed via API.

####🔹 **Step 1: Deploy Kafka & Kinesis**
```sh
cd terraform
terraform apply
```
####🔹 **Step 2: Produce a message**
```sh
aws kinesis put-record --stream-name recommendation-stream --data "user_id=123,item_id=456"
```
####🔹 **Step 3: Consume messages**
```sh
aws kinesis get-shard-iterator --stream-name recommendation-stream --shard-id shardId-000000000000 --shard-iterator-type LATEST
```

### Final Validation


✅ Batch Data is Processed & Stored in S3



✅ Embeddings are Uploaded to Vector DB


✅ Recommender System Works via Lambda


✅ Streaming Pipeline Provides Real-Time Recommendations
