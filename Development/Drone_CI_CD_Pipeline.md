# Development
## How to build, configure and setup pipelines in a Drone environment

### Step-by-step guide

#### Build and Installation (GUI):

1.  Create an AWS instance by logging into the AWS management console, selecting EC2 from the 'Compute' section of the Services menu and clicking the 'Launch Instance' button.
2.  From the quick-start menu, select the top level choice (Amazon Linux AMI) and click the Select button.
3.  Unless you're considering using a large amount of resources, click on the 't2.micro' instance type (default) and click on the 'Next:Configure Instance Details' button.
4.  Leave the next page at all defaults and click 'Next: Add Storage' button.
5.  Amend the size of the provisioned drive from 8 to 24 Gigabytes. Click the 'Next: Add Tags' button.
6.  Add three tags with the following information: **Name : PROJ-DRONE-X** - subsituting a number for the 'X'; **project : PROJ** ; **owner : your-name**. Click the 'Next: Configure Security Group' button.
7.  Tick the 'Select an existing security group' radio button and select the 'sg-32d43a5aPROJ-drone-sg' group. Click the 'Review and Launch' button.  
    
8.  Check the full details are correct and click on the Launch button to start the instance.
9.  Assign the 'PROJ.pem' certificate to the new EC2 instance.
10.  Log into the newly created instance by running an ssh connection to the new machine (eg: `ssh -i "PROJ.pem"` [root@ec2-52-56-236-213.eu-west-2.compute.amazonaws.com](mailto:root@ec2-52-56-236-213.eu-west-2.compute.amazonaws.com)")
11.  From the shell run `sudo yum -y update` to update the local machine's installed software.
12.  Once the update is complete, run 'sudo yum -y install docker'. Wait for the Docker installation to finish.
13.  Run a git-clone to the following: [https://bitbucket.org/PROJ-civica/PROJ-drone-config/src/55b8b72ff2ae?at=master](https://bitbucket.org/PROJ-civica/PROJ-drone-config/src/55b8b72ff2ae?at=master)

#### Build and Installation (AWS Script):

1.  Prerequisites: AWS CLI tools are installed on your system and configured to connect to your AWS service. Git client is also installed on your system.
2.  Run a git clone to the following: [https://bitbucket.org/PROJ-civica/PROJ-drone-config/src/55b8b72ff2ae?at=master](https://bitbucket.org/PROJ-civica/PROJ-drone-config/src/55b8b72ff2ae?at=master)
3.  Go into the newly created 'PROJ-drone-config' folder.
4.  Execute the following line (as specified in the README in the folder): `aws ec2 run-instances --image-id ami-xxxxxxxx --count 1 --instance-type t2.micro --key-name PROJ --subnet-id subnet-xxxxxxxx --security-group-ids sg-xxxxxxxx --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Hostname}]' --user-data <a rel="nofollow">file://install-drone.txt</a>`.Replace all the relevant details (image-id, subnet-id, etc) with the appropriate information for your environment.
5.  Log into the newly created instance using the PROJ.pem certificate to authenticate.

#### Configure the Drone environment:

1.  Amend the `drone-start.sh` script to reflect the environment you're working in, substituting your Bitbucket credentials, for instance.
2.  In your Bitbucket account settings, click on the settings tab and 'OAuth' in 'Access Management' section.
3.  Click on the 'Add Consumer' button. Enter the name and description in the appropriate fields.
4.  Under 'Callback URL', enter the IP address or DNS name of your Drone server with the port number and 'authorize' entry. For example: '<a rel="nofollow">http://drone.PROJteam.co.uk:8000/authorize'.</a>
5.  In URL, enter the company URL ([www.civica.co.uk).](http://www.civica.co.uk).)
6.  Tick all the available radio buttons, allowing the connection to read/write to the repo, run pulls, etc.
7.  Save all the changes. When you expand the newly created connection, you will see the Key and Secret required for the Drone config environment. Make a note of these.
8.  Edit the `drone-start.sh` script, putting the Key and Secret entries from Bitbucket into the...
9.  Run `chmod +x drone-start.sh` to set the file to exectute. Run `./drone-start.sh` to start the Drone instance.

#### Prepare the repository:

1.  Before a repo can be enabled within Drone, the following files should be present in the repo:
    1.  `.drone.yml`: The pipeline definitions go here.
    2. `docker-build.sh`: The script containing the build instructions.
    3.  `utils/get-site-version.sh`: The script from which the release version is obtained.
    4.  `docker-deploy.sh` : The script which deploys the compiled Docker image to the image repository.
2.  The `.drone.yml` file (with a leading dot) can be set with the following general configuration:  
    ![](https://civicadigital.atlassian.net/wiki/download/attachments/12910792/image2017-11-23_13-56-47.png?version=1&modificationDate=1511445409472&cacheVersion=1&api=v2)
3.  The `docker-build.sh` file contains just two lines, one which gets the release version number (`get-site-version.sh`) and the `docker build` command. The following is an example for the PROJ-public-ui container: ![](https://civicadigital.atlassian.net/wiki/download/attachments/12910792/image2017-11-23_13-59-10.png?version=1&modificationDate=1511445553146&cacheVersion=1&api=v2)
4.  The `get-site-version.sh` script scrapes the version number from the package.json file in the 'utils' folder. At present, this is particular to the PROJ-public-ui container, but can be amended to allow for other containers: ![](https://civicadigital.atlassian.net/wiki/download/attachments/12910792/image2017-11-23_14-0-47.png?version=1&modificationDate=1511445649180&cacheVersion=1&api=v2)
5.  The `docker-deploy.sh` script tags the image and issues the `docker push` command to the registry (pipe.PROJteam.co.uk): ![](https://civicadigital.atlassian.net/wiki/download/attachments/12910792/image2017-11-23_14-2-12.png?version=1&modificationDate=1511445736914&cacheVersion=1&api=v2)

#### Access the Drone Management Console:

1.  Open a browser and point it to the following URL: `http://your.drone.server.address:8000`, replacing `your.drone.server.address` with either the external IP of the server or it's DNS name, assuming it has one.
2.  Confirm that you are synchronising the system with the Bitbucket account and click through to view the repositories. All the repos in Bitbucket should show in the Management console.
3.  To enable a repo, click the grey slider button to the right of each repo. This will turn green and the settings option will appear as a link to the left of the button.
4.  Click on the settings icon next to the (now green) activation button.
5.  Tick all the radio buttons, leaving 'Project Visibility' as private and the Timeout option to the default 60 minutes.
6.  The repo is now activated and ready to process changes made to the master or branches of that repository.
7.  The docker image repository - on AWS - is accessed via an intermediate registry server (pipe.PROJteam.co.uk), which has the authentication details to allow pulls and pushes to S3 included in the start parameters set when the registry container is running. The setup and operating instructions for the registry server can be found here: [Docker Registry Server for S3](/wiki/spaces/PROJ/pages/12845244/Docker+Registry+Server+for+S3)

#### Initiating a build:

Drone automatically executes a build of the container once a change has been made to a Bitbucket repository - master or branch.The steps are:

1.  Drone downloads the contents of the target repository on Bitbucket.
2.  It then runs up the container from the definition given in the Dockerfile within the repository and confirms the container can run.
3.  Once the second step is complete, it then pushes the compiled Docker container to the S3 repo on AWS via the [Docker Registry Server for S3](/wiki/spaces/PROJ/pages/12845244/Docker+Registry+Server+for+S3).
4.  The image is now available to pull down through the same registry server from any of the Docker/Kubernetes servers/clusters. This can be run manually or initiated on a scheduled basis.

#### Setting Secrets:

Drone can parse secrets held on the Drone server into the environments of the containers that Drone runs to initiate builds. Secrets can be used to parse sensitive data in public repositories, such as passwords and usernames.

#### The process is as follows:

1.  Log into Drone and activate a repository, if you haven't arleady activated it. This is done by selecting the repo from the list and sliding the button to the right until it turns green.
2.  Click on the repository settings in Drone. Click on the three-bar icon next to your account icon in the upper-right-hand-corner of the screen and select 'Secrets'.
3.  ![](https://civicadigital.atlassian.net/wiki/download/attachments/12910792/image2018-1-2_16-33-57.png?version=1&modificationDate=1514910838844&cacheVersion=1&api=v2)<span class="confluence-embedded-file-wrapper confluence-embedded-manual-size" resolved="">![](https://civicadigital.atlassian.net/wiki/download/thumbnails/12910792/image2018-1-2_16-33-8.png?version=1&modificationDate=1514910790915&cacheVersion=1&api=v2&width=268&height=250)
4.  Enter the secret name and the value in the corresponding fields. Click Save to save the entry.
5.  ![](https://civicadigital.atlassian.net/wiki/download/attachments/12910792/image2018-1-2_16-35-38.png?version=1&modificationDate=1514910941158&cacheVersion=1&api=v2)
6.  To call the secret from within your environment, you have to put a 'secrets' line in your .drone.yml file. Below is an example:
7.  ![](https://civicadigital.atlassian.net/wiki/download/attachments/12910792/image2018-1-2_16-37-34.png?version=1&modificationDate=1514911056326&cacheVersion=1&api=v2)
8.  in the above, the secrets referred to are 'access_key' and 'access_key_id'. These have been entered as secrets in the repo entry in Drone.
9.  IMPORTANT NOTE: When referring to secrets within any part of the environment where they are called, they will be accessible as upper-case environment variables. For instance, the two secrets defined in the screen-shot above would be accessed as '$ACCESS_KEY' and '$ACCESS_KEY_ID'. So when the 'build-deploy.sh' script above calls these values, they are called as defined earlier in this entry (eg: $ACCESS_KEY).
