---
title: "api-catalog: a well-known URI and link relation to help discovery of APIs"
abbrev: api-catalog well-known URI
docname: draft-ietf-httpapi-api-catalog-06
date: {DATE}
area: IETF
category: std

ipr: trust200902
keyword: internet-draft

stand_alone: yes
smart_quotes: no
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
  HTTP: RFC9110
  WELL-KNOWN: RFC8615
  WEB-LINKING: RFC8288

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

This document defines the "api-catalog" well-known URI and link relation. It is intended to facilitate automated discovery and usage of the APIs published by a given organisation or individual. A request to the api-catalog resource will return a document providing information about, and links to, the publisher's APIs.


--- middle

# Introduction {#introduction}

An organisation or individual may publish Application Programming Interfaces (APIs) to encourage requests for interaction from external parties. Such APIs must be discovered before they may be used - i.e., the external party needs to know what APIs a given publisher exposes, their purpose, any policies for usage, and the endpoint to interact with each API. To facilitate automated discovery of this information, and automated usage of the APIs, this document proposes:

- a well-known URI {{WELL-KNOWN}}, 'api-catalog', encoded as a URI reference for an API Catalog document describing a Publisher's API endpoints.
- a link relation {{WEB-LINKING}}, 'api-catalog', of which the target resource is the Publisher's API Catalog document.
                    
## Goals and non-goals {#goals}

The primary goal is to facilitate the automated discovery of a Publisher's public API endpoints, along with metadata that describes the purpose and usage of each API, by specifying a well-known URI that returns an API catalog document. The API catalog document is primarily machine-readable to enable automated discovery and usage of APIs, and it may also include links to human-readable documentation.

Non-goals: this document does not mandate paths for API endpoints. i.e., it does not mandate that my_example_api's endpoint should be https://www.example.com/.well-known/api-catalog/my_example_api , nor even to be hosted at www.example.com (although it is not forbidden to do so).

## Notational Conventions

{::boilerplate bcp14+}

The term "content negotiation" and "status code" are from {{HTTP}}.
The term "well-known URI" is from {{WELL-KNOWN}}.
The term "link relation" is from {{WEB-LINKING}}.

The term "Publisher" refers to an organisation, company or individual that publishes one or more APIs for usage by external third parties.

In this document, "API" means the specification resources required for an external party (or in the case of 'private' APIs, an internal party) to implement software which uses the Publisher's Application Programming Interface.  

# Using the 'api-catalog' well-known URI {#usage}

The api-catalog well-known URI is intended for HTTP(S) servers that publish APIs. 

* The API Catalog MUST be named "api-catalog" in a well-known location as described by {{WELL-KNOWN}}.
* The location of the API Catalog document is decided by the Publisher: the /.well-known/api-catalog URI provides a convenient reference to that location. 

A Publisher supporting this URI:
* SHALL resolve an HTTP(S) GET request to /.well-known/api-catalog and return an API catalog document ( as described in {{API-CATALOG}} ). 
* SHOULD resolve an HTTP(S) HEAD request to /.well-known/api-catalog with a response including a Link header with the relation(s) defined in {{LINK-RELATION}}


# Link relations {#LINK-RELATION}

The following link relations {{WEB-LINKING}}} are of most relevance to this document: 

* "api-catalog": the 'api-catalog' link relation identifies a target resource that represents a list of APIs available from the Publisher of the context resource. The target resource URI may be ./well-known/api-catalog , or any other URI chosen by the Publisher. For example, the Publisher 'example' could include the api-catalog link relation in the HTTP header and/or content payload when responding to a request to https://www.example.com : 

~~~ http-message
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Location: /index.html
Link: </my_api_catalog.json>; rel=api-catalog
Content-Length: 356

<!DOCTYPE HTML>
  <html>
    <head>
      <title>Welcome to Example Publisher</title>
    </head>
    <body>
      <p><a href="my_api_catalog.json" rel="api-catalog">Example Publisher's APIs</a>.</p>
      <p>(remainder of content)</p>
    </body>
  </html>

~~~

* "item" {{!RFC6573}}. When used in an API Catalog document, the 'item' link relation identifies a target resource that represents an API that is a member of the API Catalog.

Other link relations may be utilised in an API Catalog to convey metadata descriptions for API links. 

# The API Catalog document {#API-CATALOG}    

The API Catalog is a document listing hyperlinks to a Publisher's APIs. The Publisher may host this API Catalog document at any URI(s) they choose. As illustration, the API Catalog document URI of https://www.example.com/my_api_catalog.json can be requested directly, or via a request to https://www.example.com/.well-known/api-catalog, which the Publisher will resolve to https://www.example.com/my_api_catalog.
            
The Publisher MUST publish the API Catalog document in the Linkset format application/linkset+json (section 4.2 of {{!RFC9264}}). In addition, the Publisher MAY make additional formats available via content negotiation (section 5.3 of {{!HTTP}}) to their /.well-known/api-catalog location. A non-exhaustive list of such formats that support the automated discovery, and machine (and human) usage of a Publisher's APIs, is listed below.

The API Catalog document MUST include hyperlinks to API endpoints, and is RECOMMENDED to include useful metadata, such as usage policies, API version information, links to the OpenAPI Specification [OAS] definitions for each API, etc..  If the Publisher does not include these metadata directly in the API Catalog document, they SHOULD make that metadata available at the API endpoint URIs they have listed (see {{api-catalog-example-linkset-bookmarks}} for an example).
            
Some suitable API Catalog document formats include: 

* a linkset in JSON Document format (section 4.2 of {{!RFC9264}}) of API endpoints and information to facilitate API usage. The linkset SHOULD include a profile parameter (section 5 of {{!RFC9264}}) with a Profile URI {{!RFC7284}} value of 'THIS-RFC-URL' to indicate the linkset is representing an API Catalog document as defined above. Appendix A includes example API Catalog documents based on the linkset format. 
* An APIs.json document [APIsjson].
* API bookmarks that represent an API entry-point URI, which may be followed to discover purpose and usage.
* A RESTDesc semantic description for hypermedia APIs [RESTdesc].
* A Hypertext Application Language document [HAL].
* An extension to the Schema.org WebAPI type [WebAPIext].

If a Publisher already lists their APIs in a format other than linkset but wish to utilise the /.well-known/api-catalog URI, then:

* they MUST also implement a linkset with, at minimum, hyperlinks to API endpoints - see the example of  {{api-catalog-example-linkset-bookmarks}} in Appendix A.
* they MAY support content negotiation at the /.well-known/api-catalog URI to allow their existing format to be returned.

## Nesting API Catalog links {nest}

An API Catalog may itself contain links to other API Catalogs, by using the 'api-catalog' relation type for each link.
An example of {{#api-catalog-example-linkset-nesting}} is given in Appendix A.

# Operational considerations {#OPERATIONS}

## Accounting for APIs distributed across multiple domains {#multiple_domains}

A Publisher ('example') may have their APIs hosted across multiple domains that they manage: e.g., at https://www.example.com, https://developer.example.com, https://apis.example.com, https://apis.example.net etc. They may also use a third-party API hosting provider which hosts APIs on a distinct domain.
                
To account for this scenario, it is RECOMMENDED that:

* the Publisher also publish the api-catalog well-known URI at each of their API domains e.g. https://apis.example.com/.well-known/api-catalog, https://developer.example.net/.well-known/api-catalog etc. 
* an HTTP GET request to any of these URIs returns the same result, namely, the API Catalog document. 
* since the physical location of the API Catalog document is decided by the Publisher, and may change, the Publisher choose one of their instances of /.well-known/api-catalog as a canonical reference to the location of the latest API Catalog. The Publisher's other instances of ./well-known/api-catalog SHOULD redirect to this canonical instance of /.well-known/api-catalog to ensure the latest API Catalog is returned.

For example, if the Publisher's primary API portal is https://apis.example.com, then https://apis.example.com/.well-known/api-catalog SHOULD resolve to the location of the Publisher's latest API Catalog document. If the Publisher is also the domain authority for https://www.example.net, which also hosts a selection of their APIs, then a request to www.example.net/.well-known/api-catalog SHOULD redirect to https://apis.example.com/.well-known/api-catalog .

## Internal use of api-catalog for private APIs {#INTERNAL-USE}

A Publisher may wish to use the api-catalog well-known URI on their internal network, to signpost authorised users (e.g. company employees) towards internal/private APIs not intended for third-party use. This scenario may incur additional security considerations, as noted in {{security}}.

## Scalability guidelines {#SCALABILITY}

In cases where a Publisher has a large number of APIs, potentially deployed across multiple domains, then two challenges may arise: 
* maintaining the catalog entries to ensure they are up to date and any errors corrected
* restricting the catalog size to help reduce network and client-processing overheads.

In both cases a Publisher may benefit from grouping their APIs, providing an API Catalog document for each group - and use the main API Catalog hosted at /.well-known/api-catalog to provide links to these. A Publisher may decide to group their APIs according to a business domain (e.g. 'gaming APIs', 'anti-fraud APIs' etc.) or a technology domain (e.g. ''IOT', 'networks', 'AI' etc.), or any other criterion. This grouping may already be implicit where the Publisher has split APIs across multiple domains, e.g. https://gaming.example.com, https://iot.example.com. 

The {{nest}} section below shows how the API Catalog at /.well-known/api-catalog can use the api-catalog link relation to point to other API Catalogs.

In addition, the Publisher should consider caching and compression techniques to reduce the network overhead of large API Catalogs.

## Monitoring and maintenance {#MAINTENANCE}

Publishers are RECOMMENDED to follow operational best practice when hosting API Catalog(s), including but not limited to:
* health. The Publisher SHOULD monitor availability of the API Catalog, and consider alternate means to resolve requests to /.well-known/api-catalog during planned downtime of hosts.  
* performance. Although the performance of APIs listed in an API Catalog can demand high transactions per second and low-latency response, the retrieval of the API Catalog itself to discover those APIs is less likely to incur strict  performance demands. That said, the Publisher SHOULD monitor the response time to fulfil a request for the API Catalog, and determine any necessary improvements (as with any other Web resource the Publisher serves). For large API Catalogs, the Publisher SHOULD consider the techniques described in {{SCALABILITY}}.
* usage. Since the goal of the api-catalog well-known URI is to facilitate discovery of APIs, the Publisher may wish to correlate requests to the /.well-known/api-catalog URI with subsequent requests to the API URIs listed in the catalog.
* current data. The Publisher SHOULD include the removal of stale API entries from the API Catalog as part of their API release lifecycle. The Publisher MAY decide to include metadata regarding legacy API versions or deprecated APIs to help users of those APIs discover up-to-date alternatives.
* correct metadata. The Publisher SHOULD include human and/or automated checks for syntax errors in the API Catalog. Automated checks include format validation (e.g. to ensure valid JSON syntax) and linting to enforce business rules - such as removing duplicate entries and ensuring descriptions are correctly named with valid values. A proofread of the API Catalog as part of the API release lifecycle is RECOMMENDED to detect any errors in business grammar (for example, an API entry that is described with valid syntax, but has been allocated an incorrect or outdated description.)
* security best practice, as set out in {{#security}}

## Integration with existing API management frameworks {#INTEGRATION}

A Publisher may already utilise an API management framework to produce their API portfolio. These frameworks typically include the publication of API endpoint URIs, deprecation and redirection of legacy API versions, API usage policies and documentation, plus other functions not relevant to the api-catalog topic. The api-catalog well-known URI and API Catalog document are intended to complement API management frameworks by facilitating the discovery of the framework's outputs - API endpoints, usage policies and documentation - and are not intended to replace any existing discovery mechanisms the framework has implemented.

Providers of such frameworks may include the production of an API Catalog and the publication of the /.well-known/api-catalog URI as a final pre-release (or post-release) step in the release management workflow. The following steps are recommended:

1. if ./well-known/api-catalog URI has not been published previously, the framework provider should: 
* collate and check the metadata for each API that will be included in the API Catalog. This metadata is likely to already exist in the framework.
* determine which metadata to include in the API Catalog, following the requirements set out in {{#API-CATALOG}} and the considerations set out in {{#OPERATIONS}}.
* map the chosen metadata to the format(s) described in {{#API-CATALOG}}. Where only the hyperlinks to APIs are to be included in the API Catalog, then the structure suggested in {{#api-catalog-example-linkset-bookmarks}} may be followed. Where possible the API Catalog SHOULD include further metadata per the guidance in {{#API-CATALOG}}, in which case the structure suggested in {{#api-catalog-example-linkset}} can be utilised and adapted (ensuring compliance to {{!RFC9264}}) to reflect the nature of the chosen metadata.  
* publish the /.well-known/api-catalog URI following the guidance set out in {{#usage}}.

2. if the ./well-known/api-catalog URI has previously been published, the framework provider should: 
* Include a step in the release management lifecycle to refresh the API Catalog following any changes in API hyperlinks or published metadata. This could include placing triggers on certain metadata fields, so that as they are updated in pre-production on the API framework, the updates are pushed to a pre-production copy of the API Catalog to be pushed live when the release is published by the framework.   

# Conformance to RFC8615 {#CONFORM-RFC8615}

The requirements in section 3 of {{WELL-KNOWN}} for defining Well-Known Uniform Resource Identifiers are met as described in the following sub-sections.

## Path suffix

The api-catalog URI SHALL be appended to the /.well-known/ path-prefix for "well-known locations".

## Formats and associated media types

A /.well-known/api-catalog location MUST support the Linkset {{!RFC9264}} format of application/linkset+json, and MAY
also support the other formats via content negotiation.

## Registration of the api-catalog well-known URI

See {{IANA}} considerations below.

# IANA Considerations {#IANA}

## The api-catalog well-known URI

This specification registers the "api-catalog" well-known URI in the Well-Known URI Registry as defined by {{WELL-KNOWN}}.

* URI suffix: api-catalog
* Change Controller: IETF
* Specification document(s):  THIS-RFC
* Status: permanent

## The api-catalog link relation

This specification registers the "api-catalog" link relation by following the procedures per section 2.1.1.1 of {{WEB-LINKING}}

* Relation Name:  api-catalog
* Description:  The link target identifies a catalog of the APIs published by the owner of the link target domain.
* Reference:  THIS-RFC

## The api-catalog Profile URI

 This specification registers "THIS-RFC-URL" in the "Profile URIs" registry according to {{!RFC7284}}.

* Profile URI: THIS-RFC-URL
* Common Name: API Catalog
* Description: A profile URI to request or signal a linkset representing an API Catalog.
* Reference: THIS-RFC

RFC Editor's Note: IANA is kindly requested to replace all instances of THIS-RFC and THIS-RFC-URL with the actual RFC number/URL once assigned.

# Security Considerations {#security}

For all scenarios: 

* TLS SHOULD be used, i.e. make /.well-known/api-catalog available exclusively over HTTPS,  to ensure no tampering of the API Catalog.
* The Publisher SHOULD take into account the Security Considerations from {{WELL-KNOWN}}.
* The Publisher SHOULD perform a security and privacy review of the API Catalog prior to deployment, to ensure it does not leak personal, business or other sensitive metadata, nor expose any vulnerability related to the APIs listed. 
* The Publisher SHOULD enforce read-only privileges for external requests to .well-known/api-catalog, and for internal systems and roles that monitor the .well-known/api-catalog URI. Write privileges SHOULD only be granted to roles that perform updates to the API Catalog and/or the forwarding rewrite rules for the .well-known/api-catalog URI.
* As with any Web offering, it is RECOMMENDED to apply rate-limiting measures to help mitigate abuse and prevent Denial-of-Service attacks on the API Catalog endpoint.

For the public-facing APIs scenario: security teams SHOULD additionally audit the API Catalog to ensure no APIs intended for internal use only have been mistakenly included. For example, a catalog hosted on https://developer.example.com should not expose unnecessary metadata about any internal domains (such as https://internal.example.net).

        
For the internal/private APIs scenario: the Publisher SHOULD take steps to ensure that appropriate access controls are in place to ensure only authorised roles and systems access an internal api-catalog well-known URI. 




--- back

# Example API Catalog documents {#api-catalog-example-linkset}

This section is informative and provides and example of an API Catalog document using the RECOMMENDED linkset format.

## Using Linkset with RFC8615 relations

This example uses the linkset format {{!RFC9264}}, and the following link relations defined in {{?RFC8631}}:

* "service-desc", used to link to a description of the API that is primarily intended for machine consumption.
* "service-doc", used to link to API documentation that is primarily intended for human consumption.
* "service-meta", used to link to additional metadata about the API, and is primarily intended for machine consumption.
* "status", used to link to the API status (e.g. API "health" indication etc.) for machine and/or human consumption.

Client request:

~~~ http-message
GET .well-known/api-catalog HTTP/1.1
Host: example.com
Accept: application/linkset+json
~~~

Server response:

~~~ http-message
HTTP/1.1 200 OK
Date: Mon, 01 Jun 2023 00:00:01 GMT
Server: Apache-Coyote/1.1
Content-Type: application/linkset+json;
    profile="THIS-RFC-URL"
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

This example also uses the linkset format {{!RFC9264}}, listing the API endpoints in an array of bookmarks. Each link shares the same context (the API Catalog) and "item" {{!RFC9264}} link relation (to indicate they are an item in the catalog). The intent is that by following a bookmark link, a machine-client can discover the purpose and usage of each API, hence the document targeted by the bookmark link should support this.

Note in the example below, the context anchor is www.example/com/.well-known/api-catalog, however as explained above the context anchor may be any other URI at which the api-catalog is available. 


Client request:

~~~ http-message
GET .well-known/api-catalog HTTP/1.1
Host: example.com
Accept: application/linkset+json
~~~

Server response:

~~~ http-message
HTTP/1.1 200 OK
Date: Mon, 01 Jun 2023 00:00:01 GMT
Server: Apache-Coyote/1.1
Content-Type: application/linkset+json;
    profile="THIS-RFC-URL"
~~~

~~~
{ "linkset":
 [
   { "anchor": "https://www.example.com/.well-known/api-catalog",
     "item": [
       {"href": "https://developer.example.com/apis/foo_api"},
       {"href": "https://developer.example.com/apis/bar_api"},
       {"href": "https://developer.example.com/apis/cantona_api"}
     ]
   }
 ]
}
~~~

## Nesting API Catalog links {#api-catalog-example-linkset-nesting}

In this example, a request to the /.well-known/api-catalog URI returns an array of links of relation type 'api-catalog'. This can be useful to Publishers with a large number of APIs, who wish to group them in smaller catalogs (as described in {{#SCALABILITY}}). 


Client request:

~~~ http-message
GET .well-known/api-catalog HTTP/1.1
Host: example.com
Accept: application/linkset+json
~~~

Server response:

~~~ http-message
HTTP/1.1 200 OK
Date: Mon, 01 Jun 2023 00:00:01 GMT
Server: Apache-Coyote/1.1
Content-Type: application/linkset+json;
    profile="THIS-RFC-URL"
~~~

~~~
{ "linkset":
 [
   { "anchor": "https://www.example.com/.well-known/api-catalog",
     "api-catalog": [
       {"href": "https://apis.example.com/iot/api-catalog"},
       {"href": "https://ecommerce.example.com/api-catalog"},
       {"href": "https://developer.example.com/gaming/api-catalog"}
     ]
 ]
}

~~~

# Acknowledgements

Thanks to Phil Archer, Ben Bucksch, Sanjay Dalal, Max Maton, Darrel Miller, Mark Nottingham,  Roberto Polli, Rich Salz, Herbert Van De Sompel and Erik Wilde for their suggestions and feedback.
