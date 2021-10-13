---
title: "Kubernetes Metrics"
date: 2021-5-01T09:00:00-00:00
weight: 60
draft: false
---

In this section of the workshop, we'll configure the AWS Distro for OpenTelemetry Collector to scrape metrics from the
[Kubernetes Control Plane `/metric` endpoint](https://kubernetes.io/docs/concepts/cluster-administration/system-metrics/).

This entails configuring a Collector to receive the EKS Control Plane metrics, and an exporter to send the metrics
to AWS Managed Service for Prometheus.

Due to a limitation with the current version of the Prometheus receiver,
we will need to deploy an additional OpenTelemetry Collector as a single replica
[deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).
> "When running multiple replicas of the collector with the same config, it will scrape the targets multiple times."
> [src](https://pkg.go.dev/go.opentelemetry.io/collector/receiver/prometheusreceiver#section-readme)

