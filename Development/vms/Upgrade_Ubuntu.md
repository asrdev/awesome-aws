# Development / VMs
## Upgrade Ubuntu
Check the version of Linux
```
uname -r
```
Then check the release
```
lsb_release -a
```
This process will upgrade your version of Linux. Advice on the upgrade process can be found on line (ie here).
```
sudo -s
apt install update-manager-core
cp /etc/update-manager/release-upgrades /etc/update-manager/release-upgrades.bak
vi /etc/update-manager/release-upgrades
```
Change the line “Prompt=LTS” to “Prompt=normal”, save and exit

The next setup can take a while, 30+ mins, also there will be some prompts along the way.
```
sudo do-release-upgrade
```
During installation you may be prompted to update a number of files. Please respond in the following way if and when prompted:

GRUB Install - Do not install `startwm.sh` - Keep current version release-upgrades - Update


