# Receipt Automated Processing Tool

A production-grade, event-driven serverless AWS architecture designed to automate financial receipt ingestion, structural data extraction, persistent archival, and real-time transaction reporting.

---

## 🚀 System Architecture Flow

The entire application operates on a decoupled, serverless loop triggered by filesystem events:

1. **Ingestion:** A raw receipt image (PNG, JPEG, PDF) is uploaded to a specific folder in an **Amazon S3** bucket.
2. **Event Emission:** S3 registers the `ObjectCreated` state change and asynchronously invokes the orchestrating **AWS Lambda** function, passing the file metadata payload.
3. **Machine Learning Extraction:** The Lambda function normalizes the file path and calls **Amazon Textract** (`analyze_expense` API) to perform layout-aware deep learning extraction, categorizing vendor names, receipt dates, totals, and itemized line-item arrays.
4. **Persistent Archival:** The structured data is indexed and written directly to an **Amazon DynamoDB** database table.
5. **Push Notification:** The system dynamically compiles an HTML expense summary report and dispatches it immediately to the user's verified inbox via **Amazon SES**.

---

## 🛠️ Tech Stack & Requirements

### Infrastructure Requirements
*   **Platform Environments:** Amazon Web Services (AWS Core Sandbox)
*   **Core Services:** Amazon S3, AWS Lambda, Amazon Textract, Amazon DynamoDB, Amazon SES, Amazon CloudWatch

### Software & Environment Runtimes
*   **Primary Runtime:** AWS Lambda **Python 3.12** or **Python 3.13** standard runtime
*   **Primary SDK:** AWS SDK for Python (`boto3`)
*   **Built-in Modules:** `json`, `os`, `uuid`, `datetime`, `urllib.parse`

---

## 📂 Code Structure & Implementation

The core automation workflow is encapsulated within a single Python script divided into four modular components:

*   **Lambda Handler:** The foundational orchestrator that extracts the S3 bucket details, URL-decodes the incoming object key, verifies file existence via a `head_object` check, and executes downstream service blocks.
*   **Textract Processing:** Interacts with Textract's pre-trained computer vision model using an explicit region routing override (`region_name='ap-south-1'`) to map out summary fields and line-item transactional rows.
*   **DynamoDB Storage:** Handles the database persistence layer, formatting parsed records alongside a dynamic ISO-8601 execution timestamp and linking them back to the immutable primary hash key (`receipt_id`).
*   **Email Notification:** Formats an automated, human-readable HTML document string containing tabular receipt variables and dispatches the payload via the SES mail client engine.

---

## ⚙️ Configuration & Environment Variables

To run this pipeline without modifying the core script logic, ensure the following environment variables are securely mapped within your AWS Lambda configuration panel:

| Environment Variable | Description | Example Value |
| :--- | :--- | :--- |
| `DYNAMODB_TABLE` | Target DynamoDB table name for archival index | `Receipts` |
| `SES_SENDER_EMAIL` | The verified email identity acting as the outbound envelope | `your-email@gmail.com` |
| `SES_RECIPIENT_EMAIL` | The designated destination email address for push notifications | `recipient-email@gmail.com` |

---

## ⚠️ Important Deployment Notes

*   **Regional Endpoint Configurations:** Since Amazon Textract is unavailable in certain local zones like Hyderabad (`ap-south-2`), the script explicitly targets the Mumbai gateway (`ap-south-1`) for ML computer vision extraction.
*   **SES Sandbox Identity Verification:** While your AWS account resides in the SES Sandbox environment, both the `SES_SENDER_EMAIL` and `SES_RECIPIENT_EMAIL` addresses must be explicitly added and confirmed under the **SES > Verified Identities** dashboard before emails can be successfully sent.
*   **Execution Timeout:** Ensure the Lambda function's general timeout property is raised from the default 3 seconds to at least **30 seconds** to accommodate multi-second Textract ML processing windows.
