# Kubernetes
## Operating scripts and general structure
### General Notes

The latest Kubernetes insallation runs across three hosts - a master and two slave nodes (minions). These machines are earmarked as:

*   **master.PROJteam.co.uk** - Kubernetes master server
*   **minion01.PROJteam.co.uk** - Kubernetes slave (node)
*   **minion02.PROJteam.co.uk** - Kubernetes slave (node)

These machines have DNS entries in AWS/Route53, so they can be accessed directly via their FQDNs. The login credentials are 'centos' with the PROJ.pem certificate.

Kubernetes can be managed from the centos - non-root - account. All scripts to simplify the most commonly used Kubernetes commands are in the home folder of the 'centos' account. These include:

*   **PROJ-startup.sh** - Shell script to start all presently provisioned pods and containers.
*   **PROJ-shutdown.sh** - Shell script to stop all provisioned pods and containers.
*   **status.sh** - Shell script to view the present operational status of the server across all containers, services and pods. This script runs 'kubectl get deployment,svc,pods,pvc' - meaning 'Show all deployments, services, pods and persistent volumes'.
*   **getpublicui.sh** - Shell script to launch a port forward for the public-ui on the locahost interface. An Nginx proxy running on the master forwards requests to the localhost interface to provide access to the public-ui from outside the system.
*   **gensecret.sh** - Shell script to ceate a base64 hash of any parameter, such as a password or username, for use in the Kubernetes secrets file. See the section on Kubernetes [secrets](/wiki/spaces/PROJ/pages/23396517/Kubernetes+-+Managing+Secrets) for more info.

Each API is called from a downloaded copy of the associated repository from Bitbucket. So, for instance, 'PROJ-public-ui' has been cloned from Bitbucket at the /home/centos folder level. Within each repository are the command-line scripts which call the associated yaml configs for each API/component. The folder structure for all of the repos on the server is as follows:

*   **<repository-name>/scripts/bash/start<repo>.sh** - Shell script (called by the PROJ-startup.sh script) to start the pod.
*   **<repository-name>/scripts/bash/stop<repo>.sh** - Shell script (called by the PROJ-shutdown.sh script) to stop the pod.
*   **<repository-name>/scripts/kube/<repository-name>.yaml** - The deployment file for the pod.
*   **<repository-name>/scripts/kube/<repository-name>-template.yaml** - The template for the deployment file. This contains the version number of the API as an environment parameter, which is parsed into a newly created yaml definition at startup.
*   **<repostitory-name>/scripts/kube/<repostory-name>-service.yaml** - The service file for the pod, allowing access to the pod from other pods within the cluster and - in some cases - allow access to pods from outside the cluster.

For example, the location API and it's startup file structure  is located here:

*   /home/centos/PROJ-location-api/ - root folder
*   /home/centos/PROJ-location-api/scripts - scripts folder
*   /home/centos/PROJ-location-api/scripts/bash - startup scripts folder
*   /home/centos/PROJ-location-api/scripts/kube - Kubernetes definitions

Container versioning is defined in the /home/centos/PROJ-startup.sh script, where the API version is set as an environment parameter and parsed through the API's yaml template file into a Kubernetes configuration file. At the time of writing, the startup file is as follows:

![](https://civicadigital.atlassian.net/wiki/download/thumbnails/25427969/image2018-1-16_10-37-17.png?version=1&modificationDate=1516099039047&cacheVersion=1&api=v2&width=600&height=509)

The startup scripts for the K8s deployment on the development environment are in Bitbucket here: [https://bitbucket.org/PROJ-civica/PROJ-kubernetes-scripts/overview](https://bitbucket.org/PROJ-civica/PROJ-kubernetes-scripts/overview)

Changing the version number in this file will result in the server attempting the download of the nominated version from the repository and running it in the environment. Taking the 'submission-api' container as an example, the 'submission-api-deployment-template.yaml' file has the following entry:

![](https://civicadigital.atlassian.net/wiki/download/attachments/25427969/image2018-1-16_10-41-22.png?version=1&modificationDate=1516099284570&cacheVersion=1&api=v2)

This is parsed through an 'envsubst' command to replace the $SUBMISSION_API_VER environment variable with it's value (eg: 4.1.10, as shown above). This allows for versioning to be handled via the one startup script. Below is the example startup script from 'submission-api':

![](https://civicadigital.atlassian.net/wiki/download/attachments/25427969/image2018-1-16_10-44-19.png?version=1&modificationDate=1516099461830&cacheVersion=1&api=v2)

All startup scripts for individual APIs are found in their respective Bitbucket repositories. For instance, the 'PROJ-files-api' startup scripts can be found in this Bitbucket repository: 
[https://bitbucket.org/PROJ-civica/PROJ-files-api/src/b7e83c931e4e?at=kubernetes](https://bitbucket.org/PROJ-civica/PROJ-files-api/src/b7e83c931e4e?at=kubernetes)

### Additional Information for Operation:

#### Node Selector/Volume Mounts.

The K8s system is dependent on a number of provisos. In particular, the Keycloak service requires that files are written to a non-volatile filesystem, meaning those files have to exist somewhere physically within the cluster. To achieve this, I have used a **Node Selector** as an option to host the files on a particular node within the cluster - in this case, 'minion01'. In order to to this, a directive must be run from the Kubernetes master, as follows;

kubectl label nodes minion01 name=minion01

In the above example, the Node I have nominated as the NodeSelector subject is 'minion01'. In the above directive, I have adhered a label to the node, following the same naming convention as the node itself, thought in practice the name given can be anything. The node is then included in a 'nodeSelector' directive in the yaml file for the Keycloak deployment, as follows:

![](https://civicadigital.atlassian.net/wiki/download/attachments/25427969/image2018-1-25_10-1-20.png?version=1&modificationDate=1516874482434&cacheVersion=1&api=v2)

Further down in the same yaml config for the Keycloak deployment, I have included lines to denote the volumeMounts and the volumes those mounts will utilise on the node selected (minion01). Below is an illustration of the volumes and volume mounts defined in the deployment yaml file for Keycloak:

![](https://civicadigital.atlassian.net/wiki/download/thumbnails/25427969/image2018-1-25_10-4-10.png?version=1&modificationDate=1516874652071&cacheVersion=1&api=v2&width=668&height=640)

The Volumes section defines the volume name against the path on the node where the files/folders are located. The VolumeMounts section defines the path within the container where those volumes are mapped.
# Kubernetes
## Operating scripts and general structure

### Name Resolution:

Keycloak and the Public-UI containers have to occupy the same cluster and namespace as each other. This brings up certain difficulties, in that each reference each other via DNS (eg: develop.PROJteam.co.uk and auth.PROJteam.co.uk). Both these hostnames need to resolve against one another so that flow between each other in terms of Keycloak authentication and subsequent service access are maintained. While name resolution externally is quite straightforward, internally the containers running the applications have to also resolve against each other, but using 'internal' addressing as opposed to external (internet) addressing. To acheive this, split DNS was used by making the K8s server a nameserver with locally held addresses for the 'PROJteam.co.uk' domain, utilising Bind as the name service. Each of the nodes running from the master were then set to use the master as their primary DNS server. The Bind configuration was set to act solely as a forwarder by default, with the exception of the 'PROJteam.co.uk' domain, which contained the local addresses for the relevant hosts (develop, auth and pipe). This way, external DNS names would still resolve while the 'PROJteam.co.uk' hosts would resolve correctly internally. The Bind config (zone file and named.conf) for master is given below:

![](https://civicadigital.atlassian.net/wiki/download/thumbnails/25427969/image2018-1-25_11-50-18.png?version=1&modificationDate=1516881021099&cacheVersion=1&api=v2&width=363&height=163)

_/etc/named.conf_ entries.

![](https://civicadigital.atlassian.net/wiki/download/thumbnails/25427969/image2018-1-25_11-51-59.png?version=1&modificationDate=1516881121169&cacheVersion=1&api=v2&width=462&height=266)

_/var/named/fwd.PROJteam.co.uk.db_ entries

The entry above for 'pipe.PROJteam.co.uk' is required in order for updated Docker images to be pulled from the Docker repository. The other two define the internal addresses for both 'auth' and 'develop'.

Lastly, the /etc/resolv.conf file on both nodes (minion01 and minion02) are updated to use the K8s master as their default name server. **This is because pods spun up on any node in a cluster use the same /etc/resolv.conf settings as the node they run on.**

The /etc/resolv.conf file in an EC2 instance by default is set via DHCP. So when a node is rebooted, the additional entry pointing to the K8s master/DNS server can be lost. To overcome this you can write a script which will add the entry at startup. The entry in my example is shown below, where the DNS server (K8s master) is 172.31.19.54:

![](https://civicadigital.atlassian.net/wiki/download/thumbnails/25427969/image2018-1-25_12-13-26.png?version=1&modificationDate=1516882408390&cacheVersion=1&api=v2&width=335&height=62)

### Keycloak:

Keycloak runs separately from the rest of the containers, in that it has to be run and left to run for several minutes prior to the rest of the containers. Keycloak requires a period of initialisation before it can be accessed by the other pods, so it's important that if you have to restart the server or the Keycloak services that this is observered.

To start Keycloak, navigate to the **/home/centos/PROJ-keycloak/scripts/bash** folder and run the 'startkc.sh' script.

![](https://civicadigital.atlassian.net/wiki/download/attachments/25427969/image2018-2-1_10-47-33.png?version=1&modificationDate=1517482056263&cacheVersion=1&api=v2)

To stop Keycloak, run the 'stopks.sh' script in the same folder.

Once Keycloak has run for a few minutes, you can run the PROJ-startup.sh script as outlined at the top of this document.

<