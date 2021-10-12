---
title: "Deploying the Collector"
date: 2021-5-01T09:00:00-00:00
weight: 43
draft: false
---

### Deploying Open Telemetry Collector

Set variables in the OTEL collector's kubernetes manifest file:
```bash
envsubst < kubernetes/adot/otel-container-insights-infra.yaml | sponge kubernetes/adot/otel-container-insights-infra.yaml
```

Install the OTEL collector to the EKS Cluster as a DaemonSet:
```bash
kubectl apply -f kubernetes/adot/otel-container-insights-infra.yaml
```

Check to see that the OTEL collector’s DaemonSet is running on each node:
{{< output >}}$ kubectl get nodes
NAME                                           STATUS   ROLES    AGE    VERSION
ip-192-168-29-122.us-west-2.compute.internal   Ready    <none>   2d6h   v1.21.2-eks-55daa9d
ip-192-168-43-204.us-west-2.compute.internal   Ready    <none>   2d6h   v1.21.2-eks-55daa9d
ip-192-168-70-26.us-west-2.compute.internal    Ready    <none>   2d6h   v1.21.2-eks-55daa9d

$ kubectl get pods -n aws-otel-eks -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP               NODE                                           NOMINATED NODE   READINESS GATES
aws-otel-eks-ci-47x6x   1/1     Running   0          9m26s   192.168.39.13    ip-192-168-43-204.us-west-2.compute.internal   <none>           <none>
aws-otel-eks-ci-hj8vs   1/1     Running   0          9m26s   192.168.13.246   ip-192-168-29-122.us-west-2.compute.internal   <none>           <none>
aws-otel-eks-ci-vhcrs   1/1     Running   0          9m26s   192.168.78.46    ip-192-168-70-26.us-west-2.compute.internal    <none>           <none>
{{< /output >}}


At this point the OTEL collector is installed and sending CloudWatch Insights metrics to CloudWatch.
Open your CloudWatch console, and head to Metrics > All Metrics. And you'll notice :
![CloudWatch screenshot](/images/observability-with-adot/container-insights-cloudwatch-metrics.png)
