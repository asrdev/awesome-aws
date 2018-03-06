# Development / VMs
## Upgrading XRDP
### First...
First you need to upgrade to the latest version of Ubuntu.
```
sudo -s
apt install update-manager-core
cp /etc/update-manager/release-upgrades /etc/update-manager/release-upgrades.bak
vi /etc/update-manager/release-upgrades
```
Change the line “Prompt=LTS” to “Prompt=normal”, save and exit

The next setup can take a while, 30+ mins, also there will be some prompts along the way.
```
sudo do-release-upgrade -d
```
During installation you may be prompted to update a number of files. Please respond in the following way if and when prompted:

GRUB Install - Do not install startwm.sh - Keep current version release-upgrades - Update

### Setting Xsession up correctly...
Backup the original
```
cp /etc/X11/Xsession /etc/X11/Xsession.backup
```
Get the Xsession script from https://civica-digital-internal.visualstudio.com/_git/Sapphire%20(9700-028)
Copy it to `/etc/X11/Xsession`.

Then
```
chmod uga+rx /etc/X11/Xsession
chmod u+w /etc/X11/Xsession
```
Note: To be on the safe-side run:
```
sudo apt-get install dos2unix
sudo dos2unix /etc/X11/Xsession /etc/X11/Xsession
restart xrdp just to be sure....
sudo service xrdp restart
```
Once restarted it is worth checking that all related xrdp services have started successfully. It can be the case that the xrdp-sesman service does not start or is disabled. Run the following to check the status of the services and if need be enable and restart the xrdp-sesman service.
```
sudo service xrdp status
sudo systemctl enable xrdp-sesman.service
sudo service xrdp-sesman start
```
### Connecting from windows
In windows RDP client settings, ensure that under the "Experience" tab that it's set to "Detect connection quality automatically" before you connect.

Once established ensure you use the "sesman-xorgxrdp" profile when logging in.

To get your keyboard settings back
"Applications->Settings->Keyboard" under layout tab. Change the model to "Generic 105-key (Intl)". Under the Layout section add "English (UK)" and move it to the top of the list.

Next...
```
sudo apt-get install xfce4-xkb-plugin
```
Next, `Applications->Settings->Panel` items tab, add "Keyboards Layout". Now you will see a flag icon on your task bar, click it until it shows the union jack.


