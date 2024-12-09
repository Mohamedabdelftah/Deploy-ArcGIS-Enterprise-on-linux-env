# Machine preparation 
## Update and Install Required Packages
```bash
sudo apt update
sudo apt upgrade
sudo apt install net-tools
# If OpenSSH is not installed
sudo apt install openssh-server
```

## Change Hostname
```bash
# Replace <your FQDN> with your Fully Qualified Domain Name
sudo hostnamectl set-hostname <your FQDN>
sudo nano /etc/hostname # Update to <your FQDN>
sudo nano /etc/hosts # Update 127.0.1.1 to <your FQDN> and delete the old name
sudo systemctl restart systemd-hostnamed
sudo reboot
# After reboot, check /etc/hostname to ensure changes were applied
```

## Update ulimit
```bash
sudo nano /etc/security/limits.conf
# Add the following lines
arcgis   soft   nofile   65535
arcgis   hard   nofile   65535

sudo nano /etc/pam.d/common-session
sudo nano /etc/pam.d/common-session-noninteractive
# Add this line to both files:
session required pam_limits.so
```


## Extract Installation Files
```bash
cd /home/arcgis/
mkdir sourceFile
cd sourceFile
mkdir server portal datastore webadaptor

sudo tar -xzvf serverFile.tar.gz -C /home/arcgis/sourceFile/server
sudo tar -xzvf datastore.tar.gz -C /home/arcgis/sourceFile/datastore
sudo tar -xzvf webadaptor.tar.gz -C /home/arcgis/sourceFile/webadaptor
sudo tar -xzvf portal.tar.gz -C /home/arcgis/sourceFile/portal

# Remove tar files
sudo rm serverFile.tar.gz datastore.tar.gz webadaptor.tar.gz portal.tar.gz
```
