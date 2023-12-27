# Deploy a multi-container Pod

## Choose a Strategy:

### Sidecar
A secondary container that provides enhanced functionality to the main container, but is not necessarily a part of the application.
It should generally be decoupled and not required for the main container to work.

- Ex: A log processer that reads and stores logs from the main container

### Adapter
Used to standardize the output by the primary container.
Standardizing refers to formatting the output in a specific manner that fits the requirements across your applications.

- Ex: A log formatter that processes the logs from the main container and formats the output in a predictable and repeatable way

### Ambassador
Container that can proxy network requests on behalf of the main application.
It allows other containers to connect to a port on the localhost.

- Ex: A proxy that handles requests on behalf of the main container

### Extra: Init Containers
A container that is run to completion before the creation of the main container.
Usually used to initialize tasks, configure dependencies or wait for other services to be created beforing starting the main container.


## Share a volume between containers in this Pod
- Use emptyDir{} for ephmeral storage
- Use PersistentVolumes and PersistentVolumeClaims
- Use 3rd Party Volume Plugins

## Resources
1. No Requests | No Limits - Default / can suffocate other Pods
2. No Requests | Limits - Requests equal Limits
3. Requests | Limits - Use Request, can go up to Limit ( may stifle resources if another Pod needs more )
4. Requests | No Limits - Allows Pods to grow if needed ( Most Ideal usually )

# Limit Ranges
Only affect Pods that were created after LimitRange is created

# Restrict Total Amount of Resources that can be Consumed by Apps in the Cluster
Ex: All Pods together shouldn't consume more than X CPU or Memory

## Resource Quotas at Namespace Level
Limit total CPU/Memory usage by all Pods in a Namespace to X CPU/Memory
