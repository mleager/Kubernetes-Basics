# Pods & Services

Perform the following actions on your minikube:
- Create an Nginx Pod that's a part of a Deployment
- Expose a Pod using the ClusterIP Service
- Launch one more Pod. Connect to it using kubectl exec. Use curl to test that Nginx service is working

## Create Nginx Deployment

`kubectl create deploy nginx --image=nginx`

## Expose a Pod using the ClusterIP Service

`kubectl expose deploy nginx --port=80`

<img width="546" alt="Pods - deploy   svc" src="https://github.com/mleager/Kubernetes-Basics/assets/106631893/07fea975-e971-46cf-9e74-c126060af7d7">


## Scale the Replica to 2 Pods. Curl the 2nd Pod to test the Nginx Service is working

`kubectl scale nginx --replicas=2`

* Curl from Localhost:
`kubectl exec nginx-7854ff8877-mzb99 -- curl 127.0.0.1:80`

<img width="586" alt="Pods - curl nginx pod" src="https://github.com/mleager/Kubernetes-Basics/assets/106631893/ea777baf-1e35-480c-a565-4fc951167015">

* Curl from ClusterIP:
`kubectl exec nginx-7854ff8877-mzb99 -- curl 10.97.124.208:80`

<img width="615" alt="Pods - curl nginx pod #2" src="https://github.com/mleager/Kubernetes-Basics/assets/106631893/a18c8d05-4401-4ccb-8665-82ccac143883">


The Nginx Pod returns the content and is working correctly.
