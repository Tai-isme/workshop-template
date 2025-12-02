---
title : "SAM Deploy"
date : "2025-09-15"
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

#### Deploy với AWS SAM

SAM CLI sẽ package code và deploy toàn bộ infrastructure lên AWS thông qua CloudFormation.

#### Deploy Lần Đầu (Guided Mode)

Sử dụng `--guided` để SAM hỏi từng tham số cấu hình.

```powershell
sam deploy --guided
```

#### Trả Lời Các Câu Hỏi

SAM sẽ hỏi một loạt câu hỏi. Dưới đây là gợi ý trả lời:

```
Setting default arguments for 'sam deploy'
=========================================

Stack Name [sam-app]: excel-import-workshop
```
> 💡 Nhập: `excel-import-workshop` (hoặc tên bạn muốn)

---

```
AWS Region [us-east-1]: us-east-1
```
> 💡 Nhập: `us-east-1` (N. Virginia) hoặc region gần bạn nhất

---

```
Parameter Environment [dev]: dev
```
> 💡 Nhập: `dev` (environment name)

---

```
#Shows you resources changes to be deployed and require a 'Y' to initiate deploy
Confirm changes before deploy [Y/n]: Y
```
> 💡 Nhập: `Y` (để review changes trước khi deploy)

---

```
#SAM needs permission to be able to create roles to connect to the resources in your template
Allow SAM CLI IAM role creation [Y/n]: Y
```
> 💡 Nhập: `Y` (cho phép SAM tạo IAM roles cho Lambda)

---

```
#Preserves the state of previously provisioned resources when an operation fails
Disable rollback [y/N]: N
```
> 💡 Nhập: `N` (enable rollback nếu deploy thất bại)

---

```
RegisterFunction may not have authorization defined, Is this okay? [y/N]: y
ConfirmFunction may not have authorization defined, Is this okay? [y/N]: y
LoginFunction may not have authorization defined, Is this okay? [y/N]: y
```
> 💡 Nhập: `y` cho tất cả (các functions này intentionally không cần auth)

---

```
Save arguments to configuration file [Y/n]: Y
```
> 💡 Nhập: `Y` (lưu config để lần sau không cần nhập lại)

---

```
SAM configuration file [samconfig.toml]: samconfig.toml
```
> 💡 Enter (dùng default)

---

```
SAM configuration environment [default]: default
```
> 💡 Enter (dùng default)

---

#### Quá Trình Deploy

Sau khi trả lời xong, SAM sẽ bắt đầu deploy:

**Phase 1: Preparing CloudFormation**

```
Uploading to excel-import-workshop/xxxxx  45678912 / 45678912  (100.00%)

Waiting for changeset to be created..
```

> ⏱ Phase này mất ~2-3 phút

---

**Phase 2: CloudFormation Change Set**

SAM sẽ hiển thị danh sách resources sẽ được tạo:

```
CloudFormation stack changeset
-------------------------------------------------------------------------------------------------
Operation                     LogicalResourceId             ResourceType                  
-------------------------------------------------------------------------------------------------
+ Add                         Api                           AWS::ApiGateway::RestApi      
+ Add                         ConfirmFunction               AWS::Lambda::Function         
+ Add                         CoursesTable                  AWS::DynamoDB::Table          
+ Add                         GenerateUploadUrlFunction     AWS::Lambda::Function         
+ Add                         GetJobStatusFunction          AWS::Lambda::Function         
+ Add                         ImportBucket                  AWS::S3::Bucket               
+ Add                         ImportJobsTable               AWS::DynamoDB::Table          
+ Add                         ImportS3TriggerFunction       AWS::Lambda::Function         
+ Add                         ListImportJobsFunction        AWS::Lambda::Function         
+ Add                         LoginFunction                 AWS::Lambda::Function         
+ Add                         LogoutFunction                AWS::Lambda::Function         
+ Add                         RegisterFunction              AWS::Lambda::Function         
+ Add                         StudentsTable                 AWS::DynamoDB::Table          
+ Add                         UserPool                      AWS::Cognito::UserPool        
+ Add                         UserPoolClient                AWS::Cognito::UserPoolClient  
+ Add                         [... IAM Roles ...]           AWS::IAM::Role                
-------------------------------------------------------------------------------------------------

Changeset created successfully. arn:aws:cloudformation:us-east-1:123456789012:changeSet/...
```

**Confirm deploy:**

```
Deploy this changeset? [y/N]: y
```
> 💡 Nhập: `y` để deploy

---

**Phase 3: CloudFormation Execution**

```
2025-12-01 10:30:00 - Waiting for stack create/update to complete

CloudFormation events from stack operations
-------------------------------------------------------------------------------------------------
ResourceStatus                ResourceType                  LogicalResourceId             
-------------------------------------------------------------------------------------------------
CREATE_IN_PROGRESS            AWS::CloudFormation::Stack    excel-import-workshop         
CREATE_IN_PROGRESS            AWS::DynamoDB::Table          StudentsTable                 
CREATE_IN_PROGRESS            AWS::DynamoDB::Table          CoursesTable                  
CREATE_IN_PROGRESS            AWS::DynamoDB::Table          ImportJobsTable               
CREATE_IN_PROGRESS            AWS::Cognito::UserPool        UserPool                      
CREATE_IN_PROGRESS            AWS::S3::Bucket               ImportBucket                  
CREATE_COMPLETE               AWS::DynamoDB::Table          StudentsTable                 
CREATE_COMPLETE               AWS::DynamoDB::Table          CoursesTable                  
CREATE_COMPLETE               AWS::DynamoDB::Table          ImportJobsTable               
CREATE_COMPLETE               AWS::Cognito::UserPool        UserPool                      
CREATE_IN_PROGRESS            AWS::Cognito::UserPoolClient  UserPoolClient                
CREATE_COMPLETE               AWS::Cognito::UserPoolClient  UserPoolClient                
CREATE_COMPLETE               AWS::S3::Bucket               ImportBucket                  
CREATE_IN_PROGRESS            AWS::IAM::Role                RegisterFunctionRole          
CREATE_COMPLETE               AWS::IAM::Role                RegisterFunctionRole          
CREATE_IN_PROGRESS            AWS::Lambda::Function         RegisterFunction              
CREATE_COMPLETE               AWS::Lambda::Function         RegisterFunction              
...
CREATE_IN_PROGRESS            AWS::ApiGateway::RestApi      Api                           
CREATE_COMPLETE               AWS::ApiGateway::RestApi      Api                           
CREATE_COMPLETE               AWS::CloudFormation::Stack    excel-import-workshop         
-------------------------------------------------------------------------------------------------

Successfully created/updated stack - excel-import-workshop in us-east-1
```

> ⏱ Phase này mất ~8-10 phút

---

#### Lưu Outputs Quan Trọng

Sau khi deploy xong, SAM hiển thị **Outputs** - các giá trị bạn cần cho frontend:

```
-------------------------------------------------------------------------------------------------
Outputs                                                                                      
-------------------------------------------------------------------------------------------------
Key                 ApiUrl                                                                 
Description         API Gateway URL                                                        
Value               https://abc123def4.execute-api.us-east-1.amazonaws.com/dev             

Key                 BucketName                                                             
Description         S3 Bucket Name                                                         
Value               workshop-excel-imports-123456789012                                    

Key                 UserPoolId                                                             
Description         Cognito User Pool ID                                                   
Value               us-east-1_xYzAbC123                                                    

Key                 UserPoolClientId                                                       
Description         Cognito User Pool Client ID                                            
Value               1a2b3c4d5e6f7g8h9i0j1k2l3m                                             
-------------------------------------------------------------------------------------------------

Stack excel-import-workshop outputs:
Key                  Value
ApiUrl               https://abc123def4.execute-api.us-east-1.amazonaws.com/dev
BucketName           workshop-excel-imports-123456789012
UserPoolId           us-east-1_xYzAbC123
UserPoolClientId     1a2b3c4d5e6f7g8h9i0j1k2l3m
```

> ⚠️ **Quan trọng:** Copy và lưu lại 4 giá trị này! Bạn sẽ cần chúng để cấu hình frontend ở bước 6.5.

**Cách lưu outputs:**

Tạo file `outputs.txt`:
```powershell
# Lưu outputs vào file
sam list stack-outputs --stack-name excel-import-workshop --output json > outputs.txt
```

Hoặc copy thủ công vào notepad.

---

#### Deploy Lần Sau (Không Cần Guided)

Nếu bạn đã deploy một lần với `--guided`, lần sau chỉ cần:

```powershell
# Build changes
sam build

# Deploy với config đã lưu
sam deploy
```

SAM sẽ đọc config từ `samconfig.toml` và deploy ngay.

---

#### Xem Outputs Sau Khi Deploy

Nếu bạn quên copy outputs, có thể xem lại:

**Option 1: SAM CLI**

```powershell
sam list stack-outputs --stack-name excel-import-workshop
```

**Option 2: AWS CLI**

```powershell
aws cloudformation describe-stacks --stack-name excel-import-workshop --query "Stacks[0].Outputs" --output table
```

**Option 3: AWS Console**

1. Mở [CloudFormation Console](https://console.aws.amazon.com/cloudformation)
2. Click stack `excel-import-workshop`
3. Tab **Outputs**
4. Copy các giá trị

![CloudFormation Outputs](/images/6-Excel-Workshop/cf-outputs.png)

---

#### Troubleshooting

**Lỗi: "Error: Unable to upload artifact... Access Denied"**

Nguyên nhân: AWS credentials không có quyền tạo/upload S3.

Solution:
```powershell
# Verify credentials
aws sts get-caller-identity

# Check IAM permissions (cần S3 write access)
```

---

**Lỗi: "CREATE_FAILED ... Resource already exists"**

Nguyên nhân: Đã có stack với tên này, hoặc resource bị trùng.

Solution:
```powershell
# Check existing stacks
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE

# Delete old stack nếu cần
sam delete --stack-name excel-import-workshop

# Deploy lại
sam deploy --guided
```

---

**Lỗi: "Rate exceeded" hoặc "Throttling"**

Nguyên nhân: Quá nhiều requests cùng lúc (hiếm gặp).

Solution:
- Chờ 1-2 phút
- Chạy lại `sam deploy`

---

**Lỗi: "Rollback complete" - Stack deploy failed**

Nguyên nhân: Một resource nào đó không tạo được (thường là IAM permissions).

Solution:
```powershell
# Xem chi tiết lỗi
aws cloudformation describe-stack-events --stack-name excel-import-workshop --query "StackEvents[?ResourceStatus=='CREATE_FAILED']" --output table

# Fix issue và deploy lại
sam deploy
```

---

#### ✅ Checklist

Trước khi chuyển sang bước tiếp theo:

- [ ] `sam deploy --guided` chạy thành công
- [ ] CloudFormation stack status: `CREATE_COMPLETE`
- [ ] Đã lưu 4 outputs: ApiUrl, BucketName, UserPoolId, UserPoolClientId
- [ ] File `samconfig.toml` đã được tạo

#### 🚀 Tiếp Theo

Deploy thành công! Hãy verify resources trên AWS Console.

[➡️ Tiếp theo: Verify Resources](../6.4.3-verify-resources/)
