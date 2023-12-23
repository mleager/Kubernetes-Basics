# ETCD Encryption at Rest

Kubernetes Docs - Encrypting Confidential Data at Rest: <br>
https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/

1. Determine whether Encryption at Rest is already enabled
2. Create an Encryption Configuration File
3. Update Kube-APIserver Configuration
4. Verify that the newly written data is Encrypted


## Determine whether Encryption at Rest is already enabled

1. Create a basic Secret <br>
`$ kubectl create secret generic secret-1 --from-literal=mykey-1=bigsecret`

This Secret Object can be decoded by anyone that has Read access.

2. View the Secret using the ETCD Client to determine if it's encrypted, or stored as plain text
```
$ apt-get install etcd-client
$ export ETCDCTL_API=3
$ etcdctl \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key  \
    get registry/secrets/default/secret-1 | hexdump -C
```
* Note: ETCD stores Secrets at the path registry/secrets/default/<secret name>

The output on the right-most column will include the Secret's key, and either include the value of the Secret as plain text, or as an encrypted value.
(Example of encrypted key shown at the bottom)

3. Check if the Kube-APIserver is using the argument "--encryption-provider-config"

Manually check the Kube-APIserver Definition file or search for the APIserver's processes. <br>
a. `$ kubectl -n kube-system describe pod kube-apiserver-controlplane` <br>
b. (Kubeadm) `$ cat /etc/kubernetes/manifests/kube-apiserver.yaml` <br>
c. `$ ps aux | grep kube-apiserver-controlplane | grep "encryption-provider-config"`

If no results are returned by 'c.', it is not supplied as an argument to the Kube-APIserver.

## Create an Encryption Configuration File

1. Create an EncryptionConfiguration Object using a Definition File
```
# enc.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: IYyzqPEeIIaPM/OkPgfG1aeTsHfaMUOj63nituLs7Es=  #--- Entered after generating key from step 2
      - identity: {}
```
* Note: The `providers` field is an ordered list. The 1st item in the array is the method of encrpytion. All the following items are methods for decryption. 

2. Generate a Random Key and Base64 encode it in order to encrpyt Secrets stored by ETCD <br>
`$ head -c 32 /dev/urandom | base64` <br>
Example Key: <b>IYyzqPEeIIaPM/OkPgfG1aeTsHfaMUOj63nituLs7Es=</b>

* Note: There are multiple encryption options, each with different byte-size requirements for the key.

## Update Kube-APIserver Configuration

1. Move the EncryptionConfiguration's Definition file to be used as a Volume by the Kube-APIserver <br>
`$ mkdir /etc/kuberentes/enc` <br>
`$ mv env.yaml /etc/kubernetes/enc/`

2. Edit the Kube-APIserver Configuration File
```
# Code shortened for clarity
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 10.20.30.40:443
  creationTimestamp: null
  labels:
    app.kubernetes.io/component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - ...
    - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml  # add this line
    - ...
    volumeMounts:
    ...
    - name: enc                           # add this line
      mountPath: /etc/kubernetes/enc      # add this line
      readOnly: true                      # add this line
    ...
  volumes:
  ...
  - name: enc                             # add this line
    hostPath:                             # add this line
      path: /etc/kubernetes/enc           # add this line
      type: DirectoryOrCreate             # add this line
  ...
```

2. Wait for the Kube-APIserver to Update, or Restart manually
* If using containerd, view Pods' status: `crictl pods`

3. Confirm the Kube-APIserver is up, and includes the `--encryption-provider-config` argument <br>
`$ ps aux | grep kube-apiserver | grep "encrpytion-provider-config"`

## Verify that the newly written data is Encrypted

1. Create another Secret <br>
`$ kubectl create secret generic secret-2 --from-literal=mykey-2=biggersecret`

2. Verify if this new Secret is Encrypted in ETCD
```
$ etcdctl \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key  \
    get registry/secrets/default/secret-2 | hexdump -C
```
* Example of the Returned Secret from ETCD: 
```
|/registry/secret|
|s/default/secret|
|1.k8s:enc:aescbc|
|:v1:key2:.l.....|
|%Q...l..Mz.=..|n|
|.y..(..._5.,...>|
|#:..(.H-k-F.r.pL|
|..5C.N`..o......|
|...S..>f..|
```

3. Verify new Secret can be decrypted from ETCD <br>
`$ kubectl get secret secret-2 -o yaml` <br>
<b>key2: YmlnZ2Vyc2VjcmV0Cg==</b>

* This should return the Base64 encoded version of the Secret

    `$ echo YmlnZ2Vyc2VjcmV0Cg== | base64 --decode` <br>
    <b>biggersecret</b>

## Secrets Objects in ETCD Before Encryption at Rest

As of now, `secret-2` is encrypted when stored in ETCD, but `secret-1` is not.
This is because the encryption was applied after `secret-1` was already stored inside ETCD.
So only Secrets stored after the encryption will be encrypted.

To encrypt `secret-1` we only need to update/reapply the Secret. <br>
`$ kubectl get secrets --all-namespaces -o json | kubectl replace -f -`
