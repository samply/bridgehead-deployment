# Introduction
Samply.Store is implemented as a web service and does not have a graphical UI yet. Data is modelled as XML and transferred via REST interface.


# Data model
![Datamodel](datamodel.png)


# REST interface
At the first start of the Tomcat server, the schema is created in the database. In addition, the default user "local_admin" with password "local_admin" is created until the first login in a different webapp called Connector, which uses this default user for the first connection in which the Connector creates the logged-in user and deletes the default user in the Store database. So you can create new users in the Connector and login in to activate these credentials for the import into the Store.

For a data import into the store, you send a corresponding XML file as a POST request to the REST interface: http://localhost:8081/import. In the header the user must be transmitted with his password as "**Basic Auth**". (Header:"Authorization", "Basic "+CredentialsAsBase64)

The technical means are freely selectable, e.g. Talend Open Studio has a rest interface. For the manual test we recommend Postman or Insomnia.

The XML is validated against an XSD file during import. This XSD can be obtained via GET request from the REST interface:

http://localhost:8081/importXSD

In addition, the entries are validated against the MDR by default. If you use a proxy, you must specify it at startup.

Alternatively, you can use a "curl" command. For Windows this command has to be installed (if Visual C is asked: vc_redist.x64.exe), for Ubuntu and Debian curl has to be installed by (sudo apt-get install curl).

Exporting the XSD
```
curl --output importXSD.xsd -X GET http://localhost:8081/importXSD
```

Importing an XML
```
curl -H "Content-Type:application/xml" -d @export.xml http://USERNAME:PASSWORD@localhost:8081/import
```


# Structure of the XML

An example for the import file can be found at the bottom.

The outer tag is a \<store xmlns="http://schema.samply.de/store"> tag. 

Inside each entity can be specified with a tag corresponding to its name. In addition, the id and foreign keys are always transferred to other entities in the XML. The actual fields are transmitted by a \<genericAttribute key="urn:mdr16:dataelement:23:1"> tag, where the "key" stands for the field in the MDR whose value is always a string.

For example, an entry for a collection (for a biobank with id="Bio11") looks as follows:

```
<collection id="Col22" biobankId="Bio11"> 
<genericAttribute key="urn:mdr16:dataelement:2:1">PD Dr. Schulz</genericAttribute>
</collection>
```

Currently, the entities exist: Biobank, Collection, Sample, SampleContext, Event and Donor. 



## IDs

The ID's of the XML file play a decisive role in the import. They are the values specified in the tags of the individual entities - as id (id) or foreign key (e.g. biobankId). They are stored as key/value pairs in JSON format in the data database. With the fixed key: "samply_store_unique_name". (This key will be renamed to "local_id" in future releases.)

Samply.Store automatically generates a consecutive number for each record (id in the database). It is only used internally (for example, for foreign key relationships) and does not appear externally. 

Important: The ids from the XML (id) and foreign key (biobankId) refer to the entries in the XML. These determine the 1 to n relationships between the entities and the XSD is used to check whether all foreign keys are valid. However, no content validation takes place. 


## Check

There is currently no way to check the data in the store. However, you can do this directly on the PostgreSQL database. This can be done by using the tool "pgAdmin", which is not included in the installation and has to be installed manually.

But the Connector displays each found patient and creates diagrams if you select an inquiry under http://localhost:8082/user/inquiries_list.xhtml and search for "Ergebnisliste anfordern", wait a few minutes.


## Update behavior

If an entity is already stored in the store and sent again with an XSD, the fields are updated as follows:

If an attribute already exists, it is replaced by the newly transmitted values (one or more).
If an attribute does not yet exist, it is added to the list.
If an attribute was present but is not sent along with it, it is retained.
The entity is uniquely identified by its "samply_store_unique_name" (see above).


Example:

A biobank was imported with the following attributes:

key="xyz:def:ghi:1:5"       
key="xyz:def:ghi:2:5"       
key="xyz:def:ghi:2:5"       
key="xyz:def:ghi:3:5"       


The biobank is imported again with additional attributes:

key="xyz:def:ghi:1:5"       
key="xyz:def:ghi:2:5"       
key="xyz:def:ghi:4:5"       


Result:

key="xyz:def:ghi:1:5"       
key="xyz:def:ghi:2:5"       
key="xyz:def:ghi:3:5"       
key="xyz:def:ghi:4:5"       


# Example import file 

```
<?xml version="1.1" encoding="UTF-8"?>
<store xmlns="http://schema.samply.de/store">

<biobank id="Bio11">
<genericAttribute key="urn:mdr16:dataelement:2:1">Prof. Andrea Maria Musterfrau</genericAttribute>
<genericAttribute key="urn:mdr16:dataelement:3:1">http://www.google.de</genericAttribute>
</biobank>

<collection id="Col21" biobankId="Bio11">

<genericAttribute key="urn:mdr16:dataelement:5:1">Breast cancer study</genericAttribute>
</collection>

<collection id="Col22" biobankId="Bio11">
<genericAttribute key="urn:mdr16:dataelement:5:1">Prevention for smokers</genericAttribute>

</collection>

<sampleContext id="CON31" donorId="SNS51">
<genericAttribute key="urn:mdr16:dataelement:20:1">liquid_sample_context</genericAttribute>
</sampleContext>

<sampleContext id="CON32" donorId="SNS51">
<genericAttribute key="urn:mdr16:dataelement:20:1">liquid_sample_context</genericAttribute>
</sampleContext>

<sample id="S43" sampleContextId="CON32" collectionId="Col22">

<!--Sampling date-->

<genericAttribute key="urn:mdr16:dataelement:12:1">01.01.2002</genericAttribute>
</sample>

<sample id="S44" sampleContextId="CON32" collectionId="Col22">

<!--Sampling date-->

<genericAttribute key="urn:mdr16:dataelement:12:1">01.01.2003</genericAttribute>
</sample>

<event id="EvN61" donorId="SNS51">

<genericAttribute key="urn:mdr16:dataelement:26:1">30.05.2002</genericAttribute>

<!--Diagnosis date-->
<genericAttribute key="urn:mdr16:dataelement:27:1">A15.1</genericAttribute>
</event>

<event id="EvN62" donorId="SNS51">
<genericAttribute key="urn:mdr16:dataelement:26:1">01.05.2003</genericAttribute>

<!--Diagnosis date-->
<genericAttribute key="urn:mdr16:dataelement:27:1">I02.0</genericAttribute>
</event>

<donor id="SNS51">
<genericAttribute key="urn:mdr16:dataelement:22:1">24.12.1950</genericAttribute>

<!--Date of birth-->
<genericAttribute key="urn:mdr16:dataelement:23:1">female</genericAttribute>
<!--Biological gender-->
</donor>

</store>
```
