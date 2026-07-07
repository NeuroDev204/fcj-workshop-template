---
title : "AWS CodeBuild"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.7.2. </b> "
---

Pipeline dùng **3** CodeBuild project (đều do `cicd-pipeline.yml` tạo, đều chạy trên `aws/codebuild/amazonlinux2-x86_64-standard:5.0`, đều dùng chung `periodiq-codebuild-role-dev`). Trang này nói về `periodiq-backend-build-dev` và `periodiq-frontend-build-dev`; project thứ 3, `periodiq-sam-deploy-dev`, nằm ở [5.7.3](../5.7.3-cloudformation-sam/) vì nó hoàn toàn về việc deploy.

#### `periodiq-backend-build-dev` - `buildspec-backend.yml`

Project này build backend .NET 10. Phase `install` pin đúng .NET 10 SDK (file `global.json` mặc định của CodeBuild bị xoá trước để không xung đột), sau đó `pre_build`/`build` restore, build, **unit-test**, và `dotnet publish` project API:

```yaml
build:
  commands:
    - $DOTNET_ROOT/dotnet build PeriodIQ.Backend/PeriodIQ.sln --configuration Release --no-restore
    - $DOTNET_ROOT/dotnet test PeriodIQ.Backend/PeriodIQ.sln --configuration Release --no-build --logger trx --results-directory ./test-results --collect "XPlat Code Coverage"
    - $DOTNET_ROOT/dotnet publish PeriodIQ.Backend/PeriodIQ.Api/PeriodIQ.Api.csproj --configuration Release --output ./publish --self-contained true --runtime linux-x64
```

Kết quả test được báo về CodeBuild dưới dạng report `VisualStudioTrx` (`periodiq-test-report`), và package NuGet được cache ở `/root/.nuget/packages` giữa các lần build.

Đọc log từ lần chạy pipeline gần nhất:

```bash
aws codebuild batch-get-builds --ids periodiq-backend-build-dev:bebd4b0b-8441-490d-a0bd-033832b02936 \
  --query "builds[0].logs.[groupName,streamName]"
aws logs get-log-events --log-group-name /aws/codebuild/periodiq-backend-build-dev \
  --log-stream-name bebd4b0b-8441-490d-a0bd-033832b02936
```

![Log build backend - dotnet build thành công, 49/49 test pass](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.2-CodeBuild/01-real-backend-build-tests.png)

**49/49 unit test pass, 0 fail** - đây là kết quả test của `PeriodIQ.Api.Tests.dll` từ lần merge PR #49.

#### `periodiq-frontend-build-dev` - `buildspec-frontend.yml`

Project này build frontend React 19 + Vite bằng `pnpm`, nhưng phần thú vị là *cách* nó lấy cấu hình runtime. Thay vì hardcode Cognito ID hay API URL, phase `build` lấy trực tiếp từ stack `periodiq-dev` đã deploy xong (deploy ở stage trước đó, bởi `Deploy`) và export thành biến môi trường `VITE_*` **trong cùng một lệnh shell** với `pnpm run build`, vì Vite chỉ bake vào bundle những biến `VITE_*` có mặt lúc build:

```bash
export VITE_COGNITO_USER_POOL_ID=$(aws cloudformation describe-stacks --stack-name "$STACK_NAME" \
  --query "Stacks[0].Outputs[?OutputKey=='CognitoUserPoolId'].OutputValue" --output text)
export VITE_COGNITO_CLIENT_ID=$(aws cloudformation describe-stacks --stack-name "$STACK_NAME" \
  --query "Stacks[0].Outputs[?OutputKey=='CognitoUserPoolClientId'].OutputValue" --output text)
CF_DOMAIN=$(aws cloudformation describe-stacks --stack-name "$STACK_NAME" \
  --query "Stacks[0].Outputs[?OutputKey=='CloudFrontDomainName'].OutputValue" --output text)
export VITE_API_URL="https://$CF_DOMAIN"
# nếu bất kỳ biến nào rỗng, build cố tình fail thay vì deploy nhầm một bundle rỗng
pnpm run build
```

Phase `post_build` sau đó sync `dist/` lên bucket frontend và invalidate CloudFront để bản build mới được phục vụ ngay thay vì `index.html` cũ còn cache:

```bash
aws s3 sync dist/ s3://${FRONTEND_BUCKET_NAME}/ --delete --cache-control "max-age=86400"
aws s3 cp dist/index.html s3://${FRONTEND_BUCKET_NAME}/index.html --cache-control "no-cache, no-store, must-revalidate"
DIST_ID=$(aws cloudfront list-distributions --query "DistributionList.Items[?DomainName=='$CF_DOMAIN'].Id | [0]" --output text)
aws cloudfront create-invalidation --distribution-id "$DIST_ID" --paths "/*"
```

Đọc log từ cùng lần chạy pipeline:

```bash
aws logs get-log-events --log-group-name /aws/codebuild/periodiq-frontend-build-dev \
  --log-stream-name 1de32fc1-9e21-4155-94d6-85b78f931872
```

![Log build frontend - build thành công, sync lên S3, đã tạo invalidation CloudFront](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.2-CodeBuild/02-real-frontend-build-deploy.png)

Log cho thấy một invalidation CloudFront (`I8NE2A60VGS4W13MR9G59MOHY6`) được tạo trên distribution (`E2LYL2VAX7BYMQ`) đang phục vụ `d1di1pzmfypszp.cloudfront.net`.

> **CLI Note:** mọi lệnh trên trang này đều đọc từ log build CodeBuild đã hoàn thành sẵn (`aws codebuild batch-get-builds` + `aws logs get-log-events`) - không có gì ở đây được chạy lại hay mô phỏng cho workshop.
