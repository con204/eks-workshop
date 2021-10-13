---
title: "Tracing"
date: 2021-5-01T09:00:00-00:00
weight: 50
draft: false
---

The microservice demo we have installed is configured to generate traces and send them to Jaeger. The traces are 
generated using the OpenTelemetry specification and use X-Ray compatible IDs and propagators. This allows for
using both Jaegor and X-Ray for storing and visualizing the trace across services.


Let's go setup our Open Telemetry collector to recieve these traces, batch the traces into bulk requests, and send the traces to AWS X-Ray.

![X-Ray product page diagram](/images/observability-with-adot/product-page-diagram_xray.png)


