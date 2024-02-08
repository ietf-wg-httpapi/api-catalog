---
title: "api-catalog: a well-known URI to help discovery of APIs"
abbrev: api-catalog well-known URI
docname: draft-ietf-httpapi-api-catalog-latest
date: {DATE}
area: IETF
category: std

ipr: trust200902
keyword: internet-draft
pi: [toc, tocindent, sortrefs, symrefs, strict, compact, comments, inline]

venue:
  group: "Building Blocks for HTTP APIs"
  type: "Working Group"
  mail: "httpapi@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/httpapi/"
  github: "ietf-wg-httpapi/api-catalog"
  latest: "https://ietf-wg-httpapi.github.io/api-catalog/draft-ietf-httpapi-api-catalog.html"

author:
 -
    ins: K. Smith
    name: Kevin Smith
    organization: Vodafone
    email: kevin.smith@vodafone.com
    uri: https://www.vodafone.com

normative:


informative:
  OAS:
    title: OpenAPI Specification 3.1.0
    date: 2021-02-15
    target: https://spec.openapis.org/oas/latest
    author:
    - ins: Darrel Miller
    - ins: Jeremy Whitlock
    - ins: Marsh Gardiner
    - ins: Mike Ralphson
    - ins: Ron Ratovsky
    - ins: Uri Sarid

  APIsjson:
    title: APIs.json
    date: 2020-09-15
    target: http://apisjson.org/format/apisjson_0.16.txt
    author:
    - ins: Kin Lane
    - ins: Steve Willmott

  HAL:
    title: JSON Hypertext Application Language
    date: 2020-09-15
    target: https://datatracker.ietf.org/doc/html/draft-kelly-json-hal-11
    author:
    - ins: Mike Kelly
  
  RESTdesc:
    title: RESTdesc
    date: 2023-09-15
    target: http://apisjson.org/format/apisjson_0.16.txt
    author:
    - ins: Ruben Verborgh
    - ins: Erik Mannens
    - ins: Rick Van de Walle
    - ins: Thomas Steiner
    
  WebAPIext:
    title: WebAPI type extension
    date: 2020-07-08
    target: https://webapi-discovery.github.io/rfcs/rfc0001.html
    author:
    - ins: Mike Ralphson
    - ins: Nick Evans

--- abstract

This document defines the "api-catalog" well-known URI. It is intended to facilitate automated discovery and usage of the APIs published by a given organisation or individual.


--- middle

# Introduction {#introduction}

An organisation or individual may publish Application Programming Interfaces (APIs) to encourage requests for interaction from external parties. Such APIs must be discovered before they may be used - i.e., the external party needs to know what APIs a given publisher exposes, their purpose, any policies for usage, and the endpoint to interact with each APIs. To facilitate automated discovery of this information, and automated usage of the APIs, this document proposes a well-known URI, 'api-catalog', as a location where a Publisher's API endpoints are described in an API catalog document.

## Terminology {#terminology}

* 'Publisher' - an organisation, company or individual that publishes one or more APIs for usage by external third parties.
                    
## Goals and non-goals {#goals}

The primary goal is to facilitate the automated discovery of a Publisher's public API endpoints, along with metadata that describes the purpose and usage of each API, by specifying a well-known URI {{!RFC8615}} that returns an API catalog document. The API catalog document is primarily machine-readable to enable automated discovery and usage of APIs, and it may also include links to human-readable documentation.

Non-goals: this document does not mandate paths for API endpoints. i.e., it does not mandate that my_example_api's endpoint should be example.com/.well-known/api-catalog/my_example_api , nor even to be hosted at example.com (although it is not forbidden to do so). This document does not mandate a specific format for the API catalog document, although it does suggest some existing formats and provide a recommendation.

## Requirements Language {#requirements}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED",
"MAY", and "OPTIONAL" in this document are to be interpreted as
described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they
appear in all capitals, as shown here.

# Using the 'api-catalog' well-known URI {#usage}
The api-catalog well-known URI is intended for HTTP(S) servers that publish APIs. As the key aim is  to facilitate their discovery and usage, a Publisher supporting this URI:

* SHOULD host the /.well-known/api-catalog URI at a predictable location. For example as companies typically own a .com TLD, a predictable location for the company 'example' would be   https://www.example.com/.well-known/api-catalog
* SHALL resolve an HTTP(S) GET request to /.well-known/api-catalog and return an API catalog document, as described in {{API-CATALOG}}
* SHOULD resolve an HTTP(S) HEAD request to /.well-known/api-catalog with a response including a Link header with the relation(s) defined in {{LINK-RELATION}}

# Accounting for APIs distributed across multiple domains {#multiple_domains}

A Publisher ('example') may have their APIs hosted across multiple domains that they manage: e.g., at example.com, developer.example.com, apis.example.com, apis.example.net etc. They may also use a third party API hosting provider which hosts APIs on a distinct domain.
                
To account for this scenario, it is recommended that:

* the Publisher publish the api-catalog well-known URI at a predictable location, as described above.
* the Publisher also publish the api-catalog well-known URI at each of their API domains e.g. apis.example.com/.well-known/api-catalog, developer.example.com/.well-known/api-catalog etc. 
* only one of these locations needs to host the canonical API Catalog document. The other locations SHOULD redirect any requests they receive for /.well-known/api-catalog to that primary location, using HTTP Status Code 308 Permanent Redirect {{!RFC9110}}.

As illustration, if the Publisher's primary API portal is apis.example.com, then apis.example.com/.well-known/api-catalog should be the location to host the API Catalog document. If the Publisher is also the domain authority for example.net, which also hosts a selection of their APIs, then a request to www.example.net/.well-known/api-catalog SHOULD return a redirect as follows.

Clienr request:

~~~ http-message
GET /.well-known/api-catalog HTTP/1.1
User-Agent: curl/7.16.3 libcurl/7.16.3 OpenSSL/0.9.7l zlib/1.2.3
Host: www.example.net
~~~  

Server response:

~~~ http-message
HTTP/1.1 308 Permanent Redirect
Content-Type: text/html; charset=UTF-8
Location: http\://apis.example.com/.well-known/api-catalog
Content-Length: 356

<!DOCTYPE HTML>
  <html>
    <head>
      <title>Permanent Redirect</title>
      <meta http-equiv="refresh" content="0; url=https://apis.example.com/.well-known/api-catalog">
    </head>
    <body>
      <p>The document has been moved to <a href=https://apis.example.com/.well-known/api-catalog>https://apis.example.com/.well-known/api-catalog</a>.</p>
    </body>
  </html>

~~~




# Internal use of api-catalog for private APIs {#INTERNAL-USE}

A Publisher may wish to use the api-catalog well-known URI on their internal network, to signpost authorised users (e.g. company employees) towards internal/private APIs not intended for third-party use. This scenario may incur additional security considerations, as noted in {{security}}

# The API Catalog {#API-CATALOG}    

The API Catalog is a document listing hyperlinks to a Publisher's APIs.
            
There is no mandated format for the API Catalog document: the Publisher is free to choose any format that supports the automated discovery, and machine (and human) usage of their APIs. However, it is RECOMMENDED to use a linkset {{!RFC9264}} of API endpoints (see {{api-catalog-example-linkset}} for an example).
            
The API Catalog document MUST include hyperlinks to API endpoints, and is RECOMMENDED to include useful metadata, such as usage policies, API version information, links to the OpenAPI Specification [OAS] definitions for each API, etc. . If the Publisher does not include these metadata directly in the API Catalog document, they SHOULD make that metadata available at the API endpoint URIs they have listed (see {{api-catalog-example-linkset-bookmarks}} for an example).
            
Some suitable API Catalog document formats include: 

* (RECOMMENDED) A linkset {{!RFC9264}} of API endpoints and information to facilitate API usage 
* An APIs.json document [APIsjson]
* API bookmarks that represent an API entry-point URI, which may be followed to discover purpose and usage
* A RESTDesc semantic description for hypermedia APIs [RESTdesc]
* A Hypertext Application Language document [HAL]
* An extension to the Schema.org WebAPI type [WebAPIext]
            
Appendix A includes example API Catalog documents based on the linkset format.   


# Link relations {#LINK-RELATION}

* "item" {{!RFC9264}}. When used in an API Catalog document, the 'item' link relation identifies a target resource that represents a API that is a member of the API Catalog.
* "api-catalog": the 'api-catalog' link relation identifies a target resource that represents a list of APIs of which the context resource is a member.     

# Conformance to RFC8615  {#CONFORM-RFC8615}

The requirements in section 3 of {{!RFC8615}} for defining Well-Known Uniform Resource Identifiers are met as follows:

## Path prefix

The api-catalog URI SHALL be appended to the /.well-known/ path-prefix for "well-known locations".

## Supported URI schemes

The api-catalog well-known URI may be used with the HTTP and HTTPS URI schemes.

## Registration of the api-catalog well-known URI

See {{IANA}} considerations below.

# IANA Considerations {#IANA}

## The api-catalog well-known URI

This specification registers the "api-catalog" well-known URI in the Well-Known URI Registry as defined by {{!RFC6415}} .

URI suffix: api-catalog

Specification document(s):  draft-ietf-httpapi-api-catalog-01

Related information:  The "api-catalog" documents obtained from the same host using the HTTP and HTTPS protocols (using default ports) MUST be identical.

## The api-catalog link relation

This specification registers the "api-catalog" link relation by following the procedures per section 4.2.2 of {{!RFC8288}}

* Relation Name:  api-catalog
* Description:  Identifies a catalog of APIs published by the context Publisher.
* Reference:  draft-ietf-httpapi-api-catalog-01

# Security Considerations {#security}

For all scenarios: the Publisher SHOULD perform a security and privacy review of the API Catalog prior to deployment, to ensure it does not leak personal, business or other metadata, nor expose any vulnerability related to the APIs listed.         

For the internal/private APIs scenario: the Publisher SHOULD take steps to ensure that appropriate access controls are in place to ensure only authorised users access the internal api-catalog well-known URI. 

--- back

# Example API Catalog document {#api-catalog-example-linkset}

This section is informative, and provides and example of an API Catalog document using the RECOMMENDED linkset format.

## Using Linkset with RFC8615 relations

This example uses the linkset format {{!RFC9264}}, and the following link relations defined in {{?RFC8631}}:

* "service-desc", used to link to a description of the API that is primarily intended for machine consumption.
* "service-doc", used to link to API documentation that is primarily intended for human consumption.
* "service-meta", used to link to additional metadata about the API, and is primarily intended for machine consumption.
* "status", used to link to the API status (e.g. API "health" indication etc.) for machine and/or human consumption.

Client request:

~~~ http-message
GET .well-know/api-catalog HTTP/1.1
Host: example.com
Accept: application/linkset+json
~~~

Server response:

~~~ http-message
HTTP/1.1 200 OK
Date: Mon, 01 Jun 2023 00:00:01 GMT
Server: Apache-Coyote/1.1
Content-Type: application/linkset+json
~~~

~~~
{
  "linkset": [
  {
    "anchor": "https://developer.example.com/apis/foo_api",
    "service-desc": [
      {
        "href": "https://developer.example.com/apis/foo_api/spec",
        "type": "application/yaml"
      }
    ],
    "status": [
      {
        "href": "https://developer.example.com/apis/foo_api/status",
        "type": "application/json"
      }
    ],
    "service-doc": [
      {
        "href": "https://developer.example.com/apis/foo_api/doc",
        "type": "text/html"
      }
    ],
    "service-meta": [
      {
        "href": "https://developer.example.com/apis/foo_api/policies",
        "type": "text/xml"
      }
    ]
  },
  {
    "anchor": "https://developer.example.com/apis/bar_api",
    "service-desc": [
      {
        "href": "https://developer.example.com/apis/bar_api/spec",
        "type": "application/yaml"
      }
    ],
    "status": [
      {
        "href": "https://developer.example.com/apis/bar_api/status",
       "type": "application/json"
      }
    ],
    "service-doc": [
      {
        "href": "https://developer.example.com/apis/bar_api/doc",
        "type": "text/plain"
      }
    ]
  },
  {
    "anchor": "https://apis.example.net/apis/cantona_api",
    "service-desc": [
      {
        "href": "https://apis.example.net/apis/cantona_api/spec",
        "type": "text/n3"
      }
    ],
    "service-doc": [
      {
        "href": "https://apis.example.net/apis/cantona_api/doc",
        "type": "text/html"
      }
    ]
  }
  ]
}
~~~ 

## Using Linkset with bookmarks {#api-catalog-example-linkset-bookmarks}

This example also uses the linkset format {{!RFC9264}}, listing the API endpoints in an array of bookmarks. Each link shares the same context (the API Catalog) and "item" {{!RFC9264}} link relation (to indicate they are an item in the catalog).The intent is that by following a bookmark link, a machine-client can discover the purpose and usage of each API, hence the document targeted by the bookmark link should support this.

Client request:

~~~ http-message
GET .well-know/api-catalog HTTP/1.1
Host: example.com
Accept: application/linkset+json
~~~

Server response:

~~~ http-message
HTTP/1.1 200 OK
Date: Mon, 01 Jun 2023 00:00:01 GMT
Server: Apache-Coyote/1.1
Content-Type: application/linkset+json
~~~

~~~
[
  { "anchor": "https://example.com/.well-known/api-catalog",
    "item": [
      {"href": "https://developer.example.com/apis/foo_api"},
      {"href": "https://developer.example.com/apis/bar_api"}
      {"href": "https://developer.example.com/apis/cantona_api"}
    ]
  }
]
~~~

# Acknowledgements

Thanks to Phil Archer, Ben Bucksch, Sanjay Dalal, Max Maton, Darrel Miller, Mark Nottingham,  Roberto Polli, Rich Salz, Herbert Van De Sompel and Erik Wilde for their suggestions and feedback.
