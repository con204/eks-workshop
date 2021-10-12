---
title: "OTEL Collector"
date: 2021-5-01T09:00:00-00:00
weight: 40
draft: false
---


In this section we will install [AWS Distro for Open Telemetry](https://aws.amazon.com/otel/),
a secure, production-ready, AWS-supported distribution of the OpenTelemetry project. We will also
be configuring [Container Insights](https://aws.amazon.com/about-aws/whats-new/2021/07/aws-distro-for-opentelemetry-adds-support-for-container-metrics-in-amazon-cloudwatch-container-insights-preview/),
which collects container metrics and analyzes them along with other metrics in Amazon CloudWatch.

First let's get our EKS Cluster ARN:
```bash
export CLUSTER_NAME=$(aws eks list-clusters --output=json | jq '.clusters[0]' -r)
echo -e "EKS Cluster Name: ${CLUSTER_NAME}"
```

Now let's start configuring our IAM permissions

