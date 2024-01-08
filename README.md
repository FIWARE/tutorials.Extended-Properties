# NGSI-LD Property subclasses
[<img src="https://img.shields.io/badge/NGSI-LD-d6604d.svg" width="90"  align="left" />](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.07.01_60/gs_cim009v010701p.pdf)
[<img src="https://fiware.github.io/tutorials.Getting-Started/img/Fiware.png" align="left" width="162">](https://www.fiware.org/)<br/>

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/FIWARE/tutorials.Getting-Started.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
[![JSON LD](https://img.shields.io/badge/JSON--LD-1.1-f06f38.svg)](https://w3c.github.io/json-ld-syntax/) <br/>
[![Documentation](https://img.shields.io/readthedocs/ngsi-ld-tutorials.svg)](https://ngsi-ld-tutorials.rtfd.io)

This tutorial examines the keyword syntax tokens of JSON-LD and introduces custom property types which extend NGSI-LD properties to cover multilingual capabilities and preferred enumeration names whilst reusing the data from the [Smart Farm example](https://github.com/FIWARE/tutorials.Getting-Started/tree/NGSI-LD). The tutorial uses
[cUrl](https://ec.haxx.se/) commands throughout.

[<img src="https://run.pstmn.io/button.svg" alt="Run In Postman" style="width: 128px; height: 32px;">](https://god.gw.postman.com/run-collection/217860-3b538d21-0f19-4c63-a9d6-e184ef829ca7?action=collection%2Ffork&source=rip_markdown&collection-url=entityId%3D217860-3b538d21-0f19-4c63-a9d6-e184ef829ca7%26entityType%3Dcollection%26workspaceId%3Db6e7fcf4-ff0c-47cb-ada4-e222ddeee5ac)
[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://github.com/flopezag/tutorials.Multilanguage/tree/develop)

## Contents

<details>
<summary><strong>Details</strong></summary>

-   [NGSI-LD Rules](#ngsi-ld-rules)
-   [Prerequisites](#prerequisites)
  -   [Docker Engine](#docker-engine-img-srchttpswwwdockercomfaviconico-alignleft--height30-width30-styleborder-right-stylesolid-border-right-width10px-border-colortransparent-background-transparent)
  -   [jq](#jq-img-srchttpsjqlanggithubiojqjqpng-alignleft--height19-width50-styleborder-right-stylesolid-border-right-width10px-border-colortransparent-background-transparent)
  -   [Postman](#postman-img-srchttpswwwpostmancom_ar-assetsimagesfavicon-1-48png-alignleft--height25-width35-styleborder-right-stylesolid-border-right-width10px-border-colortransparent-background-transparent)
  -   [GitPod](#gitpod-img-srchttpsgitpodiofaviconico-alignleft--height30-width30)
-   [Architecture](#architecture)
  -   [Start Up](#start-up)
  -   [Checking the system](#checking-the-system)
-   [Working with multilanguage properties](#working-with-multilanguage-properties)
  -   [Creating a new data entity](#creating-a-new-data-entity)
  -   [Reading multilingual data in normalized format](#reading-multilingual-data-in-normalized-format)
  -   [Reading multilingual data in key-value format](#reading-multilingual-data-in-key-value-format)
  -   [Querying for Multilingual Data](#querying-for-multilingual-data)
-   [Using an alternative `@context`](#using-an-alternative-context)
-   [License](#license)

</details>

# Understanding JSON-LD `@keywords`

The [JSON-LD syntax](https://www.w3.org/TR/json-ld/#syntax-tokens-and-keywords) defines a series of keywords to describe the structure of the JSON displayed. Since **NGSI-LD** is just a formally structured _extended subset_ of **JSON-LD**, **NGSI-LD** should be
directly or indirectly capable of offering an equivalent for all the functions defined by JSON-LD.

As an example, JSON-LD defines `@id` to indicate the unique identifier of an Entity, and `@type` to define the type of an Entity.
The NGSI-LD core `@context` further refines this further, so that `id`/`@id` and `type`/`@type` are considered as interchangable.

Both of the following syntaxes (with and without `@`) are acceptable in NGSI-LD:

```json
{
  "id": "urn:ngsi-ld:Building:farm001",
  "type": "Building",
  "@context" : "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.7.jsonld"
}
```


```json
{
  "@id": "urn:ngsi-ld:Building:farm001",
  "@type": "Building",
  "@context" : "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.7.jsonld"
}
```


Among the keywords defined in JSON-LD, the following terms are used or mapped within the NGSI-LD core `@context` to maintain their meaning when JSON-LD data is supplied.

- `@list` - Used to express an ordered set of data.
- `@json` - Used in association with unexpandable JSON objects
- `@language` - Used to specify the language for a particular string value or string array
- `@none` - Used as a default index value, when an attribute does not have the feature being indexed.
- `@value` - Used to specify the data that is associated with a particular property
- `@vocab` - Used to expand properties and values

Certain other keywords such as `@graph`, which describe statements about relationships are accepted in NGSI-LD, but are never processed directly by NGSI-LD Context brokers

For example. Looking at the core `@context`, the GeoProperty attribute
`coordinates` is fully defined as:

```json
"coordinates": {
  "@container": "@list",
  "@id": "geojson:coordinates"
}
```

This ensure that the ordering of the values in its array (longitude, latitude) is always maintained.

All ordinary NGSI-LD **Properties** (and **GeoProperties**) have a `value`, which is the equivalent of a JSON-LD `@value` - this mean that the `value` of a Property is just the data that is associated with a particular property.

However, there are recent updates to the NGSI-LD specification which have introduced various extensions or sub-classes to this principle, allowing the creation of NGSI-LD properties which directly conform to
JSON-LD keywords other than `@value`.

- An NGSI-LD **LanguageProperty** holds a set of internationalized strings and is defined using the JSON-LD `@language` keyword.
- An NGSI-LD **VocabularyProperty**  holds is a mapping of a URI to a value within the user'`@context` and is defined using the JSON-LD `@vocab` keyword.

In each case, the meaning of the resultant payload will be altered according to the standard JSON-LD definitions, so the output NGSI-LD remains fully valid JSON-LD.

## Entities within a Farm Management Information System (FMIS)

To illustrate some extended NGSI-LD properties within an FMIS system based on NGSI-LD, we will alter the previously defined **Building** Entity type. As a reminder this has been defined as follows

-   A building, such as a barn, is a real world bricks and mortar construct. **Building** entities would have properties
    such as:
    -   A name of the building e.g. "The Big Red Barn"
    -   The category of the building (e.g. "barn")
    -   An address "Friedrichstraße 44, 10969 Kreuzberg, Berlin"
    -   A physical location e.g. _52.5075 N, 13.3903 E_
    -   A filling level - the degree to which the building is full.
    -   A temperature - e.g. _21 °C_
    -   An association to the owner of the building (a real person)
    -   ...etc.

Taking the first attribute, the Property `name` could be localized into multiple languages, for example:
  -  **Big Red Barn**  in English
  -  **Große Rote Scheune** in German
  -  **大きな赤い納屋** in Japanese

Similarly, even if all participants in a data space can agree for a common URI for the definition of the enumerations of all the different building types within `category`, internally within their own systems, they may be requied to display these enumerations with their own localised values.


For example if the FMIS follows the URIs defined by openstreetmap.org. A building designated as A _"barn"_ would actually be be defined by the URI: `https://wiki.openstreetmap.org/wiki/Tag:building%3Dbarn`. A JSON-LD `@context` could be used to shorten this as required.


If a user wanted the `category` defined as `"barn"` internally within their system, the following JSON-LD `@context` could be used:

```json
{
  "@context": {
    "barn": "https://wiki.openstreetmap.org/wiki/Tag:building%3Dbarn",
  }
}
```

If a user wanted the `category` defined as `"scheune"` internally within their system, the following JSON-LD `@context` could be used:

```json
{
  "@context": {
    "scheune": "https://wiki.openstreetmap.org/wiki/Tag:building%3Dbarn",
  }
}
```


The definition and redefinition of enumerations is not necessarily just a language localisation issue. It is possible that an FMIS may wish to use a separate code
list of values for
regulatory reasons. For example, the names of ingredients within a pesticide,
could be regulated by law and the required name could differ based on the market in which the product is sold (e.g. `Water`, `H2O`, `Hydrogen Hydroxide`, `Oxygen Dihydride`,
 `Hydric Acid`)


## Architecture

The demo application will send and receive NGSI-LD calls to a compliant context broker. Although the standardised NGSI-LD
interface is available across multiple context brokers, we only need to pick one - for example the
[Scorpio Broker](https://fiware-orion.readthedocs.io/en/latest/). The application will therefore only make use of
one FIWARE component.

Currently, the Orion Context Broker relies on open source [MongoDB](https://www.mongodb.com/) technology to hold the
current state of the context data it contains and persistent information relevant to subscriptions and registrations.
Other Context Brokers such as Scorpio or Stellio are using [PostgreSQL](https://www.postgresql.org/) for state
information.

To promote interoperability of data exchange, NGSI-LD context brokers explicitly expose a
[JSON-LD `@context` file](https://json-ld.org/spec/latest/json-ld/#the-context) to define the data held within the
context entities. This defines a unique URI for every entity type and every attribute so that other services outside of
the NGSI domain are able to pick and choose the names of their data structures. Every `@context` file must be available
on the network. In our case the tutorial application will be used to host a series of static files.

Therefore, the architecture will consist of three elements:

-   The [Scorpio Context Broker](https://scorpio.readthedocs.io/) which will receive requests using
    [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
-   The underlying [Postgres](https://www.postgresql.org/) database:
    -   Used by the Scorpio Context Broker to hold context data information such as data entities, subscriptions and
        registrations.
-   An HTTP **Web-Server** which offers static `@context` files defining the context entities within the system.

Since all interactions between the three elements are initiated by HTTP requests, the elements can be containerized and
run from exposed ports.

The necessary configuration information can be seen in the services section of the associated `scorpio.yml` file:

```yaml
  scorpio:
    labels:
      org.fiware: 'tutorial'
    image: quay.io/fiware/scorpio:java-${SCORPIO_VERSION}
    hostname: scorpio
    container_name: fiware-scorpio
    networks:
      - default
    ports:
      - "1026:9090"
    depends_on:
      - postgres

```

```yaml
  postgres:
    labels:
      org.fiware: 'tutorial'
    image: postgis/postgis
    hostname: postgres
    container_name: db-postgres
    networks:
      - default
    ports:
      - "5432"
    environment:
      POSTGRES_USER: ngb
      POSTGRES_PASSWORD: ngb
      POSTGRES_DB: ngb
    logging:
      driver: none
    volumes:
      - postgres-db:/var/lib/postgresql/data
```

```yaml
ld-context:
    labels:
      org.fiware: 'tutorial'
    image: httpd:alpine
    hostname: context
    container_name: fiware-ld-context
    ports:
        - "3004:80"
    volumes:
        - data-models:/usr/local/apache2/htdocs/
    healthcheck:
      test: (wget --server-response --spider --quiet  http://ld-context/ngsi-context.jsonld 2>&1 | awk 'NR==1{print $$2}'|  grep -q -e "200") || exit 1
```

All containers reside on the same network - the Scorpop Context Broker is listening on Port `9090` internally and `1026` externally and PostGres is
listening on the default port `5432` and the httpd web server is offering `@context` files on port `80`. All containers
are also exposing ports externally - this is purely for the tutorial access - so that cUrl or Postman can access them
without being part of the same network. The command-line initialization should be self-explanatory.


## Prerequisites

### Docker Engine <img src="https://www.docker.com/favicon.ico" align="left"  height="30" width="30" style="border-right-style:solid; border-right-width:10px; border-color:transparent; background: transparent">

To keep things simple all components will be run using [Docker](https://www.docker.com). **Docker** is a container
technology which allows different components to be isolated into their respective environments.

-   To install Docker on Windows follow the instructions [here](https://docs.docker.com/docker-for-windows/)
-   To install Docker on Mac follow the instructions [here](https://docs.docker.com/docker-for-mac/)
-   To install Docker on Linux follow the instructions [here](https://docs.docker.com/install/)

**Docker Compose** is a tool for defining and running multi-container Docker applications. A
[YAML file](/docker-compose/orionld.yml) is used to configure the required services for the application. This means all
container services can be brought up in a single command.

Compose V1 is discontinued, nevertheless Compose V2 has replaced it and is now integrated into all current Docker
Desktop versions. Therefore, it is not needed to install the extension to the docker engine to execute the
docker compose commands.


### Start Up

All services can be initialised from the command-line by running the [services](/services) Bash script provided within 
the repository. Please clone the repository and create the necessary images by running the commands as shown:

```bash
git clone http://github.com/fiware/tutorials.Extended-Properties.git
cd tutorials.Extended-Properties

./services [start]
```

> **Note:** If you want to clean up and start over again you can do so with the following command:
>
> ```
> ./services stop
> ```

---

### Checking the system

Once the services have started up, and before interacting with the context broker itself, it is useful to check that the
necessary prerequisites are in place.

### Reading `@context` files

Two `@context` files have been generated and hosted on the tutorial application. They would be used by different organizations within the data space, and internally they define the names of attributes and enumerations in different ways.

-   [`ngsi-context.jsonld`](http://localhost:3000/data-models/ngsi-context.jsonld) -The **NGSI-LD** `@context` serves to
    define all attributes when sending data to the context broker or retrieving data. This
    `@context` must be used for all **NGSI-LD** to **NGSI-LD** interactions.

-   [`alternate-context.jsonld`](http://localhost:3000/data-models/alternate-context.jsonld) is an alternative
    **JSON-LD** definition of the attributes of the data models used by a third-party. In this case we have a German speaking customer who wishes to have all attribute names and enumerations to be defined using terminology common in the German language. Effectively, internally within their their billing application a different set of short names for attributes is used. Their `@context` file reflects
    the agreed mapping between attribute names.

The full data model description for a **Building** entity as used in this tutorial is based on the standard
[Smart Data Models definition](https://github.com/smart-data-models/dataModel.Building/tree/master/Building).
A [Swagger Specification](https://petstore.swagger.io/?url=https://smart-data-models.github.io/dataModel.Building/Building/swagger.yaml)
of the same model is also available, and would be used to generate code stubs in a full application.


## Working with multilanguage properties

Sometimes, it is required to  localize strings to offer variations for different languages in the creation and consumption of Entity data. In order to
proceed, we need to create initially a new entity data that defines the new data type `LanguageProperty` and use the 
sub-attribute `LanguageMap` (and not `value`) to keep the representation of the values of this attribute in different
languages. 

This `LanguageMap` corresponds to a JSON object consisting of a series of key-value pairs where the keys shall be JSON 
strings representing [IETF RFC 5646](https://www.rfc-editor.org/info/rfc5646) language codes.

### Creating a new data entity

Let's create a farm **Building** entity in which we want to make the `name` available in three different languages, English, German, and Japanese. The process will be to
send a **POST** request to the Broker with the following information:

#### :one: Request:

```console
curl -iX POST 'http://localhost:1026/ngsi-ld/v1/entities/' \
-H 'Content-Type: application/ld+json' \
--data-raw '{
    "id": "urn:ngsi-ld:Building:farm001",
    "type": "Building",
    "category": {
        "type": "VocabularyProperty",
        "vocab": ["farm"]
    },
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Großer Stern 1",
            "addressRegion": "Berlin",
            "addressLocality": "Tiergarten",
            "postalCode": "10557"
        },
        "verified": {
            "type": "Property",
            "value": true
        }
    },
    "location": {
        "type": "GeoProperty",
        "value": {
             "type": "Point",
             "coordinates": [13.3505, 52.5144]
        }
    },
    "name": {
        "type": "LanguageProperty",
        "languageMap": {
          "en": "Victory Farm",
          "de": "Bauernhof von Sieg",
          "ja": "ビクトリーファーム"
        }
    },
    "@context": "http://context/ngsi-context.jsonld"
}'
```


#### Response:

The response that we obtain will be something similar (except the `Date` value) to the following content:

```console
HTTP/1.1 201 Created
Date: Sat, 16 Dec 2023 08:39:32 GMT
Location: /ngsi-ld/v1/entities/urn:ngsi-ld:Building:farm001
Content-Length: 0
```


#### 3️⃣ Request:

Each subsequent entity must have a unique `id` for the given `type`. Note that within a `languageMap`, the `@none` key-value pair indicates the default fallbakc value to be displayed  for unknown languages.

```console
curl -iX POST 'http://localhost:1026/ngsi-ld/v1/entities/' \
  -H 'Content-Type: application/json' \
  -H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -d '{
    "id": "urn:ngsi-ld:Building:barn002",
    "type": "Building",
    "category": {
        "type": "VocabProperty",
        "vocab": ["barn"]
    },
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Straße des 17. Juni",
            "addressRegion": "Berlin",
            "addressLocality": "Tiergarten",
            "postalCode": "10557"
        },
        "verified": {
            "type": "Property",
            "value": true
        }
    },
     "location": {
        "type": "GeoProperty",
        "value": {
             "type": "Point",
              "coordinates": [13.3698, 52.5163]
        }
    },
    "name": {
        "type": "LanguageProperty",
        "languageMap": {
          "@none": "The Big Red Barn",
          "en": "Big Red Barn",
          "de": "Große Rote Scheune",
          "ja": "大きな赤い納屋"
        }
    }
}'
```


### Reading multilingual data in normalised format

Imagining that we want to get details of a specific entity (`urn:ngsi-ld:Building:farm001`) in normalised
format and without any reference to the language that we want to obtain the data. We should execute the following 
command:

#### :three: Request:

```console
curl -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:farm001?attrs=name' \
  -H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
```

And the response that we obtain the whole `languageMap` including all the string values defined for the different languages:

#### Response:

```json
{
  "id": "urn:ngsi-ld:Building:farm001",
  "type": "Building",
  "name": {
    "type": "LanguageProperty",
    "languageMap": {
      "en": "Victory Farm",
      "de": "Bauernhof von Sieg",
      "ja": "ビクトリーファーム"
    }
  }
}
```

On the other hand, if we decided to specify that we wanted to receive the value (or values) but only in *German*
language, we should specify the corresponding query parameter `lang` equal to `de`.

#### :four: Request:

```console
curl -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:farm001?attrs=name&lang=de' \
  -H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
```

In this case, the response provides a new sub-attribute `lang` with the details of the language that was selected 
together with the sub-attribute `value` with the content of the string in the corresponding *German* language. It is
important to notice that in this response the value of `type` is  now *Property* and there is no `LanguageMap` but `value`
sub-attribute.

#### Response:

```json
{
  "id": "urn:ngsi-ld:Building:farm001",
  "type": "Building",
  "name": {
    "type": "Property",
    "lang": "de",
    "value": "Bauernhof von Sieg"
  }
}
```

### Reading multilingual data in key-value format

If we wanted to get the response in key-value format, we need to send the corresponding request parameter `options` 
equal to `keyValues`:

#### :five: Request:

```console
curl -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:farm001?attrs=name&options=keyValues' \
  -H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
```

#### Response:

```json
{
  "id": "urn:ngsi-ld:Building:farm001",
  "name": {
    "languageMap": {
      "en": "Victory Farm",
      "de": "Bauernhof von Sieg",
      "ja": "ビクトリーファーム"
    }
  }
}
```

and if we wanted to get only the corresponding value of the `name` in *English* language:

#### :six: Request:

```console
curl -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:farm001?attrs=name&options=keyValues&lang=en' \
  -H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
```

#### Response:

```json
{
  "id": "urn:ngsi-ld:Building:farm001",
  "name": "Victory Farm"
}
```



### Querying for Multilingual Data

Use the standard Object attribute bracket `[ ]` notation when querying `LanguageProperties`. For example, if we want to
obtain the Building whose name is equal to `Big Red Barn` in English.

#### :seven: Request:

```console
curl -L -g 'http://localhost:1026/ngsi-ld/v1/entities/?type=Building&attrs=name&q=name[en]%3D%3D%22Big%20Red%20Barn%22' \
  -H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -H 'Accept: application/ld+json'
```

#### Response:

```json
[
  {
    "id" : "urn:ngsi-ld:Building:barn002",
    "type" : "Building",
    "name" : {
      "type" : "LanguageProperty",
      "languageMap" : {
        "@none" : "The Big Red Barn",
        "en" : "Big Red Barn",
        "de" : "Große Rote Scheune",
        "ja" : "大きな赤い納屋"
      }
    },
    "@context" : [ "http://context/ngsi-context.jsonld", "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.7.jsonld" ]
  }
]
```

Now, I wanted to receive the response but corresponding to `Große Rote Scheune` in *Any* language:

#### :eight: Request:

```console
curl -L -g 'http://localhost:1026/ngsi-ld/v1/entities/?type=Building&attrs=name&q=name[*]%3D%3D%22Gro%C3%9Fe%20Rote%20Scheune%22' \
  -H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -H 'Accept: application/ld+json'
```


#### Response:

Using the Asterisk Syntax `*` checks for all available languages.

```json
[
  {
    "id": "urn:ngsi-ld:Building:barn002",
    "type": "Building",
    "name": {
        "type": "LanguageProperty",
        "languageMap": {
            "@none" : "The Big Red Barn",
            "en": "Big Red Barn",
            "de": "Große Rote Scheune",
            "ja": "大きな赤い納屋"
        }
    },
    "@context": [
        "http://context/ngsi-context.jsonld",
        "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.7.jsonld"
    ]
  }
]
```



### Querying for Multilingual Data

Use the standard Object attribute bracket `[ ]` notation when querying `LanguageProperties`. For example, if we want to

#### :nine: Request:

```console
curl -L -g 'http://localhost:1026/ngsi-ld/v1/entities/?type=Building&lang=fr&attrs=name&q=name[*]%3D%3D%22Gro%C3%9Fe%20Rote%20Scheune%22' \
  -H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -H 'Accept: application/ld+json'
```


#### :one::zero: Request:



```console
curl -L -g 'http://localhost:1026/ngsi-ld/v1/entities/?type=Building&attrs=name&q=name[en]==%22Big%20Red%20Barn%22&lang=fr' \
  -H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -H 'Accept: application/ld+json'
```

#### :one::zero: Request:

```console
curl -L -g 'http://localhost:1026/ngsi-ld/v1/entities/?type=Building&attrs=name&q=name[en]==%Victory%20Farm%22&lang=fr' \
  -H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -H 'Accept: application/ld+json'
```


## Using an alternative `@context`

The simple **NGSI-LD** `@context` is merely a mechanism for mapping URNs. It is therefore possible to retrieve _the same
data_ using a different set of short names, in different languages.

The `alternate-context-fi.jsonld` maps the names of various attributes to their equivalents in Finnish. The
`alternate-context-it.jsonld` provides their equivalent in Italian. If it is supplied in the request, a query can be 
made using alternate short names (e.g., `type=Building` becomes `type=PuntoDiInteresse` or
`type=KiinnostuksenKohde`).

There is a limitation in this mapping, it is not possible to change the core context attribute names, like `id`, `type`,
or `@context` for example.

Let's try to recover the information about the farm using the alternate context file for the *German*
language.

#### :nine: Request:

```console
curl -L -g 'http://localhost:1026/ngsi-ld/v1/entities/?type=Geb%C3%A4ude&q=name[*]%3D%3D%22Gro%C3%9Fe%20Rote%20Scheune%22&lang=en&attrs=name%2Ckategorie' \
  -H 'Link: <http://context/alternate-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -H 'Accept: application/ld+json'
```

#### Response:

The response is returned in JSON-LD format with short form attribute names (`kategorie`, `name`, `PuntoDiInteresse`)
which correspond to the short names provided in the alternate context. Note that core context terms (`id`, `type`, 
`value`, etc.) cannot be overridden directly but would require an additional **JSON-LD** expansion/compaction operation 
(programmatically).

```json
[
  {
    "id": "urn:ngsi-ld:Building:barn002",
    "type": "Gebäude",
    "name": {
      "type": "LanguageProperty",
      "languageMap": {
          "en": "Big Red Barn",
          "de": "Große Rote Scheune",
          "ja": "大きな赤い納屋"
      }
    },
    "kategorie": {
      "type": "VocabularyProperty",
      "vocab": "scheune"
    },
    "@context": [
        "http://context/alternate-context.jsonld",
        "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.7.jsonld"
    ]
  }
]
```

If we change the context to a *Finnish* language:

#### :ten: Request:

```console
curl -L 'http://localhost:1026/ngsi-ld/v1/entities/?type=Geb%C3%A4ude&q=kategorie%3D%3Dbauernhof&attrs=name%2Ckategorie' \
-H 'Link: <http://context/alternate-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/ld+json'
```

#### Response:

The response is returned in JSON-LD format with short form attribute names (`luokka`, `nimi`, `KiinnostuksenKohde`).

```json
[
  {
    "@context": "http://context/alternate-context-fi.jsonld",
    "id": "urn:ngsi-ld:Building:poi123456",
    "type": "KiinnostuksenKohde",
    "luokka": "107",
    "location": {
      "type": "Point",
      "coordinates": [
        60.17021,
        24.95212
      ]
    },
    "nimi": {
      "languageMap": {
        "fi": "Helsingin tuomiokirkko",
        "en": "Helsinki Cathedral",
        "it": "Duomo di Helsinki"
      }
    }
  }
]
```

---

## License

[MIT](LICENSE) © 2020-2023 FIWARE Foundation e.V.
