# Technical Design Document: Vani-Bridge AI (v2.0)

## 1. System Architecture: Serverless Event-Driven Pipeline
The platform is built on an event-driven model to ensure high availability and cost-efficiency for a solo-developer operation.

### 1.1 Components:
* **UI Layer:** A responsive React.js frontend deployed via **AWS Amplify**.
* **Orchestration Layer:** **AWS Step Functions** serves as the state machine, coordinating the flow between AI inference, audio synthesis, and scheduling.
* **AI Compute Layer:** **Amazon Bedrock** provides the foundation models (Claude 3.5 and SDXL).
* **Scheduling Layer:** **Amazon EventBridge Scheduler** manages the one-time triggers for social media publication.

## 2. The "Vani" Scheduling Workflow
1. **Selection:** The user reviews the AI-generated "Content Kit" and chooses a target platform and timestamp.
2. **Persistence:** The Step Function metadata is stored in **Amazon DynamoDB**, marking the status as `SCHEDULED`.
3. **Trigger:** **EventBridge** monitors the timestamp; upon match, it invokes the `Publisher-Lambda`.
4. **Execution:** The Lambda retrieves encrypted OAuth tokens from **AWS Secrets Manager** and the finalized assets from **Amazon S3** to execute the POST request to social APIs.

## 3. Data Schema (DynamoDB)
* **PK (Partition Key):** `UserID`
* **SK (Sort Key):** `JobID#Timestamp`
* **Attributes:** `Input_S3_URI`, `Output_Image_URI`, `Caption_Text`, `Post_Status` (Generated, Scheduled, Success, Error), `Platform_Target`.

## 4. Security Framework
* **Identity:** **AWS Cognito** for secure user authentication and session management.
* **Access Control:** All cross-service communication is governed by IAM roles following the principle of least privilege.
* **Data Integrity:** S3 buckets are configured with Block Public Access and enforced SSL/TLS for all transfers.