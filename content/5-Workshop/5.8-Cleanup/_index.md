---
title : "Clean up"
date : 2024-01-01
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
---

This workshop is different from a typical "spin up sandbox resources, then tear them down" lab: sections [5.7.1](../5.7-nguoi5-cicd/5.7.1-codepipeline/) through [5.7.4](../5.7-nguoi5-cicd/5.7.4-cloudwatch/) document the **real, already-running production pipeline and stack** (`periodiq-pipeline-dev`, `periodiq-dev`) - not a throwaway copy. There is nothing to clean up there, because that infrastructure needs to keep running: it's the live system serving **https://d1di1pzmfypszp.cloudfront.net/**.

> **Nothing about the real pipeline, the real `periodiq-dev`/`periodiq-cicd-pipeline` stacks, or any of the real CodeBuild projects/roles/buckets was deleted, disabled, or modified while writing this workshop.** Every command against them, throughout this whole section, was a read-only `describe-*` / `get-*` / `list-*` call.

#### What actually got cleaned up

While putting this workshop together, I did build and tear down a separate, clearly-namespaced set of scratch resources (prefixed `periodiq-workshop-*`, in the same account) to test how CodePipeline/CodeBuild/CloudFormation/CloudWatch fit together via the CLI, before settling on documenting the real pipeline directly. Those scratch resources - a pipeline, two CodeBuild projects, three IAM roles, an S3 bucket, a small test stack, and a log group - have all been deleted for real:

```bash
aws cloudformation list-stacks --query "StackSummaries[?starts_with(StackName,'periodiq-workshop')]"
aws codebuild list-projects --query "projects[?starts_with(@,'periodiq-workshop')]"
aws s3api list-buckets --query "Buckets[?starts_with(Name,'periodiq-workshop')]"
aws iam list-roles --query "Roles[?starts_with(RoleName,'periodiq-workshop')]"

# and confirming the real system is untouched and still healthy:
aws cloudformation describe-stacks --stack-name periodiq-dev --query "Stacks[0].StackStatus"
curl -s -o /dev/null -w "%{http_code}" https://d1di1pzmfypszp.cloudfront.net/
```

![All periodiq-workshop-* scratch resources gone; periodiq-dev still UPDATE_COMPLETE and the live site still returns 200](/images/5-Workshop/5.8-Cleanup/01-workshop-scratch-cleaned-prod-untouched.png)

All four scratch-resource checks return empty lists, and the real stack + live site are unaffected: `periodiq-dev` is still `UPDATE_COMPLETE`, and the public site still responds `200`.
