# Kubernetes
## Some General Concepts
Kubernetes is an orchestrator, for the purpose of deploying, scaling and managing Docker containers in a clustered environment. While Docker containers may be run individually on a host, Kubernetes allows for those containers to be managed and load-balanced/scaled across multiple hosts within a Kubernetes cluster.

**Terminology:**

*   **Kubernetes Master**: The main Kubernetes orchestration host. This host manages all slave hosts using the `kubectl` command set.
*   **Kubernetes Minion**: One of a multiple of `slave` hosts that is controlled and managed by the Kubernetes Master.
*   **Node**: Another name for a Kubernetes Minion.
*   **Container**: A Docker artifact which contains the micro-service required for running a specific funtion (eg: web server, database, etc). Containers are stored as artifcats on a Docker repository, such as DockerHub.
*   **Pod**: A group of containers that are deployed together on the same host. A pod may have just one container or multiple containers within it, but a Pod can only exist on one host (node) at a time.

**General Observations**:

A Kubernetes Master can orchestrate any number of hosts from one upwards. Officially, the maximum number of nodes (hosts) a Kubernetes master can manage is up to 5,000, but this is practically defined by the memory/storage limits of the machines in question.

Containers run in a single pod can reference each other as `localhost`, since they occupy the same local network. So, for instance, two Docker containers runninng in the same pod with one listening on port 80 and the other on 22 can access each other via `localhost:80` and `localhost:22` respectively.

Containers running in separate pods can reference each other via a service established to allow external communication. These services are defined in a yaml config file, indicating the listening port for that pod. Pods can listen on multiple ports, if required. When a service is established in this way, each container will have environment parameters set with the IP address and listening port of the adjacent pods. For instance, if you run the `egar-location-api` container and the `egar-person-api` containers in separate pods and create a service for each pod to reveal their listening ports, each container will have environment parameters such as `LOCATION_API_SERVICE_PORT`, `PERSON_API_SERVICE_PORT`, `LOCATION_API_SERVICE_HOST` and `PERSON_API_SERVICE_HOST`, each corresponding to the host IP and port of each pod. _Note: The envirionment parameters exist only if the service for that pod is already running prior to the startup of the adjacent pod. So the first pod which is spun up will have only it`s own environment parameters set. The next pod/service brought up with have the parameters for itself and the first pod, and so on._

Environment parameters set within the Kubernetes configuration yaml files can be accessed via the configuration by enclosing the environment parameter within a dollar sign at the beginning and enclosing the parameter within brackets. For example: `$(PARAMNAME)`. This is of particular importance when accessing the environment parameters set for the corresponding service host and port settings (see paragraph above). Secrets can also be accessed in the same way (see the Kubernetes Secrets entry in Confluence).