# Rolling Updates & Rollbacks

Perform the following actions on your minikube:

- Perform rolling update for a Deployment
`kubectl create deploy nginx --image=nginx --replicas=4`
`k set image deploy/nginx nginx=nginx:stable-alpine`
`k describe deploy nginx | grep Image`
`k rollout history deploy/nginx`

- Perform rollback for a Deployment
`kubectl rollout undo deploy/nginx`
`k describe deploy nginx | grep Image`
`k rollout history`

- Rolling Update Deployment Strategy
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
