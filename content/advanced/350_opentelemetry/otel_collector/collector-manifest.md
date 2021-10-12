---
title: "Collector Manifest"
date: 2021-5-01T09:00:00-00:00
weight: 42
draft: false
---

### Configuring AWS Distro for Open Telemetry Collector Kubernetes manifest

Open `$ vim kubernetes/adot/otel-container-insights-infra.yaml`. Inside you’ll notice an annotation for
the `AWSDistroOpenTelemetryRole` in the `ServiceAccount` resource. 
{{< output >}}---
# create cwagent service account and role binding
apiVersion: v1
kind: ServiceAccount
metadata:
  name: aws-otel-sa
  namespace: aws-otel-eks
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::${ACCOUNT_ID}:role/AWSDistroOpenTelemetryRole
{{< /output >}}

Also take a look at the `ConfigMap` outlining the OTEL `receivers`, `processors`, and `exporters`:
{{< output >}}data:
  otel-agent-config: |
    extensions:
      health_check:

    receivers:
      awscontainerinsightreceiver:

    processors:
      batch/metrics:
        timeout: 60s

    exporters:
      awsemf:
        namespace: ContainerInsights
        ...
{{< /output >}}

Scroll down further in the file, and you'll see a `pipeline`, which grabs data from the Container Insight `receiver`,
[batching](https://github.com/open-telemetry/opentelemetry-collector/blob/main/processor/batchprocessor/README.md),
and exporting to [CloudWatch (Embedded Metric)](https://aws-otel.github.io/docs/getting-started/cloudwatch-metrics#cloudwatch-emf-exporter-awsemf). 

Open Telemetry allows multiple receivers, processors, and exporters in a pipeline. For example if you wanted to send all these collected Container Insight metrics to a second destination it’d be as simple as defining the additional exporter and adding it to the pipeline.

Now Let’s close the file by hitting esc, typing :q! , and hitting enter.
