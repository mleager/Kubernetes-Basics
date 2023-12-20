# Scheduling

Perform the following actions on your minikube:
1. Label a Node<br>
`kubectl label node node01 color=blue`

<img width="575" alt="Scheduling - Label Node" src="https://github.com/mleager/Kubernetes-Basics/assets/106631893/28286a54-7320-4173-b70f-47bc1cc80fd1">

** Node `node01` is given the Label `color=blue`
<br>
<br>

2. Use nodeSelector to schedule a Pod on a particular node

<img width="634" alt="selector-created" src="https://github.com/mleager/Kubernetes-Basics/assets/106631893/0a431673-d8ca-4b0a-acb5-7ea22d9f3de5">

** This Pod is Scheduled on a Node that contains the Label `color=blue`
<br>
<br>

3. Use taints to prevent Pods from being scheduled on a particular node<br>
`k taint node node01 color=blue:NoSchedule`

<img width="634" alt="node tainted" src="https://github.com/mleager/Kubernetes-Basics/assets/106631893/bcac3cca-7048-4202-b127-4cae432999ea">

** Node `node01` is Tainted with the Key:Value pair `color: blue` with the effect of `NoSchedule`
<br>
<br>

4. Use tolerations to ignore taints

<img width="672" alt="toleration created" src="https://github.com/mleager/Kubernetes-Basics/assets/106631893/25348e12-aa35-4547-8bbd-66c6204d7a85">

** This Pod is Scheduled on a Node that matches the Taint's Key:Value pair `color:blue`
<br>
<br>

5. Use podAntiAffinity to make sure that pods in the same deployments are not scheduled on the same node

a. Pod named `group1` has the Label `group:1`<br>
b. Pod `group1a` is scheduled on the same Node that contains Pods that have the Label `group:1`

<img width="707" alt="pod affinity created" src="https://github.com/mleager/Kubernetes-Basics/assets/106631893/105b8c1d-0aa5-4517-87ac-b152aa4cf6fe">

** NOTE: The `controlplane` Node had its Taints removed in order to schedule Pods. Do not do this outside of a learning environment.
<br>
<br>

6. Use podAffinity to make sure that pods from separate deployments are scheduled on the same node
`pod-anti-affinity.yaml`

a. Pod named `level1` has the Label `level:1`<br>
b. Pod `level2` is NOT scheduled on a Node that contains a Pod that includes the Label `level:1`

<img width="698" alt="pod anti affinity created" src="https://github.com/mleager/Kubernetes-Basics/assets/106631893/88ed05ae-7ba8-4d8e-8c10-f753c01af0c2">

** Pod `level1` is placed on the `controlplane` Node because of its lack of Tolerations<br>
** Pod 'level2' is placed on 'node01` because it cannot be placed on the same Node as `level1` and has Tolerations for `node01`
