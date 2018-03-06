# Kubernetes 
## Persistent Volumes
Docker container objects are what is known as 'immutable', meaning that any changes which occur within them while running will be lost when the container is brought down and back up again. This means that if, for instance, you're running a web server and require changes to be made to the root document, but you want those changes to survive beyond the next container restart, you will have to create a persistent volume, or a file space which exists outside the container.

A common way to define a persistent volume is to create a folder on one of the nodes in the Kubernetes cluster and then map a folder inside the container to that folder. To do this, you have to create a 'nodeSelector' definition within the yaml file for the deployment using it. This defines the node or slave in the cluster which will host the folder(s). The nodeSelector, however, points not to the node name itself, but the label you give a node (which can be the same as the node name if required).

For instance, if we wish to use 'node1' - a slave node in our cluster -as the host for locating some persistent volumes, then we would have to label the node accordingly:
```
kubectl label nodes node1 node-01
```
The above directive labels 'node1' with the name 'node-01'.

Next, we create some folders we want for our persistent volumes on 'node1':
```
mkdir /home/centos/folder1
mkdir /home/centos/folder2
```
Now, we simply define the nodeSelector parameter in our yaml file, using the label we've just given 'node1' and define the volumes and volumeMounts in the same file.
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    io.kompose.service: keycloak
  name: keycloak
spec:
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: keycloak
    spec:
      nodeSelector:
        name: node-01
      containers:
      - args:
        - -b 0.0.0.0 --server-config=standalone.xml
      ....
      ....
        image: quay.io/ukhomeofficedigital/keycloak-mysql:v3.1.0
        name: keycloak
        ports:
        - containerPort: 8080
        resources: {}
        volumeMounts:
        - mountPath: /etc/secrets/keycloak_folder
          name: keycloak-env
        - mountPath: /opt/jboss/keycloak/themes/govuk/
          name: keycloak-govuk
      ....
      ....
      restartPolicy: Always
      volumes:
        - name: keycloak-env
          hostPath:
            path: /home/centos/folder1
        - name: keycloak-govuk
          hostPath:
            path: /home/centos/folder2
       ....
       .... 
status: {}
```
When the container is brought up, node1 will be used as the location for the persistent volume, specifically /home/centos/folder1 and folder2, mapped to the internal drives /etc/secrets/keycloak_folder and /opt/jboss/keycloak/themes/govuk respectively.
