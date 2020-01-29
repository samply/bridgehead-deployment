# Bridgehead Deployment
Makes your Biobank findable over the [Sample Locator](https://samplelocator.bbmri.de), so samples are easily accessible for researcher to [make new treatments possible](http://www.bbmri-eric.eu).



* **Requirements**:
    * 16 GB RAM, 50 GB disk space (recommendation, depends on amount of data)
    * Outgoing http and https. Proxies are supported. No VPN or incoming ports required. Firewall to restrict access from internal network is recommended (Or secure Store with Basic-Auth)
    * Install and run Bridgehead with [**Docker-Compose**](#docker-compose).
        * For **Manual Deployment** on Windows/Linux see guides for [Store](https://alexanderkiel.gitbook.io/blaze/deployment/manual-deployment) and [Connector](Connector.md).
    * Create FHIR bundles satisfying our [profile](https://simplifier.net/bbmri.de) (e.g. with [Talend Open Studio](https://wiki.verbis.dkfz.de/pages/viewpage.action?pageId=76351392)), [validate](https://github.com/samply/bbmri-fhir-ig/blob/master/bbmri-ig/input/pagecontent/Validation.md) and [upload](https://alexanderkiel.gitbook.io/blaze/importing-data) to Store.
    * [Register](#connector-settings) at central [Sample Locator](https://samplelocator.bbmri.de)





## Docker-Compose

* [Install Docker](https://docs.docker.com/install/) and test with:

      docker run hello-world


* Download repository:

      git clone https://github.com/samply/bridgehead-deployment
      cd bridgehead-deployment


* The `docker-compose.yml` is as small as possible. Modify this file to enable proxy for instance (see below) and bring all up with:

      docker-compose up -d




#### Optional:

* Docker Environments
    * [Store](https://alexanderkiel.gitbook.io/blaze/deployment/environment-variables)
    * Connector:

| Name           | Default | Description                                                   |
| -------------- | ------- | ------------------------------------------------------------- |
| POSTGRES_HOST  |         | Base URI of Postgres                                          |
| POSTGRES_PORT  | *5432*  | Port of Postgres                                              |
| POSTGRES_DB    |         | Database name in Postgres                                     |
| POSTGRES_USER  |         | Authorized username for database                              |
| POSTGRES_PASS  |         | Password of authorized user                                   |
|                |         |                                                               |
| HTTP_PROXY     |         | Proxy server and port for outbound HTTP requests (`url:port`) |
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





## Connector-Settings

* To register a Sample Locator, login in Connector-UI at <http://localhost:8082/login.xhtml> (default usr=admin, pwd=adminpass)
    * Go to <http://localhost:8082/admin/broker_list.xhtml>
        * `https://samplelocator.bbmri.de/broker/` for "Broker Adresse"
        * **email address** to get the API-Key for registration for "Ihre Email Adresse"
        * `Nur Anzahl` (default, so you answer automatically with number of samples and donors) for "Automatisch antworten"
        * Select "Beitreten" to receive an email with the API key to paste under "Status", select "Aktivieren"
        * To unlock, **send an email** to `feedback@germanbiobanknode.de` with `used email address` and `desired biobank name`




#### Optional:
* To see all executed queries, create a new user at <http://localhost:8082/admin/user_list.xhtml>, logout and login with this normal user.
* To set Store-Basic-Auth-credentials in Connector (as default one `Lokales Datenmanagement` with dummy values was generated)
    * Login in Connector-UI at <http://localhost:8082/login.xhtml> (default usr=admin, pwd=adminpass)
    * Open <http://localhost:8082/admin/credentials_list.xhtml>
        - Delete all instances of`Lokales Datenmanagement`
        - for "Ziel" select `Lokales Datenmanagement`, provide decrypted CREDENTIALS in "Benutzername" and "Passwort", select "Zugangsdaten hinzufügen"
