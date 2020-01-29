# Windows

* Database
    * Follow instructions: [Download](https://www.enterprisedb.com/thank-you-downloading-postgresql?anid=1256732)
    * Open pgAdmin, "Servers", rightclick "PostgreSQL 9.6", "Create":
        * "Login/Group Role..."
            * "General": Name=`samply`
            * "Definition": Password=`samply` (Change if host not otherwise protected)
            * "Privileges": enable "Can Login" and "Superuser".
        * "Database..."
            * "General": Database=`samply.connector`, Owner=`samply`

* Tomcat
    * Look if ports `8082`, `8002`, `8202` are free, otherwise choose different ones in next step:
        * "Command Prompt" as administrator: `netstat -ab`
    * Follow instructions until "Configuration": [Download](http://ftp.halifax.rwth-aachen.de/apache/tomcat/tomcat-8/v8.5.47/bin/apache-tomcat-8.5.47.exe)
        * `8202` for "Server shutdown Port"
        * `8082` for "HTTP/1.1 Connector Port"
        * `8002` for "AJP" 
        * `Tomcat8Connector` for "Service name"

* Webapp
    * [Download](https://github.com/samply/bridgehead-deployment/raw/master/share-client-5.6.4.war) Connector war file
    * Delete all folders in `${tomcat}/webapps/`
    * Rename share-client-5.6.4.war to ROOT.war and move to `${tomcat}/webapps/`
    * Startup Tomcat once: Open "Command Prompt" as administrator
        * `${tomcat}/bin/startup.bat`
        * `${tomcat}/bin/shutdown.bat`

* Configuration
    * Copy files from `${tomcat}/webapps/ROOT/WEB-INF/conf/` to `${tomcat}/conf/`:
        * `log4j2.xml`,`mailSending.xml`,`samply_common_urls.xml`,`samply_common_operator.xml`, `samply_common_config.xml`, `samply_bridgehead_info.xml`
    * Copy file `${tomcat}/webapps/ROOT/META-INF/`**context.xml** to `${tomcat}/conf/`:
    * Open file `${tomcat}/conf/`**samply_bridgehead_info.xml** and set `<bi:queryLanguage>QUERY</bi:queryLanguage>` to **CQL**
    * Open file `${tomcat}/conf/`**samply_common_urls.xml** and set `<com:ldmUrl>http://localhost:8081</com:ldmUrl>` to **localhost:8080/fhir**
    * Delete configuration files if exist more than two times: "Command Prompt" as administrator:
        * `where /r c: log4j2.xml mailSending.xml samply_common_urls.xml samply_common_operator.xml samply_common_config.xml samply_bridgehead_info.xml`
    * (Optional) Set proxy in file `${tomcat}/conf/`**samply_common_config.xml**
        * Under "HTTP" and "HTTPS" tag, set url and port like this: `<Url>url:port</Url>`
    * (Optional) Set database connection settings in file `${tomcat}/conf/`**context.xml**






# Linux

* Database
```
sudo add-apt-repository "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -sc)-pgdg main"
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add
sudo apt-get install postgresql-9.6
sudo su postgres
psql
CREATE DATABASE "samply.connector";
CREATE USER samply WITH PASSWORD 'samply';
GRANT samply TO root;
GRANT ALL PRIVILEGES ON DATABASE "samply.connector" to samply;
exit
exit
```

* Tomcat
```
sudo apt install default-jdk
sudo useradd -m -U -d /opt/tomcat -s /bin/false tomcat
cd /tmp
wget http://www-us.apache.org/dist/tomcat/tomcat-8/v8.5.47/bin/apache-tomcat-8.5.47.zip
unzip apache-tomcat-8.5.47.zip
sudo mv apache-tomcat-8.5.47 tomcat-connector
sudo mkdir -p /opt/tomcat-connector
sudo mv tomcat-connector /opt/
sudo chown -R tomcat: /opt/tomcat-connector
sudo sh -c 'chmod +x /opt/tomcat-connector/bin/*.sh'
```

* Create Tomcat Service:
```
sudo bash -c 'echo "[Unit]
Description=Tomcat 8.5 Connector GBA
After=network.target
[Service]
Type=forking
User=tomcat
Group=tomcat
Environment="JAVA_HOME=/usr/lib/jvm/default-java"
Environment="JAVA_OPTS=-Djava.security.egd=file:///dev/urandom"
Environment="CATALINA_BASE=/opt/tomcat-connector"
Environment="CATALINA_HOME=/opt/tomcat-connector"
Environment="CATALINA_PID=/opt/tomcat-connector/temp/tomcat.pid"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"
ExecStart=/opt/tomcat-connector/bin/startup.sh
ExecStop=/opt/tomcat-connector/bin/shutdown.sh
[Install]
WantedBy=multi-user.target" > /etc/systemd/system/tomcat-connector.service'
```

* Look if ports `8082`, `8002`, `8102`, `8202` are free:
    * `sudo lsof -i -P -n | grep LISTEN`
        * Change Ports in `/opt/tomcat-connector/conf/server.xml`:
            ```
            ...
            ...<Server port="8202" shutdown="SHUTDOWN">...
            ...
            ...<connector port="8082" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8102" />...
            ...
            ...<connector port="8002" protocol="AJP/1.3" redirectPort="8102" /> ...
            ...
            ```
        * `sudo ufw allow 8082/tcp`

* Webapp
```
sudo rm -r /opt/tomcat-connector/webapps/*
cd /tmp
wget https://github.com/samply/bridgehead-deployment/raw/master/share-client-5.6.4.war
mv /tmp/share-client-5.6.4.war /opt/tomcat-connector/webapps/ROOT.war
sudo systemctl daemon-reload
sudo systemctl start tomcat-connector
sudo systemctl stop tomcat-connector
```

* Save configuration files in case of future Webapp updates
```
sudo cp /opt/tomcat-connector/webapps/ROOT/WEB-INF/conf/{log4j2.xml,mailSending.xml,samply_common_urls.xml,samply_common_config.xml,samply_common_operator.xml,samply_bridgehead_info.xml} /opt/tomcat-connector/conf/
sudo cp /opt/tomcat-connector/webapps/ROOT/META-INF/context.xml /opt/tomcat-connector/conf/
```

* Enable CQL
```
sudo sed -i "s%<bi:queryLanguage>QUERY</bi:queryLanguage>%<bi:queryLanguage>CQL</bi:queryLanguage>%" /opt/tomcat-connector/conf/samply_bridgehead_info.xml
```

* Set Store URL
```
sudo sed -i "s%<com:ldmUrl>http://localhost:8081</com:ldmUrl>%<com:ldmUrl>http://localhost:8080/fhir</com:ldmUrl>%" /opt/tomcat-connector/conf/samply_common_urls.xml
```

* Delete configuration files if exist more than two times
```
sudo find / -name "log4j2.xml" -o -name "mailSending.xml" -o -name "samply_common_urls.xml" -o -name "samply_common_operator.xml" -o -name "samply_bridgehead_info.xml" -o -name "samply_common_config.xml"
```

* (Optional) Set proxy in "HTTP" and "HTTPS" column like this: <Url>url:port</Url>:
```
sudo nano /opt/tomcat-connector/conf/samply_common_config.xml
```

* (Optional) Set database connection settings:
```
sudo nano /opt/tomcat-connector/conf/context.xml
```

* Enable startup after reboot
```
sudo chown -R tomcat: /opt/tomcat-connector
sudo systemctl start tomcat-connector
sudo systemctl status tomcat-connector
sudo systemctl enable tomcat-connector
```
