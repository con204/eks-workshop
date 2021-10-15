---
title: "Deploy Backend Microservices"
date: 2021-10-12T14:34:29.412Z
weight: 30
draft: false
---

## Deploy Backend Microservices

The microservice we are going to deploy looks like this one:

![Architecture Diagram of Microservices](/images/observability-with-adot/microservice.png)

Grab a copy of the microservices:
```bash
git clone https://github.com/con204/adot-demo.git
cd adot-demo
```

Deploy the microservice's databases into your EKS Cluster:
```bash
kubectl apply -f kubernetes/databases/
```

Quickly, check all the database services are deployed and ready to receive requests:
{{< output >}}$ kubectl get pods
NAME                                                 READY   STATUS    RESTARTS   AGE
employee-mongo-67cd95c7d9-jdhfb                      1/1     Running   0          13s
jaeger-tracing-salaryamount-mysql-76cf744c75-t9q85   1/1     Running   0          13s
salary-grade-postgres-database-6df4676bb-j59lp       1/1     Running   0          13s
{{< /output >}}

Deploy the backend services:
```bash
kubectl apply -f kubernetes/backend/
```

In the backend service manifest, we specified an ingress with ELB Load Balancer. Let’s wait until the ELB is fully deployed:
{{< output >}}$ kubectl get svc jaeger-tracing-nodejs-service
NAME                            TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)          AGE
jaeger-tracing-nodejs-service   LoadBalancer   10.100.104.173   a814b34247f984307ac5d078b6e3f1b0-1991136339.us-west-2.elb.amazonaws.com   8081:32521/TCP   13s
{{< /output >}}

{{% notice info %}}
If you do not see an External-IP, re-run the command and wait till it shows (this can take a minute or two)
{{% /notice %}}

