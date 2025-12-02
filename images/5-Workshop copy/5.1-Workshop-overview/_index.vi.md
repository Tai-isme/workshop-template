---
title : "Giới thiệu"
date :  "2025-09-15" 
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Giới thiệu về Workshop

Workshop này hướng dẫn xây dựng một hệ thống **Excel Import** serverless trên AWS với các tính năng authentication và data processing.

#### Mục tiêu

Sau bài workshop chúng ta sẽ:

- **Làm quen với kiến trúc serverless:** Cách thiết kế ứng dụng với Lambda, API Gateway, S3, và DynamoDB
- **Làm quen event-driven architecture:** Sử dụng S3 Event Notifications để trigger xử lý tự động
- **Implement authentication:** Tích hợp Amazon Cognito cho authentication và API security
- **Parse Excel trong Lambda:** Sử dụng Apache POI library để đọc và xử lý file .xlsx/.xls
- **Deploy với AWS SAM:** Làm quen với SAM CLI để build và deploy và áp dụng IaC cho toàn bộ hạ tầng. 

#### Các Thành Phần Chính

- **8 Lambda Functions:** Register, Confirm, Login, Logout, GenerateUploadUrl, ListImportJobs, GetJobStatus, ImportS3Trigger
- **3 DynamoDB Tables:** Students, Courses, ImportJobs
- **1 S3 Bucket:** Lưu trữ file Excel với lifecycle policy (auto-delete sau 7 ngày)
- **1 Cognito User Pool:** Quản lý users và authentication
- **1 API Gateway:** REST API với Cognito authorizer

#### Thời Gian & Chi Phí

**Thời gian hoàn thành:** ~30 phút

**Chi phí:** Miễn phí (Tất cả đều nằm trong Free Tier)

> Để tránh chi phí không mong muốn, hãy thực hiện cleanup ngay sau khi hoàn thành workshop.

#### Yêu Cầu

- Hiểu biết cơ bản về AWS (console, regions, basic services)
- Biết sử dụng terminal/command line
- Đọc hiểu ngôn ngữ Java
