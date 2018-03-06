# Kubernetes 
## Health Checking & Liveness
Kubernetes has the option to provide health checking and liveness probes for all pods running whithin a Kubernetes cluster. This is best illustrated by way of example. Below is a sample of K8s script defining a container within a pod. Some elements have been left out for simplification.

The livenessProbe entry is placed at the end of a container definition. The contents of the above lines are explained below:

| Entry | Usage |
|--|--|
|  initialDelaySeconds: | Defines the number of seconds after startup that the liveness probe will delay before checking the container's state.
|   periodSeconds: | The number of seconds after the first check and seconds between subsequent checks before each succeeding check is carried out.
|   httpGet: |The liveness check will be made by checking the availability of the interface
|   path: | The path after the URL (localhost) which is checked to confirm the health of the container.
|   port: | The port the container listens on which the liveness probe listens.
```
livenessProbe:
  initialDelaySeconds: 60
  periodSeconds: 20
  httpGet:
    path: /healthz
    port: 8080
```
In the above example, the container is spun up and 60 seconds elapse before the first liveness check. The check is then run against `localhost:8080/healthz`, to confirm the container is running. If the container doesn't respond on this port, the container is automatically restarted and the check process starts again. Subsequent checks are made every 20 seconds.

The initialDelaySeconds parameter is important, in that it can sometimes take a certain time for a service to become fully operational after start. This parameter may require tweaking initially to ensure that the checks don't start before the service has had time to completely initialise. **If the delay period is too short, this can result in the service being rebooted before becoming ready for the first check, which in turn will result in the service rebooting itself continually.**
