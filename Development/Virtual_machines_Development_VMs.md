# Development
## Virtual machines (Development VMs)

We should be using Virtual machines to perform our development
The child pages of this page contain details about which VMs are available and how to configure them.

1. [Configuring Ubuntu for Remote Desktop access](vms/Configuring_Ubuntu_for_Remote_Desktop_access.md)
1. Fixing the Copy Paste Issue
https://askubuntu.com/questions/876812/for-xrdp-how-t-copy-paste-back-to-windows-host
1. Setting the Keyboard to be UK standard
Navigate to https://github.com/billhartcivica/egar-ubuntu. Download the [km-0809.ini](https://raw.githubusercontent.com/billhartcivica/egar-ubuntu/master/km-0809.ini) file. Copy this so the `/etc/xrdp` folder and exit and re-establish the RDP session. The UK keyboard should now be enabled.
`wget -q https://raw.githubusercontent.com/billhartcivica/egar-ubuntu/master/km-0809.ini`
`cp  /km-0809.ini /etc/xrdp`
1. [Ubuntu development environment ON LOCAL HARDWARE](vms/Ubuntu_development_environment.md)
1. [Upgrade Ubuntu](vms/Upgrade_Ubuntu.md)
1. [Upgrading XRDP](vms/Upgrading_XRDP.md)