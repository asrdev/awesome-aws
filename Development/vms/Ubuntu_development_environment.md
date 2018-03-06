# Development / VMs
## Ubuntu development environment ON LOCAL HARDWARE
### Credentials and specs
The development environment has been set up on an Ubuntu 16.04 virtual machine with a 4-core processor, 8GB of RAM and 100GB disk space.
### Changes
The Desktop Environment has been changed from Unity (default) to XFCE.

Unity did not support RDP but XFCE does.

When initially given access to the VM from the ISU it had a VNC service installed. It was not appropriate to continue using VNC instead of RDP as using VNC only allowed a single low-resolution viewport (1024x768) and applications such as Eclipse require much more screen real estate.
When using VNC to connect to the Unity DE we experienced a lot of latency.
XFCE is very light-weight and Unity is anything but. Unity provides no additional features that a developer may find useful, apart from a familiar user interface to the Ubuntu environment.
### Installed tools
The tools that were identified as needed in any Java microservice-oriented project were installed through the use of bash scripts. These scripts can be found on GitHub. The tools that were installed were:

- Docker
- Git
- Google Chrome
- Java 8
- Maven
- Spring Tool Suite
- 
All these tools have been installed using the latest version, and the scripts can easily be upgraded by editing the VERSION variable inside each script. The checked out repository can be found in /home/localuser/dev-env. Here you can also find additional scripts for tools that developers may choose to install, but these tools should be left up to the individual developers. More information can be found in the README.md file in the Gir repository.

The tools listed in dev-env have all been verified against both Debian Jessie and Ubuntu 16.04. A couple of the scripts will ask for the distribution upon execution.

### Concerns and comments
The current way we have Ubuntu set up should more than suffice as a proof of concept in the short term and good enough for a trial run with microservices and Docker with the training team.

*Note* that from conversations with the ISU the virtualization software (Hyper-V Server 2012 R2) that we use to spin up these VMs do not support any other Linux images than Ubuntu 16.04/Ubuntu 14.04 or an old version of CentOS. It is highly recommended that this software is updated to a more current version. Initially the idea was to use a Debian based system, as it is famously stable, and is often used as a server for deploying applications and thus developing on the same system would be an advantage. As Debian is more light-weight than Ubuntu, it is still recommended to go this route, should it be possible in the future. Should we be locked to Ubuntu, we should consider changing to Xubuntu, which is the 'native' version of Ubuntu + XFCE. The way we have gone about installing XFCE on the VM is not ideal, and we expect to find tweaks that needs to be made due to the somewhat forced nature of the Unity-to-XFCE transition.

During our setup and continued use of this VM we found that we were depending on the ISU fairly often. We have no control over the virtualization tool, and if, for example, the computer needs to be restarted after it freezes then we would have no way of starting the VM after having shut it down. A viable solution going forward would require more control of the VMs/Virtualization software. Ideally each developer would have access to their own VM. In particular, if we were to try and automate the setup of the development environment from a base install, we would need repeated access to the virtualization software to test the scripts/recipes used.



