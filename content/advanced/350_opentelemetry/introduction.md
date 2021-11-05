---
title: "Introduction"
date: 2018-11-13T16:36:24+09:00
weight: 10
draft: false
---

[AWS Distro for OpenTelemetry](https://aws.amazon.com/otel/) is a secure, production-ready,
AWS-supported distribution of the OpenTelemetry project. Part of the Cloud Native Computing Foundation,
OpenTelemetry provides open source APIs, libraries, and agents to collect distributed traces and metrics
for application monitoring. 


Benefits of using ADOT:
* AWS Distro for OpenTelemetry empowers you to implement broad yet efficient, secure yet flexible,
  observability solutions
* It helps you optimize your production environments by ensuring predictable resource utilization, and can
  increase your analytical visibility while protecting your investment in standardized observability tools
* It is backed by AWS Support, testing, and certification.


The following diagram shows how we'll be configuring our OpenTelemetry Collector in this module:

![Observability with AWS Distro for Open Telemetry Workshop Architecture](/images/observability-with-adot/overview-architecture-diagram.png)

## Demo Microservice infrastructure

The microservice we are going to deploy looks like this one:

![Architecture Diagram of Microservices](/images/observability-with-adot/microservice.png)

