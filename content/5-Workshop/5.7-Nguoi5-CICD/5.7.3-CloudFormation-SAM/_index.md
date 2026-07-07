---
title : "CloudFormation / SAM"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.7.3. </b> "
---

The `Deploy` stage's CodeBuild project, `periodiq-sam-deploy-dev`, is where `sam build`/`sam deploy` actually run - unlike the other two CodeBuild projects, its buildspec is written **inline inside `cicd-pipeline.yml`** itself rather than as a separate file in the repo, since it's infrastructure-deployment logic that belongs with the pipeline definition.

#### What this stage actually does, in order

**1. Install phase** - installs the .NET 10 SDK, `Amazon.Lambda.Tools`, and the AWS SAM CLI fresh in every build:
```bash
/tmp/dotnet-install.sh --channel 10.0 --install-dir /root/dotnet10
/root/dotnet10/dotnet tool install --tool-path /root/dotnet10/tools Amazon.Lambda.Tools
pip install aws-sam-cli
```

**2. Deploy the WAF stack, in `us-east-1`** - CloudFront-scope WAF WebACLs are a global/`us-east-1` requirement, so this has to be a separate stack from the main one (which runs in `ap-southeast-1`):
```bash
sam deploy -t waf-cloudfront.yml --stack-name periodiq-waf-cloudfront-dev \
  --region us-east-1 --resolve-s3 --capabilities CAPABILITY_IAM \
  --no-confirm-changeset --no-fail-on-empty-changeset \
  --parameter-overrides Environment=dev
```

**3. Fetch the WAF ARN** from that stack's output, then **`sam build` + `sam deploy` the main stack** in `ap-southeast-1`, passing the WAF ARN in as a parameter so the main template can attach it to CloudFront:
```bash
WAF_ARN=$(aws cloudformation describe-stacks --stack-name periodiq-waf-cloudfront-dev --region us-east-1 \
  --query "Stacks[0].Outputs[?OutputKey=='WebAclArn'].OutputValue" --output text)

sam build
sam deploy --no-confirm-changeset --no-fail-on-empty-changeset \
  --parameter-overrides Environment=dev AlertEmail=syvan.claude@outlook.com WafWebAclArn=$WAF_ARN
```

**4. Post-build health check** - hits the freshly-deployed API's `/api/health` endpoint and warns (without failing the build) if it doesn't respond:
```bash
API_URL=$(aws cloudformation describe-stacks --stack-name periodiq-dev \
  --query "Stacks[0].Outputs[?OutputKey=='ApiUrl'].OutputValue" --output text)
curl -f "$API_URL/api/health" || echo "CANH BAO: Health check THAT BAI"
```

#### Reading the real log from the last real deployment

```bash
aws logs get-log-events --log-group-name /aws/codebuild/periodiq-sam-deploy-dev \
  --log-stream-name d6895bf1-a210-4e4c-b7d7-82ed49b5ba8b
```

![Real SAM deploy log - WAF stack up to date, main stack created/updated, health check passed](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.3-CloudFormation-SAM/01-real-sam-deploy-log.png)

The WAF stack reported "No changes to deploy" (it was already correct from a previous run), the main stack was created/updated successfully, and the health check returned a genuine healthy response:
```json
{"status":"healthy","timestamp":"2026-07-06T08:12:58.9320196Z","version":"1.0.0.0","environment":"dev","commit":"local","buildNumber":"0"}
```

#### Checking the deployed stack directly

```bash
aws cloudformation describe-stacks --stack-name periodiq-dev --query "Stacks[0].[StackStatus,Outputs]"
```

![Real periodiq-dev stack - UPDATE_COMPLETE, with all real outputs](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.3-CloudFormation-SAM/02-real-stack-outputs.png)

This confirms the stack is `UPDATE_COMPLETE` and exposes exactly the resources described in the [Proposal](../../../2-proposal/): Cognito User Pool (`ap-southeast-1_3l2EERPJk`), the WAF WebACL ARN attached to CloudFront, the SQS queue, the SNS alerts topic, and - matching the live site - **`CloudFrontDomainName: d1di1pzmfypszp.cloudfront.net`**.

> **CLI Note:** this is the same live CloudFront domain referenced throughout the rest of this report - there is exactly one `periodiq-dev` stack, and it's the one actually serving the public site.
