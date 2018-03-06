# Kubernetes
## Managing Secrets
When defining a Kubernetes deployment or pod, there may be a need to include passwords, codes or usernames in the yaml file. Also, secrets may be required to be accessed within the container itself once it`s spun up. For this purpose, secrets can be created and shared across the Kubernetes estate.

**Encrypt the parameters first:**

1.  From the command-line, encode your secret parameters parsing into base64, using the `echo -n` command (_**Note**: The `-n` removes the carriage return from the exported string_)
```
echo -n "secretpassword" | base64
c2VjcmV0cGFzc3dvcmQ=
```
2.  Create a secrets yaml fle, such as `secrets.yaml`. Naming of the file is up to you. The format should be similar to below:
```
apiVersion: v1
kind: Secret
metadata:
  name: proj-secret
type: Opaque
data:
  password: c2VjcmV0cGFzc3dvcmQ=
  ussername: c2E=
```
4.  The structure is similar to most Kubernetes yaml config files. The encoded entries are entered alongside the parameter names. Save this file. (_**Note**: Ensure this file is kept safe and is inaccessible, as the encoded entries can be easily converted back to plaintext_)
6.  To load the secrets into Kubernetes, type `kubectl create -f <filename>`, substituting the <filename> for the actual name of the yaml file.
7.  You can confirm the secrets are loaded by typing `kubectl get secrets`. You can also type `kubectl describe secrets`, which will give some further information.
```
kubectl get secrets
NAME                  TYPE                                  DATA      AGE
default-token-wkgcd   kubernetes.io/service-account-token   3         3d
proj-secret           Opaque                                1         5s
kubectl describe secrets
Name:           proj-secret
Namespace:      default
Labels:         <none>
Annotations:    <none>

Type:   Opaque

Data
====
password:         62 bytes
username:         17 bytes
```  
8.  To use these secrets in your Kubernetes configuration, edit the yaml file which loads your deployment/pod and include the following entries in the definition for the container. We will use the example shown above for `dbuser` and `dbpass`:
10.  As you can see, the `env:` (environment) section is where you define the environment parameters within the `person-api` container above. The name given is `DB_USER`, it`s using a secrets definition with the name `egar-secret` and the key (value) is `dbuser`. So the encoded entry for `dbuser` will be parsed into the environment parameter `DB_USER` in unencrypted form.
12.  Secrets can also be accessed from within the container via shell scripts running within the containerised environment. In this instance, simply reference them normally within the shell script (eg: `$DB_USER`).
13.  You can define multiple secrets in the secrets yaml file, but you can only reference those secrets which are directly defined within the yaml file for each pod/deployment.
14.  **Secrets must be loaded prior to loading the pods/deployments which utilise them, so ensure you run kubectl against the secrets yaml file before creating any other pods**.

You may also want to use visual panels to communicate related information, tips or things users need to be aware of.

