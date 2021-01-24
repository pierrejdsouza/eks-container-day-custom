# Deploy the Metrics Server
Metrics Server is a scalable, efficient source of container resource metrics for Kubernetes built-in autoscaling pipelines.

These metrics will drive the scaling behavior of the deployments.

We will deploy the metrics server using Kubernetes Metrics Server.
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.4.1/components.yaml
```
Lets' verify the status of the metrics-server APIService (it could take a few minutes).
```
kubectl get apiservice v1beta1.metrics.k8s.io -o json | jq '.status'
```
```
{
  "conditions": [
    {
      "lastTransitionTime": "2020-11-10T06:39:13Z",
      "message": "all checks passed",
      "reason": "Passed",
      "status": "True",
      "type": "Available"
    }
  ]
}
```
