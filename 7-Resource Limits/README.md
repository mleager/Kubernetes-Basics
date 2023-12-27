# Create a Deployment
`kubectl create deploy nginx --image=nginx --replicas=4`

# Use Requests and Limits to put Resource restrictions on a Deployment
```
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: 1
        memory: "256Mi"
      limits:
        cpu: 1
        memory: "512Mi"
```

# Scale the Deployment and check what will happen when a Node does not have enough Resources
"Insufficient resources to schedule a Pod"
- Pod responsiveness can become throttled due to high CPU usage
- Pods can freeze or crash from high Memory usage
- Pods may be evicted
- New Pods become UnSchedulable 


## Resources
1. No Requests | No Limits - Default / can suffocate other Pods
2. No Requests | Limits - Requests equal Limits
3. Requests | Limits - Use Request, can go up to Limit ( may stifle resources if another Pod needs more )
4. Requests | No Limits - Allows Pods to grow if needed ( Most Ideal usually )

## Limit Ranges
Only affect Pods that were created after LimitRange is created

### Restrict Total Amount of Resources that can be Consumed by Apps in the Cluster
Ex: All Pods together shouldn't consume more than X CPU or Memory

## Resource Quotas at Namespace Level
Limit total CPU/Memory usage by all Pods in a Namespace to X CPU/Memory
