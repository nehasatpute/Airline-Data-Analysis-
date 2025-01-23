# Airline-Data-Analysis- Flight Booking Data Pipeline with Airflow & CICD 
This project involves ingesting and processing airline data from two CSV files (one containing flight details and the other containing airport information) using a variety of AWS services. Here's a breakdown of the stack used and the overall process:

### Project Overview: **Airline Data Ingestion**

#### **Data Files:**
1. **Flight Data CSV:**
   - Columns: `Carrier`, `OriginAirportID`, `DestAirportID`, `DepDelay`, `ArrDelay`
   - This contains information about flight delays, carriers, and airport IDs.

2. **Airport Information CSV:**
   - Columns: `airport_id`, `city`, `state`, `name`
   - This contains information about airports, including their ID, location (city, state), and name.

### **Tech Stack:**
1. **S3 (Amazon Simple Storage Service):**
   - The CSV files will be stored in an S3 bucket. S3 provides durable, scalable, and secure storage for data.
   
2. **S3 CloudTrail Notification:**
   - Whenever new files are added to the S3 bucket (e.g., when a new CSV file is uploaded), an event notification is triggered. CloudTrail logs are used for auditing and tracking changes to the S3 bucket. This ensures that any changes to the data, such as file uploads, are logged for traceability.

3. **EventBridge Pattern Rule:**
   - EventBridge listens for specific events, such as new files being uploaded to the S3 bucket. The EventBridge rule is set to trigger whenever a new file is detected in the S3 bucket. It then sends the event to the next step in the pipeline (AWS Glue).

4. **Glue Crawler:**
   - AWS Glue is a managed ETL (Extract, Transform, Load) service. The Glue Crawler scans the CSV files stored in S3 and automatically detects the schema (column names, data types) of the data. It then creates a metadata table in the Glue Data Catalog that helps Glue and other AWS services understand the structure of the data.

5. **Glue Visual ETL (With PySpark):**
   - AWS Glue's Visual ETL tool allows you to design ETL workflows without writing complex code. In this case, PySpark (a powerful data processing library) is used to transform the data.
     - **Join Operation:** The flight data (which contains `OriginAirportID` and `DestAirportID`) will be joined with the airport information CSV (based on `airport_id` and the airport ID columns in the flight data) to enrich the flight details with airport names, cities, and states.
     - **Delay Calculations:** You can perform transformations to analyze flight delays, like calculating average delays for specific airlines, airports, or regions.

6. **SNS (Simple Notification Service):**
   - After the ETL process is completed, you can use SNS to notify stakeholders (such as data analysts or team members) about the success or failure of the data pipeline. For example, an SNS topic can send an email notification or trigger a Lambda function for further processing.

7. **Redshift:**
   - Once the data is processed and transformed, it can be loaded into Amazon Redshift, a managed data warehouse. Redshift is used for querying large datasets efficiently.
   - After loading the data into Redshift, you can run complex analytical queries, like identifying patterns in flight delays, analyzing carrier performance, or determining the best airports based on delay metrics.

8. **Step Functions:**
   - AWS Step Functions are used to manage and orchestrate the workflow of various services in the pipeline. A Step Function state machine can ensure that the process flows sequentially: from data ingestion in S3 to the Glue ETL process, and then loading the data into Redshift.
   - You can set up error handling, retries, and conditional logic based on the success or failure of each step in the pipeline.

### **Process Flow:**
1. A new CSV file is uploaded to an S3 bucket.
2. The CloudTrail notification is triggered and sends an event to EventBridge.
3. EventBridge invokes the Glue Crawler, which scans the file, detects its schema, and stores metadata in the Glue Data Catalog.
4. The Glue Visual ETL job is triggered to process the data using PySpark:
   - The flight data and airport data are joined, and transformations (such as delay analysis) are performed.
5. After successful data transformation, an SNS notification is sent to stakeholders.
6. The processed data is loaded into Redshift for analytical querying.
7. Step Functions orchestrate the entire workflow, ensuring all steps occur in the correct sequence and handling failures or retries as necessary.

### **Outcome:**
This pipeline enables automated ingestion, processing, transformation, and storage of airline data, creating a robust system for analyzing flight delays, airline performance, and airport metrics in near real-time. Redshift can be used for advanced analytics, reporting, and insights into the airline data.
