---
title: "Deploy Frontend"
date: 2021-10-12T14:34:29.412Z
weight: 40
draft: false
---

## Deploy Frontend

Now that we deployed our backend microservices, let's go and deploy our frontend stack.

Grab the Backend Service ELB, and deploy our Front End:
```bash
export API_BACKEND=$(kubectl get svc jaeger-tracing-nodejs-service --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
echo -e "API_BACKEND: $API_BACKEND"
envsubst < kubernetes/frontend/jaeger-tracing-frontend-service-deployment.yaml | sponge kubernetes/frontend/jaeger-tracing-frontend-service-deployment.yaml
kubectl apply -f kubernetes/frontend/
```

Now we wait for all the pods to be up and running:
```bash
kubectl get pods
```
{{< output >}} 
NAME                                                 READY   STATUS    RESTARTS   AGE
employee-mongo-67cd95c7d9-26w8x                      1/1     Running   0          29m
jaeger-tracing-frontend-service-5dd45d5bb-rdppb      1/1     Running   0          22m
jaeger-tracing-go-service-95556f866-b67zh            1/1     Running   0          29m
jaeger-tracing-java-service-74fcd985f6-9qnj4         1/1     Running   0          29m
jaeger-tracing-nodejs-service-75fdff7c8c-pfpd4       1/1     Running   0          29m
jaeger-tracing-python-service-6fbfbf9b8b-5dgnw       1/1     Running   0          29m
jaeger-tracing-salaryamount-mysql-76cf744c75-4d477   1/1     Running   0          29m
salary-grade-postgres-database-6df4676bb-dbtzh       1/1     Running   0          29m
{{< /output >}}

{{% notice info %}}
Note: It can take a few minutes for all the pods to show a `Status` of `Running` and have a `READY` of `1/1`
{{% /notice %}}

Check the frontend's ingress Load Balancer was provisioned:
```bash
kubectl get svc jaeger-tracing-frontend-service
```
{{< output >}} 
NAME         TYPE         CLUSTER-IP  EXTERNAL-IP                                                               PORT(S)      AGE
front-end-lb LoadBalancer 10.100.2.16 *a9e7c954d96114324b80da103b888b79**-902876639.us-west-2.elb.amazonaws.com 80:31627/TCP 2m16s
{{< /output >}}

{{% notice info %}}
Note: If you do not see an External-IP, re-run the command and wait till it appears (this can take a minute or two)
{{% /notice %}}

Get the URL for demo microservice.
```bash
export SERVICE_IP=$(kubectl get svc jaeger-tracing-frontend-service --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}") 
echo http://${SERVICE_IP}/
```

Now check the website is able to load. Note: it can take a few minutes for the ELB’s DNS records to propagate. 
![frontend webpage](/images/observability-with-adot/frontend-webpage.png)

