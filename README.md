# Bridgehead Deployment
Combines Store and Connector, running in Tomcat 8, using two Postgres databases, monitored by Prometheus and visualized by [Grafana](#grafana).

To make samples easily accessible for researcher to help [making new treatments possible.](http://www.bbmri-eric.eu/)

**Minimal requirements** for productive use:

- 16 GB RAM, 50 GB disk space (recommendation, depends on amount of data)

- Network Communication/Firewall: Outgoing http and https. Proxies are supported. No VPN or incoming ports required.



**Connect your Biobank**:

- Install and run Bridgehead ([Docker-Compose](#docker-compose) which is recommended, or [manual installation](#manual-installation))
- Create xml and [import into Store](IMPORT.md)
- Connect to central [Sample Locator](#connect-sample-locator)


## Docker-Compose

1. [Install Docker](https://docs.docker.com/install/)
   And test Docker in any command line with:

      docker run hello-world

2. After checking for free [ports](#Ports-outside-container), bring all up with:

   ```
   git clone https://github.com/samply/bridgehead-deployment
   cd bridgehead-deployment
   docker-compose up
   ```

Docker compose will start all containers and print the logs to the console.

If you see database connection errors from the store or the connector, open a second terminal and run `docker-compose stop` followed by `docker-compose start`. Database connection problems should only occur at the first start because the store and the connector doesn't wait for the databases to be ready. Both try to connect at startup which might be to early.



If one needs only one of them, on can bring up only the specific one with:

```sh
docker-compose up store
docker-compose up connector
```

To stop Store and Connector:

```
docker-compose down
```

And to empty databases:

```
docker volume rm bridgeheaddeployment_store-db-data
docker volume rm bridgeheaddeployment_connector-db-data
docker volume rm bridgeheaddeployment_grafana-data
```



### Environment Variables

The Docker containers and docker-compose.yml accept certain environment variables:

#### Ports outside container

- PORT_STORE - defaults to `8081`
- PORT_CONNECTOR - defaults to `8082`

- PORT_STORE_METRICS - defaults to `9101`
- PORT_CONNECTOR_METRICS - defaults to `9102`

- PORT_PROMETHEUS - defaults to `9090`
- PORT_GRAFANA - defaults to `3000`

You can either stop all services occupying those ports or [change and save your personal environments](#save-your-environments-optional) 



#### Store specific (Optional)

- STORE_CATALINA_OPTS - JVM options for Tomcat, defaults to `-Xmx1g`

- STORE_MDR_NAMESPACE - current namespace which changes after every change depending data-elements, defaults to `mdr16`

- STORE_MDR_VALIDATION - validation against mdr during store import, defaults to `true`

- STORE_POSTGRES_PASS - the database password, defaults to `samply`


#### Connector specific (Optional)

- CONNECTOR_CATALINA_OPTS - JVM options, defaults to `-Xmx1g`

- CONNECTOR_POSTGRES_PASS - the database password, defaults to `samply`


#### Common (Optional)

- PROXY_URL - the URL of the HTTP proxy to use for outgoing connections, "url:port"; enables proxy usage if set, defaults to ``

- PROXY_USER - the user of the proxy account, defaults to ``

- PROXY_PASS - the password of the proxy account, defaults to ``


#### Save your environments (Optional)

In the repo directory, create a file called `.env`, here you can save your environments if default values change.
Docker-Compose will find this file at startup.

`.env` example:

```
CONNECTOR_POSTGRES_PASS=geheim
STORE_POSTGRES_PASS=mega_geheim
```



## Manual Installation
+++ See READMEs of Store and Connector. Under construction, these repositories need to be pushed to github first +++
ITCs, see:
https://code.mitro.dkfz.de/projects/SHAR/repos/samply.share.client.v2/commits/24b6d9449c3c3b30d5155856d004cb7d25f0972f
https://code.mitro.dkfz.de/projects/STOR/repos/samply.store.rest/commits/ebd6acf0e8d0254128fffac294fb8cd7952673ba





## Connect Sample Locator

You can access the Connector under http://localhost:8082 and login under <http://localhost:8082/login.xhtml> (default credentials are **admin**, **adminpass**). If the Store runs while logging in the first time, the default credentials of the Store get deactivated. From now on, Connector and Store share the same credentials.

Register a Sample Locator under <http://localhost:8082/admin/broker_list.xhtml>

- Broker Adresse = <https://search.germanbiobanknode.de/broker/>
- Ihre Email Adresse = your email address to get the API-Key for registration
- Automatisch antworten = Nur Anzahl (default, so you answer automatically with number of samples)

You will receive an email with API-key generated by our Sample Locator, paste these eight numbers into the provided field and press "ok". Contact us by sending an email from the registered email address to itc_intern@germanbiobanknode.de and let us know the name of your Biobank. An ITC will validate your request so researcher can find your samples.

To see all executed queries, or to create new credentials for the [import](IMPORT.md), create a new user at <http://localhost:8082/admin/user_list.xhtml>, logout, and login as normal user.


### Grafana

You can access Grafana under http://localhost:3000. The login credentials are **admin**, **admin**.

There are two dashboards available. One for the Store and one for the Connector. Currently, they only show JVM metrics.

- Store dashboard: http://localhost:3000/d/wEuwMIpmz
- Connector dashboard: http://localhost:3000/d/wEuwMIpmy