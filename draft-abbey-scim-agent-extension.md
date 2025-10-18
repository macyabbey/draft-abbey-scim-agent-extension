---
title: "SCIM Agents and Agentic Applications Extension"
abbrev: "SCIM Agents and Agentic Applications Extension"
category: std
docname: draft-abbey-scim-agent-extension-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
workgroup: SCIM
keyword:
  - SCIM
  - Agents
  - Agentic applications
  - Workloads
stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "macyabbey/draft-abbey-scim-agent-extension"
  latest: "https://macyabbey.github.io/draft-abbey-scim-agent-extension/draft-abbey-scim-agent-extension.html"

author:
  - fullname: "Macy Abbey"
    organization: Okta
    email: "macy.abbey@gmail.com"
  - fullname: "Rafael S. Cohen"
    organization: Okta
    email: "rafael.cohen@okta.com"

# Normative/information references documented better here
#  https://github.com/cabo/kramdown-rfc
#
# WELL KNOWN
#  Lookup here: https://bib.ietf.org/
#  normative:
#    TCP: RFC0793
#  informative:
#    SST: DOI.10.1145/1282427.1282421
#
# FILL OUT YOURSELF
#   REST:
#     target: http://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf
#     title: Architectural Styles and the Design of Network-based Software Architectures
#     author:
#       ins: R. Fielding
#       name: Roy Thomas Fielding
#       org: University of California, Irvine
#     date: 2000
#     seriesinfo:
#       "Ph.D.": "Dissertation, University of California, Irvine"
#     format:
#       PDF: http://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf
normative:
  RFC7643:
informative:
  RFC7642:
  RFC7644:
  ENTITLEMENTS:
   target: https://github.com/ietf-scim-wg/draft-ietf-scim-roles-entitlements/blob/main/draft-ietf-scim-roles-entitlements.md
   title: SCIM Roles and Entitlements Extension
   author:
      - name: Danny Zollner
        org: Microsoft
        email: zollnerd@microsoft.com
      - name: Unmesh Vartak
        org: Okta
        email: unmesh.vartak@okta.com
#   draft-grizzle-scim-pam-ext-01:
#     target: https://datatracker.ietf.org/doc/id/draft-grizzle-scim-pam-ext-01.txt
#     format:
#       TXT: https://datatracker.ietf.org/doc/id/draft-grizzle-scim-pam-ext-01.txt

--- abstract

The System for Cross-domain Identity Management (SCIM) specification
{{!RFC7643}} provides schemas that represent common identity information
about users and groups, as well as a protocol for communicating that information
between systems.

The systems that tend to implement SCIM clients and servers are identity providers,
and service providers. These are the same systems that are now need to manage agents
and agentic applications across domains.

This document describes a SCIM 2.0 extension for agents and agentic applications,
which includes extensions to the core User and Group
objects, and new resource types and schemas for agentic constructs.

This extension is intended to provide greater interoperability between Identity
providers, agentic applications, agents and their clients while reducing the
responsibilities assumed by the every growing list of new protocols for agents.

--- middle

# Introduction

The SCIM protocol was originally developed to address an **abundance** of
complex standards for describing and exchanging user information.

As stated in the introduction of
[RFC7643#Section-1.1](https://datatracker.ietf.org/doc/html/rfc7643#section-1.1)

> While there are existing standards for describing and exchanging user
information, many of these standards can be difficult to implement and/or use...

> This increases both the cost and complexity associated with organizations adopting
products and services from multiple cloud providers, as they must perform
redundant integration development...SCIM seeks to simplify this problem through
an easily implemented specification suite...

With the rise of AI, agents, and agentic applications, we see another abundance
of protocols emerging, with varying levels of industry adoption,
as well as implementation complexity as many brilliant and enthusiastic
early adopters rush to define new standards for identity interopability.

This includes but is not limited to:

- [ACP](https://agentcommunicationprotocol.dev/core-concepts/agent-discovery)
- [A2A](https://a2a-protocol.org/latest/topics/agent-discovery/)
- [ANS](https://genai.owasp.org/resource/agent-name-service-ans-for-secure-al-agent-discovery-v1-0/)
- [AGNTCY](https://docs.agntcy.org/dir/overview/)

The intent of this SCIM extension is to offer a viable path for the industry
to re-leverage the well known core SCIM specifications, as well as existing
implementations of SCIM clients and SCIM servers, to solve for agent cross domain
management.

In doing so, we can free the emerging standards in the agentic AI space
to focus on truly novel concerns, instead of addressing the problems already
solved by SCIM for user and groups.

For example, in the A2A protocol, instead of [describing a very high level
concept of Curated registries](https://a2a-protocol.org/latest/topics/agent-discovery/#2-curated-registries-catalog-based-discovery)
we could offer more concrete guidance by stating Agent Cards may be discovered
by a SCIM client accessing any SCIM server that implements this extension.

# Conventions

{::boilerplate bcp14-tagged}

# Definitions

Agent: A workload with its own identifier, metadata and privileges which are
independent of a particular runtime environment or containing application.
An agent is distinct from a traditional software workloads (lambdas, services, etc...)
due to varying degrees of unpredictable behavior caused by delegation of
control flow to artificial intelligence models.

Agentic application: An application exposing one or more agents to its users.
An agentic application is similar to a traditional native or web application,
in that there are pre-defined ways authenticate and interact with the application;
however, as soon as the application exposes agents, there are additional
considerations for managing access to that application.

# Core Schema Extensions

## ServiceProviderConfig

SCIM endpoints that support Agent extensions MUST advertise this support
in the ServiceProviderConfig endpoint as defined:


      agentExtension
         A complex type that specifies Agent Extension configuration options.

         supported Boolean value specifying whether any aspect of the extension is supported.

         agentsSupported Boolean value specifying whether the agent resource type
                         is supported

         agenticApplicationsSupported Boolean value specifying whether the agent
                                      resource type is supported

This is required so that:

1) Clients may know if the server supports the concept of Agents.
2) Servers discourage clients from confusing users and agents.

If the server does not support the concept of agents, a SCIM client MAY choose
to create a User representation in the server for an Agent. All the reasons it
may choose to do so are beyond the scope of this document. If the client does so,
the client SHOULD indicate the user is linked to an agent using a LinkedObject
from [draft-grizzle-scim-pam-ext-01](https://datatracker.ietf.org/doc/id/draft-grizzle-scim-pam-ext-01.txt) This would allow a SCIM server that
supports that SCIM extension to add support for this extension and determine
what users in the server should be mapped to agents when support is added.

# Additional ResourceTypes and Schemas

This SCIM Agent extension defines additional
ResourceTypes and Schemas that MAY be implemented by the service
provider.  If implemented, these ResourceTypes SHOULD support all
SCIM operations {{RFC7644}}.  All attributes defined in the schemas are
optional unless explicitly marked as REQUIRED.

## Agent

This extension adds a new resource type of "Agent".

Pursuant to {{RFC7643}}
[Section 3.2 Defining New Resource Types](https://datatracker.ietf.org/doc/html/rfc7643#section-3.2)
this document define the ResourceType, Schema and Extensions for Agent.

### Agent Resource Type

The Agent Resource Type schema is:

      {
         "schemas": ["urn:ietf:params:scim:schemas:core:2.0:ResourceType"],
         "id": "Agent",
         "name": "Agent",
         "endpoint": "/Agents",
         "description": "Agent identities",
         "schema": "urn:ietf:params:scim:schemas:core:2.0:Agent"
      }

### Agent filtering

Clients MAY have a reference to the Agent name or externalId but not the ID.
For this reason, it is RECOMMENDED that service providers implement
filtering that allows equality matching on the "name" and "externalId" attributes.

Example (note that escaping has been removed for readability):

      GET /scim/v2/Agents?filter=name eq 'Helpdesk bot'

      GET /scim/v2/Agents?filter=externalId eq '8ccc535b-716d-4d32-b3e9-57c8be449c82'

### Agent Common Attributes

The agent resource type contains the common SCIM resource type attributes
defined in {{RFC7643}} [Section 3.1 Common Attributes](https://datatracker.ietf.org/doc/html/rfc7643#section-3.1)

They are listed here for completeness:

   + id
   + externalId
   + meta

### Agent Core Schema

The core agent schema provides the minimal representation of a resource "Agent".

It contains only those attributes that any agent may need, and only one
attribute is required.  It is identified using the schema URI:

"urn:ietf:params:scim:schemas:core:2.0:Agent"

The following attributes are defined in the core agent schema.

      name  The name of the Agent.  REQUIRED

      displayName
         The display name of the Agent.  If displayName is unassigned,
         the name MAY be used as the display name.

      active
         A Boolean value indicating the agent's administrative status.  The
         definitive meaning of this attribute is determined by the service
         provider.  As a typical example, a value of true implies that the
         agent is able to authenticate, while a value of false implies that the
         agent's account has been suspended and the agent will be unable to
         authenticate.

      description
         The description of the Agent.

      agentType
         The type of agent. There are no canonical values defined
         for type, but service providers MAY choose to define the valid
         types.

      active
         A Boolean value indicating the agent's administrative status.  The
         definitive meaning of this attribute is determined by the service
         provider.  As a typical example, a value of true implies that the
         agent is running, while a value of false implies that the
         agent has been suspended.

      entitlements
         An optional complex object that indicates entitlements the agent has.
         Its form is precisely the same as that defined in Section 4.1.2 of
         {{RFC7643}}.

      roles:
         An optional complex object that indicates roles the agent assumes.
         Its form is precisely the same as that defined in Section 4.1.2 of
         {{RFC7643}}.

      groups:
         An optional read-only complex object that indicates group
         membership.  Its form is precisely the same as that defined in
         Section 4.1.2 of {{RFC7643}}.

      applications
         A complex multi-valued attribute referencing applications this agent
         shares a trust boundary with. See "Agentic Application" section of
         this document.

      subject
         An optional attribute that clients may specify when
         provisioning an agent so that
         service providers implementing inbound token federation
         may correlate the agent with the `sub` claim in
         an inbound token from an OpenID connect provider.

     x509Certificates
         A list of certificates associated with the resource (e.g., a
         User).  Each value contains exactly one DER-encoded X.509
         certificate (see Section 4 of [RFC5280]), which MUST be base64
         encoded per Section 4 of [RFC4648].  A single value MUST NOT
         contain multiple certificates and so does not contain the encoding
         "SEQUENCE OF Certificate" in any guise.

      protocols
         A complex multi-value attribute that informs service providers of the
         various communication protocols an agent may support.
         This information can help service providers automatically
         support agent to agent or human to agent communication scenarios.
         An agent that supports no protocols is understood to the service provider
         to be inaccessible. For example, when an agent can only
         be accessed via its containing agentic application.

         The following sub-attributes are defined.

            type The type of the protocol. A number of canonical values
                 are provided based on known agent protocols. They are:
                 A2A, OpenAPI, MCP-Server

            specificationUrl The URL the service provider may retrieve the
                             specification document describing the agent's specific
                             information for that protocol.

      parent
         A complex attribute that defines the parent Agent of this
         Agent if the service provider supports hierarchies of
         agents.

         The following sub-attributes are defined.

            value  The ID of the agent that is the parent of this
               Agent in the hierarchy.

            $ref  A URI reference to the Agent that is the parent of this
               Agent in the hierarchy.

            display  The display name of the Agent that is the parent of
            this Agent in the hierarchy.

      owners
         A complex multi-valued attribute that defines the User or Group objects
         that are owners of this Agent.  OPTIONAL.  The following sub-attributes are
         defined for each value object.

         value  The ID of the User that owns this Agent.

         $ref  A URI reference to the User that owns this Agent.

         display  The display name of the user that owns this Agent.


#### JSON Representation

##### Minimal Agent Representation

The following is a non-normative example of the minimal required SCIM representation in JSON format.

      {
         "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Agent"],
         "id": "2819c223-7f76-453a-919d-413861904646",
         "name": "Clippy 2.0",
         "meta": {
            "resourceType": "Agent",
            "created": "2010-01-23T04:56:22Z",
            "lastModified": "2011-05-13T04:42:34Z",
            "version": "W\/\"3694e05e9dff590\"",
            "location": "https://example.com/v2/Agents/2819c223-7f76-453a-919d-413861904646"
         }
      }

##### Full Agent Representation

The following is a non-normative example of the fully populated SCIM
representation in JSON format.

      {
         "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Agent"],
         "id": "2819c223-7f76-453a-919d-413861904646",
         "externalId": "clpy2001",
         "name": "Clippy 2.0",
         "active":true,
         "agentType": "Assistant",
         "groups": [
            {
               "value": "e9e30dba-f08f-4109-8486-d5c6a331660a",
               "$ref":
         "https://example.com/v2/Groups/e9e30dba-f08f-4109-8486-d5c6a331660a",
               "display": "The next generation"
            },
            {
               "value": "fc348aa8-3835-40eb-a20b-c726e15c55b5",
               "$ref":
         "https://example.com/v2/Groups/fc348aa8-3835-40eb-a20b-c726e15c55b5",
               "display": "Animated assistants"
             },
             {
               "value": "71ddacd2-a8e7-49b8-a5db-ae50d0a5bfd7",
               "$ref":
         "https://example.com/v2/Groups/71ddacd2-a8e7-49b8-a5db-ae50d0a5bfd7",
               "display": "AI clippers"
             }
         ],
         "x509Certificates": [
         {
            "value":
             "MIIDQzCCAqygAwIBAgICEAAwDQYJKoZIhvcNAQEFBQAwTjELMAkGA1UEBhMCVVMx
              EzARBgNVBAgMCkNhbGlmb3JuaWExFDASBgNVBAoMC2V4YW1wbGUuY29tMRQwEgYD
              VQQDDAtleGFtcGxlLmNvbTAeFw0xMTEwMjIwNjI0MzFaFw0xMjEwMDQwNjI0MzFa
              MH8xCzAJBgNVBAYTAlVTMRMwEQYDVQQIDApDYWxpZm9ybmlhMRQwEgYDVQQKDAtl
              eGFtcGxlLmNvbTEhMB8GA1UEAwwYTXMuIEJhcmJhcmEgSiBKZW5zZW4gSUlJMSIw
              IAYJKoZIhvcNAQkBFhNiamVuc2VuQGV4YW1wbGUuY29tMIIBIjANBgkqhkiG9w0B
              AQEFAAOCAQ8AMIIBCgKCAQEA7Kr+Dcds/JQ5GwejJFcBIP682X3xpjis56AK02bc
              1FLgzdLI8auoR+cC9/Vrh5t66HkQIOdA4unHh0AaZ4xL5PhVbXIPMB5vAPKpzz5i
              PSi8xO8SL7I7SDhcBVJhqVqr3HgllEG6UClDdHO7nkLuwXq8HcISKkbT5WFTVfFZ
              zidPl8HZ7DhXkZIRtJwBweq4bvm3hM1Os7UQH05ZS6cVDgweKNwdLLrT51ikSQG3
              DYrl+ft781UQRIqxgwqCfXEuDiinPh0kkvIi5jivVu1Z9QiwlYEdRbLJ4zJQBmDr
              SGTMYn4lRc2HgHO4DqB/bnMVorHB0CC6AV1QoFK4GPe1LwIDAQABo3sweTAJBgNV
              HRMEAjAAMCwGCWCGSAGG+EIBDQQfFh1PcGVuU1NMIEdlbmVyYXRlZCBDZXJ0aWZp
              Y2F0ZTAdBgNVHQ4EFgQU8pD0U0vsZIsaA16lL8En8bx0F/gwHwYDVR0jBBgwFoAU
              dGeKitcaF7gnzsNwDx708kqaVt0wDQYJKoZIhvcNAQEFBQADgYEAA81SsFnOdYJt
              Ng5Tcq+/ByEDrBgnusx0jloUhByPMEVkoMZ3J7j1ZgI8rAbOkNngX8+pKfTiDz1R
              C4+dx8oU6Za+4NJXUjlL5CvV6BEYb1+QAEJwitTVvxB/A67g42/vzgAtoRUeDov1
              +GFiBZ+GNF/cAYKcMtGcrs2i97ZkJMo="
         }
         ],
         "entitlements": [{
            "value": "write",
            "display": "Write permission",
            "type: "permission",
            "primary": true
         }],
         "roles": [{
            "value": "administrator",
            "display": "Administrator",
            "type: "permission",
            "primary": true
         }],
         "applications": [{
            "value": "e9e30dba-f08f-4109-8486-d5c6a331660a",
            "$ref": "https://example.com/v2/AgenticApplications/e9e30dba-f08f-4109-8486-d5c6a331660a",
            "display": "Clippy portal",
            "type": "Web"
         }],
         "subject": "clpy2001",
         "protocols: [{
            "type": "A2A",
            "specificationUrl": "https://example.com/v2/Agents/2819c223-7f76-453a-919d-413861904646/.well-known/agent-card.json"
         }],
         "owners": [{
            "value": "71ddacd2-a8e7-49b8-a5db-ae50d0a5bfd7",
            "$ref": "../Groups/71ddacd2-a8e7-49b8-a5db-ae50d0a5bfd7",
            "display": "US Employees"
         }],
         "parent": {
            "value": "71ddacd2-a8e7-49b8-a5db-ae50d0a5bfd7",
            "$ref": "https://example.com/v2/Agents/71ddacd2-a8e7-49b8-a5db-ae50d0a5bfd7",
            "display": "Clippy 1.0"
         },
         "meta": {
            "resourceType": "Agent",
            "created": "2010-01-23T04:56:22Z",
            "lastModified": "2011-05-13T04:42:34Z",
            "version": "W\/\"3694e05e9dff590\"",
            "location": "https://example.com/v2/Agents/2819c223-7f76-453a-919d-413861904646"
         }
      }

## Agentic application

An Agentic application represents a software application that hosts or provides access to one or more agents.
It serves as a container and runtime environment for agents, managing their authentication, authorization, and access to resources.

### Resource Type

The Agentic Application Resource Type schema is:

      {
         "schemas": ["urn:ietf:params:scim:schemas:core:2.0:ResourceType"],
         "id": "AgenticApplication",
         "name": "AgenticApplication",
         "endpoint": "/AgenticApplications",
         "description": "Applications that host or provide access to agents",
         "schema": "urn:ietf:params:scim:schemas:core:2.0:AgenticApplication",
      }

### Filtering

Clients MAY have a reference to the Agentic Application name, URL, or externalId but not the ID.
For this reason, it is RECOMMENDED that service providers implement
filtering that allows equality matching on the "name", "externalId", and "applicationUrls.value" attributes.

Example (note that escaping has been removed for readability):

      GET /scim/v2/AgenticApplications?filter=name eq 'AI Assistant Platform'

      GET /scim/v2/AgenticApplications?filter=externalId eq 'app-123456'

### Common attributes

The agentic application resource type contains the common SCIM resource type attributes
defined in {{RFC7643}} [Section 3.1 Common Attributes](https://datatracker.ietf.org/doc/html/rfc7643#section-3.1)

They are listed here for completeness:

   + id
   + externalId
   + meta

### Schema

The core agentic application schema provides the representation of an "AgenticApplication" resource.
It is identified using the schema URI:

"urn:ietf:params:scim:schemas:core:2.0:AgenticApplication"

The following attributes are defined in the core agentic application schema.

      name
         The name of the Agentic Application. REQUIRED.

      displayName
         The display name of the Agentic Application. If displayName is unassigned,
         the name MAY be used as the display name.

      description
         The description of the Agentic Application.

      active
         A Boolean value indicating the application's administrative status.
         The definitive meaning of this attribute is determined by the service
         provider. As a typical example, a value of true implies that the
         application is operational, while a value of false implies that the
         application has been disabled.

      applicationUrls
         A complex multi-valued attribute containing URLs associated with the application.
         The following sub-attributes are defined:

         type
            The type of URL. Canonical values are: "ssoEndpoint", "loginPage", "api", "homepage".

         primary
            A Boolean value indicating whether this is the primary URL of this type.

         value
            The URL string value.

         description
            A human-readable description of the URL.

      lastAccessed
         Timestamp of when the application was last accessed by any agent or user.
         This attribute can be used for stale access detection and least privilege enforcement.

      oAuthConfiguration
         A complex multi-valued attribute that describes the OAuth connections of the application.
         The following sub-attributes are defined:

         clientId
            The OAuth client identifier as described in section 2.2 of RFC6749.

         description
            A human-readable description of the client ID.

         audienceUri
            The OAuth audience as defined
            in the "aud" claim of section 4.1.3 of RFC7519.

         issuerUri
            The identity provider issuer URI as defined in the "iss" claim
            of section 4.1.1 of RFC7519.

         redirectUri
            A multi-valued attribute containing authorized redirect URIs.


      agents
         A complex multi-valued attribute referencing agents associated with this application.
         The following sub-attributes are defined:

         value
            The ID of an agent associated with this application.

         ref
            A URI reference to an agent associated with this application.

         display
            The display name of the agent.

         type
            The relationship type between the agent and application.
            Canonical values are: "owned", "authorized", "guest".

      externalIdentifiers
         A complex multi-valued attribute containing identifiers associated with this application. OPTIONAL
         The following sub-attributes are defined:

         type
            The type of identifier. Service providers MAY define canonical values.
            <!-- Todo: what kind? I'm thinking about the SSO URLs of that application in the IDP -->

         value
            The identifier string value.

         system
            The system or domain this identifier is valid within.

### Example

Example Agentic Application:

      {
        "schemas": [
          "urn:ietf:params:scim:schemas:core:2.0:AgenticApplication"
        ],
        "id": "b57f8a91-4bc7-4b43-9f9c-2885c4b6c1e0",
        "externalId": "app-123456",
        "meta": {
          "resourceType": "AgenticApplication",
          "created": "2024-05-01T12:34:56Z",
          "lastModified": "2024-06-15T09:20:30Z"
        },
        "name": "AI Assistant Platform",
        "displayName": "AI Assistant Platform",
        "description": "Provides authenticated access to customer-facing AI assistants.",
        "active": true,
        "applicationUrls": [
          {
            "type": "homepage",
            "primary": true,
            "value": "https://assistant.example.com",
            "description": "Primary marketing site for the application"
          },
          {
            "type": "api",
            "value": "https://api.assistant.example.com/v1",
            "description": "Public API endpoint for agent integrations"
          }
        ],
        "oAuthConfiguration": [
          {
            "clientId": "0oa1qwerty",
            "description": "Production OAuth client",
            "audienceUri": "https://assistant.example.com/api",
            "issuerUri": "https://idp.example.com",
            "redirectUri": [
              "https://assistant.example.com/oauth/callback"
            ]
          }
        ],
        "agents": [
          {
            "value": "1d2c3b4a-5678-90ab-cdef-1234567890ab",
            "$ref": "https://scim.example.com/v2/Agents/1d2c3b4a-5678-90ab-cdef-1234567890ab",
            "display": "Customer Support Agent",
            "type": "owned"
          },
          {
            "value": "9f8e7d6c-5432-10fe-dcba-0987654321ff",
            "display": "Knowledge Retrieval Agent",
            "type": "authorized"
          }
        ],
        "externalIdentifiers": [
          {
            "type": "ssoTenantId",
            "value": "00abc123",
            "system": "https://idp.example.com"
          }
        ],
        "lastAccessed": "2024-06-20T17:45:00Z"
      }

# Schema JSON Representations

This section provides the complete JSON representation for the schemas defined in this extension.

## Agent Schema JSON

The following is intended as an example of the SCIM schema
   representation in JSON format for SCIM resources.  Where permitted, individual values and schema MAY change.  This example includes schema representations for "Agent".

      {
        "id" : "urn:ietf:params:scim:schemas:core:2.0:Agent",
        "name" : "Agent",
        "description" : "An AI agent",
        "attributes" : [
          {
            "name" : "name",
            "type" : "string",
            "multiValued" : false,
            "description" : "Unique identifier for the Agent, typically used by the agent to directly authenticate to the service provider. Each Agent MUST include a non-empty name value.  This identifier MUST be unique across the service provider's entire set of Agents. REQUIRED.",
            "required" : true,
            "caseExact" : false,
            "mutability" : "readWrite",
            "returned" : "default",
            "uniqueness" : "server"
          },
          {
            "name" : "agentType",
            "type" : "string",
            "multiValued" : false,
            "description" : "Used to classify like agents.  Typical values used might be
    'Assistant', 'Reseacher', 'Chat bot', and
    'Unknown', but any value may be used.",
            "required" : false,
            "caseExact" : false,
            "mutability" : "readWrite",
            "returned" : "default",
            "uniqueness" : "none"
          },
          {
            "name" : "active",
            "type" : "boolean",
            "multiValued" : false,
            "description" : "A Boolean value indicating the Agent's
    administrative status.",
            "required" : false,
            "mutability" : "readWrite",
            "returned" : "default"
          },
          {
            "name" : "groups",
            "type" : "complex",
            "multiValued" : true,
            "description" : "A list of groups to which the user belongs,
    either through direct membership, through nested groups, or
    dynamically calculated.",
            "required" : false,
            "subAttributes" : [
              {
                "name" : "value",
                "type" : "string",
                "multiValued" : false,
                "description" : "The identifier of the User's group.",
                "required" : false,
                "caseExact" : false,
                "mutability" : "readOnly",
                "returned" : "default",
                "uniqueness" : "none"
              },
              {
                "name" : "$ref",
                "type" : "reference",
                "referenceTypes" : [
                  "User",
                  "Group"
                ],
                "multiValued" : false,
                "description" : "The URI of the corresponding 'Group'
    resource to which the user belongs.",
                "required" : false,
                "caseExact" : false,
                "mutability" : "readOnly",
                "returned" : "default",
                "uniqueness" : "none"
              },
              {
                "name" : "display",
                "type" : "string",
                "multiValued" : false,
                "description" : "A human-readable name, primarily used
    for display purposes.  READ-ONLY.",
                "required" : false,
                "caseExact" : false,
                "mutability" : "readOnly",
                "returned" : "default",
                "uniqueness" : "none"
              },
              {
                "name" : "type",
                "type" : "string",
                "multiValued" : false,
                "description" : "A label indicating the attribute's
    function, e.g., 'direct' or 'indirect'.",
                "required" : false,
                "caseExact" : false,
                "canonicalValues" : [
                  "direct",
                  "indirect"
                ],
                "mutability" : "readOnly",
                "returned" : "default",
                "uniqueness" : "none"
              }
            ],
            "mutability" : "readOnly",
            "returned" : "default"
          },
          {
            "name" : "entitlements",
            "type" : "complex",
            "multiValued" : true,
            "description" : "A list of entitlements for the User that
    represent a thing the User has.",
            "required" : false,
            "subAttributes" : [
              {
                "name" : "value",
                "type" : "string",
                "multiValued" : false,
                "description" : "The value of an entitlement.",
                "required" : false,
                "caseExact" : false,
                "mutability" : "readWrite",
                "returned" : "default",
                "uniqueness" : "none"
              },
              {
                "name" : "display",
                "type" : "string",
                "multiValued" : false,
                "description" : "A human-readable name, primarily used
    for display purposes.  READ-ONLY.",
                "required" : false,
                "caseExact" : false,
                "mutability" : "readWrite",
                "returned" : "default",
                "uniqueness" : "none"
              },
              {
                "name" : "type",
                "type" : "string",
                "multiValued" : false,
                "description" : "A label indicating the attribute's
    function.",
                "required" : false,
                "caseExact" : false,
                "mutability" : "readWrite",
                "returned" : "default",
                "uniqueness" : "none"
              },
              {
                "name" : "primary",
                "type" : "boolean",
                "multiValued" : false,
                "description" : "A Boolean value indicating the 'primary'
    or preferred attribute value for this attribute.  The primary
    attribute value 'true' MUST appear no more than once.",
                "required" : false,
                "mutability" : "readWrite",
                "returned" : "default"
              }
            ],
            "mutability" : "readWrite",
            "returned" : "default"
          },
          {
            "name" : "roles",
            "type" : "complex",
            "multiValued" : true,
            "description" : "A list of roles for the User that
    collectively represent who the User is, e.g., 'Student', 'Faculty'.",
            "required" : false,
            "subAttributes" : [
              {
                "name" : "value",
                "type" : "string",
                "multiValued" : false,
                "description" : "The value of a role.",
                "required" : false,
                "caseExact" : false,
                "mutability" : "readWrite",
                "returned" : "default",
                "uniqueness" : "none"
              },
              {
                "name" : "display",
                "type" : "string",
                "multiValued" : false,
                "description" : "A human-readable name, primarily used
    for display purposes.  READ-ONLY.",
                "required" : false,
                "caseExact" : false,
                "mutability" : "readWrite",
                "returned" : "default",
                "uniqueness" : "none"
              },
              {
                "name" : "type",
                "type" : "string",
                "multiValued" : false,
                "description" : "A label indicating the attribute's
    function.",
                "required" : false,
                "caseExact" : false,
                "canonicalValues" : [],
                "mutability" : "readWrite",
                "returned" : "default",
                "uniqueness" : "none"
              },
              {
                "name" : "primary",
                "type" : "boolean",
                "multiValued" : false,
                "description" : "A Boolean value indicating the 'primary'
    or preferred attribute value for this attribute.  The primary
    attribute value 'true' MUST appear no more than once.",
                "required" : false,
                "mutability" : "readWrite",
                "returned" : "default"
              }
            ],
            "mutability" : "readWrite",
            "returned" : "default"
          },
          {
            "name" : "x509Certificates",
            "type" : "complex",
            "multiValued" : true,
            "description" : "A list of certificates issued to the User.",
            "required" : false,
            "caseExact" : false,
            "subAttributes" : [
              {
                "name" : "value",
                "type" : "binary",
                "multiValued" : false,
                "description" : "The value of an X.509 certificate.",
                "required" : false,
                "caseExact" : false,
                "mutability" : "readWrite",
                "returned" : "default",
                "uniqueness" : "none"
              },
              {
                "name" : "display",
                "type" : "string",
                "multiValued" : false,
                "description" : "A human-readable name, primarily used
    for display purposes.  READ-ONLY.",
                "required" : false,
                "caseExact" : false,
                "mutability" : "readWrite",
                "returned" : "default",
                "uniqueness" : "none"
              },
              {
                "name" : "type",
                "type" : "string",
                "multiValued" : false,
                "description" : "A label indicating the attribute's
    function.",
                "required" : false,
                "caseExact" : false,
                "canonicalValues" : [],
                "mutability" : "readWrite",
                "returned" : "default",
                "uniqueness" : "none"
              },
              {
                "name" : "primary",
                "type" : "boolean",
                "multiValued" : false,
                "description" : "A Boolean value indicating the 'primary'
    or preferred attribute value for this attribute.  The primary
    attribute value 'true' MUST appear no more than once.",
                "required" : false,
                "mutability" : "readWrite",
                "returned" : "default"
              }
            ],
            "mutability" : "readWrite",
            "returned" : "default"
          },
          {
            "name": "applications",
            "type" : "complex",
            "multiValued" : true,
            "description" : "A list of applications to which the agent belongs.",
            "required" : false,
            "subAttributes" : [
              {
                "name" : "value",
                "type" : "string",
                "multiValued" : false,
                "description" : "The identifier of the Agent's application.",
                "required" : false,
                "caseExact" : false,
                "mutability" : "readOnly",
                "returned" : "default",
                "uniqueness" : "none"
              },
              {
                "name" : "$ref",
                "type" : "reference",
                "referenceTypes" : [
                  "AgenticApplication"
                ],
                "multiValued" : false,
                "description" : "The URI of the corresponding 'AgenticApplication'
    resource to which the user belongs.",
                "required" : false,
                "caseExact" : false,
                "mutability" : "readOnly",
                "returned" : "default",
                "uniqueness" : "none"
              },
              {
                "name" : "display",
                "type" : "string",
                "multiValued" : false,
                "description" : "A human-readable name, primarily used
    for display purposes.  READ-ONLY.",
                "required" : false,
                "caseExact" : false,
                "mutability" : "readOnly",
                "returned" : "default",
                "uniqueness" : "none"
              }
            ],
            "mutability" : "readOnly",
            "returned" : "default"
          },
          {
             "name": "subject",
             "type" : "string",
             "multiValued" : false,
             "description" : "The subject to use for this agent in inbound tokens READ-ONLY.",
             "required" : false,
             "caseExact" : false,
             "mutability" : "readOnly",
             "returned" : "default",
             "uniqueness" : "none"
          },
          {
            "name": "owners",
            "type" : "complex",
            "multiValued" : true,
            "description" : "A list of users or groups that are the accountable parties for the agent.",
            "required" : false,
            "subAttributes" : [
              {
                "name" : "value",
                "type" : "string",
                "multiValued" : false,
                "description" : "The identifier of the Agent's application.",
                "required" : false,
                "caseExact" : false,
                "mutability" : "readOnly",
                "returned" : "default",
                "uniqueness" : "none"
              },
              {
                "name" : "$ref",
                "type" : "reference",
                "referenceTypes" : [
                  "User",
                  "Group"
                ],
                "multiValued" : false,
                "description" : "The URI of the corresponding 'User' or 'Group'",
                "required" : false,
                "caseExact" : false,
                "mutability" : "readOnly",
                "returned" : "default",
                "uniqueness" : "none"
              },
              {
                "name" : "display",
                "type" : "string",
                "multiValued" : false,
                "description" : "A human-readable name, primarily used
    for display purposes.  READ-ONLY.",
                "required" : false,
                "caseExact" : false,
                "mutability" : "readOnly",
                "returned" : "default",
                "uniqueness" : "none"
              }
            ],
            "mutability" : "readOnly",
            "returned" : "default"
          },
          {
             "name": "protocols",
             "type" : "complex",
             "multiValued" : true,
             "description" : "A list of protocols to communicate with the Agent.",
             "required" : false,
             "subAttributes" : [
                 {
                   "name" : "type",
                   "type" : "string",
                   "multiValued" : false,
                   "description" : "One of the canonical protocol types.",
                   "required" : false,
                   "caseExact" : false,
                   "mutability" : "readOnly",
                   "returned" : "default",
                   "uniqueness" : "none"
                 },
                 {
                   "name" : "specifiationUrl",
                   "type" : "string",
                   "multiValued" : false,
                   "description" : "URL of the specification for the protocol for this agent.",
                   "required" : false,
                   "caseExact" : false,
                   "mutability" : "readOnly",
                   "returned" : "default",
                   "uniqueness" : "none"
                 }
               ],
               "mutability" : "readOnly",
               "returned" : "default"
          },
          {
            "name": "parent",
            "type" : "complex",
            "multiValued" : false,
            "description" : "Parent agent.",
            "required" : false,
            "subAttributes" : [
              {
                "name" : "value",
                "type" : "string",
                "multiValued" : false,
                "description" : "The identifier of the parent Agent",
                "required" : false,
                "caseExact" : false,
                "mutability" : "readOnly",
                "returned" : "default",
                "uniqueness" : "none"
              },
              {
                "name" : "$ref",
                "type" : "reference",
                "referenceTypes" : [
                  "Agent"
                ],
                "multiValued" : false,
                "description" : "The URI of the corresponding 'Agent'",
                "required" : false,
                "caseExact" : false,
                "mutability" : "readOnly",
                "returned" : "default",
                "uniqueness" : "none"
              },
              {
                "name" : "display",
                "type" : "string",
                "multiValued" : false,
                "description" : "A human-readable name, primarily used
    for display purposes.  READ-ONLY.",
                "required" : false,
                "caseExact" : false,
                "mutability" : "readOnly",
                "returned" : "default",
                "uniqueness" : "none"
              }
            ],
            "mutability" : "readOnly",
            "returned" : "default"
          }
        ],
        "meta" : {
          "resourceType" : "Schema",
          "location" :
            "/v2/Schemas/urn:ietf:params:scim:schemas:core:2.0:Agent"
        }
      }

## Agentic Application Schema JSON

     {
       "id" : "urn:ietf:params:scim:schemas:core:2.0:AgenticApplication",
       "name" : "AgenticApplication",
       "description" : "An application that hosts or provides access to agents.",
       "attributes" : [
         {
           "name" : "name",
           "type" : "string",
           "multiValued" : false,
           "description" : "Unique identifier for the Agentic Application. Each Agentic Application MUST include a non-empty name value. This identifier MUST be unique across the service provider's entire set of Agentic Applications. REQUIRED.",
           "required" : true,
           "caseExact" : false,
           "mutability" : "readWrite",
           "returned" : "default",
           "uniqueness" : "server"
         },
         {
           "name" : "displayName",
           "type" : "string",
           "multiValued" : false,
           "description" : "A human-readable name, primarily used for display purposes. If not present, the value of name MAY be used instead.",
           "required" : false,
           "caseExact" : false,
           "mutability" : "readWrite",
           "returned" : "default",
           "uniqueness" : "none"
         },
         {
           "name" : "description",
           "type" : "string",
           "multiValued" : false,
           "description" : "A description of the Agentic Application.",
           "required" : false,
           "caseExact" : false,
           "mutability" : "readWrite",
           "returned" : "default",
           "uniqueness" : "none"
         },
         {
           "name" : "active",
           "type" : "boolean",
           "multiValued" : false,
           "description" : "A Boolean value indicating the Agentic Application's administrative status.",
           "required" : false,
           "mutability" : "readWrite",
           "returned" : "default"
         },
         {
           "name" : "applicationUrls",
           "type" : "complex",
           "multiValued" : true,
           "description" : "A list of URLs associated with the Agentic Application.",
           "required" : false,
           "subAttributes" : [
             {
               "name" : "type",
               "type" : "string",
               "multiValued" : false,
               "description" : "Type of URL.",
               "required" : false,
               "caseExact" : false,
               "canonicalValues" : [
                 "ssoEndpoint",
                 "loginPage",
                 "api",
                 "homepage"
               ],
               "mutability" : "readWrite",
               "returned" : "default",
               "uniqueness" : "none"
             },
             {
               "name" : "primary",
               "type" : "boolean",
               "multiValued" : false,
               "description" : "A Boolean value indicating whether this is the primary URL of this type.",
               "required" : false,
               "mutability" : "readWrite",
               "returned" : "default"
             },
             {
               "name" : "value",
               "type" : "string",
               "multiValued" : false,
               "description" : "The URL value.",
               "required" : false,
               "caseExact" : false,
               "mutability" : "readWrite",
               "returned" : "default",
               "uniqueness" : "none"
             },
             {
               "name" : "description",
               "type" : "string",
               "multiValued" : false,
               "description" : "A human-readable description of the URL.",
               "required" : false,
               "caseExact" : false,
               "mutability" : "readWrite",
               "returned" : "default",
               "uniqueness" : "none"
             }
           ],
           "mutability" : "readWrite",
           "returned" : "default"
         },
         {
           "name" : "lastAccessed",
           "type" : "dateTime",
           "multiValued" : false,
           "description" : "Timestamp of when the Agentic Application was last accessed.",
           "required" : false,
           "mutability" : "readOnly",
           "returned" : "default"
         },
         {
           "name" : "oAuthConfiguration",
           "type" : "complex",
           "multiValued" : true,
           "description" : "OAuth client configuration for the Agentic Application.",
           "required" : false,
           "subAttributes" : [
             {
               "name" : "clientId",
               "type" : "string",
               "multiValued" : false,
               "description" : "The OAuth client identifier.",
               "required" : false,
               "caseExact" : false,
               "mutability" : "readWrite",
               "returned" : "default",
               "uniqueness" : "none"
             },
             {
               "name" : "description",
               "type" : "string",
               "multiValued" : false,
               "description" : "A human-readable description of the OAuth client.",
               "required" : false,
               "caseExact" : false,
               "mutability" : "readWrite",
               "returned" : "default",
               "uniqueness" : "none"
             },
             {
               "name" : "audienceUri",
               "type" : "string",
               "multiValued" : false,
               "description" : "The OAuth audience URI.",
               "required" : false,
               "caseExact" : false,
               "mutability" : "readWrite",
               "returned" : "default",
               "uniqueness" : "none"
             },
             {
               "name" : "issuerUri",
               "type" : "string",
               "multiValued" : false,
               "description" : "The identity provider issuer URI.",
               "required" : false,
               "caseExact" : false,
               "mutability" : "readWrite",
               "returned" : "default",
               "uniqueness" : "none"
             },
             {
               "name" : "redirectUri",
               "type" : "reference",
               "referenceTypes" : [
                 "uri"
               ],
               "multiValued" : true,
               "description" : "Authorized redirect URIs for the OAuth client.",
               "required" : false,
               "caseExact" : false,
               "mutability" : "readWrite",
               "returned" : "default",
               "uniqueness" : "none"
             }
           ],
           "mutability" : "readWrite",
           "returned" : "default"
         },
         {
           "name" : "agents",
           "type" : "complex",
           "multiValued" : true,
           "description" : "Agents associated with this Agentic Application.",
           "required" : false,
           "subAttributes" : [
             {
               "name" : "value",
               "type" : "string",
               "multiValued" : false,
               "description" : "Identifier of the associated Agent.",
               "required" : false,
               "caseExact" : false,
               "mutability" : "readWrite",
               "returned" : "default",
               "uniqueness" : "none"
             },
             {
               "name" : "$ref",
               "type" : "reference",
               "referenceTypes" : [
                 "Agent"
               ],
               "multiValued" : false,
               "description" : "The URI of the associated Agent resource.",
               "required" : false,
               "caseExact" : false,
               "mutability" : "readOnly",
               "returned" : "default",
               "uniqueness" : "none"
             },
             {
               "name" : "display",
               "type" : "string",
               "multiValued" : false,
               "description" : "Display name of the associated Agent.",
               "required" : false,
               "caseExact" : false,
               "mutability" : "readOnly",
               "returned" : "default",
               "uniqueness" : "none"
             },
             {
               "name" : "type",
               "type" : "string",
               "multiValued" : false,
               "description" : "Relationship between the Agentic Application and the Agent.",
               "required" : false,
               "caseExact" : false,
               "canonicalValues" : [
                 "owned",
                 "authorized",
                 "guest"
               ],
               "mutability" : "readWrite",
               "returned" : "default",
               "uniqueness" : "none"
             }
           ],
           "mutability" : "readWrite",
           "returned" : "default"
         },
         {
           "name" : "externalIdentifiers",
           "type" : "complex",
           "multiValued" : true,
           "description" : "External identifiers for the Agentic Application.",
           "required" : false,
           "subAttributes" : [
             {
               "name" : "type",
               "type" : "string",
               "multiValued" : false,
               "description" : "Type of identifier.",
               "required" : false,
               "caseExact" : false,
               "mutability" : "readWrite",
               "returned" : "default",
               "uniqueness" : "none"
             },
             {
               "name" : "value",
               "type" : "string",
               "multiValued" : false,
               "description" : "Identifier value.",
               "required" : false,
               "caseExact" : false,
               "mutability" : "readWrite",
               "returned" : "default",
               "uniqueness" : "none"
             },
             {
               "name" : "system",
               "type" : "string",
               "multiValued" : false,
               "description" : "System or domain within which the identifier is valid.",
               "required" : false,
               "caseExact" : false,
               "mutability" : "readWrite",
               "returned" : "default",
               "uniqueness" : "none"
             }
           ],
           "mutability" : "readWrite",
           "returned" : "default"
         }
       ],
       "meta" : {
         "resourceType" : "Schema",
         "location" : "/v2/Schemas/urn:ietf:params:scim:schemas:core:2.0:AgenticApplication"
       }
     }

# Security Considerations
-> fill out

# IANA Considerations

This document has no IANA actions.

# Change Log

-01

+ Macy finish up Agent schema description, JSON representation and schema
+ Rafael contribution of agent app

-00

+ Initial draft extension.

--- back

# Acknowledgments

We would like to thanks the authors of the
[SCIM Extension for Privileged Access Management](https://datatracker.ietf.org/doc/id/draft-grizzle-scim-pam-ext-01.txt) and [Device Schema Extensions to the SCIM model](https://datatracker.ietf.org/doc/draft-ietf-scim-device-model/)
which served as excellent guidance on how to document proposed extension to
the SCIM protocol.

Additionaly, we would like to thank all the contributors the emerging agent
standards which inspired this extension, including:

- [Agent communication protocol](https://agentcommunicationprotocol.dev/core-concepts/agent-discovery)
- [Agent 2 Agent](https://a2a-protocol.org/latest/topics/agent-discovery/)
- [Agent name service](https://genai.owasp.org/resource/agent-name-service-ans-for-secure-al-agent-discovery-v1-0/)
- [AGNTCY directory](https://docs.agntcy.org/dir/overview/)
