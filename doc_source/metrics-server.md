# Installing the Kubernetes Metrics Server<a name="metrics-server"></a>

The Kubernetes Metrics Server is an aggregator of resource usage data in your cluster, and it isn't deployed by default in Amazon EKS clusters\. For more information, see [Kubernetes Metrics Server](https://github.com/kubernetes-sigs/metrics-server) on GitHub\. The Metrics Server is commonly used by other Kubernetes add ons, such as the [Horizontal Pod Autoscaler](horizontal-pod-autoscaler.md) or the [Kubernetes Dashboard](eks-managing.md)\. For more information, see [Resource metrics pipeline](https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-metrics-pipeline/) in the Kubernetes documentation\. This topic explains how to deploy the Kubernetes Metrics Server on your Amazon EKS cluster\.

**Important**  
The metrics are meant for point\-in\-time analysis and aren't an accurate source for historical analysis\. They can't be used as a monitoring solution or for other non\-auto scaling purposes\. For information about monitoring tools, see [Observability in Amazon EKS](eks-observe.md)\.

**Deploy the Metrics Server**

1. Update aws-auth configmap to include the username field so that fetching metrics will be authorized.

```
Name:         aws-auth
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>

Data
====
mapRoles:
----
...
-
  groups:
  - system:masters
  rolearn: arn:aws:iam::123456789123:role/kubernetes-devops
  username: devops:{{SessionName}}  # Ensure this has been specified.
```

2. Deploy the Metrics Server with the following command:

   ```
   kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
   ```

3. Verify that the `metrics-server` deployment is running the desired number of pods with the following command\.


   ```
   kubectl get deployment metrics-server -n kube-system
   ```

   An example output is as follows\.

   ```
   NAME             READY   UP-TO-DATE   AVAILABLE   AGE
   metrics-server   1/1     1            1           6m
   ```
