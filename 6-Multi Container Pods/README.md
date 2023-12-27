# Deploy a Multi-Container Pod

## Choose a Strategy:

### Sidecar
A secondary container that provides enhanced functionality to the main container, but is not necessarily a part of the application. <br>
It should generally be decoupled and not required for the main container to work.

- Ex: A log processer that reads and stores logs from the main container

### Adapter
Used to standardize the output by the primary container. <br>
Standardizing refers to formatting the output in a specific manner that fits the requirements across your applications.

- Ex: A log formatter that processes the logs from the main container and formats the output in a predictable and repeatable way

### Ambassador
Container that can proxy network requests on behalf of the main application. <br>
It allows other containers to connect to a port on the localhost.

- Ex: A proxy that handles requests on behalf of the main container

### Extra: Init Containers
A container that is run to completion before the creation of the main container. <br>
Usually used to initialize tasks, configure dependencies or wait for other services to be created beforing starting the main container.

## Share a Volume between Containers in this Pod
- Use `emptyDir{}` for ephmeral storage
- Use `PersistentVolumes` and `PersistentVolumeClaims`
- Use a `3rd Party Storage Driver`

This allows both containers to share and access the same data. <br>
Configure a Volume and define the named Volume under each container's `volumeMounts` section. <br>
Each container can have the data mounted on specific locations in their container, and access data written by the other. <br>
