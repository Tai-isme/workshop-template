---
title: "Dọn dẹp tài nguyên"
date: "2025-12-01"
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

#### Dọn Dẹp Tài Nguyên AWS

Sau khi hoàn thành workshop, hãy xóa tất cả resources để tránh phát sinh chi phí không mong muốn.

#### 🎉 Chúc Mừng Hoàn Thành Workshop!

Trong workshop này, bạn đã học được:

**✅ Serverless Architecture:**
- Event-driven design với S3 Events trigger Lambda
- Asynchronous processing patterns
- Managed services giảm operational overhead

**✅ AWS Services Integration:**
- **Lambda:** 8 functions cho auth và processing
- **API Gateway:** REST API với Cognito authorizer
- **S3:** File storage với pre-signed URLs và event notifications
- **DynamoDB:** NoSQL database với GSI
- **Cognito:** User authentication và JWT tokens

**✅ Development Skills:**
- Deploy với AWS SAM (Infrastructure as Code)
- Parse Excel files trong Lambda (Apache POI)
- React frontend integration với AWS backend
- CloudWatch Logs cho debugging

**✅ Best Practices:**
- Security: JWT authentication, IAM least privilege
- Performance: Batch writes, GSI queries
- Cost optimization: On-demand pricing, S3 lifecycle

---

#### ⚠️ Quan Trọng: Xóa Resources Ngay

Nếu không xóa, bạn có thể bị charge cho:
- API Gateway requests (sau 12 tháng Free Tier)
- Lambda invocations (sau 1M requests/tháng)
- DynamoDB storage (sau 25GB Free Tier)
- S3 storage (sau 5GB Free Tier)
- CloudWatch Logs storage

> 💰 **Estimated cost nếu không cleanup:** $5-20/tháng (tùy usage)

---

#### 🗑️ Các Bước Cleanup

#### **Bước 1: Empty S3 Bucket (Quan Trọng Nhất!)**

CloudFormation **không thể xóa** bucket nếu có files bên trong.

**Option A: Xóa qua AWS Console**

1. Mở [S3 Console](https://s3.console.aws.amazon.com)
2. Tìm bucket: `workshop-excel-imports-{AccountId}`
3. Select bucket → Click **"Empty"**
4. Nhập: `permanently delete`
5. Click **"Empty"**

![Empty S3 Bucket](/images/6-Excel-Workshop/empty-s3-bucket.png)

---

**Option B: Xóa qua AWS CLI (Nhanh hơn)**

```powershell
# Get bucket name
$bucketName = aws cloudformation describe-stacks --stack-name excel-import-workshop --query "Stacks[0].Outputs[?OutputKey=='BucketName'].OutputValue" --output text

# Empty bucket
aws s3 rm s3://$bucketName --recursive

# Verify
aws s3 ls s3://$bucketName
# Output: (empty)
```

---

**Option C: PowerShell Script**

```powershell
# Empty bucket script
$bucketName = "workshop-excel-imports-YOUR_ACCOUNT_ID"

Write-Host "Emptying bucket: $bucketName" -ForegroundColor Yellow

# List và delete all objects
$objects = aws s3api list-objects-v2 --bucket $bucketName --query 'Contents[].Key' --output text

if ($objects) {
    $objects -split "`t" | ForEach-Object {
        aws s3api delete-object --bucket $bucketName --key $_
        Write-Host "Deleted: $_" -ForegroundColor Green
    }
    Write-Host "Bucket emptied successfully!" -ForegroundColor Green
} else {
    Write-Host "Bucket already empty" -ForegroundColor Cyan
}
```

---

#### **Bước 2: Xóa CloudFormation Stack**

**Option A: SAM CLI (Recommended)**

```powershell
# Di chuyển vào backend directory
cd excel-import-workshop

# Delete stack
sam delete --stack-name excel-import-workshop --region us-east-1 --no-prompts
```

Expected output:
```
Are you sure you want to delete the stack excel-import-workshop in the region us-east-1 ? [y/N]: y
Are you sure you want to delete the folder excel-import-workshop in S3 which contains the artifacts? [y/N]: y

Deleting stack excel-import-workshop...

CloudFormation stack changeset
-------------------------------------------------------------------------------------------------
Operation                     LogicalResourceId             ResourceType                  
-------------------------------------------------------------------------------------------------
- Delete                      Api                           AWS::ApiGateway::RestApi      
- Delete                      ConfirmFunction               AWS::Lambda::Function         
- Delete                      CoursesTable                  AWS::DynamoDB::Table          
...
-------------------------------------------------------------------------------------------------

Successfully deleted stack excel-import-workshop
```

> ⏱ Quá trình xóa mất ~5-8 phút.

---

**Option B: AWS CLI**

```powershell
# Delete stack
aws cloudformation delete-stack --stack-name excel-import-workshop --region us-east-1

# Wait for deletion complete
aws cloudformation wait stack-delete-complete --stack-name excel-import-workshop --region us-east-1

Write-Host "Stack deleted successfully!" -ForegroundColor Green
```

---

**Option C: AWS Console**

1. Mở [CloudFormation Console](https://console.aws.amazon.com/cloudformation)
2. Chọn stack: `excel-import-workshop`
3. Click **"Delete"**
4. Confirm: **"Delete stack"**

![Delete CloudFormation Stack](/images/6-Excel-Workshop/delete-cf-stack.png)

**Monitor deletion progress:**
- Tab **Events** hiển thị deletion progress
- Status: `DELETE_IN_PROGRESS` → `DELETE_COMPLETE`
- Nếu có lỗi: Status = `DELETE_FAILED` (xem Events để debug)

---

#### **Bước 3: Verify Resources Đã Xóa**

**Check Lambda Functions:**

```powershell
aws lambda list-functions --query "Functions[?starts_with(FunctionName, 'Register') || starts_with(FunctionName, 'Import')].FunctionName" --region us-east-1
```

Expected output: `[]` (empty array)

---

**Check DynamoDB Tables:**

```powershell
aws dynamodb list-tables --query "TableNames[?contains(@, 'workshop')]" --region us-east-1
```

Expected output: `[]`

---

**Check S3 Buckets:**

```powershell
aws s3 ls | Select-String "workshop-excel"
```

Expected output: (no results)

---

**Check API Gateway:**

```powershell
aws apigateway get-rest-apis --query "items[?name=='excel-import-workshop'].id" --region us-east-1
```

Expected output: `[]`

---

**Check Cognito User Pool:**

```powershell
aws cognito-idp list-user-pools --max-results 60 --query "UserPools[?Name=='ExcelWorkshopUsers'].Id" --region us-east-1
```

Expected output: `[]`

---

#### **Bước 4: (Optional) Xóa CloudWatch Log Groups**

CloudFormation **không tự động xóa** CloudWatch Logs. Nếu muốn xóa hoàn toàn:

**List log groups:**

```powershell
aws logs describe-log-groups --query "logGroups[?contains(logGroupName, '/aws/lambda/Register') || contains(logGroupName, '/aws/lambda/Import')].logGroupName" --region us-east-1
```

**Delete từng log group:**

```powershell
# Delete all Lambda log groups
$logGroups = @(
    "/aws/lambda/Register",
    "/aws/lambda/Confirm",
    "/aws/lambda/Login",
    "/aws/lambda/Logout",
    "/aws/lambda/GenerateUploadUrl",
    "/aws/lambda/ListImportJobs",
    "/aws/lambda/GetJobStatus",
    "/aws/lambda/ImportS3Trigger"
)

foreach ($logGroup in $logGroups) {
    aws logs delete-log-group --log-group-name $logGroup --region us-east-1
    Write-Host "Deleted: $logGroup" -ForegroundColor Green
}
```

> 💡 **Note:** Log groups không tốn nhiều tiền (5GB Free Tier), có thể giữ lại để reference sau.

---

#### **Bước 5: (Optional) Delete SAM Build Artifacts**

Xóa local build artifacts:

```powershell
# Trong backend directory
Remove-Item -Recurse -Force .aws-sam
Remove-Item -Recurse -Force target
Remove-Item samconfig.toml

Write-Host "Local build artifacts cleaned!" -ForegroundColor Green
```

---

#### **Bước 6: Stop Frontend Dev Server**

Nếu `npm run dev` vẫn đang chạy:

```powershell
# Trong terminal đang chạy npm run dev
Ctrl + C

# Confirm: Y
```

---

#### Troubleshooting Cleanup

**Lỗi: "Stack deletion failed - Resource XXX cannot be deleted"**

Nguyên nhân phổ biến:

1. **S3 bucket not empty:**
   ```
   Error: The bucket you tried to delete is not empty
   ```
   Solution: Quay lại Bước 1, empty bucket và delete stack lại.

2. **Lambda ENI cleanup:**
   ```
   Error: Network interface eni-xxx is currently in use
   ```
   Solution: Chờ 5-10 phút để AWS cleanup ENI, sau đó delete stack lại.

3. **IAM role in use:**
   ```
   Error: Role XXX cannot be deleted while in use
   ```
   Solution: Chờ vài phút, AWS sẽ detach role tự động.

---

**Lỗi: "Access Denied" khi delete resources**

Nguyên nhân: IAM user không có quyền delete.

Solution:
```powershell
# Verify IAM permissions
aws sts get-caller-identity

# Cần policy:
# - cloudformation:DeleteStack
# - lambda:DeleteFunction
# - dynamodb:DeleteTable
# - s3:DeleteBucket
# - etc.
```

---

**Force delete stack (nếu cần):**

```powershell
# Retain failed resources và delete stack anyway
aws cloudformation delete-stack --stack-name excel-import-workshop --retain-resources RegisterFunction --region us-east-1
```

Sau đó delete resource thủ công:
```powershell
aws lambda delete-function --function-name RegisterFunction --region us-east-1
```

---

#### ✅ Cleanup Checklist

Đảm bảo tất cả đã xóa:

- [ ] S3 bucket emptied và deleted
- [ ] CloudFormation stack deleted (status: DELETE_COMPLETE)
- [ ] Lambda functions: 0 functions còn lại
- [ ] DynamoDB tables: 0 tables còn lại
- [ ] API Gateway: API deleted
- [ ] Cognito User Pool: Pool deleted
- [ ] CloudWatch Log Groups: Deleted (optional)
- [ ] Local build artifacts: Cleaned (optional)
- [ ] Frontend dev server: Stopped

---

#### Verify No Charges

**Check AWS Cost Explorer:**

1. Mở [Cost Explorer](https://console.aws.amazon.com/cost-management/home#/cost-explorer)
2. Timeframe: Last 7 days
3. Group by: Service
4. Filter: Services used in workshop
   - Lambda, API Gateway, DynamoDB, S3, Cognito

Expected: Minimal/zero cost nếu cleanup đúng.

**Set up Billing Alert (Optional nhưng khuyến nghị):**

```powershell
# Create billing alarm (notify nếu > $5/month)
aws cloudwatch put-metric-alarm `
  --alarm-name WorkshopBillingAlert `
  --alarm-description "Alert nếu vượt $5" `
  --metric-name EstimatedCharges `
  --namespace AWS/Billing `
  --statistic Maximum `
  --period 21600 `
  --evaluation-periods 1 `
  --threshold 5 `
  --comparison-operator GreaterThanThreshold `
  --region us-east-1
```

---

#### 📚 Next Steps

**Bạn muốn học thêm?**

- ✅ Deploy production version với custom domain (Route 53 + ACM)
- ✅ Add CI/CD pipeline (CodePipeline + CodeBuild)
- ✅ Implement caching (ElastiCache hoặc DynamoDB DAX)
- ✅ Add monitoring & alerting (CloudWatch Alarms, SNS)
- ✅ Scale up: Handle large files (S3 Select, Step Functions)
- ✅ Add data validation & transformation (AWS Glue)

**Resources hữu ích:**

- [AWS Serverless Application Repository](https://serverlessrepo.aws.amazon.com/)
- [AWS SAM Examples](https://github.com/aws/aws-sam-cli-app-templates)
- [DynamoDB Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)
- [Lambda Performance Optimization](https://aws.amazon.com/blogs/compute/operating-lambda-performance-optimization-part-1/)

---

#### 🎊 Cảm Ơn!

Cảm ơn bạn đã hoàn thành workshop **Excel-to-DynamoDB on AWS using S3 Notifications**!

Nếu có feedback hoặc câu hỏi, hãy để lại comment hoặc tạo issue trên repository.

**Happy Serverless Building! 🚀**

---

#### 💡 Pro Tips

**Muốn giữ lại code để tham khảo sau:**

```powershell
# Backup code trước khi cleanup
$backupPath = "D:\Backups\excel-workshop-$(Get-Date -Format 'yyyyMMdd')"
Copy-Item -Recurse workshop\ $backupPath
Write-Host "Code backed up to: $backupPath"
```

**Muốn deploy lại sau này:**

1. Keep source code
2. Update `samconfig.toml` nếu cần thay region/stack name
3. Run: `sam build && sam deploy`
4. Update frontend `config.js` với outputs mới

**Share workshop với team:**

1. Push code lên Git repository (private/public)
2. Document môi trường cần thiết (Prerequisites)
3. Add CI/CD để auto-deploy khi có commits

---

**🏁 END OF WORKSHOP 🏁**
