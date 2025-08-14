# ETL Pipeline - AWS Serverless Data Processing

## Overview

This ETL (Extract, Transform, Load) pipeline is a serverless solution built on AWS that automatically processes order data from JSON format to optimized Parquet format, enabling efficient data analytics and warehousing.

## Architecture Overview

```
S3 (Source) → Lambda Function → S3 (Staging) → Glue Crawler → Data Catalog
```

## Services Used

### 1. **Amazon S3 (Simple Storage Service)**
- **Source Bucket**: Stores incoming JSON order data files
- **Staging Bucket**: Stores processed Parquet files in the `orders_parquet_datalake/` folder
- **Event-Driven**: Triggers Lambda function when new files are uploaded

### 2. **AWS Lambda**
- **Runtime**: Python 3.x
- **Trigger**: S3 event notifications
- **Functionality**: 
  - Extracts JSON data from S3
  - Transforms/flattens nested order data
  - Converts to Parquet format
  - Loads processed data back to S3
  - Triggers Glue crawler for metadata discovery

### 3. **AWS Glue**
- **Crawler**: `etl_pipeline_crawler`
- **Purpose**: Automatically discovers and catalogs new data schemas
- **Output**: Updates AWS Glue Data Catalog with table definitions
- **Integration**: Works with Amazon Athena, Redshift, and other analytics services

### 4. **Pandas & PyArrow**
- **Data Processing**: Handles data transformation and flattening
- **Parquet Conversion**: Converts data to columnar Parquet format for optimal storage and query performance
- **Memory Efficient**: Uses in-memory buffers without disk I/O

## Data Flow

### 1. **Extract Phase**
- Lambda function is triggered by S3 upload event
- Reads JSON file from source S3 bucket
- Parses JSON content into Python objects

### 2. **Transform Phase**
- **Data Flattening**: Converts nested JSON structure to flat tabular format
- **Schema Normalization**: Extracts customer and product information into separate columns
- **Data Type Handling**: Ensures consistent data types across records

### 3. **Load Phase**
- Converts flattened data to Parquet format using PyArrow engine
- Generates timestamped filename for versioning
- Uploads processed Parquet file to staging S3 bucket
- Triggers Glue crawler to update data catalog

## Data Schema

### Input JSON Structure
```json
{
  "order_id": "string",
  "order_date": "string",
  "total_amount": "number",
  "customer": {
    "customer_id": "string",
    "name": "string",
    "email": "string",
    "address": "string"
  },
  "products": [
    {
      "product_id": "string",
      "name": "string",
      "category": "string",
      "price": "number",
      "quantity": "number"
    }
  ]
}
```

### Output Parquet Schema
| Column | Type | Description |
|--------|------|-------------|
| order_id | string | Unique order identifier |
| order_date | string | Date of order placement |
| total_amount | double | Total order value |
| customer_id | string | Customer identifier |
| customer_name | string | Customer's full name |
| email | string | Customer's email address |
| address | string | Customer's delivery address |
| product_id | string | Product identifier |
| product_name | string | Product name |
| category | string | Product category |
| price | double | Product unit price |
| quantity | int64 | Quantity ordered |

## Benefits

### 1. **Performance**
- **Parquet Format**: Columnar storage for faster analytical queries
- **Compression**: Efficient storage with reduced costs
- **Partitioning**: Ready for data lake partitioning strategies

### 2. **Scalability**
- **Serverless**: Automatically scales with data volume
- **Event-Driven**: Processes data as it arrives
- **No Infrastructure Management**: Fully managed AWS services

### 3. **Cost Efficiency**
- **Pay-per-use**: Only pay for actual processing time
- **Storage Optimization**: Parquet format reduces storage costs
- **No Idle Resources**: Lambda functions only run when needed

### 4. **Data Quality**
- **Schema Validation**: Consistent data structure
- **Metadata Management**: Automated catalog updates via Glue
- **Audit Trail**: Timestamped file versions

## Use Cases

- **E-commerce Analytics**: Order processing and customer behavior analysis
- **Data Warehousing**: Preparing data for business intelligence tools
- **Real-time Analytics**: Near real-time data processing pipelines
- **Machine Learning**: Clean, structured data for ML model training

## Monitoring & Observability

- **CloudWatch Logs**: Lambda execution logs and errors
- **S3 Event Notifications**: File processing triggers
- **Glue Crawler Metrics**: Data catalog update status
- **Lambda Metrics**: Function performance and error rates

## Security Features

- **IAM Roles**: Least privilege access to AWS services
- **S3 Encryption**: Data encryption at rest and in transit
- **VPC Integration**: Can be configured for private network access
- **CloudTrail**: API call logging for audit purposes

## Future Enhancements

- **Data Quality Checks**: Add validation rules and error handling
- **Partitioning Strategy**: Implement date-based partitioning
- **Error Handling**: Dead letter queues for failed processing
- **Monitoring Dashboard**: CloudWatch dashboards for pipeline health
- **Data Lineage**: Track data transformations and dependencies 