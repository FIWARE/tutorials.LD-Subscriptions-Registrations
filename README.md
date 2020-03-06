[![FIWARE Banner](https://fiware.github.io/tutorials.LD-Subscriptions-Registrations/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Relationships-Linked-Data.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://nexus.lab.fiware.org/repository/raw/public/badges/stackoverflow/fiware.svg)](https://stackoverflow.com/questions/tagged/fiware)
[![NGSI LD](https://img.shields.io/badge/NGSI-linked_data-red.svg)](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.01.01_60/gs_CIM009v010101p.pdf)
<br/> [![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

This tutorial discusses the usage of subscriptions and registrations within NGSI-LD and highlights the similarities and
differences between the equivalent NGSI-v2 and NGSI-LD operations. The tutorial is an analogue of the original
context-provider and subscriptions tutorials but uses API calls from the **NGSI-LD** interface throughout.

The tutorial uses [cUrl](https://ec.haxx.se/) commands throughout, but is also available as
[Postman documentation](https://fiware.github.io/tutorials.LD-Subscriptions-Registrations/)

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/7ae2d2d3f42bbdf59c45)

## Contents

<details>
<summary><strong>Details</strong></summary>

-   [Understanding Linked Data Subscriptions and Registrations](#understanding-linked-data-subscriptions-and-registrations)
    -   [Entities within a stock management system](#entities-within-a-stock-management-system)
    -   [Stock Management frontend](#stock-management-frontend)
-   [Prerequisites](#prerequisites)
    -   [Docker](#docker)
    -   [Cygwin](#cygwin)
-   [Architecture](#architecture)
-   [Start Up](#start-up)
-   [Interactions between Components](#interactions-between-components)
    -   [Using Subscriptions with NGSI-LD](#using-subscriptions-with-ngsi-ld)
        -   [Create a Subscription (Store 1) - Low Stock](#create-a-subscription-store-1---low-stock)
        -   [Create a Subscription (Store 2) - Low Stock](#create-a-subscription-store-2---low-stock)
        -   [Read Subscription Details](#read-subscription-details)
    -   [Using Registrations with NGSI-LD](#using-registrations-with-ngsi-ld) +
        [Create a Registration](#create-a-registration) + [Read Registration Details](#read-registration-details) +
        [Read from Store 1](#read-from-store-1) +
        [Read direct from the Context Provider](#read-direct-from-the-context-provider) +
        [Direct update of the Context Provider](#direct-update-of-the-context-provider) +
        [Forwarded Update](#forwarded-update)
        </details>

# Understanding Linked Data Subscriptions and Registrations

> “Do not repeat after me words that you do not understand. Do not merely put on a mask of my ideas, for it will be an
> illusion and you will thereby deceive yourself.”
>
> ― Jiddu Krishnamurti

NGSI-LD Subscriptions and Registrations provide the basic mechanism to allow the components within a Smart Linked Data
Solution to interact with each other.

As a brief reminder, within a distributed system, subscriptions inform a third party component that a change in the
context data has occurred (and the component needs to take further actions), whereas registrations tell the context
broker that additional context information is available from another source.

Both of these operations require that the receiving component fully understands the requests it receives, and is capable of
creating and interpreting the resultant payloads. The differences here between NGSI-v2 and NGSI-LD operations is small,
but there has been a minor amendment to facilite the incorporation of linked data concepts, and therefore the contract
between the various components has changed.

## Entities within a stock management system

The relationship between our entities is defined as shown:

![](https://fiware.github.io/tutorials.LD-Subscriptions-Registrations/img/entities.png)

## Stock Management frontend

In the previous [tutorial](https://github.com/FIWARE/tutorials.Working-with-Linked-Data/), the simple Node.js Express
application was updated to use NGSI-LD. This tutorial will use the monitor page to watch the status of recent requests,
and a store page to buy products. Once the services are running these pages can be accessed from the following URLs:

#### Event Monitor

The event monitor can be found at: `http://localhost:3000/app/monitor`

![FIWARE Monitor](https://fiware.github.io/tutorials.LD-Subscriptions-Registrations/img/monitor.png)

#### Store 001

Store001 can be found at: `http://localhost:3000/app/store/urn:ngsi-ld:Building:store001`

![Store](https://fiware.github.io/tutorials.LD-Subscriptions-Registrations/img/store1.png)

# Prerequisites

## Docker

To keep things simple all components will be run using [Docker](https://www.docker.com). **Docker** is a container
technology which allows to different components isolated into their respective environments.

-   To install Docker on Windows follow the instructions [here](https://docs.docker.com/docker-for-windows/)
-   To install Docker on Mac follow the instructions [here](https://docs.docker.com/docker-for-mac/)
-   To install Docker on Linux follow the instructions [here](https://docs.docker.com/install/)

**Docker Compose** is a tool for defining and running multi-container Docker applications. A
[YAML file](https://raw.githubusercontent.com/fiware/tutorials.Relationships-Linked-Data/master/docker-compose.yml) is
used configure the required services for the application. This means all container services can be brought up in a
single command. Docker Compose is installed by default as part of Docker for Windows and Docker for Mac, however Linux
users will need to follow the instructions found [here](https://docs.docker.com/compose/install/)

## Cygwin

We will start up our services using a simple bash script. Windows users should download [cygwin](http://www.cygwin.com/)
to provide a command-line functionality similar to a Linux distribution on Windows.

# Architecture

The demo Supermarket application will send and receive NGSI-LD calls to a compliant context broker. Since the NGSI-LD
interface is available on an experimental version of the
[Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/), the demo application will only make use of one
FIWARE component.

Currently, the Orion Context Broker relies on open source [MongoDB](https://www.mongodb.com/) technology to keep
persistence of the context data it holds. To request context data from external sources, a simple Context Provider NGSI
proxy has also been added. To visualize and interact with the Context we will add a simple Express application

Therefore, the architecture will consist of four elements:

-   The [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) which will receive requests using
    [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/gitlab/NGSI-LD/NGSI-LD/raw/master/spec/updated/full_api.json)
-   The underlying [MongoDB](https://www.mongodb.com/) database :
    -   Used by the Orion Context Broker to hold context data information such as data entities, subscriptions and
        registrations
-   The **Context Provider NGSI** proxy which will:
    -   receive requests using
        [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/gitlab/NGSI-LD/NGSI-LD/raw/master/spec/updated/full_api.json#/)
    -   makes requests to publicly available data sources using their own APIs in a proprietary format
    -   returns context data back to the Orion Context Broker in
        [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/gitlab/NGSI-LD/NGSI-LD/raw/master/spec/updated/full_api.json#/)
        format.
-   The **Stock Management Frontend** which will:
    -   Display store information
    -   Show which products can be bought at each store
    -   Allow users to "buy" products and reduce the stock count.

Since all interactions between the elements are initiated by HTTP requests, the entities can be containerized and run
from exposed ports.

![](https://fiware.github.io/tutorials.LD-Subscriptions-Registrations/img/architecture.png)

The necessary configuration information can be seen in the services section of the associated `docker-compose.yml` file.
It has been described in a [previous tutorial](https://github.com/FIWARE/tutorials.Working-with-Linked-Data/)

# Start Up

All services can be initialised from the command-line by running the
[services](https://github.com/FIWARE/tutorials.LD-Subscriptions-Registrations/blob/master/services) Bash script provided
within the repository. Please clone the repository and create the necessary images by running the commands as shown:

```bash
git clone https://github.com/FIWARE/tutorials.LD-Subscriptions-Registrations.git
cd tutorials.LD-Subscriptions-Registrations

./services orion
```

> **Note:** If you want to clean up and start over again you can do so with the following command:
>
> ```
> ./services stop
> ```

---

# Interactions between Components

## Using Subscriptions with NGSI-LD

Goto `http://localhost:3000/app/store/urn:ngsi-ld:Building:store001` to display and interact with the Supermarket data.

### Create a Subscription (Store 1) - Low Stock

#### :one: Request:

```console
curl -L -X POST 'http://localhost:1026/ngsi-ld/v1/subscriptions/' \
-H 'Content-Type: application/ld+json' \
--data-raw '{
  "description": "Notify me of low stock in Store 001",
  "type": "Subscription",
  "entities": [{"type": "Shelf"}],
  "watchedAttributes": ["numberOfItems"],
  "q": "numberOfItems<10;locatedIn==urn:ngsi-ld:Building:store001",
  "notification": {
    "attributes": ["numberOfItems", "stocks", "locatedIn"],
    "format": "keyValues",
    "endpoint": {
      "uri": "http://tutorial:3000/subscription/low-stock-store001",
      "accept": "application/json"
    }
  },
   "@context": "https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld"
}'
```

### Create a Subscription (Store 2) - Low Stock

#### :two: Request:

```console
curl -L -X POST 'http://localhost:1026/ngsi-ld/v1/subscriptions/' \
-H 'Content-Type: application/json' \
--data-raw '{
  "description": "Notify me of low stock in Store 002",
  "type": "Subscription",
  "entities": [{"type": "Shelf"}],
  "watchedAttributes": ["numberOfItems"],
  "q": "numberOfItems<10;locatedIn==urn:ngsi-ld:Building:store002",
  "notification": {
    "attributes": ["numberOfItems", "stocks", "locatedIn"],
    "format": "keyValues",
    "endpoint": {
      "uri": "http://tutorial:3000/subscription/low-stock-store002",
      "accept": "application/json"
    }
  },
   "@context": "https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld"
}'
```

### Read Subscription Details

#### :three: Request:

```console
curl -L -X GET 'http://localhost:1026/ngsi-ld/v1/subscriptions/' \
-H 'Content-Type: application/json'
```

#### Response:

```jsonld
[
    {
        "id": "urn:ngsi-ld:Subscription:5e62405ee232da3a07b5fa7f",
        "type": "Subscription",
        "description": "Notify me of low stock in Store 001",
        "entities": [
            {
                "type": "Shelf"
            }
        ],
        "watchedAttributes": [
            "numberOfItems"
        ],
        "q": "https://fiware.github.io/tutorials.Step-by-Step/schema/numberOfItems<10;https://fiware.github.io/tutorials.Step-by-Step/schema/locatedIn==urn:ngsi-ld:Building:store001",
        "notification": {
            "attributes": [
                "numberOfItems",
                "stocks",
                "locatedIn"
            ],
            "format": "keyValues",
            "endpoint": {
                "uri": "http://tutorial:3000/subscription/low-stock-store001",
                "accept": "application/json"
            }
        },
        "@context": "https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld"
    },
    {
        "id": "urn:ngsi-ld:Subscription:5e624063e232da3a07b5fa80",
        "type": "Subscription",
        "description": "Notify me of low stock in Store 002",
        "entities": [
            {
                "type": "Shelf"
            }
        ],
        "watchedAttributes": [
            "numberOfItems"
        ],
        "q": "https://fiware.github.io/tutorials.Step-by-Step/schema/numberOfItems<10;https://fiware.github.io/tutorials.Step-by-Step/schema/locatedIn==urn:ngsi-ld:Building:store002",
        "notification": {
            "attributes": [
                "numberOfItems",
                "stocks",
                "locatedIn"
            ],
            "format": "keyValues",
            "endpoint": {
                "uri": "http://tutorial:3000/subscription/low-stock-store002",
                "accept": "application/json"
            }
        },
        "@context": "https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld"
    }
]
```

## Using Registrations with NGSI-LD

### Create a Registration

#### :four: Request:

```console
curl -L -X POST 'http://localhost:1026/ngsi-ld/v1/csourceRegistrations/' \
-H 'Content-Type: application/json' \
-H 'Link: <https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
--data-raw ' {
    "type": "ContextSourceRegistration",
    "information": [
        {
            "entities": [
                {
                    "type": "Building",
                    "id": "urn:ngsi-ld:Building:store001"
                }
            ],
            "properties": [
                "tweets"
            ]
        }
    ],
    "endpoint": "http://context-provider:3000/static/tweets"
}'
```

### Read Registration Details

#### :five: Request:

```console
curl -L -X GET 'http://localhost:1026/ngsi-ld/v1/csourceRegistrations/' \
-H 'Accept: application/ld+json' \
-H 'Link: <https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
```

#### Response:

```jsonld
[
    {
        "@context": "https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld",
        "id": "urn:ngsi-ld:ContextSourceRegistration:5e6242179c26be5aef9991d4",
        "type": "ContextSourceRegistration",
        "endpoint": "http://context-provider:3000/static/tweets",
        "information": [
            {
                "entities": [
                    {
                        "id": "urn:ngsi-ld:Building:store001",
                        "type": "Building"
                    }
                ],
                "properties": [
                    "tweets"
                ]
            }
        ]
    }
]
```

### Read from Store 1

#### :six: Request:

```console
curl -L -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:store001' \
-H 'Link: <https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json'
```

#### Response:

```jsonld
{
    "@context": "https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld",
    "id": "urn:ngsi-ld:Building:store001",
    "type": "Building",
    "furniture": {
        "type": "Relationship",
        "object": [
            "urn:ngsi-ld:Shelf:unit001",
            "urn:ngsi-ld:Shelf:unit002",
            "urn:ngsi-ld:Shelf:unit003"
        ]
    },
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        },
        "verified": {
            "type": "Property",
            "value": true
        }
    },
    "name": {
        "type": "Property",
        "value": "Bösebrücke Einkauf"
    },
    "category": {
        "type": "Property",
        "value": "commercial"
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [
                13.3986,
                52.5547
            ]
        }
    },
    "tweets": {
        "type": "Property",
        "value": [
            "It has great practical value – you can wrap it around you for warmth as you bound across the cold moons of Jaglan Beta;",
            "You can lie on it on the brilliant marble-sanded beaches of Santraginus V, inhaling the heady sea vapours;",
            "You can sleep under it beneath the stars which shine so redly on the desert world of Kakrafoon;",
            "Use it to sail a mini raft down the slow heavy river Moth;",
            "Wet it for use in hand-to-hand-combat;",
            "Wrap it round your head to ward off noxious fumes or to avoid the gaze of the Ravenous Bugblatter Beast of Traal  (a mindboggingly stupid animal, it assumes that if you can’t see it, it can’t see you – daft as a bush, but very, very ravenous);",
            "You can wave your towel in emergencies as a distress signal, and of course dry yourself off with it if it still seems to be clean enough."
        ]
    }
}
```


### Read direct from the Context Provider

#### :seven: Request:

```console
curl -L -X GET 'http://localhost:3000/static/tweets/ngsi-ld/v1/entities/urn:ngsi-ld:Building:store001' \
-H 'Link: <https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/ld+json'
```

#### Response:

```jsonld
{
    "@context": "https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld",
    "id": "urn:ngsi-ld:Building:store001",
    "type": "Building",
    "tweets": {
        "type": "Property",
        "value": [
            "It has great practical value – you can wrap it around you for warmth as you bound across the cold moons of Jaglan Beta;",
            "You can lie on it on the brilliant marble-sanded beaches of Santraginus V, inhaling the heady sea vapours;",
            "You can sleep under it beneath the stars which shine so redly on the desert world of Kakrafoon;",
            "Use it to sail a mini raft down the slow heavy river Moth;",
            "Wet it for use in hand-to-hand-combat;",
            "Wrap it round your head to ward off noxious fumes or to avoid the gaze of the Ravenous Bugblatter Beast of Traal  (a mindboggingly stupid animal, it assumes that if you can’t see it, it can’t see you – daft as a bush, but very, very ravenous);",
            "You can wave your towel in emergencies as a distress signal, and of course dry yourself off with it if it still seems to be clean enough."
        ]
    }
}
```

### Direct update of the Context Provider

#### :eight: Request:

```console
curl -L -X PATCH 'http://localhost:3000/static/tweets/ngsi-ld/v1/entities/urn:ngsi-ld:Building:store001/attrs' \
-H 'Link: <https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
--data-raw '{
  "tweets": {
    "type": "Property",
    "value": [
      "Space is big.",
      "You just won'\''t believe how vastly, hugely, mind-bogglingly big it is.",
      "I mean, you may think it'\''s a long way down the road to the chemist'\''s, but that'\''s just peanuts to space."
    ]
  }
}'
```


#### :nine: Request:

```console
curl -L -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:store001?attrs=tweets&options=keyValues' \
-H 'Link: <https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
```

#### Response:

```jsonld
{
    "@context": "https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld",
    "id": "urn:ngsi-ld:Building:store001",
    "type": "Building",
    "tweets": [
        "Space is big.",
        "You just won't believe how vastly, hugely, mind-bogglingly big it is.",
        "I mean, you may think it's a long way down the road to the chemist's, but that's just peanuts to space."
    ]
}
```

### Forwarded Update

#### :one::zero: Request:

```console
curl -L -X PATCH 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:store001/attrs/tweets' \
-H 'Link: <https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
--data-raw '{
  "type": "Property",
  "value": [
    "This must be Thursday",
    "I never could get the hang of Thursdays."
  ]
} '
```


#### :one::one: Request:

```console
curl -L -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:store001?attrs=tweets&options=keyValues' \
-H 'Link: <https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json'
```


#### Response:

```jsonld
{
    "@context": "https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld",
    "id": "urn:ngsi-ld:Building:store001",
    "type": "Building",
    "tweets": [
        "This must be Thursday",
        "I never could get the hang of Thursdays."
    ]
}
```

---

## License

[MIT](LICENSE) © 2020 FIWARE Foundation e.V.
