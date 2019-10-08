# Bridgehead Deployment
Makes your Biobank visible on the [Sample Locator](https://search.germanbiobanknode.de) to make samples easily accessible for researcher to help [making new treatments possible.](https://www.bbmri-eric.eu)

This Docker Compose combines Store and Connector, using two Tomcat and two Postgres instances, monitored by Prometheus and visualized by [Grafana](#grafana).

**Minimal requirements** for productive use:

- 16 GB RAM, 50 GB disk space (recommendation, depends on amount of data)

- Network Communication/Firewall: Outgoing http and https. Proxies are supported. No VPN or incoming ports required.


**Connect your Biobank**:

- Install and run the Bridgehead (recommendation: [Docker-Compose](#docker-compose). For manual deployment on Windows/Linux see [Store](doc/Store.md) and [Connector](doc/Connector.md).)
- Set credentials for [Connector-Store-Authorization](#connector-store-authorization)
- [Create xml and import](doc/IMPORT.md) into Store
- [Connect](#connect-sample-locator) to central Sample Locator


## Docker-Compose
1. [Install Docker](https://docs.docker.com/install/) and test with:

    docker run hello-world

2. The Docker Compose occupies the ports 8081, 8082, 3000, 9090, 9101 and 9102, bring all up with:

       git clone https://github.com/samply/bridgehead-deployment
       cd bridgehead-deployment
       docker-compose up -d

Docker compose will start all containers and print the logs to the console.

If you see database connection errors of Store or Connector, open a second terminal and run `docker-compose stop` followed by `docker-compose start`. Database connection problems should only occur at the first start because the store and the connector doesn't wait for the databases to be ready. Both try to connect at startup which might be to early.

If one needs only one of them, on can bring up only the specific one with:

    docker-compose up store
    docker-compose up connector

To stop Store and Connector:

    docker-compose down

And to empty databases:

    docker volume rm bridgeheaddeployment_store-db-data
    docker volume rm bridgeheaddeployment_connector-db-data
    docker volume rm bridgeheaddeployment_grafana-data


### Environment Variables

| Name                     | Default  | Description                                                  |
| ------------------------ | -------- | ------------------------------------------------------------ |
| PROXY_URL                |          | URL of the HTTP proxy to use for outgoing connections, "url:port"; enables proxy usage if set |
| PROXY_USER               |          | User of the proxy account                                    |
| PROXY_PASS               |          | Password of the proxy account                                |
|                          |          |                                                              |
| STORE_CATALINA_OPTS      | "-Xmx1g" | JVM options for Tomcat of Store                              |
| STORE_MDR_NAMESPACE      | mdr16    | Current namespace which changes after every update depending data-elements |
| STORE_MDR_VALIDATION     | "true"   | Validation against MDR during Store import                   |
| STORE_POSTGRES_PASS      | samply   | Database password for Store                                  |
|                          |          |                                                              |
| CONNECTOR_CATALINA_OPTS  | "-Xmx1g" | JVM options for Tomcat of Connector                          |
| CONNECTOR_QUERY_LANGUAGE | QUERY    | `QUERY` for Samply Store, `CQL` for Blaze                    |
| CONNECTOR_POSTGRES_PASS  | samply   | Database password for Connector                              |

To change passwords and save your proxy configurations if necessary, create a file called `.env` in the repo directory, here you can save your environments like this:

    STORE_POSTGRES_PASS=TopSecret
    CONNECTOR_POSTGRES_PASSWORD=samply
    PROXY_URL=117.156.15.21:7808
    PROXY_USER=Max
    PROXY_PASS=VerySecret


## Connector-Store-Authorization

Before the first login on the Connector UI, the Store accepts for import **local_admin:local_admin**.
After the first login on the Connector UI and have the Store running, the credentials for **local_admin** get deactivated for import.

* Login under <http://localhost:8082/login.xhtml> (default credentials are **admin**, **adminpass**).
* To change password for securing the Store, select the username `admin` in the right upper corner and select "Passwort ändern".
* Under <http://localhost:8082/admin/credentials_list.xhtml>, for "Ziel" select `Lokales Datenmanagement`, for "Benutzername" `admin` and for "Passwort" your password.
* From now on, the Store accepts for any connection only `admin` and your password.

To add users to the Store database, create one on the Connector UI under <http://localhost:8082/admin/user_list.xhtml> and login at least once.


## Connect Sample Locator

Login at <http://localhost:8082/login.xhtml>, register the Sample Locator under <http://localhost:8082/admin/broker_list.xhtml>

- Broker Adresse = <https://samplelocator.bbmri.de/broker/>
- Ihre Email Adresse = your email address to get the API-Key for registration
- Automatisch antworten = Nur Anzahl (default, so you answer automatically with number of samples and donors)

You will receive an email with API-key generated by the Sample Locator, paste these eight numbers into the provided field and press "ok".

Contact us by sending an email from the registered email address to itc_intern@germanbiobanknode.de and let us know the name of your Biobank. An ITC will validate your request, so researcher can find your samples.

To see all executed queries, create a new user at <http://localhost:8082/admin/user_list.xhtml>, logout, and login as normal user.


### Grafana

You can access Grafana under http://localhost:3000. The login credentials are **admin**, **admin**.

There are two dashboards available. One for the Store and one for the Connector. Currently, they only show JVM metrics.

- Store dashboard: http://localhost:3000/d/wEuwMIpmz
- Connector dashboard: http://localhost:3000/d/wEuwMIpmy
