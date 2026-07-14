# Receipt-Automated-Processing-Tool
A production-grade serverless AWS architecture for automated financial receipt processing. The pipeline utilizes Amazon S3 for ingestion, triggering Python Lambda functions to extract structured data via Amazon Textract. Records are indexed in DynamoDB, and real-time transaction summaries are dispatched to users via Amazon SES.
