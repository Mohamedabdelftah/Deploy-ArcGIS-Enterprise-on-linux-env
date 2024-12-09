
## Configure Tomcat Service
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

