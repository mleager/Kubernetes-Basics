# Scheduling

Perform the following actions on your minikube:
- Label a Node
`kubectl label node node01 color=blue`
<label node>

* Node `node01` is given the Label `color=blue`

- Use nodeSelector to schedule a Pod on a particular node
`node-selector.yaml`
<selector created>

* This Pod is Scheduled on a Node that contains the Label `color=blue`

- Use taints to prevent Pods from being scheduled on a particular node
`k taint node node01 color=blue:NoSchedule`
<node tainted>

* Node `node01` is Tainted with the Key:Value pair `color: blue`

- Use tolerations to ignore taints
`tolerations.yaml`
<toleration created>

* This Pod is Scheduled on a Node that matches the Taint's Key:Value pair `color:blue`

- Use podAntiAffinity to make sure that pods in the same deployments are not scheduled on the same node
`pod-affinity.yaml`

1. Pod named `group1` has the Label `group:1`
2. Pod `group1a` is scheduled on the same Node that contains Pods that have the Label `group:1`

<>

- Use podAffinity to make sure that pods from separate deployments are scheduled on the same node
`pod-anti-affinity.yaml`

1. Pod named `level1` has the Label `level:1`
2. Pod `level2` is NOT scheduled on a Node that contains a Pod that includes the Label `level:1`

<>
