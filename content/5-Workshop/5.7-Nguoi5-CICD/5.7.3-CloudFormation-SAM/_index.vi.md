---
title : "CloudFormation / SAM"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.7.3. </b> "
---

CodeBuild project của stage `Deploy`, `periodiq-sam-deploy-dev`, là nơi `sam build`/`sam deploy` thực sự chạy - khác với 2 CodeBuild project còn lại, buildspec của nó được viết **inline ngay trong `cicd-pipeline.yml`** thay vì một file riêng trong repo, vì đây là logic deploy hạ tầng nên thuộc về chính định nghĩa pipeline.

#### Stage này làm gì, theo thứ tự

**1. Phase install** - cài .NET 10 SDK, `Amazon.Lambda.Tools`, và AWS SAM CLI mới hoàn toàn ở mỗi lần build:
```bash
/tmp/dotnet-install.sh --channel 10.0 --install-dir /root/dotnet10
/root/dotnet10/dotnet tool install --tool-path /root/dotnet10/tools Amazon.Lambda.Tools
pip install aws-sam-cli
```

**2. Deploy stack WAF, ở `us-east-1`** - WebACL của WAF scope CloudFront bắt buộc phải ở global/`us-east-1`, nên đây phải là một stack riêng biệt với stack chính (chạy ở `ap-southeast-1`):
```bash
sam deploy -t waf-cloudfront.yml --stack-name periodiq-waf-cloudfront-dev \
  --region us-east-1 --resolve-s3 --capabilities CAPABILITY_IAM \
  --no-confirm-changeset --no-fail-on-empty-changeset \
  --parameter-overrides Environment=dev
```

**3. Lấy WAF ARN** từ output của stack đó, rồi **`sam build` + `sam deploy` stack chính** ở `ap-southeast-1`, truyền WAF ARN vào làm tham số để template chính gắn nó vào CloudFront:
```bash
WAF_ARN=$(aws cloudformation describe-stacks --stack-name periodiq-waf-cloudfront-dev --region us-east-1 \
  --query "Stacks[0].Outputs[?OutputKey=='WebAclArn'].OutputValue" --output text)

sam build
sam deploy --no-confirm-changeset --no-fail-on-empty-changeset \
  --parameter-overrides Environment=dev AlertEmail=syvan.claude@outlook.com WafWebAclArn=$WAF_ARN
```

**4. Health check sau khi build xong** - gọi vào endpoint `/api/health` của API vừa deploy và cảnh báo (không làm fail build) nếu không phản hồi:
```bash
API_URL=$(aws cloudformation describe-stacks --stack-name periodiq-dev \
  --query "Stacks[0].Outputs[?OutputKey=='ApiUrl'].OutputValue" --output text)
curl -f "$API_URL/api/health" || echo "CANH BAO: Health check THAT BAI"
```

#### Đọc log từ lần deploy gần nhất

```bash
aws logs get-log-events --log-group-name /aws/codebuild/periodiq-sam-deploy-dev \
  --log-stream-name d6895bf1-a210-4e4c-b7d7-82ed49b5ba8b
```

![Log SAM deploy - stack WAF up to date, stack chính tạo/cập nhật xong, health check pass](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.3-CloudFormation-SAM/01-real-sam-deploy-log.png)

Stack WAF báo "No changes to deploy" (đã đúng sẵn từ lần chạy trước), stack chính tạo/cập nhật thành công, và health check trả về response khoẻ mạnh:
```json
{"status":"healthy","timestamp":"2026-07-06T08:12:58.9320196Z","version":"1.0.0.0","environment":"dev","commit":"local","buildNumber":"0"}
```

#### Kiểm tra trực tiếp stack đã deploy

```bash
aws cloudformation describe-stacks --stack-name periodiq-dev --query "Stacks[0].[StackStatus,Outputs]"
```

![Stack periodiq-dev - UPDATE_COMPLETE, kèm toàn bộ output](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.3-CloudFormation-SAM/02-real-stack-outputs.png)

Kết quả xác nhận stack đang `UPDATE_COMPLETE` và có đúng các resource đã mô tả ở [Proposal](../../../2-proposal/): Cognito User Pool (`ap-southeast-1_3l2EERPJk`), ARN của WAF WebACL gắn vào CloudFront, SQS queue, SNS alerts topic, và - khớp với site đang chạy - **`CloudFrontDomainName: d1di1pzmfypszp.cloudfront.net`**.

> **CLI Note:** đây chính là domain CloudFront live được nhắc đến xuyên suốt phần còn lại của báo cáo này - chỉ có đúng một stack `periodiq-dev`, và đó chính là stack đang phục vụ site công khai.
