
# Install Portal for ArcGIS
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
