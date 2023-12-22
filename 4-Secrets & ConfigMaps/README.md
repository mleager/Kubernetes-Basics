# Secrets & ConfigMaps

Perform the following actions on your minikube:
1. Create a secret
2. Access a secret from a Pod
3. Create a configmap
4. Access a configmap from a Pod


### Note on Secrets:
- Secrets and NOT Encrypted, they're only encoded ( anyone can decode )
- Do NOT check-in Secret Objects to your Repos
- Secrets are not Encrypted be default in ETCD ( must enable Encryption-at-Rest )
- Anyone able to create Pods/Deploy in a Namespace can access Secret Objects in that Namespace
- Configure RBAC for access to Secrets
- Consider External 3rd Party Secret Store ( AWS, Azure, GCP, Vault )


## -- Create a Secret --

### Imperative

- Inline Arguments <br>
<b>`kubectl create secret generic <secret name> --from-literal=<key>=<value>`</b>

  `kubectl create secret generic app-secret --from-literal=DB_HOST=mysql`

- Input from a File<br>
<b>`kubectl create secret generic <secret name> --from-file=<path/to/file>`</b>

  `kubectl create secret generic app-secret --from-file=./app-secret.properties`


### Declarative

* Must ENCODE the values before using them in a Definition File <br>
`echo 'mysql' | base64`   -->  `bXlzcWw=` <br>
`echo 'root' | base64`    -->  `cm9vdA==` <br>
`echo 'passwrd' | base64` -->  `cGFzc3dyZA==` <br>

```
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_HOST: bXlzcWw=
  DB_USER: cm9vdA==
  DB_PASSWORD: cGFzc3dyZA==
```
`$ kubectl create -f secret.yaml`

* To DECODE Secret Values <br>
`echo 'bXlzcWw=' | base64 --decode`     -->  `mysql` <br>
`echo 'cm9vdA==' | base64 --decode`     -->  `root` <br>
`echo 'cGFzc3dyZA==' | base64 --decode` -->  `passwrd`


## -- Access a Secret from a Pod --

### Inject Secret Object into a Pod
```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - ...
    ...
    envFrom:
    - secretRef:
        name: app-secret
```

### Inject a Single Secret as an ENV into a Pod
```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - ...
    ...
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: DB_PASSWORD
```

### Inject Secret as a Volume
```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - ...
    ...
  volumes:
  - name: app-secret-volume
    secret:
      secretName: app-secret
```

* Each attribute in the Secret is created as a file, 
  and the Value of the Secret it its content

 `$ ls /opt/app-secret-volume` <br>
    <b>DB_HOST    DB_PASSWORD    DB_USER</b>

 `$ cat /opt/app-secret-volume/DB_PASSWORD` <br>
    <b>passwrd</b>

## -- Create a ConfigMap --

### Imperative

- Inline Arguments <br>
<b>`kubectl create configmap <cm-name> --from-literal=<key>=<value>` </b>

  `kubectl create cm app-config --from-literal=APP_GROUP=1 --from-literal=APP_ENV=prod`

- Input from a File <br>
<b>`kubectl create configmap <cm-name> --from_file=<path/to/file>` </b>

  `kubectl create cm app-config --from_file=/tmp/app-configmap.properties`


### Declarative
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_GROUP: 1
  APP_ENV: prod
  PORT: 3306
  max_allowed_packet: 128M
```
`$ kubectl create -f configmap.yaml`


## -- Access a ConfigMap from a Pod --

### Inject ConfigMap in a Pod
```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - ...
    ...
    envFrom:
    - configMapRef:
        name: app-config
```


### Use a Single ConfigMap Value as an ENV in a Pod
```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - ...
    ...
    env:
    - name: APP_GROUP
      valueFrom:
       configMapKeyRef:
         name: app-config
         key: APP_GROUP
```

### Inject ConfigMap as a Volume
```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - ...
    ...
  volumes:
  - name: app-config-map
    configMap:
      name: app-config
```
