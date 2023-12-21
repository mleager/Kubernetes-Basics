# Rolling Updates & Rollbacks

Perform the following actions on your minikube:

## Perform rolling update for a Deployment<br>
`kubectl create deploy nginx --image=nginx --replicas=4`<br>

<img width="573" alt="Rolling Updates - create deploy" src="https://github.com/mleager/Kubernetes-Basics/assets/106631893/0b2790fe-539f-47ec-a0d5-c7b925e725e4">
<br>
<br>

`kubectl set image deploy/nginx nginx=nginx:stable-alpine`<br>
`kubectl describe deploy nginx | grep Image`<br>
`kubectl rollout history deploy/nginx`<br>

<img width="587" alt="rolling update" src="https://github.com/mleager/Kubernetes-Basics/assets/106631893/187c91cc-8611-4f7e-b1c3-ed692dd53650">
<br>
<br>

## Perform rollback for a Deployment
`kubectl rollout undo deploy/nginx`<br>
`kubectl describe deploy nginx | grep Image`<br>
`kubectl rollout history`<br>

<img width="587" alt="rollback" src="https://github.com/mleager/Kubernetes-Basics/assets/106631893/a374de39-322e-40ef-8e96-36ebe52c59a9">
<br>
<br>

## Rolling Update Deployment Strategy
- Rolling Deployment Strategy can be defined manually in a manifest file by including spec.strategy.rollingUpdate
- You can define maxSurge and maxUnavailble to determine the number of Pods that are taken down and replaced at a time
```
apiVersion: apps/v1
kind: Deployment
...
spec:
  replicas: 4
  selector:
    matchLabels:
      name: webapp
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    ...
    spec:
      containers:
      - image: <image name>
        name: <container name>
```
