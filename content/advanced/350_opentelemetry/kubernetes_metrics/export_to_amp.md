---
title: "Export to AMP"
date: 2021-5-01T09:00:00-00:00
weight: 62
draft: false
---

In previous section we learned how to receive metrics from the Kubernetes Control Plane and export them to
CloudWatch using the OpenTelemetry collector. In this section we are going to demonstrate a core benefit of the
OpenTelemetry Collector pattern, by configuring it to send the Kubernetes Control Plane metrics to an additional
destination, AWS Managed Service for Prometheus (AMP).

Let's create an AMP workspace:
```bash:
aws amp create-workspace --alias eks-workshop --region $AWS_REGION
```

#### IAM Policy & IAM Role

Now we create a new IAM Policy (`AWSManagedPrometheusWriteAccessPolicy`) which grants AMP read and write permission:
```bash
read -r -d '' PERMISSION_POLICY <<EOF
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Effect":"Allow",
         "Action":[
            "aps:RemoteWrite",
            "aps:QueryMetrics",
            "aps:GetSeries",
            "aps:GetLabels",
            "aps:GetMetricMetadata"
         ],
         "Resource":"*"
      }
   ]
}
EOF

echo "${PERMISSION_POLICY}" > AMPPolicy.json
export SERVICE_ACCOUNT_IAM_POLICY_ARN="arn:aws:iam::${ACCOUNT_ID}:policy/AWSManagedPrometheusWriteAccessPolicy"
aws iam create-policy --policy-name AWSManagedPrometheusWriteAccessPolicy --policy-document file://AMPPolicy.json
```

Then the `AWSManagedPrometheusWriteAccessPolicy` IAM Policy is attached to the IAM Role (`AWSDistroOpenTelemetryRole`)
we previously created for our OpenTelemetry collector.
```bash
aws iam attach-role-policy --role-name AWSDistroOpenTelemetryRole --policy-arn=${SERVICE_ACCOUNT_IAM_POLICY_ARN}
```

#### Exporter

Open `kubernetes/adot/otel-prometheus.yaml`, and view lines 63-67. This describes an exporter which sends metrics to an AMP workspace.
{{< output >}} exporters:
      #awsprometheusremotewrite:
      #  endpoint: https://aps-workspaces.eu-central-1.amazonaws.com/workspaces/${WORKSPACE_ID}/api/v1/remote_write
      #  aws_auth:
      #    service: "aps"
      #    region: ${AWS_REGION}
{{< /output >}}

Let's un-comment lines 63-67 with the following command:
```bash
sed -i '63,67 s/#//g' kubernetes/adot/otel-prometheus.yaml
```


### Pipeline

Now in `kubernetes/adot/otel-prometheus.yaml` let's view lines 116-119. This outlines a new pipeline, which
takes in the same prometheus metrics we send to CloudWatch, processes the metrics into batched segments, then exports
the metrics to our AMP workspace. 
{{< output >}}   service:
      pipelines:
        metrics:
          receivers: [prometheus]
          processors: [resourcedetection, resource]
          exporters: [awsemf]
        #metrics/prom:
        #  receivers: [prometheus]
        #  processors: [batch]
        #  exporters: [awsprometheusremotewrite]
{{< /output >}}

Let's un-comment lines 116-119 with the following command:
```bash
sed -i '116,119 s/#//g' kubernetes/adot/otel-prometheus.yaml
```

#### Updating Prometheus Metrics Collector

Now that the Prometheus Metrics Collector's configuration was updated to send metrics to Prometheus, let's update our deployment.

First, let's grab our AMP Workspace ID:
```bash
export WORKSPACE_ID=$(aws amp --region=$AWS_REGION list-workspaces --alias eks-workshop | jq .workspaces[0].workspaceId -r)
echo "AMP Workspace ID: ${WORKSPACE_ID}"
```

Now we'll update the manifest with our environment variables, deploy the new manifest, and like before force-recreate the collector to speed up the rollout.
```bash
envsubst < kubernetes/adot/otel-prometheus.yaml | sponge kubernetes/adot/otel-prometheus.yaml
kubectl apply -f kubernetes/adot/otel-prometheus.yaml
kubectl delete pod -n aws-otel-eks -l name=aws-otel-eks-prometheus
```

Now our OpenTelemetry Collector is sending Kubernetes Control Plane metrics to both CloudWatch and Prometheus.
