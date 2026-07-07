---
title : "AWS CodeBuild"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.7.2. </b> "
---

The real pipeline uses **three** CodeBuild projects (all created by `cicd-pipeline.yml`, all running on `aws/codebuild/amazonlinux2-x86_64-standard:5.0`, all under the shared `periodiq-codebuild-role-dev`). This page covers `periodiq-backend-build-dev` and `periodiq-frontend-build-dev`; the third, `periodiq-sam-deploy-dev`, is covered in [5.7.3](../5.7.3-cloudformation-sam/) since it's entirely about deployment.

#### `periodiq-backend-build-dev` - `buildspec-backend.yml`

This project builds the .NET 10 backend. Its `install` phase pins a specific .NET 10 SDK (CodeBuild's own `global.json` gets deleted first so it doesn't conflict), then `pre_build`/`build` restore, build, **unit-test**, and `dotnet publish` the API project:

```yaml
build:
  commands:
    - $DOTNET_ROOT/dotnet build PeriodIQ.Backend/PeriodIQ.sln --configuration Release --no-restore
    - $DOTNET_ROOT/dotnet test PeriodIQ.Backend/PeriodIQ.sln --configuration Release --no-build --logger trx --results-directory ./test-results --collect "XPlat Code Coverage"
    - $DOTNET_ROOT/dotnet publish PeriodIQ.Backend/PeriodIQ.Api/PeriodIQ.Api.csproj --configuration Release --output ./publish --self-contained true --runtime linux-x64
```

Test results are reported back to CodeBuild as a `VisualStudioTrx` report (`periodiq-test-report`), and NuGet packages are cached under `/root/.nuget/packages` between builds.

Reading the real log from the pipeline's last real execution:

```bash
aws codebuild batch-get-builds --ids periodiq-backend-build-dev:bebd4b0b-8441-490d-a0bd-033832b02936 \
  --query "builds[0].logs.[groupName,streamName]"
aws logs get-log-events --log-group-name /aws/codebuild/periodiq-backend-build-dev \
  --log-stream-name bebd4b0b-8441-490d-a0bd-033832b02936
```

![Real backend build log - dotnet build succeeded, 49/49 tests passed](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.2-CodeBuild/01-real-backend-build-tests.png)

**49 of 49 unit tests passed, 0 failed** - this is the actual `PeriodIQ.Api.Tests.dll` test run from the real merge of PR #49.

#### `periodiq-frontend-build-dev` - `buildspec-frontend.yml`

This project builds the React 19 + Vite frontend with `pnpm`, but the interesting part is *how* it gets its runtime configuration. Rather than hardcoding Cognito IDs or an API URL, the `build` phase pulls them live from the already-deployed `periodiq-dev` stack (deployed one stage earlier, by `Deploy`) and exports them as `VITE_*` environment variables **in the same shell invocation** as `pnpm run build`, since Vite only bakes in `VITE_*` variables that are present at build time:

```bash
export VITE_COGNITO_USER_POOL_ID=$(aws cloudformation describe-stacks --stack-name "$STACK_NAME" \
  --query "Stacks[0].Outputs[?OutputKey=='CognitoUserPoolId'].OutputValue" --output text)
export VITE_COGNITO_CLIENT_ID=$(aws cloudformation describe-stacks --stack-name "$STACK_NAME" \
  --query "Stacks[0].Outputs[?OutputKey=='CognitoUserPoolClientId'].OutputValue" --output text)
CF_DOMAIN=$(aws cloudformation describe-stacks --stack-name "$STACK_NAME" \
  --query "Stacks[0].Outputs[?OutputKey=='CloudFrontDomainName'].OutputValue" --output text)
export VITE_API_URL="https://$CF_DOMAIN"
# if any of these are empty, the build fails on purpose rather than shipping an empty bundle
pnpm run build
```

The `post_build` phase then syncs `dist/` to the real frontend bucket and invalidates CloudFront so the new build is served immediately instead of a stale cached `index.html`:

```bash
aws s3 sync dist/ s3://${FRONTEND_BUCKET_NAME}/ --delete --cache-control "max-age=86400"
aws s3 cp dist/index.html s3://${FRONTEND_BUCKET_NAME}/index.html --cache-control "no-cache, no-store, must-revalidate"
DIST_ID=$(aws cloudfront list-distributions --query "DistributionList.Items[?DomainName=='$CF_DOMAIN'].Id | [0]" --output text)
aws cloudfront create-invalidation --distribution-id "$DIST_ID" --paths "/*"
```

Reading the real log from the same pipeline execution:

```bash
aws logs get-log-events --log-group-name /aws/codebuild/periodiq-frontend-build-dev \
  --log-stream-name 1de32fc1-9e21-4155-94d6-85b78f931872
```

![Real frontend build log - build succeeded, synced to S3, real CloudFront invalidation created](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.2-CodeBuild/02-real-frontend-build-deploy.png)

The log shows a genuine CloudFront invalidation (`I8NE2A60VGS4W13MR9G59MOHY6`) created against the real distribution (`E2LYL2VAX7BYMQ`) that serves `d1di1pzmfypszp.cloudfront.net`.

> **CLI Note:** every command on this page reads from real, already-completed CodeBuild build logs (`aws codebuild batch-get-builds` + `aws logs get-log-events`) - nothing here was re-run or simulated for the workshop.
