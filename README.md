# Multilingual tutorial
[<img src="https://img.shields.io/badge/NGSI-LD-d6604d.svg" width="90"  align="left" />](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.07.01_60/gs_cim009v010701p.pdf)
[<img src="https://fiware.github.io/tutorials.Getting-Started/img/Fiware.png" align="left" width="162">](https://www.fiware.org/)<br/>

[![License: MIT](https://img.shields.io/github/license/FIWARE/tutorials.Getting-Started.svg)](https://opensource.org/licenses/MIT)
[![JSON LD](https://img.shields.io/badge/JSON--LD-1.1-f06f38.svg)](https://w3c.github.io/json-ld-syntax/) <br/>

This tutorial examines the management multilingual capabilities inside the ETSI NGSI-LD API. The tutorial uses 
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

## NGSI-LD Rules

**NGSI-LD** is a formally structured _extended subset_ of **JSON-LD**. Therefore, **NGSI-LD** offers all the
interoperability and flexibility of **JSON-LD** itself. It also defines its own core `@context` which cannot be
overridden for **NGSI-LD** operations. This means that **NGSI-LD** users agree to a common well-defined set of rules for
structuring their data, and then supplement this with the rest of the **JSON-LD** specification.

Whilst interacting directly with **NGSI-LD** interface of the context broker the additional **NGSI-LD** rules must be
respected. However after the data has been extracted it is possible to loosen this requirement and pass the results to
third parties as **JSON-LD**.

This tutorial is a simple introduction to the rules and restrictions behind **NGSI-LD** and will create some **NGSI-LD**
entities and then extract the data in different formats. The two main data formats are _normalized_ and
_key-value-pairs_. Data returned in the _normalised_ format respects the **NGSI-LD** rules and may be used directly by
another context broker (or any other component offering an **NGSI-LD** interface). Data returned in the
_key-value-pairs_ format is by definition not **NGSI-LD**.

## Prerequisites

### Docker Engine <img src="https://www.docker.com/favicon.ico" align="left"  height="30" width="30" style="border-right-style:solid; border-right-width:10px; border-color:transparent; background: transparent">

To keep things simple all components will be run using [Docker](https://www.docker.com). **Docker** is a container
technology which allows to different components isolated into their respective environments.

-   To install Docker on Windows follow the instructions [here](https://docs.docker.com/docker-for-windows/)
-   To install Docker on Mac follow the instructions [here](https://docs.docker.com/docker-for-mac/)
-   To install Docker on Linux follow the instructions [here](https://docs.docker.com/install/)

**Docker Compose** is a tool for defining and running multi-container Docker applications. A
[YAML file](/docker-compose/orionld.yml) is used configure the required services for the application. This means all 
container services can be brought up in a single command. 

Compose V1 is discontinued, nevertheless Compose V2 has replaced it and is now integrated into all current Docker 
Desktop versions. Therefore, it is not needed to install anymore any extension to the docker engine to execute the 
docker compose commands.

### jq <img src="https://jqlang.github.io/jq/jq.png" align="left"  height="19" width="50" style="border-right-style:solid; border-right-width:10px; border-color:transparent; background: transparent">

[jq](https://jqlang.github.io/jq/) is a lightweight and flexible command-line JSON processor which can be used to format
the JSON responses received from the context broker and other FIWARE components. More information about how to use jq
can be found [here](https://www.digitalocean.com/community/tutorials/how-to-transform-json-data-with-jq). `jq-1.6` is
recommended.

### Postman <img src="https://www.postman.com/_ar-assets/images/favicon-1-48.png" align="left"  height="25" width="35" style="border-right-style:solid; border-right-width:10px; border-color:transparent; background: transparent">

The tutorials which use HTTP requests supply a collection for use with the Postman utility. Postman is a testing
framework for REST APIs. The tool can be downloaded from [www.getpostman.com](https://www.getpostman.com). All the
FIWARE Postman collections can be downloaded directly from the
[Postman API network](https://explore.postman.com/team/3mM5EY6ChBYp9D).

### GitPod <img src="https://gitpod.io/favicon.ico" align="left"  height="30" width="30">

[Gitpod](https://github.com/gitpod-io/gitpod) is an open-source Kubernetes application for ready-to-code cloud
development environments that spins up an automated dev environment for each task, in the cloud. It enables you to run
the tutorials in a cloud development environment directly from your browser or your Desktop IDE. The default environment
is based on Ubuntu and includes Java `11.0.16` and Maven `3.8.6`.

## Architecture

The demo application will send and receive NGSI-LD calls to a compliant context broker. Since the standardized NGSI-LD
interface is available across multiple context brokers, so we only need to pick one - for example the
[Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/). The application will therefore only make use of
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

-   The [OrionLD Context Broker](https://fiware-orion.readthedocs.io/en/latest/) which will receive requests using
    [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
-   The underlying [MongoDB](https://www.mongodb.com/) database:
    -   Used by the OrionLD Context Broker to hold context data information such as data entities, subscriptions and
        registrations.
-   An HTTP **Web-Server** which offers static `@context` files defining the context entities within the system.

Since all interactions between the three elements are initiated by HTTP requests, the elements can be containerized and
run from exposed ports.

The necessary configuration information can be seen in the services section of the associated `orion-ld.yml` file:

```yaml
orionld:
    labels:
      org.fiware: 'tutorial'
    platform: linux/amd64
    image: quay.io/fiware/orion-ld:1.4.0
    hostname: orionld
    container_name: fiware-orionld
    depends_on:
      - mongo-db
    networks:
      - default
    ports:
      - 1026:1026
    command: -dbhost mongo-db -logLevel DEBUG -forwarding -experimental
    healthcheck:
      test: curl --fail -s http://orionld:1026/version || exit 1
      interval: 5s
```

```yaml
mongo-db:
    labels:
      org.fiware: 'tutorial'
    image: mongo:4.4
    hostname: mongo-db
    container_name: db-mongo
    expose:
      - "27017"
    ports:
      - "27017:27017"
    networks:
      - default
    volumes:
      - mongo-db:/data/db
      - mongo-config:/data/configdb
    healthcheck:
      test: [ "CMD", "mongo", "--quiet", "127.0.0.1/test", "--eval", "'quit(db.runCommand({ ping: 1 }).ok ? 0 : 2)'"]
      interval: 5s
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

All containers are residing on the same network - the OrionLD Context Broker is listening on Port `1026` and MongoDB is
listening on the default port `27017` and the httpd web server is offering `@context` files on port `80`. All containers
are also exposing ports externally - this is purely for the tutorial access - so that cUrl or Postman can access them
without being part of the same network. The command-line initialization should be self-explanatory.

### Start Up

All services can be initialised from the command-line by running the [services](/services) Bash script provided within 
the repository. Please clone the repository and create the necessary images by running the commands as shown:

```bash
git clone git@github.com:flopezag/tutorials.Multilanguage.git
cd tutorials.Multilanguage

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

Three `@context` files have been generated and hosted on the tutorial application. They serve different roles.

-   [`ngsi-context.jsonld`](http://localhost:3000/data-models/ngsi-context.jsonld) -The **NGSI-LD** `@context` serves to
    define all attributes when sending data to the context broker or retrieving data in _normalized_ format. This
    `@context` must be used for all **NGSI-LD** to **NGSI-LD** interactions.

-   [`alternate-context.jsonld`](http://localhost:3000/data-models/alternate-context.jsonld) is an alternative
    **JSON-LD** definition of the attributes of the data models used by a third-party. Internally their billing 
    application used different short names for attributes depending of the language. Their `@context` file reflects 
    the agreed mapping between attribute names.

The full data model description for a **PointOfInterest** entity as used in this tutorial is based on the standard 
[Smart Data Models definition](https://github.com/smart-data-models/dataModel.PointOfInterest/tree/master/PointOfInterest). 
A [Swagger Specification](https://petstore.swagger.io/?url=https://smart-data-models.github.io/dataModel.PointOfInterest/PointOfInterest/swagger.yaml)
of the same model is also available, and would be use to generate code stubs in a full application.

### Checking the service health

You can check if the OrionLD Context Broker is running by making an HTTP request to the exposed port:

#### :one: Request:

```console
curl -X GET \
  'http://localhost:1026/version'
```

#### Response:

The response will look similar to the following:

```json
{
  "orionld version": "1.4.0",
  "orion version":   "1.15.0-next",
  "uptime":          "0 d, 0 h, 49 m, 50 s",
  "git_hash":        "746e13b343987d846b3451fe1f943600c4b2abe9",
  "compile_time":    "Sat Aug 26 06:19:10 UTC 2023",
  "compiled_by":     "root",
  "compiled_in":     "",
  "release_date":    "Sat Aug 26 06:19:10 UTC 2023",
  "doc":             "https://fiware-orion.readthedocs.org/en/master/"
}
```

The format of the version response has not changed. The `release_date` must be 26th August 2023 or later to be able to
work with the requests defined below.

## Working with multilanguage properties

Sometimes, it is required the use of different language in the creation and consumption of Entity data. In order to 
proceed, we need to create initially a new entity data that define the new data type `LanguageProperty` and use the 
sub-attribute `LanguageMap` (and not value) to keep the representation of the values of this attribute in different 
languages. 

This `LanguageMap` corresponds to a JSON object consisting of a series of key-value pairs where the keys shall be JSON 
strings representing [IETF RFC 5646](https://www.rfc-editor.org/info/rfc5646) language codes.

### Creating a new data entity

Let's create a Poit of Interest data in which we want to keep the detail information about the Helsinki Cathedral, gut
for the value of the name, we use three different languages, English, Finnish, and Italian. The process will be to send 
a request to the Broker with the following information:

#### :two: Request:

```console
curl -iX POST 'http://localhost:1026/ngsi-ld/v1/entities/' \
  -H 'Content-Type: application/ld+json' \
  -d '{
  "id": "urn:ngsi-ld:PointOfInterest:poi123456",
  "type": "PointOfInterest",
  
  "category": {
      "type": "Property",
      "value": ["107"]
  },

  "location": {
      "type": "GeoProperty",
      "value": {
          "type": "Point",
          "coordinates": [60.17021,24.95212]
      }
  },

  "name": {
      "type": "LanguageProperty",
      "languageMap": {
          "fi": "Helsingin tuomiokirkko",
          "en": "Helsinki Cathedral",
          "it": "Duomo di Helsinki"
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
Location: /ngsi-ld/v1/entities/urn:ngsi-ld:PointOfInterest:poi123456
Content-Length: 0
```

### Reading multilingual data in normalized format

Imaging that we want to get details of a specific entity (`urn:ngsi-ld:PointOfInterest:poi123456`) in normalized format
and without any reference to the language that we want to obtain the data. We should execute the following command:

#### :three: Request:

```console
curl -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:PointOfInterest:poi123456?attrs=name' \
  -H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' | jq .
```

And the response that we obtain include all the string values defined for the different languages:

#### Response:

```jsonld
{
  "id": "urn:ngsi-ld:PointOfInterest:poi123456",
  "type": "PointOfInterest",
  "name": {
    "type": "LanguageProperty",
    "languageMap": {
      "fi": "Helsingin tuomiokirkko",
      "en": "Helsinki Cathedral",
      "it": "Duomo di Helsinki"
    }
  }
}
```

On the other side, if we decided to specify that we wanted to receive the value (or values) but only in *Italian* 
language, we should specify the corresponding query parameter `lang` equal to `it`.

#### :four: Request:

```console
curl -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:PointOfInterest:poi123456?attrs=name&lang=it' \
  -H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' | jq .
```

In this case, the response provides a new sub-attribute `lang` with the details of the language that was selected 
together with the sub-attribute `value` with the content of the string in the corresponding *Italian* language. It is 
important to notice that in this response the value of `type` is *Property* and there is no `LanguageMap` but `value` 
sub-attribute.

#### Response:

```jsonld
{
  "id": "urn:ngsi-ld:PointOfInterest:poi123456",
  "type": "PointOfInterest",
  "name": {
    "type": "Property",
    "lang": "it",
    "value": "Duomo di Helsinki"
  }
}
```

### Reading multilingual data in key-value format

If we wanted to get the response in key-value format, we need to send the corresponding request parameter `options` 
equal to `keyValues`:

#### :five: Request:

```console
curl -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:PointOfInterest:poi123456?attrs=name&options=keyValues' \
  -H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' | jq .
```

#### Response:

```jsonld
{
  "id": "urn:ngsi-ld:PointOfInterest:poi123456",
  "type": "PointOfInterest",
  "name": {
    "languageMap": {
      "fi": "Helsingin tuomiokirkko",
      "en": "Helsinki Cathedral",
      "it": "Duomo di Helsinki"
    }
  }
}
```

and if we wanted to get only the corresponding value of the `name` in *English* language:

#### :six: Request:

```console
curl -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:PointOfInterest:poi123456?attrs=name&options=keyValues&lang=en' \
  -H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' | jq .
```

#### Response:

```jsonld
{
  "id": "urn:ngsi-ld:PointOfInterest:poi123456",
  "type": "PointOfInterest",
  "name": "Helsinki Cathedral"
}
```


### Querying for Multilingual Data

Use the standard Object attribute bracket `[ ]` notation when querying `LanguageProperties`. For example, if we want to
obtain the PointOfInterest whose name is equal to `Helsinki Cathedral` in English.

#### :seven: Request:

```console
curl -X GET 'http://localhost:1026/ngsi-ld/v1/entities?type=PointOfInterest&q=name\[it\]=="Duomo+di+Helsinki"' \
  -H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' | jq .
```

#### Response:

```jsonld
[
  {
    "id": "urn:ngsi-ld:PointOfInterest:poi123456",
    "type": "PointOfInterest",
    "category": {
      "type": "Property",
      "value": "107"
    },
    "location": {
      "type": "GeoProperty",
      "value": {
        "type": "Point",
        "coordinates": [
          60.17021,
          24.95212
        ]
      }
    },
    "name": {
      "type": "LanguageProperty",
      "languageMap": {
        "fi": "Helsingin tuomiokirkko",
        "en": "Helsinki Cathedral",
        "it": "Duomo di Helsinki"
      }
    }
  }
]
```

Now, I wanted to receive the response but corresponding to the *Finish* language:

#### :eight: Request:

```console
curl -X GET 'http://localhost:1026/ngsi-ld/v1/entities?type=PointOfInterest&q=name\[it\]=="Duomo+di+Helsinki"&lang=fi' \
  -H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' | jq .
```

#### Response:

```jsonld
[
  {
    "id": "urn:ngsi-ld:PointOfInterest:poi123456",
    "type": "PointOfInterest",
    "category": {
      "type": "Property",
      "value": "107"
    },
    "location": {
      "type": "GeoProperty",
      "value": {
        "type": "Point",
        "coordinates": [
          60.17021,
          24.95212
        ]
      }
    },
    "name": {
      "type": "Property",
      "lang": "fi",
      "value": "Helsingin tuomiokirkko"
    }
  }
]
```

## Using an alternative `@context`

The simple **NGSI-LD** `@context` is merely a mechanism for mapping URNs. It is therefore possible to retrieve _the same
data_ using a different set of short names, in different languages.

The `alternate-context-fi.jsonld` maps the names of various attributes to their equivalents in Finnish. The
`alternate-context-it.jsonld` provides their equivalent in Italian. If it is supplied in the request, a query can be 
made using alternate short names (e.g., `type=PointOfInterest` becomes `type=PuntoDiInteresse` or 
`type=KiinnostuksenKohde`).

There is a limitation in this mapping, it is not possible to change the core context attribute names, like _id_, _type_,
or _context_ for example.

Let's try to recover the information about the point of interest using the alternate context file for the *Italian* 
language.

#### :nine: Request:

```console
curl -G -X GET \
    'http://localhost:1026/ngsi-ld/v1/entities' \
-H 'Link: <http://context/alternate-context-it.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/ld+json' \
    -d 'type=PuntoDiInteresse' \
    -d 'options=keyValues'
```

#### Response:

The response is returned in JSON-LD format with short form attribute names (`categoria`, `nome`, `PuntoDiInteresse`) 
which correspond to the short names provided in the alternate context. Note that core context terms (`id`, `type`, 
`value`, etc.) cannot be overridden directly but would require an additional **JSON-LD** expansion/compaction operation 
(programmatically).

```jsonld
[
  {
    "@context": "http://context/alternate-context-it.jsonld",
    "id": "urn:ngsi-ld:PointOfInterest:poi123456",
    "type": "PuntoDiInteresse",
    "categoria": "107",
    "location": {
      "type": "Point",
      "coordinates": [
        60.17021,
        24.95212
      ]
    },
    "nome": {
      "languageMap": {
        "fi": "Helsingin tuomiokirkko",
        "en": "Helsinki Cathedral",
        "it": "Duomo di Helsinki"
      }
    }
  }
]
```

If we change the context to a *Finnish* language:

#### :ten: Request:

```console
curl -G -X GET \
    'http://localhost:1026/ngsi-ld/v1/entities' \
-H 'Link: <http://context/alternate-context-fi.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/ld+json' \
    -d 'type=KiinnostuksenKohde' \
    -d 'options=keyValues'
```

#### Response:

The response is returned in JSON-LD format with short form attribute names (`luokka`, `nimi`, `KiinnostuksenKohde`).

```jsonld
[
  {
    "@context": "http://context/alternate-context-fi.jsonld",
    "id": "urn:ngsi-ld:PointOfInterest:poi123456",
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
