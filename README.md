# Bridgehead Deployment


## Goal
Allow the Sample Locator to search for patients and samples in your biobanks, giving researchers easy access to your resources.


## Quick start
If you simply want to set up a test installation, without exploring all of the possibilities offered by the Bridgehead, then the sections you need to look at are:
* [Starting a Bridgehead](#starting-a-bridgehead)
* [Register with a Sample Locator](#register-with-a-sample-locator)
* [Checking your newly installed Bridgehead](#checking-your-newly-installed-bridgehead)


## Background
The **Sample Locator** is a tool that allows researchers to make searches for samples over a large number of geographically distributed biobanks. Each biobank runs a so-called **Bridgehead** at its site, which makes it visible to the Sample Locator.  The Bridgehead is designed to give a high degree of protection to patient data. Additionally, a tool called the [Negotiator][negotiator] puts you in complete control over which samples and which data are delivered to which researcher.

You will most likely want to make your biobanks visible via the [publicly accessible Sample Locator][sl], but the possibility also exists to install your own Sample Locator for your site or organization, see the GitHub pages for [the server][sl-server-src] and [the GUI][sl-ui-src].

The Bridgehead has two primary components:
* The **Blaze Store**. This is a highly responsive FHIR data store, which you will need to fill with your data via an ETL chain.
* The **Connector**. This is the communication portal to the Sample Locator, with specially designed features that make it possible to run it behind a corporate firewall without making any compromises on security.

This document will show you how to:
* Install the components making up the Bridgehead.
* Register your Bridgehead with the Sample Locator, so that researchers can start searching your resources.


## Requirements
For data protection concept, server requirements, validation or import instructions, see [the list of general requirements][requirements].


## Starting a Bridgehead
The file `docker-compose.yml` contains the the minimum settings needed for installing and starting a Bridgehead on your computer. This Bridgehead should run straight out of the box. However, you may wish to modify this file, e.g. in order to:
* Enable a corporate proxy (see below).
* Set an alternative Sample Locator URL.
* Change the admin credentials for the Connector.

To start a Bridgehead on your computer, you will need to follow the following steps:

* [Install Docker][docker] and test with:

```sh
docker run hello-world
```

* Download this repository:

```sh
git clone https://github.com/samply/bridgehead-deployment
cd bridgehead-deployment
```

* Launch the Bridgehead with the following command:

```sh
docker-compose up -d
```

* First test of the installation: check to see if there is a Connector running on port 8082:

```sh
curl localhost:8082 | grep Welcome 
```

* If you need to stop the Bridgehead, from within this directory:

```sh
docker-compose down
```  

## Port usage
Once you have started the Bridgehead, the following components will be visible to you via ports on localhost:
* Blaze Store: port 8080
* Connector admin: port 8082

## Connector Administration
The Connector administration panel allows you to set many of the parameters regulating your Bridgehead. Most especially, it is the place where you can register your site with the Sample Locator. It is accessible via your browser under the URL http://localhost:8082.

You will first need to log in as an administrator with the URL http://localhost:8082/login.xhtml. The default user is `admin`, password: `adminpass` (you can change these defaults in the docker-compose.yml file). Once you have logged in, the following options are open to you:

### Register with a Sample Locator
* Go to the registration page http://localhost:8082/admin/broker_list.xhtml.
* To register with the production GBA Sample Locator, enter the following values in the three fields under "Join new Searchbroker":
  * "Address": `https://samplelocator.bbmri.de/broker/` (`https://samplelocator.test.bbmri.de/broker/` for the test Sample Locator)
  * "Your email address": this is the email to which the registration token will be returned.
  * "Automatic reply": Set this to be `Total Size`
* Click "Join" to start the registration process.
* You should now have a list containing exactly one broker. You will notice that the "Status" box is empty.
* Send an email to `feedback@germanbiobanknode.de` and let us know that you want to register with one of our Sample Locators. We will then contact you and arrange an appointment for an online registration session, where you will be given a registration token and we will than complete the registration process together. If you wish to keep in contact with us, feel free to request access to our Zulip channel for Sample Locator support.

### Monitoring
* Activate Monitoring (Icinga will send a test query periodically to send you an email if errors occur)
* Open the config page http://localhost:8082/admin/configuration.xhtml to enable three buttons under "Reporting to central services" and scroll down to save with button "Save"

### User
* To enable a user to access the connector, a new user can be created under http://localhost:8082/admin/user_list.xhtml.
This user has the possibility to view incoming queries

### Jobs
* The connector uses [Quartz Jobs](http://www.quartz-scheduler.org/) to do things like collect the queries from the searchbroker or execute the queries.
The full list of jobs can be viewed under the job page http://localhost:8082/admin/job_list.xhtml.

### Tests
* The Connector connectivity checks can be found under the test page http://localhost:8082/admin/tests.xhtml.

## Checking your newly installed Bridgehead
We will load some test data and then run a query to see if it shows up.

First, install [bbmri-fhir-gen][bbmri-fhir-gen]. Run the following command:

```sh
mkdir TestData
bbmri-fhir-gen TestData -n 10
```

This will generate test data for 10 patients, and place it in the directory `TestData`.

Next, install [blazectl][blazectl]. Run the following commands:

```sh
blazectl --server http://localhost:8080/fhir upload TestData
blazectl --server http://localhost:8080/fhir count-resources
```

If both of them complete successfully, it means that the test data has successfully been uploaded to the Blaze Store.

Open the [Sample Locator][sl] and hit the "SEND" button. You may need to wait for a minute before all results are returned. Click the "Login" link to log in via the academic institution where you work (AAI). You should now see a list of the biobanks known to the Sample Locator.

If your biobank is present, and it contains non-zero counts of patients and samples, then your installation was successful.

If you wish to remove the test data, you can do so by simply deleting the Docker volume for the Blaze Store database:

```sh
docker-compose down
docker volume rm store-db-data
```

## Manual installation
The installation described here uses Docker, meaning that you don't have to worry about configuring or installing the Bridgehead components - Docker does it all for you. If you do not wish to use Docker, you can install all of the software directly on your machine, as follows:

* Install the [Blaze Store][man-store]
* Install the [Connector][man-connector]
* Register with the Sample Locator (see above)


Source code for components deployed by `docker-compose.yml`:

* [Store][store-src]
* [Connector][connector-src]


## Optional configuration:

#### Proxy example
Add environments variables in `docker-compose.yml` (remove user and password environments if not available):
"http://proxy.example.de:8080", user "testUser", password "testPassword"
      
      version: '3.4'
      services:
        store:
          container_name: "store"
          image: "samply/blaze:0.10.3"
          environment:
            BASE_URL: "http://store:8080"
            JAVA_TOOL_OPTIONS: "-Xmx4g"
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
      


#### General information on Docker environment variables used in the Bridgehead

* [Store][env-store]
* [Connector][env-connector]


## Notes
* If you see database connection errors of Store or Connector, open a second terminal and run `docker-compose stop` followed by `docker-compose start`. Database connection problems should only occur at the first start because the store and the connector doesn't wait for the databases to be ready. Both try to connect at startup which might be to early.

* If one needs only one of the the Bridgehead components, they can be started individually:

```sh
docker-compose up store -d
docker-compose up connector -d
```

* To shut down all services (but keep databases):

```sh
docker-compose down
```  

* To delete databases as well (destroy before):

```sh
docker volume rm store-db-data
docker volume rm connector-db-data
```

* To see all executed queries, create a [new user][connector-user], logout and login with this normal user.

* To set Store-Basic-Auth-credentials in Connector (as default `Lokales Datenmanagement` with dummy values was generated)
    * Login at [Connector-UI][connector-login] (default usr=admin, pwd=adminpass)
    * Open [credentials page][connector-credentials]
        - Delete all instances of`Lokales Datenmanagement`
        - for "Ziel" select `Lokales Datenmanagement`, provide decrypted CREDENTIALS in "Benutzername" and "Passwort", select "Zugangsdaten hinzufügen"


## Useful Links
* [FHIR Quality Reporting Authoring UI][quality-ui-github]
* [How to join Sample Locator][join-sl]
* [Samply code repositories][samply]


[sl]: <https://samplelocator.bbmri.de>
[sl-ui-src]: <https://github.com/samply/sample-locator>
[sl-server-src]: <https://github.com/samply/share-broker-rest>
[negotiator]: <https://negotiator.bbmri-eric.eu/login.xhtml>
[bbmri]: <http://www.bbmri-eric.eu>
[docker]: <https://docs.docker.com/install>

[connector-user]:<http://localhost:8082/admin/user_list.xhtml>
[connector-login]:<http://localhost:8082/admin/login.xhtml>
[connector-credentials]:<http://localhost:8082/admin/credentials_list.xhtml>

[requirements]: <https://samply.github.io/bbmri-fhir-ig/howtoJoin.html#general-requirements>

[man-store]: <https://github.com/samply/blaze/blob/master/docs/deployment/manual-deployment.md>
[env-store]: <https://github.com/samply/blaze/blob/master/docs/deployment/environment-variables.md>
[env-connector]: <https://github.com/samply/share-client/blob/master/docs/deployment/docker-deployment.md>

[bbmri-fhir-gen]: <https://github.com/samply/bbmri-fhir-gen>
[blazectl]: <https://github.com/samply/blazectl>

[man-connector]: <Connector.md>

[store-src]: <https://github.com/samply/blaze>
[connector-src]: <https://github.com/samply/share-client>

[quality-ui-github]:<https://github.com/samply/blaze-quality-reporting-ui>
[join-sl]: <https://samply.github.io/bbmri-fhir-ig/howtoJoin.html>
[samply]: <https://github.com/samply>
