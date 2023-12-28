# Create a Deployment
`kubectl create deploy liveliness --image=nginx --replicas=2`

# Create a Liveness Probe
Check if container is running, or needs to be restarted

## Liveliness Command
1. Busybox container creates `/tmp/healthy` and after 30 seconds, deletes it.
2. 5 seconds after container is created, run the command `cat /tmp/healthy`. <br>
If command returns 0, then the container is considered healthy.
3. After the `/tmp/healthy` directory is deleted, container will return an error. <br>
The container will be decalred unhealthy and be restarted.
```
apiVersion: v1
kind: Pod
metadata:
  name: lively-command
spec:
  containers:
  - name: busybox
    image: busybox
    args:               ## Create /tmp/healthy directoy
    - sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 60
    livelinessProbe:
      exec:
        command:        ## Output the contents of the /tmp/healthy directory
        - cat           ## If /tmp/healthy does not exist, return failure code
        - /tmp/healthy
      initalDelaySeconds: 5     ## Wait 5 seconds before performing the 1st probe
      periodSeconds: 5          ## Perform check every 5 seconds
```

## Liveliness HTTP Request
1. Kubelet send's an HTTP Get request to the server running in the container.
2. If the handler for the defined path returns a successful code (200-400), the container is considered alive and healthy.
```
apiVersion: v1
kind: Pod
metadata:
  name: lively-http
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 8080
    livelinessProbe:        ## Check if the container is serving traffic
      httpGet:
        path: /usr/share/nginx/html/index.html
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
```

## Extra: Startup Probe
Provides a larger inital timeframe for the Kubelet to begin sending health checks to the container.
```
...
spec:
  containers:
  - ...
    ports:
    - containerPort: 8080
    startupProbe:
      httpGet:
        path: /
        port: 8080
      failureThreshold: 30      # Startup Probe will have 300 seconds (30x10)
      periodSeconds: 10
```

# Create a Readiness Probe
Check if container is able to accept traffic

## Readiness Command
Same approach and application as Liveliness Probe, except uses `readinessProbe` instead of `livelinessProbe`
```
...
spec:
  containers:
  - ...
    ports:
    - containerPort: 80
    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

## Readiness HTTP Request
```
...
spec:
  containers:
  - ...
    ports:
    - containerPort: 8080
    readinessProbe:
      httpGet:
        path: /usr/share/nginx/html/index.html
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
```

# Purposely fail a Liveness or a Readiness probe and check what will happen
1. Pod is started <br>
`kubectl describe pod`
<br>
<img width="682" alt="liveness cmd" src="https://github.com/mleager/Kubernetes-Basics/assets/106631893/8103bc47-ec5e-45e2-8804-fbea01456e5f">
<br>
<br>

3. Liveliness Probe's command returns a successful response and Pod is alive <br>
<img width="751" alt="container started" src="https://github.com/mleager/Kubernetes-Basics/assets/106631893/6b062680-aec2-445a-9880-e0c6ebdb61c9">
<br>
<br>

4. Liveliness Probe's command fails, signals the Kubelet to restart the container
<br>
<img width="838" alt="container unhealthy" src="https://github.com/mleager/Kubernetes-Basics/assets/106631893/2af12f83-1c37-43b3-a335-96e2f9ba4750">
<br>
<br>

5. Pod is restarted successfully <br>
<br>
<img width="367" alt="container restarted" src="https://github.com/mleager/Kubernetes-Basics/assets/106631893/2de19598-bb22-4569-9415-b3b0e68a54bd">