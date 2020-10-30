[sl]: <https://samplelocator.bbmri.de>
[bbmri]: <http://www.bbmri-eric.eu>
[docker]: <https://docs.docker.com/install>

[store]: <http://localhost:8080/fhir>
[man-store]: <https://github.com/samply/blaze/blob/master/docs/deployment/manual-deployment.md>
[env-store]: <https://github.com/samply/blaze/blob/master/docs/deployment/environment-variables.md>

[connector]: <http://localhost:8082/>
[man-connector]: <Connector.md>
[connector-user]: <http://localhost:8082/admin/user_list.xhtml>
[connector-login]: <http://localhost:8082/login.xhtml>
[connector-register]: <http://localhost:8082/admin/broker_list.xhtml>
[connector-credentials]: <http://localhost:8082/admin/credentials_list.xhtml>
[connector-config]: <http://localhost:8082/admin/configuration.xhtml>
[connector-jobs]: <http://localhost:8082/admin/job_list.xhtml>

[quality-ui]: <http://localhost:8081>
[quality-ui-github]:<https://github.com/samply/blaze-quality-reporting-ui>


# Bridgehead Deployment


## Goal
Makes your Biobank findable over the [Sample Locator][sl], so samples are easily accessible for researcher to [make new treatments possible][bbmri].


## Requirements
For data protection concept, server requirements, validation or import instructions, see https://samply.github.io/bbmri-fhir-ig/howtoJoin.html#general-requirements


## Docker-Compose

* [Install Docker][docker] and test with

      docker run hello-world

* Download repository

      git clone https://github.com/samply/bridgehead-deployment
      cd bridgehead-deployment

* The `docker-compose.yml` is as small as possible. Modify this file to enable proxy for instance (see below) and bring all up with

      docker-compose up -d

* To connect to the Sample Locator, login at [Connector-UI][connector-login] (default usr=admin, pwd=adminpass)
    * Go to [register page][connector-register] and enter these values:
        * "Broker Adresse": `https://samplelocator.bbmri.de/broker/`
        * "Ihre Email Adresse": your email address to get an API key
        * "Automatisch antworten": `Nur Anzahl` (default, so you answer automatically with number of samples and donors)
        * Select "Beitreten" to receive an email with the API key to paste under "Status", then select "Aktivieren"
        * At least, **send an email** to `feedback@germanbiobanknode.de` with `your used email address` and `desired biobank name`

* Activate Monitoring (Heidelberg will send a test query periodically to send you an email if errors occur)
    * [Open][connector-config] to enable three buttons under "Übermittlung von Metadaten" and scroll down to save with button "Speichern"

## Manual installation

* [Store][man-store]
* [Connector][man-connector]
* Connect the Sample Locator (see above)


#### Installed components:

* Store: [store]
* Connector: [connector]
* [FHIR Quality Reporting Authoring UI][quality-ui-github]: [Open][quality-ui]


## Optional configuration:

#### Proxy example
Add Environments in docker-compose.yml (remove user and password environments if not available):
"http://proxy.example.de:8080", user "testUser", password "testPassword"
      
      version: '3.4'
      services:
        store:
          container_name: "store"
          image: "samply/blaze:0.8.0"
          environment:
            BASE_URL: "http://store:8080"
            DB_DIR: "/app/data/db"
            JAVA_TOOL_OPTIONS: "-Xmx4g -XX:+UseG1GC"
            PROXY_HOST: "http://proxy.example.de"
            PROXY_PORT: "8080"
            PROXY_USER: "testUser"
            PROXY_PASSWORD: "testPassword"
          networks:
            - "samply"
            
      .......
      
      connector:
          container_name: "connector"
          image: "samply/connector:6.13.0"
          environment:
            POSTGRES_HOST: "connector-db"
            POSTGRES_DB: "samply.connector"
            POSTGRES_USER: "samply"
            POSTGRES_PASS: "samply"
            STORE_URL: "http://store:8080/fhir"
            QUERY_LANGUAGE: "CQL"
            MDR_URL: "https://mdr.germanbiobanknode.de/v3/api/mdr"
            HTTP_PROXY: "http://proxy.example.de:8080"
            PROXY_USER: "testUser"
            PROXY_PASS: "testPassword
          networks:
            - "samply"
            
      .......
      


#### Docker Environments

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
