# Reference Data Service

Repository hosting public material and documentation for the [Reference Data Service (RDS)](https://rds-global.swissartresearch.net/), a project of the [University of Zürich](https://www.uzh.ch/) provided by the [Swiss Art Research Infrastructure (SARI)](http://swissartresearch.net/) and powered by [metaphacts](https://metaphacts.com/).

The Reference Data Service (the so-called RDS-Global system) provides access to datasets like AAT, ULAN, GeoNames, Wikidata, GND EntityFacts, and a few more.

## How to start you own RDS-Local instance

You can also launch your own local service which fetches data from the upstream RDS-Global system.

### Prerequisites
First of all you should ensure that [docker](https://www.docker.com) and [docker-compose](https://docs.docker.com/compose/install/) are installed on your computer. Once this step is done, you can move to the second one.

### Installation

1. clone or download this repository
1. In your local copy, navigate to the `docker-compose` folder and execute the following command: `docker-compose up -d`. This command will start the platform in a few minutes. 
1. Once the system is running you can access the platform using the url: [http://localhost:8080](http://localhost:8080) or [https://localhost:8443](https://localhost:8443).
1. The default credentials are username `admin` and password `admin`. This can be changed by going to the settings menu (gear icon on the top-right) and from there to _Security_

### Configuration

Some additional configuration is recommended in order to use the full functionality of the platform. Example configurations are provided as `.ttl` files in the `/configuration` folder.

1. Log in as Administrator and navigate to the Administration page by clicking on the cog wheel in the top right of the menu bar
1. Click on _Data Import & Export_
1. Upload the `dataset-metadata.ttl` file using the File Upload interface
    * This will display additional information when searching through external datasets
1. Click on _Advanced Options_ and enter the following URI in the field _URI of the target NamedGraph_: `http://schema.swissartresearch.net/rds/type-mapping`. Then upload the `type-mapping.ttl` file through the File Upload interface
    * This will ensure the correct data types are used when searching for existing and newly created records

If you intend to use your RDS-Local instance for data creation, perform the following steps. 

1. Navigate to the _Administration_ page and access the _Security_ configuration.
1. For editing records a user needs to have the `rds-editor` role. Click on the `admin` user to assign the role to this user.
1. The form under the list will change to _Update Account_. Add the `rds-editor` to the list of roles and click _Update_.
1. Add the `rds-editor` to any user that requires editing rights.
1. Return to the _Administration_ page and navigate to the _Data Import & Export_ page.
1. Upload the `user-roles.ttl` file.
    * This defines the roles of individual users in the editing workflow.

### Operations

* To stop the platform you can execute the `docker-compose stop` command. 
* After changing any container configuration the platform needs to be re-created using `docker-compose up -d` again.
* The container configuration can be customized (injection of volumes, memory settings, ...) in the file `docker-compose.override.yml`. Please do not touch `docker-compose.yml`, rather copy sections to `docker-compose.override.yml` and change them there
* The logs are available using `docker-compose logs -f`



# RDS lookup service API

## Implementation

The RDS lookup service follow the Reconciliation API developed by the W3C "Working Group Reconciliation Service API". Full specs of the API can be found here

- [https://reconciliation-api.github.io/specs/latest/](https://reconciliation-api.github.io/specs/latest/)
- [https://github.com/OpenRefine/OpenRefine/wiki/Reconciliation-Service-API](https://github.com/OpenRefine/OpenRefine/wiki/Reconciliation-Service-API)
While a live test bench for the API is available at the address below
- [https://reconciliation-api.github.io/testbench/](https://reconciliation-api.github.io/testbench/)

The current implementation of the Reconciliation API only includes a reconciliation service (Look-up service). Only the following rest endpoints are available:

	GET /rest/reconciliation (without any parameters) - returns manifest of the service in JSON(P) format (Depending on Accept field in the header).

	GET /rest/reconciliation?query={JSON_QUERY} - where JSON_QUERY is a url-encoded query-json-object


| Parameter     	| Description                                                                                                                                                                                                                                                         	| parameter details                                                                                                                                                                                                                                                                	| Examples                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               	|
|---------------	|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| "query"       	| A string to search for. Required.                                                                                                                                                                                                                                   	| query: "string"                                                                                                                                                                                                                                                                  	| https://rds-qa.swissartresearch.net/rest/reconciliation?queries={"q0":{"query":"Meinheim"}}                                                                                                                                                                                                                                                                                                                                                                                                           	|
| "limit"       	| An integer to specify how many results to return. Optional.                                                                                                                                                                                                         	| query: "string" limit: "number of results the query should display"                                                                                                                                                                                                              	| https://rds-qa.swissartresearch.net/rest/reconciliation?queries={"q0":{"query":"Meinheim", "limit":3}}                                                                                                                                                                                                                                                                                                                                                                                                	|
| "type"        	| A single string, or an array of strings, specifying the types of result e.g., person, product, ... The actual format of each type depends on the service (e.g., "Q515" as a Wikidata type). Optional. (Currently we treat this parameter as rdf:typeproperty value) 	| query: "string" type: "instance of rdf:type to be retrieved"                                                                                                                                                                                                                     	| https://rds-qa.swissartresearch.net/rest/reconciliation?queries=%7B%22q0%22%3A%7B%22query%22%3A%22Meinheim%22%2C%22type%22%3A%22http%3A%2F%2Fwww.geonames.org%2Fontology%23Feature%22%7D%7D&callback=alert                                                                                                                                                                                                                                                                                            	|
| "type_strict" 	| A string, one of "any", "all", "should". Optional.                                                                                                                                                                                                                  	| type_strict: "additional properties to be used for retrieving value"?any: the target candidate should have any or none of provided propertiesall: the target candidate should have all of provided propertiesshould: the target candidate should have one of provided properties 	| https://rds-qa.swissartresearch.net/rest/reconciliation?queries=%7B%22q0%22%3A%7B%22query%22%3A%22Shakarla%22%2C%22type%22%3A%22http%3A%2F%2Fwww.geonames.org%2Fontology%23Feature%22%2C%22type_strict%22%3A%22any%22%2C%22properties%22%3A%5B%7B%22pid%22%3A%20%22http%3A%2F%2Fwww.geonames.org%2Fontology%23countryCode%22%2C%22v%22%3A%20%22RU%22%7D%2C%20%7B%22pid%22%3A%20%22http%3A%2F%2Fwww.geonames.org%2Fontology%23officialName%22%2C%22v%22%3A%20%22Sjakarla%22%7D%5D%7D%7D&callback=alert 	|
| "properties"  	| Array of json object literals or links. Optional. Properties can be one of two types:literal property: {pid: string, v: string}orobject property: {pid: string, v: {id: string}} - where id: is IRI currently                                                       	| Additional properties to be used when retrieving values. How is that different from the one above?                                                                                                                                                                               	| https://rds-qa.swissartresearch.net/rest/reconciliation?queries=%7B%22q0%22%3A%7B%22query%22%3A%22Shakarla%22%2C%22type%22%3A%22http%3A%2F%2Fwww.geonames.org%2Fontology%23Feature%22%2C%22properties%22%3A%5B%7B%22pid%22%3A%20%22http%3A%2F%2Fwww.geonames.org%2Fontology%23countryCode%22%2C%22v%22%3A%20%22RU%22%7D%2C%20%7B%22pid%22%3A%20%22http%3A%2F%2Fwww.geonames.org%2Fontology%23officialName%22%2C%22v%22%3A%20%22Sjakarla%22%7D%5D%7D%7D                                                	|
|               	|                                                                                                                                                                                                                                                                     	|                                                                                                                                                                                                                                                                                  	|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        	|
|               	|                                                                                                                                                                                                                                                                     	|                                                                                                                                                                                                                                                                                  	|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        	|

And response is a list of reconciliation candidates described in the documentation:
For multiple queries, the response is a JSON literal object with the same keys as in the request

```
{
    "q0" : {
      "result" : { ... }
    },
    "q1" : {
      "result" : { ... }
    }
  }
```



Each result consists of a JSON literal object with the structure

```
{
    "result" : [
      {
        "id" : ... string, database ID ...
        "name" : ... string ...
        "type" : ... array of strings ...
        "score" : ... double ...
        "match" : ... boolean, true if the service is quite confident about the match ...
      },
      ... more results ...
    ],
    ... potentially some useful envelope data, such as timing stats ...
  }

```

The results should be sorted by decreasing score. The service must also support JSONP through a callback parameter ie &callback=foo.
POST /rest/reconciliation - support similar (to previous endpoint) functionality, the only difference is that JSON_QUERY is provided as JSON-object in the body of POST-request.

## Examples (cURL):


1. To get manifest use:

```bash
curl --location --request GET 'https://rds-qa.swissartresearch.net/rest/reconciliation' --header 'Content-Type: application/json' --header 'Accept: application/json'
```

2. To get response via GET use:

```bash
curl --location --request GET 'https://rds-qa.swissartresearch.net/rest/reconciliation?queries=%7B%22q0%22%3A%7B%22query%22%3A%22Meinheim%22%2C%22type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2004%2F02%2Fskos%2Fcore%23OrderedCollection%22}}&callback=alert' 
```

3. To get response via POST(raw-json) use:

```bash
curl --location --request POST 'https://rds-qa.swissartresearch.net/rest/reconciliation' --header 'Content-Type: application/json' --header 'Accept: application/json' --data-raw '{"q2":{"query":"Car","limit":3},"q3":{"query":"Home","limit":3},"q0":{"query":"Meinheim","limit":3}}'
```

4. To get response via POST(form) use:

```bash
curl --location --request POST 'https://rds-qa.swissartresearch.net/rest/reconciliation' --header 'Content-Type: application/json' --header 'Accept: application/json' --form 'queries={"q2":{"query":"Car","limit":3},"q3":{"query":"Home","limit":3},"q0":{"query":"Meinheim","limit":3}}'
```

5. To get response via POST(url-encoded form) use:

```bash
curl --location --request POST 'https://rds-qa.swissartresearch.net/rest/reconciliation' --header 'Content-Type: application/json' --header 'Accept: application/json' --data-urlencode 'queries={"q2":{"query":"Car","limit":3},"q3":{"query":"Home","limit":3},"q0":{"query":"Meinheim","limit":3}}'
```
## Label Service
Label service is a central service of the platform which maps labels to resources by their iris. (This service can be configured using preferredLabels configuration in the UI props)

To get labels for your resources use following endpoint:

POST: `{RDS-L/RDS-G}/rest/data/rdf/utils/getLabelsForRdfValue` - where the set of resource IRIs should be provide as JSON-array in the body of the POST request.

Example:
```
curl --location --request POST 'https://rds-mph.swissartresearch.net/rest/data/rdf/utils/getDescriptionForRdfValue' -
-header 'Accept: application/json' --header 'Content-Type: application/json' --data-raw '["http://example.com/resource-iri"]'
```

## Description Service
Description service is very similar to the label service. It maps descriptions to resources by their iris. (This service can be configured using preferredDescription configuration in the UI props)

To get descriptions for your resources use following endpoint:

POST: `{RDS-L/RDS-G}/rest/data/rdf/utils/getDescriptionForRdfValue` - where the set of resource IRIs should be provide as array in the body of POST request.
Example:
```
curl --location --request POST 'https://rds-global.swissartresearch.net:443/rest/data/rdf/utils/getDescriptionForRdfValue'  --header 'Content-Type: application/json' --data-raw '["http://example.com/resource-iri"]'
```