# Docker
## Deploying a Docker Image to a Node
Once a Docker image is pushed to a repository - in our case, AWS S3 - it needs to be deployed to whichever host or node is hosting the container instance. This can be done a number of ways, two of which are detailed below.

## Step-by-step guide

Manually pull the image down from the private repo:

1.  SSH onto your Docker host.
2.  From the command line, type `docker pull pipe.PROJteam.co.uk/imagename:version`, where `imagename` is the name of the Docker image posted to the S3 repo (eg: PROJ-public-ui), and `version` is the version number (eg: 3.0.7) or otherwise type `latest` to pull the latest image.
3.  Once the pull is complete, type `docker images` and <enter> to see if the image you pulled is available to your host.

Deploy the image within your docker-compose.yml file:

1.  Edit your `docker-compose.yml` file.
2.  The line which contains image: references a remote image the deployment uses. Ensure this line reads `image: pipe.PROJteam.co.uk/imagename:version` , where `imagename` is the name of the Docker image you're using and `version` the version number (or latest) of the container you`re referencing.
3.  Once you've saved the change to your docker-compose.yml file, type `docker-compose down` and then `docker-compose up` to bring the new container up.

Deploy the image within your Dockerfile:

1.  Edit the Dockerfile in the folder where you downloaded the image information from Bitbucket.
2.  The first line reads `FROM`. Amend this line to reference the image you want to use. (eg: FROM pipe.PROJteam.co.uk/nodejs-base:v6.11.1).
3.  Save the Dockerfile and type `docker build .`, ensuring to include the period at the end, to customise the image from the downloaded image.
4.  Once finished building, type `docker images` to check the image has been built. If so, type `docker run -it imagename, where `imagename` is the name/version of the image you`re running.

Deploy the image via a startup script:

1.  After downloading the docker image, type `docker images` to confirm the image is available.
2.  In the root folder, locate the startup script for the image (eg: start-PROJ-public-ui.sh, or start-PROJ-workflow-api.sh).
3.  Edit the script, commenting out the last line and adding a new line to run the new version of the image.
4.  From the command-line, type `docker stop <name>, where `name` is the name given for the running container (this is defined in the startup script).
5.  From the command-line, type `docker rm <name>`. This disassociates the name with the last running container (if you try running the new container without doing this, you may get an error saying the name is already associated with another container).
6.  Next, run the startup script with the added entry for the new image. The new container will come up immediately.
7.  Note: You may get an error message which reads "Error response from daemon: Conflict. The name "container-name" is already in use by container `e5d65778c6aa2476796a4b20ebcdbf5 b5fb15c01c7f5c52d0483a84e79ac4b10\`. You have to remove (or rename) that container to be able to reuse that name.." In this instance, you need to remove the name from the previous container which used it by typing `docker rm container-name`, replacing container-name with the name you gave the container on the last run.

Node names and addresses:

1.  The node earmarked as `dev.PROJteam.co.uk` is on IP X.XX.XX.01. The ssh key used to connect to it is PROJ.pem.
2.  The node earmarked as `demo.PROJteam.co.uk` is on IP XX.XX.XX.02. The ssh key used to connect to it is also PROJ.pem.
3.  Once logged in, for simplicity sudo to root (`sudo su -`). This will take you to the location of the startup scripts for each of the services (public-ui, workflow-api and keycloak-proxy). The scripts are self-explanatory. When a new image is pulled to the host, the startup script simply needs amending to spin up the correct verion of the container and then executed.
