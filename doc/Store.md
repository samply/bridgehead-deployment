# Store

The Store stores patient data as part of the [GBA-Bridgehead](https://github.com/samply/bridgehead-deployment).

You can access the Store under `http://localhost:8081`. A simple test is fetching the import XSD under `http://localhost:8081/importXSD`.

The interface uses the OSSE XML schemas defined [here](http://schema.samply.de/osse/).

Most work is to import patient/sample data into the store. You have to create a xml file which will be tested against our Metadata Repository.
The current namespace, where all data-elements are defined and all GBA-Components work with, can be found under http://mdr.germanbiobanknode.de/view.xhtml?namespace=mdr16

Instructions for import can be found [here (IMPORT.md)](IMPORT.md)


## Run ([Docker](#docker) or [Manual](#manual))

### Docker

Use the Docker-Compose of the [GBA-Bridgehead](https://github.com/samply/bridgehead-deployment) and run only the Store with:

```
docker-compose up store
```

### Manual

Requirements:

- [Database](#database)
- [Tomcat](#tomcat)
- The Store webapp as .war file: [download](https://maven.samply.de/nexus/content/repositories/oss-releases/de/samply/store-rest/4.2.6/store-rest-4.2.6.war)

Steps:

- Delete folder ${tomcat.home}/webapps/ROOT.
- Rename .war file to ROOT.war
- Copy ROOT.war to ${tomcat.home}/webapps/ 

Start tomcat by executing ${tomcat.home}/bin/startup.sh (Windows: startup.bat) or by running the tomcat-service if you [created one.](#tomcat-service-for-autostart)


## Environment

### Database

The Open-Source database Postresql 9.6 is used. The database connection uses the connection pool of Tomcat. 

This webapp needs schema '**samply**' in the database '**samply.store**' under user '**samply**' and password '**samply**' under port `5432`.

To change these settings, see context.xml (described under [Configurations](#Configurations)).

- Follow installation for port **5432**

  - Windows: https://www.enterprisedb.com/downloads/postgres-postgresql-downloads
  - Linux Mint:

  ```
  sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main" > /etc/apt/sources.list.d/postgresql.list'
  
  wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
  
  sudo apt-get install postgresql-9.6
  ```

  ​	Other Linux:

  ```
  sudo add-apt-repository "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -sc)-pgdg main"
  
  wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
  
  sudo apt-get install postgresql-9.6
  ```

- Create database and user:

  - pgAdmin installed: Having Server opened, under "Databases": rightclick on "Login/Group Roles". Select "Create"?"Login/Group Role". Tab Generel: Enter Name. Tab Definition: Enter Password. Tab Privileges: enable "Can Login?" and "Superuser". By creating new Databases, select this user as "Owner"*

  - command line: 

    ```
    (sudo su postgres)
    psql
    CREATE DATABASE "samply.store";
    CREATE USER samply WITH PASSWORD 'samply';
    GRANT ALL PRIVILEGES ON DATABASE "samply.store" to samply;
    ```


### Tomcat

Requirements:

   - [Java 8](#java)

1. Download and unzip Tomcat: https://tomcat.apache.org/download-80.cgi

2. Change ports: Every webapp has its own tomcat, so change ports for Store-Tomcat in ${tomcat.base}/conf/server.xml:

      ```
      ...
      ...<connector port="8081" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8101" />...
      ...
      ...<connector port="8001" protocol="AJP/1.3" redirectPort="8101" /> ...
      ...
      ...<Server port="8201" shutdown="SHUTDOWN">...
      ...
      ```

   

   ### Java

   Is a dependency of Tomcat,

   if you install different jre versions on this machine, set jre 8 for tomcat by creating a so called "setenv.sh".

   Linux: [OpenJDK](https://openjdk.java.net/install/)

   Windows: [Oracle](https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html) 

### Configurations

These configuration files are used:

```
src/main/java/webapp/WEB-INF/conf/
(backend.xml, log4j2.xml, mdrconfig.xml, query.configs.sql, user.sql)

src/main/java/webapp/META-INF/
(context.xml)
```

The `context.xml` will be auto-copied by tomcat at startup to `${tomcat.base}/conf/Catalina/localhost/ROOT.xml`.
This file will not be overwritten by updating the WAR file due to tomcat settings.

All files under `WEB-INF/conf` will always be found from FileFinder as ultimate fallback.

If you want to save your configurations, copy all files under `WEB-INF/conf` (tomcat or code source) to `${tomcat.base}/conf`.

**IntelliJ** creates a *tomcat.base* directory for every startup of the application. So save your configuration files to *tomcat.home* and it will copy these files and logs every time to *tomcat.base*. You will see the paths at startup in the first lines of the console output.

According to the `log4j2.xml`, all logs can be found in ${tomcat.base}/logs/connector.

To use a **proxy**, set your url in file **proxy.xml** and in **mdrconfig.xml** activate proxy and set path to your **proxy.xml** which should stay in ${tomcat.base}.

### Connections

Ingoing (secured with basic auth):

```
POST /import | import patient data
GET /importXSD | Return xsd for import

POST /requests
GET  /requests/{id}/result
GET  /requests/{id}/stats
GET  /info

POST /delete
GET  /access_token

GET  /testAuth
POST /saveOrUpdateUser
```


Outgoing:

```
MDR | url and proxy in (mdrconfig.xml)

```


### Productive Settings

#### Tomcat service for autostart

​	Linux:

​		Remember path of output:

```
sudo update-java-alternatives -l

```

​		Create new service file:

```
sudo nano /etc/systemd/system/tomcat-store.service

```

​		Copy the remembered path to JAVA_HOME and add `/jre` to the end of this path, also check 		tomcat path:

```
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64/jre
Environment=CATALINA_PID=/opt/tomcat-store/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat-store
Environment=CATALINA_BASE=/opt/tomcat-store
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/opt/tomcat-store/bin/startup.sh
ExecStop=/opt/tomcat-store/bin/shutdown.sh

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target

```



​	Windows: 

​		Follow installer on: https://tomcat.apache.org/download-80.cgi

​		And check service (one per app/tomcat): http://www.ansoncheunghk.info/article/5-steps-install-multiple-apache-tomcat-instance-windows