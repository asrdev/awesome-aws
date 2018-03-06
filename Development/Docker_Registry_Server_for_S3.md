# Development
## Docker Registry Server for S3
How to build, configure and use a registry server for pushing/pulling Docker containers to an AWS S3 repository.

### Step-by-step guide

#### Building the base server:

1.  Create an AWS instance by logging into the AWS management console, selecting EC2 from the `Compute` section of the Services menu and clicking the `Launch Instance` button.
2.  From the quick-start menu, select the top level choice (Amazon Linux AMI) and click the Select button.
3.  Unless you're considering using a large amount of resources, click on the `t2.micro` instance type (default) and click on the `Next:Configure Instance Details` button.
4.  Leave the next page at all defaults and click `Next: Add Storage` button.
5.  Amend the size of the provisioned drive from 8 to 24 Gigabytes. Click the `Next: Add Tags` button.
6.  Add three tags with the following information:**Name : PROJ-REG-X**- subsituting a number for the 'X';**project : PROJ**;**owner : your-name**. 'Click the Next: Configure Security Group' button.
7.  Tick the `Select an existing security group` radio button and select the `sg-32d43a5aPROJ-drone-sg` group. Click the `Review and Launch` button. 
8.  Check the full details are correct and click on the Launch button to start the instance.
9.  Assign the `PROJ.pem` certificate to the new EC2 instance.
10.  Log into the newly created instance by running an ssh connection to the new machine (eg: ssh -i "PROJ.pem"[root@ec2-52-56-236-213.eu-west-2.compute.amazonaws.com](mailto:root@ec2-52-56-236-213.eu-west-2.compute.amazonaws.com)")
11.  From the shell run `sudo yum -y update` to update the local machine's installed software.
12.  Once the update is complete, run `sudo yum -y install docker'. Wait for the Docker installation to finish.
13.  Run a git clone to the Bitbucket repo containing the startup script for the registry: (git clone [https://bhartcivica@bitbucket.org/PROJ-civica/PROJ-s3registry-startup.git](https://bhartcivica@bitbucket.org/PROJ-civica/PROJ-s3registry-startup.git))

#### Configure the server:
1.  Move into the newly created `PROJ-s3registry-startup` folder from the git clone in the previous instruction.
2.  Edit the docker-reg.sh script, replacing the `XXX's with the corresponding AWS accesskey and secretkey.
3.  ![](https://civicadigital.atlassian.net/wiki/download/attachments/12845244/image2017-11-23_15-16-22.png?version=1&modificationDate=1511450185265&cacheVersion=1&api=v2)
4.  Start the system by setting the execute bit on the `docker-reg.sh` file (`chmod +x docker-reg.sh`), and then run it (`sudo ./docker-reg.sh`).
5.  Test that the system is running by opening your browser and navigating to the following URL: [http://pipe.PROJteam.co.uk/v2/_catalog](http://pipe.PROJteam.co.uk/v2/_catalog)
6.  You should see a simple list of the available repositories in JSON format.

#### Important note:
When pulling or pushing images through the registry server, it is important to note that the server requires no username or password, since these are provided in the startup script for the server itself. Also, the connection to the server is not via SSL (HTTPS). In order for Docker to allow images to be transferred through the registry, an extra setting must be made within the Docker configuration. This configuration is required in the /etc/docker/daemon.json file:

![](https://civicadigital.atlassian.net/wiki/download/thumbnails/12845244/image2018-2-20_15-34-55.png?version=1&modificationDate=1519140897475&cacheVersion=1&api=v2&width=406&height=54)

As shown above, the registry server - in this case `pipe.PROJteam.co.uk` - is defined as an insecure registry, allowing images to be pulled and pushed through it without credentials or SSL. Unless this setting is included, no images will be able to be transferred.

Once the `daemon.json` file is amended, the Docker daemon needs to be restarted (`systemctl docker restart`).


