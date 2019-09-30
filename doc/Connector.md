# Connector

The Connector (or "Samply Share Client" or "Teiler" ) connects Store and [Searchbroker](https://code.mitro.dkfz.de/projects/SHAR/repos/samply.share.broker.rest) as part of the [Bridgehead-Deployment](https://github.com/samply/bridgehead-deployment).
Currently the [classic Store](https://code.mitro.dkfz.de/projects/STOR/repos/samply.store.rest) is used, in future [Blaze](https://github.com/life-research/blaze).

You can access the Connector under http://localhost:8082 and login under <http://localhost:8082/login.xhtml> (default credentials are **admin**, **adminpass**).
By default the Connector and Store use the same default credentials (**local_admin**, **local_admin**).

To change the default credentials for Connector & Store add a new user to the Connector and add credentials satisfying 

|Key|Value|
|---|---|
|Target| Local Datamanagement|
|CredentialType| Basic |
|Username| [username of new user] |
|Password| [password of new user] |
|Domain||
|Workstation||

If the Store runs while logging in the first time with this new user, the default credentials of the Store get deactivated. From now on, Connector and Store share the same new credentials instead of the old ones.

To register a Searchbroker, see [Bridgehead-Deployment](https://github.com/samply/bridgehead-deployment#connect-sample-locator).


## Run ([Docker](#docker) or [Manual](#manual))

### Docker

Docker network:

    docker network create gba

Database:

    docker run \
    --name pg-connector \
    --network=gba \
    -e POSTGRES_USER=samply \
    -e POSTGRES_DB=samply.connector \
    -e POSTGRES_PASSWORD=samply \
    -p 5432:5432 \
    postgres:9.6

Connector:

    docker run \
    --name=connector \
    --network=gba \
    -p 8082:8080 \
    -e POSTGRES_HOST='pg-connector' \
    -e POSTGRES_DB='samply.connector' \
    -e POSTGRES_USER='samply' \
    -e POSTGRES_PASS='samply' \
    -e MDR_URL='https://mdr.germanbiobanknode.de/v3/api/mdr' \
    -e STORE_URL='http://store:8080' \
    -e QUERY_LANGUAGE='QUERY' \
    -e CATALINA_OPTS='"-Xmx2g"' \
    martinbreu/connector:5.6.0


### All Environments of Connector container

| Name                | Default  | Description                                                  |
| ------------------- | -------- | ------------------------------------------------------------ |
| POSTGRES_HOST       |          | Database base url                                            |
| POSTGRES_PORT       | 5432     | Database port                                                |
| POSTGRES_DB         |          | Database name                                                |
| POSTGRES_USER       |          | Database user                                                |
| POSTGRES_PASS       |          | Database password                                            |
| HTTP_PROXY          |          | URL of the HTTP proxy to use for outgoing connections, "url:port"; enables proxy usage if set |
| PROXY_USER          |          | User of the proxy account                                    |
| PROXY_PASS          |          | Password of the proxy account                                |
| STORE_URL           |          | Base url to Store                                            |
| MDR_URL             |          | Url of the MDR server                                        |
| OPERATOR_FIRST_NAME |          | First name from the bridgehead admin                         |
| OPERATOR_LAST_NAME  |          | Last name from the bridgehead admin                          |
| OPERATOR_EMAIL      |          | Email from the bridgehead admin                              |
| OPERATOR_PHONE      |          | Phone number from the bridgehead admin                       |
| MAIL_HOST           |          | Host of email server                                         |
| MAIL_PORT           | 25       | Port of email server                                         |
| MAIL_PROTOCOL       | smtp     | Protocol of email server                                     |
| MAIL_FROM_ADDRESS   |          | Address for email sender                                     |
| MAIL_FROM_NAME      |          | Name for email sender                                        |
| CATALINA_OPTS       | "-Xmx1g" | JVM options for Tomcat of Connector                          |
| QUERY_LANGUAGE      | QUERY    | `QUERY` for Samply Store, `CQL` for Blaze                    |


### Manual

Requirements:

- [Database](#database)
- [Tomcat](#tomcat)
- The Connector webapp as .war file: [download](https://maven.samply.de/nexus/content/repositories/oss-releases/de/samply/share-client/5.6.2/share-client-5.6.2.war)


Steps:

- Delete folder ${tomcat.home}/webapps/ROOT.
- Rename .war file to ROOT.war
- Copy ROOT.war to ${tomcat.home}/webapps/ 

Start tomcat by executing ${tomcat.home}/bin/startup.sh (Windows: startup.bat) or by running the tomcat-service if you [created one.](#tomcat-service-for-autostart)


## Environment

### Database

The Open-Source database Postresql 9.6 is used. The database connection uses the connection pool of Tomcat. 

This webapp needs schema '**samply**' in the database '**samply.connector**' under user '**samply**' and password '**samply**' under port `5432`. 

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
    CREATE DATABASE "samply.searchbroker";
    CREATE USER samply WITH PASSWORD 'samply';
    GRANT ALL PRIVILEGES ON DATABASE "samply.searchbroker" to samply;
    ```

### Tomcat

Requirements:

- [Java 8](#java)

1. Download and unzip Tomcat: https://tomcat.apache.org/download-80.cgi

2. Change ports: Every webapp has its own tomcat, so change ports for Store-Tomcat in ${tomcat.base}/conf/server.xml:

   ```
   ...
   ...<connector port="8082" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8102" />...
   ...
   ...<connector port="8002" protocol="AJP/1.3" redirectPort="8102" /> ...
   ...
   ...<Server port="8202" shutdown="SHUTDOWN">...
   ...
   ```

### Java

Is a dependency of tomcat,

if you install different jre versions on this machine, set jre 8 for tomcat by creating a so called "setenv.sh".

Linux: [OpenJDK](https://openjdk.java.net/install/)

Windows: [Oracle](https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html) 


### Configurations

These configuration files are used:

```
src/main/java/webapp/WEB-INF/conf/
(log4j2.xml, mailSending.xml, samply_bridgehead_info.xml, samply_common_config.xml, samply_common_operator.xml, samply_common_urls.xml, )

src/main/java/webapp/META-INF/
(context.xml)
```

The `context.xml` will be auto-copied by tomcat at startup to `${tomcat.base}/conf/Catalina/localhost/ROOT.xml`.
This file will not be overwritten by updating the WAR file due to tomcat settings.

All files under `WEB-INF/conf` will always be found from FileFinder as ultimate fallback.

If you want to save your configurations, copy all files under `WEB-INF/conf` (tomcat or code source) to `${tomcat.base}/conf`.

**IntelliJ** creates a *tomcat.base* directory for every startup of the application. So save your configuration files to *tomcat.home* and it will copy these files and logs every time to *tomcat.base*. You will see the paths at startup in the first lines of the console output.

According to the `log4j2.xml`, all logs can be found in ${tomcat.base}/logs/connector.

To use a **proxy**, set your url in file **samply_common_config.xml**.

#### Configuration files

bridgehead_info.xml:
```
<bi:bridgehead xmlns:bi="http://schema.samply.de/config/BridgeheadInfo">
    <bi:name>name of the bridgehead</bi:name>
    <bi:centralsearch>url of the centralsearch server (only for dktk)</bi:centralsearch>
    <bi:decentralsearch>url of the decentrealsearch server</bi:decentralsearch>
    <bi:queryLanguage>name of the querylanguage (CQL or QUERY)</bi:queryLanguage>
</bi:bridgehead>
```
common_urls.xml:
```
<com:urls xmlns:com="http://schema.samply.de/common">
    <com:shareUrl>Ip of the connector</com:shareUrl>
    <com:idmanagerUrl>url of the id-manager (only for dktk necessary)</com:idmanagerUrl>
    <com:ldmUrl>url of the local data management</com:ldmUrl>
    <com:mdrUrl>url of the mdr server</com:mdrUrl>
</com:urls>
```
common_operator.xml:
```
<com:operator xmlns:com="http://schema.samply.de/common">
    <!--Optional:-->
    <com:firstName>first name from the bridgehead admin</com:firstName>
    <!--Optional:-->
    <com:lastName>last name from the bridgehead admin</com:lastName>
    <!--Optional:-->
    <com:email>email from the bridgehead admin</com:email>
    <!--Optional:-->
    <com:phone>phone number from the bridgehead admin</com:phone>
</com:operator>
```

common_config.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<Configuration xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schema.samply.de/common"
               xsi:schemaLocation="http://schema.samply.de/common http://schema.samply.de/config/Common.Config.xsd ">
    <Proxy>
        <HTTP>
            <Url>url of the proxy server</Url>
            <Username>username for the proxy server</Username>
            <Password>password for the proxy server</Password>
        </HTTP>
        <HTTPS>
            <Url>url of the proxy server</Url>
            <Username>username for the proxy server</Username>
            <Password>password for the proxy server</Password>
        </HTTPS>
        <Realm/>
        <NoProxyHosts>
            <Host>hosts where the proxy should not be used<Host>
        </NoProxyHosts>
    </Proxy>
</Configuration>
```

### Connections

Ingoing (secured with basic auth):

```
GET /inquiries/active
GET /inquiries/archived
GET /inquiries/erroneous
GET /inquiries/log
```



Outgoing:

```
STORE/testAuth
STORE/saveOrUpdateUser

STORE/requests
STORE/result
STORE/stats
STORE/info

SEARCHBROKER/rest/searchbroker/banks/{bankemail} (register, activate and unregister)
SEARCHBROKER/rest/searchbroker/inquiries
SEARCHBROKER/rest/searchbroker/inquiries/{inquiryid}
SEARCHBROKER/rest/searchbroker/inquiries/{inquiryid}/replies/{bankemail}

SEARCHBROKER/rest/test/inquiries/{inquiryid}
SEARCHBROKER/referencequery
SEARCHBROKER/rest/monitoring

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
sudo nano /etc/systemd/system/tomcat-connector.service

```

​		Copy the remembered path to JAVA_HOME and add `/jre` to the end of this 		path, also check tomcat path:

```
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64/jre
Environment=CATALINA_PID=/opt/tomcat-connector/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat-connector
Environment=CATALINA_BASE=/opt/tomcat-connector
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/opt/tomcat-connector/bin/startup.sh
ExecStop=/opt/tomcat-connector/bin/shutdown.sh

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
