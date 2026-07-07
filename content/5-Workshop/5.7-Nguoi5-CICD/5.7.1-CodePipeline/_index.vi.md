---
title : "AWS CodePipeline"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.7.1. </b> "
---

Đây là quy trình từng bước đứng sau pipeline đang chạy `periodiq-pipeline-dev` - không phải bản thay thế rút gọn. Vì toàn bộ được định nghĩa dưới dạng một CloudFormation template duy nhất (`cicd-pipeline.yml`), "dựng" nó không phải là hàng chục lệnh `create-*` riêng lẻ - mà là vài bước chuẩn bị rồi **một** lệnh deploy duy nhất tạo ra mọi thứ cùng lúc.

#### Bước 1 - Tạo CodeStar (CodeConnections) connection tới GitHub

Trước khi pipeline có thể lấy source từ `PeriolIQ/PeriodIQ`, phải có một connection giữa AWS và GitHub:

```bash
aws codeconnections create-connection \
  --provider-type GitHub \
  --connection-name periodiq-github
```

Lệnh này trả về connection ở trạng thái `PENDING` - **bước duy nhất trong cả pipeline này không thể làm chỉ bằng CLI**: AWS yêu cầu một cú click thủ công một lần ("Update pending connection") trên console để xác thực connection với tài khoản/tổ chức GitHub của bạn. Sau khi xác thực xong:

```bash
aws codeconnections list-connections
```

![Connection CodeStar (CodeConnections) tới GitHub - AVAILABLE](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.1-CodePipeline/02-real-codestar-connection.png)

Connection `periodiq-github` đang ở trạng thái `AVAILABLE` - đã được xác thực và đang được dùng.

#### Bước 2 - Viết pipeline dưới dạng CloudFormation template

`cicd-pipeline.yml` trong repo định nghĩa mọi thứ pipeline cần dưới dạng infrastructure-as-code: 2 IAM role, 3 CodeBuild project, 1 S3 bucket artifact, và chính resource `AWS::CodePipeline::Pipeline` với 4 stage của nó (`Source` → `BuildBackend` → `Deploy` → `BuildFrontend`, chi tiết ở [5.7.2](../5.7.2-codebuild/) và [5.7.3](../5.7.3-cloudformation-sam/)). Nó nhận 4 tham số: `GitHubRepo`, `GitHubBranch`, `GitHubConnectionArn` (từ Bước 1), và `Environment`.

#### Bước 3 - Deploy template (một lệnh này dựng mọi thứ)

```bash
aws cloudformation deploy \
  --template-file cicd-pipeline.yml \
  --stack-name periodiq-cicd-pipeline \
  --parameter-overrides \
      GitHubConnectionArn=arn:aws:codeconnections:ap-southeast-1:345798130457:connection/b842ea5b-c94d-4ffd-8e12-088dbe9c0128 \
      GitHubRepo=PeriolIQ/PeriodIQ \
      GitHubBranch=main \
      Environment=dev \
  --capabilities CAPABILITY_NAMED_IAM
```

> **CLI Note:** `--capabilities CAPABILITY_NAMED_IAM` là bắt buộc vì template tạo IAM role với `RoleName` tường minh (`periodiq-codepipeline-role-dev`, `periodiq-codebuild-role-dev`) thay vì để CloudFormation tự sinh tên - CloudFormation luôn yêu cầu xác nhận tường minh trước khi được phép tạo identity IAM có tên cố định.

#### Bước 4 - Xác nhận lệnh đó đã dựng ra những gì

```bash
aws cloudformation list-stack-resources --stack-name periodiq-cicd-pipeline
```

![Resource của stack - 2 IAM role, 3 CodeBuild project, 1 S3 bucket, 1 CodePipeline - tất cả từ một lần deploy duy nhất](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.1-CodePipeline/03-real-cicd-stack-resources.png)

Một lệnh `deploy` đã tạo ra đủ 7 resource: `CodePipelineServiceRole`, `CodeBuildServiceRole`, `BackendBuildProject`, `SamDeployProject`, `FrontendBuildProject`, `PipelineArtifactBucket`, và chính `Pipeline` (tên vật lý `periodiq-pipeline-dev`).

#### Bước 5 - Từ đây pipeline tự chạy

Khi stack đã tồn tại, pipeline tự động kích hoạt mỗi khi có push lên `main` - không cần thêm bước thủ công nào nữa. Kiểm tra trạng thái sau một lần kích hoạt (một pull request đã merge, `#49 từ PeriolIQ/hoang`):

```bash
aws codepipeline get-pipeline-state --name periodiq-pipeline-dev
```

![Trạng thái pipeline - cả 4 stage (Source, BuildBackend, Deploy, BuildFrontend) đều Succeeded](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.1-CodePipeline/01-real-pipeline-4-stages.png)

| Stage | Action | Provider | Làm gì |
|---|---|---|---|
| `Source` | `GitHubSource` | CodeStarSourceConnection | Lấy branch `main` qua connection ở Bước 1 |
| `BuildBackend` | `BuildDotNet` | CodeBuild (`periodiq-backend-build-dev`) | Restore, build, unit-test backend .NET - [5.7.2](../5.7.2-codebuild/) |
| `Deploy` | `SamDeploy` | CodeBuild (`periodiq-sam-deploy-dev`) | Deploy stack WAF + stack SAM chính, chạy health check - [5.7.3](../5.7.3-cloudformation-sam/) |
| `BuildFrontend` | `BuildReact` | CodeBuild (`periodiq-frontend-build-dev`) | Build frontend React/Vite, sync lên S3, invalidate CloudFront - [5.7.2](../5.7.2-codebuild/) |

`Deploy` cố tình chạy **trước** `BuildFrontend`: SAM phải tạo S3 bucket cho frontend và xuất các output Cognito/API/CloudFront trước, vì `BuildFrontend` đọc các output đó lúc build để bake vào bundle Vite.

> **CLI Note:** mọi thứ ở trên - connection, deploy template, danh sách resource, trạng thái pipeline - đều là lệnh `aws` CLI gọi trực tiếp trên deployment đang chạy. Stack pipeline (`periodiq-cicd-pipeline`) đã được deploy đúng một lần theo cách này, và chạy không cần can thiệp từ đó tới nay.

#### Cùng execution đó, trên AWS Console

Output CLI ở trên và AWS Console khớp nhau từng chi tiết, vì cả hai đều chỉ đang đọc cùng một trạng thái pipeline:

![Console AWS CodePipeline - periodiq-pipeline-dev, cả 4 stage đều xanh, cùng execution ID với output CLI ở trên](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.1-CodePipeline/07-aws-console-pipeline-view.png)

Cùng execution ID (`69fa059f-9811-44ed-9bbd-70732e939a69`), cùng commit kích hoạt (`85e4521c`, PR #49 từ `hoang`), cùng 4 stage (`Source` → `BuildBackend` → `Deploy` → `BuildFrontend`) mỗi stage ứng với đúng action bên dưới (`GitHubSource`, `BuildDotNet`, `SamDeploy`, `BuildReact`) - đây là giao diện console của đúng pipeline mà các lệnh CLI trên trang này đã truy vấn.

#### Cùng execution đó, qua dashboard admin tôi đã xây dựng

Trang "Lịch sử Deploy" trong admin panel là một phần khác của hạng mục CI/CD & Monitoring - nó gọi API CodePipeline/CodeBuild từ backend và hiển thị lại lịch sử execution đó dưới dạng UI thay vì output CLI thô:

![Danh sách lịch sử deploy admin - commit, tỷ lệ thành công, thời lượng](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.1-CodePipeline/04-admin-ui-deploy-history-list.png)

Danh sách này liệt kê các execution pipeline theo commit - bao gồm nhiều lần từ chính branch `Sy-2` của tôi (`#48`, `#47`, `#46`, `#45`, `#44`) cùng với PR của đồng đội (`#49` từ `hoang`, `#43` từ `Tai`, `#42` từ `Huan`). Tỷ lệ thành công hiển thị (70%, 14/20) và thời lượng trung bình (6m 53s) đều được tính từ chính lịch sử này - và các lần fail được hiển thị trung thực chứ không giấu đi.

Bấm vào execution `#49` (đúng execution đã xem trong ảnh CLI ở trên) sẽ vào chi tiết từng stage:

![Chi tiết deploy admin - log build trực tiếp của stage BuildBackend](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.1-CodePipeline/05-admin-ui-deploy-detail-log.png)

![Chi tiết deploy admin - cả 4 stage kèm thời lượng](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.1-CodePipeline/06-admin-ui-deploy-detail-stages.png)

Thời lượng từng stage ở đây - `Source` 3s, `BuildBackend` 2m 4s, `Deploy` 4m 7s, `BuildFrontend` 1m 33s - khớp chính xác với những gì `aws codepipeline get-pipeline-state` báo cáo cho execution này, và bấm vào bất kỳ stage nào sẽ mở rộng log CodeBuild ngay tại chỗ (lấy qua `aws logs get-log-events`, cùng lệnh đã dùng ở trên trang này và ở [5.7.2](../5.7.2-codebuild/)).
