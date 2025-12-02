---
title: "Workshop"
date: "2025-09-15"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Excel-to-DynamoDB on AWS using S3 Notifications

#### Tổng quan

Trong workshop này, chúng ta sẽ xây dựng một ứng dụng serverless hoàn chỉnh để **import dữ liệu từ file Excel vào DynamoDB** sử dụng các dịch vụ AWS:
- **AWS Lambda** để xử lý logic backend
- **Amazon S3** để lưu trữ file Excel
- **S3 Event Notifications** để trigger Lambda tự động khi có file mới
- **Amazon DynamoDB** để lưu trữ dữ liệu từ excel
- **Amazon Cognito** để xác thực người dùng
- **API Gateway** để expose REST APIs

Chúng ta sẽ thực hiện:
- Deploy serverless application bằng **AWS SAM**
- Cấu hình S3 Event để trigger Lambda
- Parse file Excel trong Lambda và lưu vào DynamoDB
- Tích hợp Cognito authentication
- Theo dõi tiến trình import job

#### Kiến trúc hệ thống

Workshop này triển khai một kiến trúc serverless event-driven với các thành phần:

**API Layer:**
- API Gateway với Cognito Authorizer
- REST endpoints cho authentication và import operations

**Processing Layer:**
- Lambda functions xử lý business logic
- S3 Event Notifications trigger processing
- Apache POI library để parse Excel files

**Storage Layer:**
- S3 bucket cho file uploads
- DynamoDB tables cho các dữ liệu từ file excel và import jobs

#### Nội dung

1. [Tổng quan Workshop](5.1-workshop-overview/)
2. [Chuẩn bị môi trường](5.2-prerequisites/)
3. [Kiến trúc hệ thống](5.3-architecture/)
4. [Deploy Backend (AWS SAM)](5.4-deploy-backend/)
5. [Deploy Frontend (React)](5.5-deploy-frontend/)
6. [Test ứng dụng](5.6-test-application/)
7. [Dọn dẹp tài nguyên](5.7-cleanup/)
