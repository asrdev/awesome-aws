# Keycloak 
## Server - updating
Keycloak is the authentication system provided by the HO and operates on it's own isolated instance. These steps go about how to update the keycloak server.

### Step-by-step guide
To update the Keycloak server

1. Log onto the Keycloak server via SSH using he PROJ ssh key (ssh -i "PROJ.pem" centos@ec2-35-177-238-18.eu-west-2.compute.amazonaws.com)
1. Once logged on, switch to root (sudo su -)
1. If present, rename the `PROJ-keycloak` folder to `PROJ-keycloak-XX.XX.XX`, using today's date for the triplet of X's.
1. Run 'git clone https://bhartcivica@bitbucket.org/PROJ-civica/PROJ-keycloak.git'.
1. Move into the `PROJ-keycloak` folder (cd PROJ-keycloak).
If using a branch of `PROJ-keycloak`, switch to the git branch.
1. From the command line, while in the folder, type 'npm install'.
1. Next, run the `start-keycloak.sh` script (eg: `./start-keycloak.sh`).
1. Wait for Keycloak to come up. Three images should be running (`keycloak-mysql`, `maildev` and `mariadb`).
1. Check that all three containers are running by typing `docker ps`. This will also show which ports each container is listening on.
1. The keycloak server is reference directly by it's public domain name - `key.PROJteam.co.uk`, on port `8090`. So accessing it would mean putting the following URL in your browser: `http://key.PROJteam.co.uk:8090`.
1. Note that if the system is starting from scratch (rebooted), then the settings will have been zeroed out. To access the server via http, log into the development workstation using Microsoft's RDP client on IP `NNN.NNN.NNN.NNN`. Log in with the standard dev credentials. Navigate to `http://10.10.10.4:8090` and log in with the standard admin credentials to make the changes you require.
1. Recommended: Under 'realm settings' under the 'Login' tab, change the 'Require SSL' setting to 'none'. This will allow external machines to access the keycloak server console directly from a browser in the Civica offices.
