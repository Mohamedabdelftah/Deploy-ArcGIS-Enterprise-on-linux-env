
# Install ArcGIS Data Store
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