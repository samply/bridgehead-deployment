[sl]: <https://samplelocator.bbmri.de>
[bbmri]: <http://www.bbmri-eric.eu>
[docker]: <https://docs.docker.com/install>

[man-store]: <https://github.com/samply/blaze/blob/master/docs/deployment/manual-deployment.md>
[env-store]: <https://github.com/samply/blaze/blob/master/docs/deployment/environment-variables.md>

[man-connector]: <Connector.md>
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

## Admin Page (Conncetor)

### Register at Sample Locator
* To connect to the Sample Locator, login at Connector-UI (../login.xhtml) (default usr=admin, pwd=adminpass)
* Go to the register page (../admin/broker_list.xhtml) and enter these values:
* "Address": `https://samplelocator.bbmri.de/broker/`
* "Your email address": your email address to get an API key
* "Automatic reply": `Total Size` (default, so you answer automatically with number of samples and donors)
* Select "Join" to receive an email with the API key to paste under "Status", then select "Activate"
* At least, **send an email** to `feedback@germanbiobanknode.de` with `your used email address` and `desired biobank name`

### Monitoring
* Activate Monitoring (Icinga will send a test query periodically to send you an email if errors occur)
* Open the conig page (../admin/configuration.xhtml) to enable three buttons under "Reporting to central services" and scroll down to save with button "Save"

### Credentials
* Under (../admin/credentials_list.xhtml) you can see the credentials which the connector are using.
* It is possible to add the following credentials:
* HTTP Proxy
* Local Data Management authentication
* Directory Sync (if the feature toggle is enabled)

### User
* To enable a user to access the connector, a new user can be created under "../admin/user_list.xhtml".
This user has the possibility to view incoming queries

### Jobs
* The connector are using [Quartz Jobs](http://www.quartz-scheduler.org/) to do things like collect the queries from the searchbroker or execute the queries.
Under the job page ("../admin/job_list.xhtml") you can see the full list of the jobs.

### Tests
* The connector has connectivity checks which can be found under the test page(../admin/tests.xhtml).

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
          image: "samply/connector:7.0.0"
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

## Environment Variables

| Name | Default | Description |
| -------------- | ------- | ------------------------------------------------------------- |
| POSTGRES_HOST* | | Base URI of Postgres |
| POSTGRES_PORT | *5432* | Port of Postgres |
| POSTGRES_DB* | | Database name in Postgres |
| POSTGRES_USER* | | Authorized username for database |
| POSTGRES_PASS* | | Password of authorized user |
| HTTP_PROXY | | Proxy server and port for outbound HTTP requests, e.g. "proxy.example.de:8080" |
| PROXY_USER | | Proxy server user, if authentication is needed. |
| PROXY_PASS | | Proxy server password, if authentication is needed. |
| STORE_URL* | | The URL under which the Store is accessible by Connector |
| QUERY_LANGUAGE | *QUERY* | `QUERY` for Classic Store, `CQL` for Blaze |
| MDR_URL* | | The URL under which the Metadata Repository is accessible |
| DIRECTORY_URL | | The URL under which the BBMRI Directory is accessible |
| OPERATOR_FIRST_NAME | | The first name from the connector admin |
| OPERATOR_LAST_NAME | | The last name from the connector admin |
| OPERATOR_EMAIL | | The email from the connector admin |
| OPERATOR_PHONE | | The phone number from the connector admin |
| MAIL_HOST | | The URL of the mail server |
| MAIL_PORT | 25 | The port of the mail server |
| MAIL_PROTOCOL | smtp | The protocol of the mail server |
| MAIL_FROM_ADDRESS | | The email address that appears as sender in the email |
| MAIL_FROM_NAME | | The name that appears as sender in the email |
| LOG_LEVEL | info | Log level of tomcat |
| feature_BBMRI_DIRECTORY_SYNC | false | Feature toggle for the BBMRI directory sync |
| feature_DKTK_CENTRAL_SEARCH | false | Feature toggle for the DKTK central search |
| feature_NNGM_CTS | false | Feature toggle for the NNGM CTS |
| CATALINA_OPTS | | JVM options |
*necessary




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
