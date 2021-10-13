---
title: "Adding trace Configuration"
date: 2021-5-01T09:00:00-00:00
weight: 51
draft: false
---

Let’s update our Collector’s Manifest to collect traces and export them to x-ray. This involves un-commenting a
few sections in `kubernetes/adot/otel-container-insights-infra.yaml`. The below sections will walk through the commented out sections and provide CLI commands to un-comment them out.

#### Thrift Service

First let’s head to the end of the manifest and un-comment the Thrift service. This stands up an API endpoint to
recieve thrift traffic over the Thrift Binary protocol on port 6832, HTTP-TCP on TCP port 14268,
and gRPC on TCP 14250. Thrift supports all three of these protocols.

{{< output >}}#---
#apiVersion: v1
#kind: Service
#metadata:
#  name: aws-otel-eks
#  namespace: aws-otel-eks
#  labels:
#    name: aws-otel-eks
#spec:
#  ports:
#    - name: thrift-binary
#      port: 6832
#      protocol: UDP
#    - name: thrift-http
#      port: 14268
#    - name: grpc
#      port: 14250
#  selector:
#    name: aws-otel-eks-ci
{{< /output >}}

Uncomment these lines by running the following command:
```bash
sed -i '275,296 s/#//g' kubernetes/adot/otel-container-insights-infra.yaml
```


#### Jaeger receiver

Now let's head to lines 77 through 81 of the manifest. This defines how the Collector should receive Jaeger data.
Notice how we configure it with both the binary, http, and gRPC protocols, which our Thrift Service was also
configured to support.

{{< output >}}      #jaeger:
      #  protocols:
      #    thrift_binary:
      #    thrift_http:
      #    grpc:
{{< /output >}}

Uncomment these lines by running the following command:
```bash
sed -i '77,81 s/#//g' kubernetes/adot/otel-container-insights-infra.yaml
```


#### AWS X-Ray exporter

Head to lines 159 & 160 in the manifest. This configures how to export data to AWS X-Ray, with us specifying the
AWS Region to send data to.

{{< output >}}      #awsxray:
      #  region: ${AWS_REGION}
{{< /output >}}

Uncomment these lines by running the following command:
```bash
sed -i '159,160 s/#//g' kubernetes/adot/otel-container-insights-infra.yaml
```

#### Trace Pipeline

Now that we configured how to recieve traces, and how to export them to AWS X-Ray. Let's tell the Collector to
create a pipeline connecting these two components together.

Head to lines 168 through 171. We defined a Pipeline named `traces/applications` that recives data from the
Jaeger receiver we configured, and sends it to the AWS X-Ray exporter we configured.

{{< output >}}      #traces/applications:
        #  receivers: [jaeger]
        #  processors: []
        #  exporters: [awsxray]
{{< /output >}}

Uncomment these lines out by running the following command:
```bash
sed -i '168,171 s/#//g' kubernetes/adot/otel-container-insights-infra.yaml
```

#### Updating the Collector

We defined a Jaeger receiver, X-Ray exporter, and a Pipeline to receive and export traces. Let's update our Collector
with this new configuration. Collectors will recurringly detect and update configuration, however to speed things up
we will force Collector pod re-creation.

Let's run the following commands to update our Collector:
```bash
kubectl apply -f kubernetes/adot/otel-container-insights-infra.yaml
kubectl delete pods -n aws-otel-eks -l name=aws-otel-eks-ci
```

