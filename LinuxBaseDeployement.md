# Installing ArcGIS Enterprise 11.2 on Linux server

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

# Apply changes
ulimit -Sn 65535
ulimit -Hn 65535
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

## Install ArcGIS Server Silently
```bash
# Ensure you have the license file
cd /home/arcgis/sourceFile/server/
./Setup -m silent -l yes -a <fullpath to .ecp or .prvc file> -d /arcgis/server/ -v

# If encountering Java-related errors, run the following:
sudo apt install libxtst6 libxi6 libxrender1 libxrandr2 libgtk2.0-0
```

## Post-Installation: ArcGIS Server Service
```bash
# Switch to root admin
# Copy service file to systemctl
sudo cp /home/arcgis/server/framework/etc/scripts/arcgisserver.service /etc/systemd/system
cd /etc/systemd/system
sudo chmod 600 arcgisserver.service

systemctl enable arcgisserver.service
systemctl stop arcgisserver.service
systemctl start arcgisserver.service
systemctl status arcgisserver.service

# Uncomment and edit the following line in the arcgisserver.service file:
# TasksMax=512

# If installvariables.properties is missing or incomplete, create/edit it:
sudo nano /home/arcgis/server/.Setup/Uninstall_ArcGISServer/installvariables.properties
# Add the following:
INSTALLDIR=/home/arcgis/server
SERVERNAME=<FQDN>
SIMPLE_VERSION=<VERSION>
ESRI_PROP_FILE_PATH=/home/arcgis/.ESRI.properties.gis.esri.local.11.2
```

## Create Site Silently
```bash
cd /home/arcgis/server/tools/createsite/
./createsite.sh -u siteadmin -p siteadmin -d /home/arcgis/server/usr/directories -c /home/arcgis/server/usr/config-store
```

## Install ArcGIS Data Store
```bash
ulimit -Sn 65535
ulimit -Hn 65535
sudo tar -xzvf datastore.tar.gz -C /home/arcgis/sourceFile/datastore
sudo rm datastore.tar.gz
mkdir /home/arcgis/datastore

cd /home/arcgis/sourceFile/datastore/
./Setup -m silent -l yes -d /home/arcgis/datastore -f Relational -v
```

## Configure ArcGIS DataStore Service
```bash
cd ~/server/framework/etc/scripts/
sudo cp arcgisdatastore.service /etc/systemd/system
cd /etc/systemd/system
sudo chmod 600 arcgisdatastore.service

systemctl enable arcgisdatastore.service
systemctl stop arcgisdatastore.service
systemctl start arcgisdatastore.service
systemctl status arcgisdatastore.service
```

### Configure Data Store
```bash
cd /home/arcgis/datastore/tools/
./configuredatastore.sh https://<FQDN>:6443/ siteadmin siteadmin /home/arcgis/datastore/usr/data/ --stores relational,tileCache --mode primaryStandby
```

## Install Portal for ArcGIS
```bash
cd ~
mkdir portal
cd /filecontainFile
sudo tar -xzvf Portal_for_ArcGIS_Linux_112_188338.tar.gz -C ./portal/
cd portal/PortalForArcGIS/
./Setup -m silent -l yes -d /home/arcgis/portal/

cd ~/portal/framework/etc/
sudo cp arcgisportal.service /etc/systemd/system
cd /etc/systemd/system
sudo chmod 600 arcgisportal.service

systemctl enable arcgisportal.service
systemctl stop arcgisportal.service
systemctl start arcgisportal.service
systemctl status arcgisportal.service
```

### Create Portal Site
- Copy license file to VM
- ```bash
  cd ~/portal/tools/createportal/
  ./listadministratorusertypes.sh -lf <pathToLicenseFile>
  ```
 - the previouse tool will list for you the avilable user types choose on of them to assign it to the portaladmin at the next tool

```bash
# Select user type and run:
./createportal.sh -fn portal -ln admin -u portaladmin -p portaladmin123 -e portaladmin@esri.com -qi 1 -qa cairo -d /home/arcgis/portal/usr/arcgisportal/ -lf <pathToLicenseFile> -ut advancedUT
```

## Install Supported Tomcat 9
```bash
sudo apt update
sudo apt install openjdk-11-jdk wget tar
wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.97/bin/apache-tomcat-9.0.97.tar.gz
sudo tar xzvf apache-tomcat-9.0.97.tar.gz -C /opt
sudo mv /opt/apache-tomcat-9.0.97 /opt/tomcat9

nano ~/.bashrc
source ~/.bashrc
sudo chown -R $USER:$USER /opt/tomcat9
sudo setcap CAP_NET_BIND_SERVICE=+eip $(readlink -f $(which java))

sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw reload
```

### Configure Tomcat Service
```bash
sudo nano /etc/systemd/system/tomcat9.service
# Add the following:
[Unit]
Description=Apache Tomcat 9
After=network.target

[Service]
Type=forking
ExecStart=/opt/tomcat9/bin/startup.sh
ExecStop=/opt/tomcat9/bin/shutdown.sh
User=arcgis
Group=arcgis
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target

sudo systemctl daemon-reload
```

### Edit Tomcat Configuration
```bash
# Edit tomcat-users.xml
sudo nano /opt/tomcat9/conf/tomcat-users.xml
<tomcat-users>
    <role rolename="manager-gui"/>
    <role rolename="manager-script"/>
    <user username="admin" password="password" roles="manager-gui,manager-script"/>
</tomcat-users>

# Edit manager/web.xml
sudo nano /opt/tomcat9/webapps/manager/WEB-INF/web.xml
<security-constraint>
    <web-resource-collection>
        <web-resource-name>Manager</web-resource-name>
        <url-pattern>/manager/*</url-pattern>
    </web-resource-collection>
    <auth-constraint>
        <role-name>manager-gui</role-name>
    </auth-constraint>
    <user-data-constraint>
        <transport-guarantee>NONE</transport-guarantee>
    </user-data-constraint>
</security-constraint>

# Edit manager/META-INF/context.xml
sudo nano /opt/tomcat9/webapps/manager/META-INF/context.xml
<!-- Uncomment the following lines:
<Valve className="org.apache.catalina.valves.RemoteAddrValve"
       allow="127\\.\\d+\\.\\d+\\.\\d+|::1|0:0:0:0:0:0:0:1" />
-->

sudo systemctl restart tomcat9
```

## Create Self-Signed Certificate

```bash
keytool -genkeypair -alias tomcat -keyalg RSA -keysize 2048 -keystore /opt/tomcat9/conf/keystore.jks -validity 365
```

- Answer the questions prompted by the certificate generation.
- Use the password: `P@ssw0rd`.

---

## Update the Tomcat Configuration

Open the `server.xml` file:

```bash
sudo nano /opt/tomcat9/conf/server.xml
```

Add the following connectors:

```xml
<Connector port="80" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="443" />

<Connector protocol="org.apache.coyote.http11.Http11NioProtocol"
           port="443"
           maxThreads="150"
           maxParameterCount="1000"
           scheme="https"
           secure="true"
           SSLEnabled="true"
           keystoreFile="/opt/tomcat9/conf/keystore.jks"
           keystorePass="P@ssw0rd"
           clientAuth="false"
           sslProtocol="TLS" />
```

---

## Test the Configuration

- Test HTTP: [http://FQDN](http://FQDN)
- Test HTTPS: [https://FQDN](https://FQDN)

**Web server setup is complete.**

---

# Install Web Adaptor

Navigate to the directory containing the installation file:

```bash
cd /filecontainFile > ArcGIS_Web_Adaptor_Java_Linux_112_188341.tar.gz
```

Extract the Web Adaptor files:

```bash
sudo tar -xzvf ArcGIS_Web_Adaptor_Java_Linux_112_188341.tar.gz -C ./webadaptor/
```

Run the Web Adaptor setup:

```bash
cd /home/arcgis/sourceFile/webadaptor/WebAdaptor/
./Setup -m silent -l yes -v
```

---

# Configure Web Adaptor for Server

Copy the `arcgis.war` file and change name to server:

```bash
cp arcgis.war /opt/tomcat9/webapps/server.war
```

Run the configuration script:

```bash
cd tools
./configurewebadaptor.sh -m server -w https://gis.mapssystem.local/server/webadaptor -g https://gis.mapssystem.local:6443 -u siteadmin -p siteadmin -a true -v
```

---

# Configure Web Adaptor for Portal

Copy the `arcgis.war` file and change the name to portal :

```bash
cp arcgis.war /opt/tomcat9/webapps/portal.war
```

Run the configuration script:

```bash
cd tools
./configurewebadaptor.sh -m portal -w https://gis.mapssystem.local/portal/webadaptor -g https://gis.mapssystem.local:7443 -u portaladmin -p portaladmin123 -a true -v
```

