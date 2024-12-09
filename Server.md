
## Install ArcGIS Server Silently

```shell
ulimit -Sn 65535
ulimit -Hn 65535
```

```bash
# Ensure you have the license file
cd /home/arcgis/-sourceFile/server/
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