---
title : "AWS CodePipeline"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.7.1. </b> "
---

This is the actual step-by-step process behind the real, running pipeline `periodiq-pipeline-dev` - not a simplified stand-in. Because the whole thing is defined as one CloudFormation template (`cicd-pipeline.yml`), "building" it isn't dozens of separate `create-*` calls - it's a handful of setup steps followed by **one** deploy command that provisions everything at once.

#### Step 1 - Create the CodeStar (CodeConnections) connection to GitHub

Before the pipeline can pull source from `PeriolIQ/PeriodIQ`, a connection between AWS and GitHub has to exist:

```bash
aws codeconnections create-connection \
  --provider-type GitHub \
  --connection-name periodiq-github
```

This returns a connection in `PENDING` state - the **one step in this whole pipeline that genuinely cannot be done via CLI alone**: AWS requires a one-time manual click ("Update pending connection") in the console to authorize the connection against your GitHub account/org. Once authorized:

```bash
aws codeconnections list-connections
```

![Real CodeStar (CodeConnections) connection to GitHub - AVAILABLE](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.1-CodePipeline/02-real-codestar-connection.png)

The real connection, `periodiq-github`, shows `AVAILABLE` - it's already authorized and in use.

#### Step 2 - Write the pipeline as a CloudFormation template

`cicd-pipeline.yml` in the repo defines everything the pipeline needs as infrastructure-as-code: 2 IAM roles, 3 CodeBuild projects, 1 S3 artifact bucket, and the `AWS::CodePipeline::Pipeline` resource itself with its 4 stages (`Source` → `BuildBackend` → `Deploy` → `BuildFrontend`, detailed in [5.7.2](../5.7.2-codebuild/) and [5.7.3](../5.7.3-cloudformation-sam/)). It takes 4 parameters: `GitHubRepo`, `GitHubBranch`, `GitHubConnectionArn` (from Step 1), and `Environment`.

#### Step 3 - Deploy the template (this one command builds everything)

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

> **CLI Note:** `--capabilities CAPABILITY_NAMED_IAM` is required because the template creates IAM roles with explicit `RoleName` values (`periodiq-codepipeline-role-dev`, `periodiq-codebuild-role-dev`) rather than letting CloudFormation generate names - CloudFormation always asks for explicit acknowledgment before it's allowed to create named IAM identities.

#### Step 4 - Confirm what that one command actually built

```bash
aws cloudformation list-stack-resources --stack-name periodiq-cicd-pipeline
```

![Real stack resources - 2 IAM roles, 3 CodeBuild projects, 1 S3 bucket, 1 CodePipeline - all from a single deploy](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.1-CodePipeline/03-real-cicd-stack-resources.png)

One `deploy` call created all 7 resources: `CodePipelineServiceRole`, `CodeBuildServiceRole`, `BackendBuildProject`, `SamDeployProject`, `FrontendBuildProject`, `PipelineArtifactBucket`, and the `Pipeline` itself (physical name `periodiq-pipeline-dev`).

#### Step 5 - The pipeline runs itself from here

Once the stack exists, the pipeline triggers automatically on every push to `main` - no further manual steps. Checking its state after a real trigger (a merged pull request, `#49 from PeriolIQ/hoang`):

```bash
aws codepipeline get-pipeline-state --name periodiq-pipeline-dev
```

![Real pipeline state - all 4 stages (Source, BuildBackend, Deploy, BuildFrontend) Succeeded](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.1-CodePipeline/01-real-pipeline-4-stages.png)

| Stage | Action | Provider | What it does |
|---|---|---|---|
| `Source` | `GitHubSource` | CodeStarSourceConnection | Pulls `main` via the connection from Step 1 |
| `BuildBackend` | `BuildDotNet` | CodeBuild (`periodiq-backend-build-dev`) | Restores, builds, unit-tests the .NET backend - [5.7.2](../5.7.2-codebuild/) |
| `Deploy` | `SamDeploy` | CodeBuild (`periodiq-sam-deploy-dev`) | Deploys the WAF stack + main SAM stack, runs a health check - [5.7.3](../5.7.3-cloudformation-sam/) |
| `BuildFrontend` | `BuildReact` | CodeBuild (`periodiq-frontend-build-dev`) | Builds the React/Vite frontend, syncs to S3, invalidates CloudFront - [5.7.2](../5.7.2-codebuild/) |

`Deploy` runs **before** `BuildFrontend` on purpose: SAM has to create the frontend S3 bucket and export the Cognito/API/CloudFront outputs first, since `BuildFrontend` reads those outputs at build time to bake them into the Vite bundle.

> **CLI Note:** everything above - the connection, the template deploy, the resource list, the pipeline state - is a genuine `aws` CLI call against the real, already-running deployment. The pipeline stack (`periodiq-cicd-pipeline`) was deployed once, this way, and has been running unattended ever since.

#### The same execution, in the AWS Console

The CLI output above and the AWS Console agree pixel-for-pixel, since they're both just reading the same pipeline state:

![AWS CodePipeline console - periodiq-pipeline-dev, all 4 stages green, same execution ID as the CLI output above](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.1-CodePipeline/07-aws-console-pipeline-view.png)

Same execution ID (`69fa059f-9811-44ed-9bbd-70732e939a69`), same triggering commit (`85e4521c`, PR #49 from `hoang`), same 4 stages (`Source` → `BuildBackend` → `Deploy` → `BuildFrontend`) each mapped to its underlying action (`GitHubSource`, `BuildDotNet`, `SamDeploy`, `BuildReact`) - this is the console view of the exact same pipeline the CLI commands on this page have been querying.

#### The same execution, through the admin dashboard I built

The "Lịch sử Deploy" (Deploy History) page in the admin panel is another piece of the CI/CD & Monitoring role - it calls the CodePipeline/CodeBuild APIs from the backend and renders the same execution history as a UI instead of raw CLI output:

![Admin deploy history list - real commits, success rate, durations](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.1-CodePipeline/04-admin-ui-deploy-history-list.png)

This lists real pipeline executions by commit - including several from my own `Sy-2` branch merges (`#48`, `#47`, `#46`, `#45`, `#44`) alongside teammates' PRs (`#49` from `hoang`, `#43` from `Tai`, `#42` from `Huan`). The overall success rate shown (70%, 14/20) and average duration (6m 53s) are computed from this same real execution history - and the failed runs are shown honestly rather than hidden.

Clicking into the `#49` execution (the same one shown in the CLI screenshot above) drills into per-stage detail:

![Admin deploy detail - live build log for the BuildBackend stage](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.1-CodePipeline/05-admin-ui-deploy-detail-log.png)

![Admin deploy detail - all 4 stages with their real durations](/images/5-Workshop/5.7-Nguoi5-CICD/5.7.1-CodePipeline/06-admin-ui-deploy-detail-stages.png)

The per-stage durations here - `Source` 3s, `BuildBackend` 2m 4s, `Deploy` 4m 7s, `BuildFrontend` 1m 33s - match exactly what `aws codepipeline get-pipeline-state` reports for this same execution, and clicking any stage expands its live CodeBuild log in place (pulled via `aws logs get-log-events`, the same call used earlier on this page and in [5.7.2](../5.7.2-codebuild/)).
