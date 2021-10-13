---
title: "Vizualizing with Grafana"
date: 2021-5-01T09:00:00-00:00
weight: 64
draft: false
---

Now that Grafana is setup to read metrics from Prometheus, let's create a dashboard on our Kubernetes Cluster's
health.


#### End to End test AMP & Grafana

In the Grafana web-console, let's head to Explore
![grafana explore](/images/observability-with-adot/grafana-explore.png)

Let's run a test [PromQL query](https://prometheus.io/docs/prometheus/latest/querying/basics/) against our AMP
workspace. In the query bar type in `etcd_object_counts`, then click the Run Query button on the top right.
If this query returns data, we can confirm that we are 1) Ingesting Kubernetes Control Plane metrics to our AMP
workspace. 2) Grafana is able to connect to our AMP workspace.

![Grafana AMP Query](/images/observability-with-adot/grafana-query-amp.png)


#### Creating Kubernetes Grafana Dashboard

In the Nav select **Create**, and choose **Import**.
![Grafana Create Import](/images/observability-with-adot/grafana-create-import.png)

Enter `12006` in the field for the *Grafana.com dashboard ID*, and then click the Load button
![Grafana load Dashboard](/images/observability-with-adot/grafana-load-dashboard.png)

Click on Import
![Grafana import dashboard](/images/observability-with-adot/grafana-import-dashboard.png)


Now we can see a pre-made dashboard outlining our Kubernetes Control Plane metrics
![Grafana kubernetes dashboard](/images/observability-with-adot/grafana-k8s-dashboard.png)


Congratulations! You've now used the AWS Distro for OpenTelemetry to
1) receive metrics from Container Insights
2) receive traces from the deployed microservices
3) store the telemetry data using CloudWatch, X-Ray, and AWS Managed Service for Prometheus
4) Visualize using X-Ray, CloudWatch, and Grafana.

