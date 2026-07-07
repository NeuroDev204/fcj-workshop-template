---
title : "Dọn dẹp tài nguyên"
date : 2024-01-01
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
---

Workshop này khác với một bài lab "dựng resource sandbox rồi xoá" thông thường: các mục [5.7.1](../5.7-nguoi5-cicd/5.7.1-codepipeline/) đến [5.7.4](../5.7-nguoi5-cicd/5.7.4-cloudwatch/) ghi lại **pipeline và stack production đang chạy sẵn** (`periodiq-pipeline-dev`, `periodiq-dev`) - không phải bản sao dùng-rồi-bỏ. Không có gì để dọn dẹp ở đó cả, vì hạ tầng đó cần tiếp tục chạy: đó là hệ thống live đang phục vụ **https://d1di1pzmfypszp.cloudfront.net/**.

> **Không có gì thuộc pipeline chính, stack `periodiq-dev`/`periodiq-cicd-pipeline`, hay bất kỳ CodeBuild project/role/bucket production nào bị xoá, tắt, hay sửa đổi trong lúc viết workshop này.** Mọi lệnh chạy trên chúng, xuyên suốt cả phần này, đều là lệnh `describe-*` / `get-*` / `list-*` chỉ đọc.

#### Những gì đã được dọn dẹp

Trong lúc chuẩn bị workshop này, tôi có dựng và xoá một bộ resource nháp riêng biệt, đặt tên rõ ràng (tiền tố `periodiq-workshop-*`, trong cùng account) để thử xem CodePipeline/CodeBuild/CloudFormation/CloudWatch khớp với nhau ra sao qua CLI, trước khi quyết định ghi lại trực tiếp pipeline chính. Các resource nháp đó - một pipeline, hai CodeBuild project, ba IAM role, một S3 bucket, một stack test nhỏ, và một log group - đều đã được xoá:

```bash
aws cloudformation list-stacks --query "StackSummaries[?starts_with(StackName,'periodiq-workshop')]"
aws codebuild list-projects --query "projects[?starts_with(@,'periodiq-workshop')]"
aws s3api list-buckets --query "Buckets[?starts_with(Name,'periodiq-workshop')]"
aws iam list-roles --query "Roles[?starts_with(RoleName,'periodiq-workshop')]"

# và xác nhận hệ thống chính không bị đụng vào, vẫn khoẻ mạnh:
aws cloudformation describe-stacks --stack-name periodiq-dev --query "Stacks[0].StackStatus"
curl -s -o /dev/null -w "%{http_code}" https://d1di1pzmfypszp.cloudfront.net/
```

![Toàn bộ resource nháp periodiq-workshop-* đã xoá sạch; periodiq-dev vẫn UPDATE_COMPLETE và site live vẫn trả về 200](/images/5-Workshop/5.8-Cleanup/01-workshop-scratch-cleaned-prod-untouched.png)

Cả 4 kiểm tra resource nháp đều trả về danh sách rỗng, và stack chính + site live không bị ảnh hưởng: `periodiq-dev` vẫn `UPDATE_COMPLETE`, và site công khai vẫn phản hồi `200`.
