# Development / VMs
## Configuring Ubuntu for Remote Desktop access

Ubuntu is a Linux based operating system. The ISU can provide us with a clean Ubuntu installation running on a virtual machine, but it is not their responsibility to configure it for our needs.

This document describes the process of configuring an Ubuntu machine so it can be accessed via the Windows standard Remote Desktop Protocol (RDP).

### Known problems
The standard Ubuntu Desktop Environment, Unity, is not compatible with RDP so an alternative must be used.
Dual monitors should work, but we haven't got them working yet.
### List of Software to be installed
1. Xfce 4
This is a popular desktop environment which aims to be lightweight and easy to use.
1. Xrdp
This is a program to support Microsoft's Remote Desktop Protocol.

### Configuration
We need to be logged in to the machine in order to perform the configuration.

This can be done from a terminal on a Windows machine using SSH.
```
$ ssh <username>@<hostname>
```
Replace `<username>` and `<hostname>` with the credentials you have been provided with and enter the password when prompted.

It's good practice to make sure that the system is up to date before installing new software.
```
$ sudo apt-get update
$ sudo apt-get upgrade
```

### Install Xfce 4 and Xrdp
```
$ sudo apt-get install Xfce4 xrdp
```
#### Tell Ubuntu to use Xfce instead of Unity
This is done by editing the x session configuration file in a user's home directory.
```
$ echo xfce4-session > ~/.xsession
```
- Check the contents of the file with this command.
```
$ cat ~/.xsession
```
- Change the firewall to allow RDP connections
Ubuntu comes with a firewall which by default prevents connections on port `3389`, the default RDP port.

Why the port is not opened when installing Xrdp is a mystery. 
To open the port we use the Ubuntu command `ufw`.
```
$ sudo ufw enable # in case it is not already enabled
$ sudo ufw allow 3389/tcp
```
Ensure Xrdp runs as a service
- Check that Xrdp will run when the machine starts. This is done by checking the output of the following command.
```
$ sudo service --status-all
```
If there is an entry for Xrdp, then Xrdp will run at start up. Now restart Xrdp.
```
$ sudo service xrdp restart
```
### Connect over RDP
On a Windows machine open the Remote Desktop Connection tool. Enter the name or the IP address you have been given into the Computer field. The user name can be left blank.
- Click on the `Show Options` button to configure the connection settings to your liking.

Particularly useful are the `Display` and `Experience` tabs.
- Click the `Connect` button to connect to the Virtual Machine. You will be prompted to enter the credentials of the user you want to log in as. Once entered you should be logged in to the Xfce desktop.

### Change tab behaviour
By default Xfce uses the tab key to switch between different windows of the same application. This may be annoying to those used to tab-complete functionality. 

To disable this: 
- Go to `Application Menu > Settings > Window Manager`
- Click on the `Keyboard tab`.
- Clear the Switch window for same application setting.
- Reconnect to session after un-docking
- Run the following to find port you want to attach to (e.g. 5910):
```
netstat -plnt | grep Xvnc
```
- Edit `xrdp.ini` and change the port of [xrdp1] from -1 to the port you want to connect to:
```
sudo vim /etc/xrdp/xrdp.ini
```
- Reconnect with Remote Desktop and it will reattach to your VNC session of choice.

*Note*: Resolution can not be changed since geometry is set after the vnc server has been started.