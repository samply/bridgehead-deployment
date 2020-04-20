[sl]: <https://samplelocator.bbmri.de>
[bbmri]: <http://www.bbmri-eric.eu>
[docker]: <https://docs.docker.com/install>
[profile]: <https://simplifier.net/bbmri.de>
[talend]: <https://wiki.verbis.dkfz.de/pages/viewpage.action?pageId=76351392>
[validate]: <https://github.com/samply/bbmri-fhir-ig/blob/master/input/pagecontent/Validation.md>
             
[register]: <#connector-settings>
[compose]: <#docker-compose>

[import-store]: <https://alexanderkiel.gitbook.io/blaze/importing-data>
[man-store]: <https://alexanderkiel.gitbook.io/blaze/deployment/manual-deployment>
[env-store]: <https://alexanderkiel.gitbook.io/blaze/deployment/environment-variables>

[man-connector]: <Connector.md>
[connector-user]: <http://localhost:8082/admin/user_list.xhtml>
[connector-login]: <http://localhost:8082/login.xhtml>
[connector-register]: <http://localhost:8082/admin/broker_list.xhtml>
[connector-credentials]: <http://localhost:8082/admin/credentials_list.xhtml>

[quality-ui]:<https://github.com/samply/blaze-quality-reporting-ui>


# Bridgehead Deployment
Makes your Biobank findable over the [Sample Locator][sl], so samples are easily accessible for researcher to [make new treatments possible][bbmri].



* **Requirements**:
    * 16 GB RAM, 50 GB disk space (recommendation, depends on amount of data)
    * Outgoing http and https. Proxies are supported. No VPN or incoming ports required. Firewall to restrict access from internal network is recommended (Or secure Store with Basic-Auth)
    * Install and run Bridgehead with [**Docker-Compose**][compose]
        * For **Manual Deployment** on Windows/Linux see guides for [Store][man-store] and [Connector][man-connector]
    * Create FHIR bundles satisfying our [profile][profile] (e.g. with [Talend Open Studio][talend]), [validate][validate] and [import][import-store] to Store
    * [Register][register] the search engine [Sample Locator][sl]





## Docker-Compose

* [Install Docker][docker] and test with

      docker run hello-world


* Download repository

      git clone https://github.com/samply/bridgehead-deployment
      cd bridgehead-deployment


* The `docker-compose.yml` is as small as possible. Modify this file to enable proxy for instance (see below) and bring all up with

      docker-compose up -d


## Connector-Settings

* To register a Sample Locator, login at [Connector-UI][connector-login] (default usr=admin, pwd=adminpass)
    * Go to [register page][connector-register] and enter these values:
        * "Broker Adresse": `https://samplelocator.bbmri.de/broker/`
        * "Ihre Email Adresse": your email address to get an API key
        * "Automatisch antworten": `Nur Anzahl` (default, so you answer automatically with number of samples and donors)
        * Select "Beitreten" to receive an email with the API key to paste under "Status", then select "Aktivieren"
        * At least, **send an email** to `feedback@germanbiobanknode.de` with `your used email address` and `desired biobank name`


#### Optional:

* [FHIR Quality Reporting Authoring UI: ][quality-ui]Open http://localhost:8000 

* Docker Environments
    * [Store][env-store]
    * Connector:

| Name           | Default | Description                                                   |
| -------------- | ------- | ------------------------------------------------------------- |
| POSTGRES_HOST  |         | Base URI of Postgres                                          |
| POSTGRES_PORT  | *5432*  | Port of Postgres                                              |
| POSTGRES_DB    |         | Database name in Postgres                                     |
| POSTGRES_USER  |         | Authorized username for database                              |
| POSTGRES_PASS  |         | Password of authorized user                                   |
|                |         |                                                               |
| HTTP_PROXY     |         | Proxy server and port for outbound HTTP requests, e.g. "proxy.example.de:8080" |
| PROXY_USER     |         | Proxy server user, if authentication is needed.               |
| PROXY_PASS     |         | Proxy server password, if authentication is needed.           |
|                |         |                                                               |
| STORE_URL      |         | The URL under which the Store is accessible by Connector      |
| QUERY_LANGUAGE | *QUERY* | `QUERY` for Classic Store, `CQL` for Blaze                    |
| MDR_URL        |         | The URL under which the Metadata Repository is accessible     |
|                |         |                                                               |
| CATALINA_OPTS  |         | JVM options                                                   |



* If you see database connection errors of Store or Connector, open a second terminal and run `docker-compose stop` followed by `docker-compose start`. Database connection problems should only occur at the first start because the store and the connector doesn't wait for the databases to be ready. Both try to connect at startup which might be to early.

* If one needs only one of them

      docker-compose up store -d
      docker-compose up connector -d

* To destroy all services (which keeps databases):
  
      docker-compose down

* To delete databases (destroy before):

      docker volume rm store-db-data
      docker volume rm connector-db-data

* To see all executed queries, create a [new user][connector-user], logout and login with this normal user.

* To set Store-Basic-Auth-credentials in Connector (as default `Lokales Datenmanagement` with dummy values was generated)
    * Login at [Connector-UI][connector-login] (default usr=admin, pwd=adminpass)
    * Open [credentials page][connector-credentials]
        - Delete all instances of`Lokales Datenmanagement`
        - for "Ziel" select `Lokales Datenmanagement`, provide decrypted CREDENTIALS in "Benutzername" and "Passwort", select "Zugangsdaten hinzufügen"
