---
title: "SCIM Agents and Agentic Applications Extension"
abbrev: "SCIM Agents and Agentic Applications Extension"
category: std
docname: draft-scim-agent-extension-latest
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
  github: "macyabbey/draft-scim-agent-extension"
  latest: "https://macyabbey.github.io/draft-scim-agent-extension/draft-scim-agent-extension.html"

author:
  - fullname: "Macy Abbey"
    organization: Okta
    email: "macy.abbey@okta.com"

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
         "schema": "urn:ietf:params:scim:schemas:core:2.0:agent",
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

      description
         The description of the Agent.

      type
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

      <!-- TODO subject be in an extension instead of core? -->
      subject An optional attribute that clients may specify when
              provisioning an agent so that
              service providers implementing inbound token federation
              may correlate the agent with the `sub` claim in
              an inbound token from an OpenID connect provider.

      <!-- TODO should protocols be in an extension instead of core? -->
      protocols
         A complex attribute that informs service providers of the
         various communication protocols an agent may support.
         This information can help service providers automatically
         support agent to agent or human to agent communication scenarios.
         An agent that supports no protocols is understood to the service provider
         to not be directly accessible. For example, when an agent can only
         be accessed via its containing agentic application.

         The following sub-attributes are defined.

            type The type of the protocol. A number of canonical values
                 are provided based on known agent protocols. They are:
                 A2A, OpenAPI, MCP-Client, MCP-Server

            <!-- TODO  example values per spec type -->
            specificationUrl The URL the service provider may retrieve the
                             specification document describing the agent's specific
                             information for that protocol.


      <!-- TODO should owners be in an extension instead of core? -->
      parent
         A complex attribute that defines the parent Agent of this
         Agent if the service provider supports hierarchies of
         agents.  The following sub-attributes are defined.

         value  The ID of the agent that is the parent of this
            Agent in the hierarchy.

         $ref  A URI reference to the Agent that is the parent of this
            Agent in the hierarchy.

         display  The display name of the Agent that is the parent of
            this Agent in the hierarchy.

      <!-- TODO should owners be in an extension instead of core? -->
      owners
         A complex multi-valued attribute that defines the User or Group objects
         that are owners of this Agent.  OPTIONAL.  The following sub-attributes are
         defined for each value object.

         value  The ID of the User that owns this Agent.

         $ref  A URI reference to the User that owns this Agent.

         display  The display name of the user that owns this Agent.



#### Agent JSON Example

Example Agent representation:

```json
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Agent"],
  "id": "5d8d20c3-6a9f-4e45-b8d0-c7aa3f562acd",
  "externalId": "8ccc535b-716d-4d32-b3e9-57c8be449c82",
  "meta": {
    "resourceType": "Agent",
    "created": "2023-11-08T04:56:22Z",
    "lastModified": "2023-11-08T04:56:22Z",
    "location": "https://example.com/v2/Agents/5d8d20c3-6a9f-4e45-b8d0-c7aa3f562acd"
  },
  "name": "Customer Support Assistant",
  "displayName": "CS Assistant",
  "description": "An agent for handling customer support inquiries",
  "type": "support",
  "active": true,
  "entitlements": [
    {
      "value": "urn:example:entitlement:access-knowledge-base",
      "display": "Knowledge Base Access"
    },
    {
      "value": "urn:example:entitlement:create-support-tickets",
      "display": "Support Ticket Creation"
    }
  ],
  "roles": [
    {
      "value": "urn:example:role:support-agent",
      "display": "Support Agent"
    }
  ],
  "groups": [
    {
      "value": "e9e30dba-f08f-4109-8486-d5c6a331660a",
      "$ref": "https://example.com/v2/Groups/e9e30dba-f08f-4109-8486-d5c6a331660a",
      "display": "Customer Support Agents"
    }
  ],
  "applications": [
    {
      "value": "2819c223-7f76-453a-919d-413861904646",
      "$ref": "https://example.com/v2/AgenticApplications/2819c223-7f76-453a-919d-413861904646",
      "display": "AI Assistant Platform"
    }
  ],
  "subject": "agent:customer-support-assistant",
  "protocols": [
    {
      "type": "A2A",
      "specificationUrl": "https://example.com/a2a-specs/customer-support-assistant"
    },
    {
      "type": "OpenAPI",
      "specificationUrl": "https://example.com/openapi/customer-support-assistant"
    }
  ],
  "owners": [
    {
      "value": "b7c14771-226c-4d05-8860-134711653041",
      "$ref": "https://example.com/v2/Users/b7c14771-226c-4d05-8860-134711653041",
      "display": "John Doe"
    }
  ]
}
```

#### Agent Schema Json

The Agent schema in JSON format:

```json
{
  "id": "urn:ietf:params:scim:schemas:core:2.0:Agent",
  "name": "Agent",
  "description": "Agent Schema for SCIM",
  "attributes": [
    {
      "name": "name",
      "type": "string",
      "multiValued": false,
      "description": "The name of the Agent",
      "required": true,
      "caseExact": false,
      "mutability": "readWrite",
      "returned": "default",
      "uniqueness": "none"
    },
    {
      "name": "displayName",
      "type": "string",
      "multiValued": false,
      "description": "The display name of the Agent",
      "required": false,
      "caseExact": false,
      "mutability": "readWrite",
      "returned": "default",
      "uniqueness": "none"
    },
    {
      "name": "description",
      "type": "string",
      "multiValued": false,
      "description": "A description of the Agent",
      "required": false,
      "caseExact": false,
      "mutability": "readWrite",
      "returned": "default",
      "uniqueness": "none"
    },
    {
      "name": "type",
      "type": "string",
      "multiValued": false,
      "description": "The type of agent",
      "required": false,
      "caseExact": false,
      "mutability": "readWrite",
      "returned": "default",
      "uniqueness": "none"
    },
    {
      "name": "active",
      "type": "boolean",
      "multiValued": false,
      "description": "A Boolean value indicating the agent's administrative status",
      "required": false,
      "mutability": "readWrite",
      "returned": "default"
    },
    {
      "name": "entitlements",
      "type": "complex",
      "multiValued": true,
      "description": "Entitlements the agent has",
      "required": false,
      "subAttributes": [
        {
          "name": "value",
          "type": "string",
          "multiValued": false,
          "description": "The value of an entitlement",
          "required": false,
          "caseExact": false,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "display",
          "type": "string",
          "multiValued": false,
          "description": "A human-readable name for the entitlement",
          "required": false,
          "caseExact": false,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "type",
          "type": "string",
          "multiValued": false,
          "description": "A label indicating the attribute's function",
          "required": false,
          "caseExact": false,
          "canonicalValues": ["direct", "indirect"],
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "primary",
          "type": "boolean",
          "multiValued": false,
          "description": "A Boolean value indicating the 'primary' or preferred entitlement",
          "required": false,
          "mutability": "readWrite",
          "returned": "default"
        }
      ],
      "mutability": "readWrite",
      "returned": "default"
    },
    {
      "name": "roles",
      "type": "complex",
      "multiValued": true,
      "description": "Roles the agent assumes",
      "required": false,
      "subAttributes": [
        {
          "name": "value",
          "type": "string",
          "multiValued": false,
          "description": "The value of a role",
          "required": false,
          "caseExact": false,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "display",
          "type": "string",
          "multiValued": false,
          "description": "A human-readable name for the role",
          "required": false,
          "caseExact": false,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "type",
          "type": "string",
          "multiValued": false,
          "description": "A label indicating the attribute's function",
          "required": false,
          "caseExact": false,
          "canonicalValues": ["direct", "indirect"],
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "primary",
          "type": "boolean",
          "multiValued": false,
          "description": "A Boolean value indicating the 'primary' or preferred role",
          "required": false,
          "mutability": "readWrite",
          "returned": "default"
        }
      ],
      "mutability": "readWrite",
      "returned": "default"
    },
    {
      "name": "groups",
      "type": "complex",
      "multiValued": true,
      "description": "Groups to which the agent belongs",
      "required": false,
      "subAttributes": [
        {
          "name": "value",
          "type": "string",
          "multiValued": false,
          "description": "The identifier of the group",
          "required": false,
          "caseExact": false,
          "mutability": "readOnly",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "$ref",
          "type": "reference",
          "referenceTypes": ["Group"],
          "multiValued": false,
          "description": "The URI of the group",
          "required": false,
          "caseExact": false,
          "mutability": "readOnly",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "display",
          "type": "string",
          "multiValued": false,
          "description": "A human-readable name for the group",
          "required": false,
          "caseExact": false,
          "mutability": "readOnly",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "type",
          "type": "string",
          "multiValued": false,
          "description": "A label indicating the attribute's function",
          "required": false,
          "caseExact": false,
          "mutability": "readOnly",
          "returned": "default",
          "uniqueness": "none"
        }
      ],
      "mutability": "readOnly",
      "returned": "default"
    },
    {
      "name": "applications",
      "type": "complex",
      "multiValued": true,
      "description": "Applications this agent shares a trust boundary with",
      "required": false,
      "subAttributes": [
        {
          "name": "value",
          "type": "string",
          "multiValued": false,
          "description": "The identifier of the application",
          "required": false,
          "caseExact": false,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "$ref",
          "type": "reference",
          "referenceTypes": ["AgenticApplication"],
          "multiValued": false,
          "description": "The URI of the application",
          "required": false,
          "caseExact": false,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "display",
          "type": "string",
          "multiValued": false,
          "description": "A human-readable name for the application",
          "required": false,
          "caseExact": false,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        }
      ],
      "mutability": "readWrite",
      "returned": "default"
    },
    {
      "name": "subject",
      "type": "string",
      "multiValued": false,
      "description": "The subject identifier for token federation",
      "required": false,
      "caseExact": true,
      "mutability": "readWrite",
      "returned": "default",
      "uniqueness": "server"
    },
    {
      "name": "protocols",
      "type": "complex",
      "multiValued": true,
      "description": "Communication protocols the agent supports",
      "required": false,
      "subAttributes": [
        {
          "name": "type",
          "type": "string",
          "multiValued": false,
          "description": "The type of the protocol",
          "required": true,
          "caseExact": true,
          "canonicalValues": ["A2A", "OpenAPI", "MCP-Client", "MCP-Server"],
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "specificationUrl",
          "type": "string",
          "multiValued": false,
          "description": "URL to the protocol specification",
          "required": false,
          "caseExact": false,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        }
      ],
      "mutability": "readWrite",
      "returned": "default"
    },
    {
      "name": "parent",
      "type": "complex",
      "multiValued": false,
      "description": "The parent Agent of this Agent",
      "required": false,
      "subAttributes": [
        {
          "name": "value",
          "type": "string",
          "multiValued": false,
          "description": "The identifier of the parent agent",
          "required": false,
          "caseExact": false,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "$ref",
          "type": "reference",
          "referenceTypes": ["Agent"],
          "multiValued": false,
          "description": "The URI of the parent agent",
          "required": false,
          "caseExact": false,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "display",
          "type": "string",
          "multiValued": false,
          "description": "A human-readable name for the parent agent",
          "required": false,
          "caseExact": false,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        }
      ],
      "mutability": "readWrite",
      "returned": "default"
    },
    {
      "name": "owners",
      "type": "complex",
      "multiValued": true,
      "description": "User or Group objects that are owners of this Agent",
      "required": false,
      "subAttributes": [
        {
          "name": "value",
          "type": "string",
          "multiValued": false,
          "description": "The identifier of the owner",
          "required": false,
          "caseExact": false,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "$ref",
          "type": "reference",
          "referenceTypes": ["User", "Group"],
          "multiValued": false,
          "description": "The URI of the owner",
          "required": false,
          "caseExact": false,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "display",
          "type": "string",
          "multiValued": false,
          "description": "A human-readable name for the owner",
          "required": false,
          "caseExact": false,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        }
      ],
      "mutability": "readWrite",
      "returned": "default"
    }
  ],
  "meta": {
    "resourceType": "Schema",
    "location": "/v2/Schemas/urn:ietf:params:scim:schemas:core:2.0:Agent"
  }
}
```

### Agent extensions

<!-- See TODOs above, should some of those attributes come out and be extensions? -->
<!-- Do we need an "EnterpriseAgent" similar to "EnterpriseUser" extension? -->

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
         "schema": "urn:ietf:params:scim:schemas:core:2.0:agenticapplication",
      }

### Filtering

Clients MAY have a reference to the Agentic Application name, URL, or correlationId but not the ID.
For this reason, it is RECOMMENDED that service providers implement
filtering that allows equality matching on the "name", "correlationId", and "applicationUrls.value" attributes.

Example (note that escaping has been removed for readability):

      GET /scim/v2/AgenticApplications?filter=name eq 'AI Assistant Platform'
      
      GET /scim/v2/AgenticApplications?filter=correlationId eq 'app-123456'

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

      correlationId
         A string identifier assigned by the SCIM client, enabling correlation
         and reporting for an agentic application that has multiple identities.
         The definitive meaning of this attribute is determined by the SCIM client.

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
            The type of URL. Canonical values are: "sso", "api", "homepage", "console".
            
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
         A complex attribute that describes the OAuth parameters of the application.
         The following sub-attributes are defined:

         clientIds
            A multi-valued complex attribute containing OAuth client IDs.
            
            clientId
               The OAuth client identifier as described in section 2.2 of RFC6749.
               
            description
               A human-readable description of the client ID.
               
            environments
               A multi-valued attribute listing the environments this client ID is valid for.

         redirectUris
            A multi-valued attribute containing authorized redirect URIs.

         audiences
            A multi-valued attribute containing authorized audiences as defined
            in the "aud" claim of section 4.1.3 of RFC7519.

         issuer
            The identity provider issuer URI as defined in the "iss" claim
            of section 4.1.1 of RFC7519.

      agents
         A complex multi-valued attribute referencing agents associated with this application.
         The following sub-attributes are defined:
         
         value
            The ID of an agent associated with this application.
            
         $ref
            A URI reference to an agent associated with this application.
            
         display
            The display name of the agent.
            
         type
            The relationship type between the agent and application.
            Canonical values are: "owned", "authorized", "guest".

      identifiers
         A complex multi-valued attribute containing identifiers associated with this application.
         The following sub-attributes are defined:
         
         type
            The type of identifier. Service providers MAY define canonical values.
            
         value
            The identifier string value.
            
         system
            The system or domain this identifier is valid within.
            
### Example

Example Agentic Application:

```json
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:AgenticApplication"],
  "id": "2819c223-7f76-453a-919d-413861904646",
  "externalId": "app-123456",
  "meta": {
    "resourceType": "AgenticApplication",
    "created": "2023-11-08T04:56:22Z",
    "lastModified": "2023-11-08T04:56:22Z",
    "location": "https://example.com/v2/AgenticApplications/2819c223-7f76-453a-919d-413861904646"
  },
  "name": "AI Assistant Platform",
  "displayName": "Enterprise AI Assistant",
  "description": "Platform hosting various AI agents for enterprise use",
  "correlationId": "app-123456",
  "active": true,
  "applicationUrls": [
    {
      "type": "sso",
      "primary": true,
      "value": "https://login.example.com/aiassistant",
      "description": "SSO login URL for the AI Assistant Platform"
    },
    {
      "type": "api",
      "primary": true,
      "value": "https://api.example.com/aiassistant/v1",
      "description": "API endpoint for the AI Assistant Platform"
    },
    {
      "type": "homepage",
      "primary": true,
      "value": "https://aiassistant.example.com",
      "description": "Homepage for the AI Assistant Platform"
    }
  ],
  "lastAccessed": "2023-11-10T14:32:10Z",
  "oAuthConfiguration": {
    "clientIds": [
      {
        "clientId": "client-abc123",
        "description": "Production OAuth client",
        "environments": ["production"]
      },
      {
        "clientId": "client-test456",
        "description": "Test OAuth client",
        "environments": ["test", "staging"]
      }
    ],
    "redirectUris": [
      "https://aiassistant.example.com/oauth/callback",
      "https://test.aiassistant.example.com/oauth/callback"
    ],
    "audiences": ["https://api.example.com"],
    "issuer": "https://identity.example.com"
  },
  "agents": [
    {
      "value": "5d8d20c3-6a9f-4e45-b8d0-c7aa3f562acd",
      "$ref": "https://example.com/v2/Agents/5d8d20c3-6a9f-4e45-b8d0-c7aa3f562acd",
      "display": "Customer Support Assistant",
      "type": "owned"
    },
    {
      "value": "7e9d42b1-5a8c-4f21-a7b3-d5e8f9c12345",
      "$ref": "https://example.com/v2/Agents/7e9d42b1-5a8c-4f21-a7b3-d5e8f9c12345",
      "display": "Data Analysis Assistant",
      "type": "owned"
    }
  ],
  "identifiers": [
    {
      "type": "NHI",
      "value": "NHI12345",
      "system": "healthcare-system"
    },
    {
      "type": "internalAppId",
      "value": "int-app-789",
      "system": "enterprise-catalog"
    }
  ]
}
```

# Schema JSON Representations

This section provides the complete JSON representation for the schemas defined in this extension.

## Agent Schema JSON

The complete Agent schema in JSON format is included in the Agent Schema section above.

## Agentic Application Schema JSON

```json
{
  "id": "urn:ietf:params:scim:schemas:core:2.0:AgenticApplication",
  "name": "AgenticApplication",
  "description": "Agentic Application Schema for SCIM",
  "attributes": [
    {
      "name": "name",
      "type": "string",
      "multiValued": false,
      "description": "The name of the Agentic Application",
      "required": true,
      "caseExact": false,
      "mutability": "readWrite",
      "returned": "default",
      "uniqueness": "none"
    },
    {
      "name": "displayName",
      "type": "string",
      "multiValued": false,
      "description": "The display name of the Agentic Application",
      "required": false,
      "caseExact": false,
      "mutability": "readWrite",
      "returned": "default",
      "uniqueness": "none"
    },
    {
      "name": "description",
      "type": "string",
      "multiValued": false,
      "description": "A description of the Agentic Application",
      "required": false,
      "caseExact": false,
      "mutability": "readWrite",
      "returned": "default",
      "uniqueness": "none"
    },
    {
      "name": "correlationId",
      "type": "string",
      "multiValued": false,
      "description": "An identifier for correlation across systems",
      "required": false,
      "caseExact": true,
      "mutability": "readWrite",
      "returned": "default",
      "uniqueness": "none"
    },
    {
      "name": "active",
      "type": "boolean",
      "multiValued": false,
      "description": "A Boolean value indicating the application's administrative status",
      "required": false,
      "mutability": "readWrite",
      "returned": "default"
    },
    {
      "name": "applicationUrls",
      "type": "complex",
      "multiValued": true,
      "description": "URLs associated with the application",
      "required": false,
      "subAttributes": [
        {
          "name": "type",
          "type": "string",
          "multiValued": false,
          "description": "The type of URL",
          "required": true,
          "caseExact": true,
          "canonicalValues": ["sso", "api", "homepage", "console"],
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "primary",
          "type": "boolean",
          "multiValued": false,
          "description": "A Boolean value indicating if this is the primary URL",
          "required": false,
          "mutability": "readWrite",
          "returned": "default"
        },
        {
          "name": "value",
          "type": "string",
          "multiValued": false,
          "description": "The URL value",
          "required": true,
          "caseExact": true,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "description",
          "type": "string",
          "multiValued": false,
          "description": "A description of the URL",
          "required": false,
          "caseExact": false,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        }
      ],
      "mutability": "readWrite",
      "returned": "default"
    },
    {
      "name": "lastAccessed",
      "type": "dateTime",
      "multiValued": false,
      "description": "When the application was last accessed",
      "required": false,
      "mutability": "readWrite",
      "returned": "default",
      "uniqueness": "none"
    },
    {
      "name": "oAuthConfiguration",
      "type": "complex",
      "multiValued": false,
      "description": "OAuth configuration for the application",
      "required": false,
      "subAttributes": [
        {
          "name": "clientIds",
          "type": "complex",
          "multiValued": true,
          "description": "OAuth client IDs",
          "required": false,
          "subAttributes": [
            {
              "name": "clientId",
              "type": "string",
              "multiValued": false,
              "description": "The OAuth client identifier",
              "required": true,
              "caseExact": true,
              "mutability": "readWrite",
              "returned": "default",
              "uniqueness": "server"
            },
            {
              "name": "description",
              "type": "string",
              "multiValued": false,
              "description": "A description of the client ID",
              "required": false,
              "caseExact": false,
              "mutability": "readWrite",
              "returned": "default",
              "uniqueness": "none"
            },
            {
              "name": "environments",
              "type": "string",
              "multiValued": true,
              "description": "Environments for this client ID",
              "required": false,
              "caseExact": true,
              "mutability": "readWrite",
              "returned": "default",
              "uniqueness": "none"
            }
          ],
          "mutability": "readWrite",
          "returned": "default"
        },
        {
          "name": "redirectUris",
          "type": "string",
          "multiValued": true,
          "description": "Authorized redirect URIs",
          "required": false,
          "caseExact": true,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "audiences",
          "type": "string",
          "multiValued": true,
          "description": "Authorized audiences",
          "required": false,
          "caseExact": true,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "issuer",
          "type": "string",
          "multiValued": false,
          "description": "Identity provider issuer URI",
          "required": false,
          "caseExact": true,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        }
      ],
      "mutability": "readWrite",
      "returned": "default"
    },
    {
      "name": "agents",
      "type": "complex",
      "multiValued": true,
      "description": "Agents associated with this application",
      "required": false,
      "subAttributes": [
        {
          "name": "value",
          "type": "string",
          "multiValued": false,
          "description": "The identifier of the agent",
          "required": false,
          "caseExact": false,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "$ref",
          "type": "reference",
          "referenceTypes": ["Agent"],
          "multiValued": false,
          "description": "The URI of the agent",
          "required": false,
          "caseExact": false,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "display",
          "type": "string",
          "multiValued": false,
          "description": "A human-readable name for the agent",
          "required": false,
          "caseExact": false,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "type",
          "type": "string",
          "multiValued": false,
          "description": "The relationship type between agent and application",
          "required": false,
          "caseExact": true,
          "canonicalValues": ["owned", "authorized", "guest"],
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        }
      ],
      "mutability": "readWrite",
      "returned": "default"
    },
    {
      "name": "identifiers",
      "type": "complex",
      "multiValued": true,
      "description": "Identifiers associated with this application",
      "required": false,
      "subAttributes": [
        {
          "name": "type",
          "type": "string",
          "multiValued": false,
          "description": "The type of identifier",
          "required": true,
          "caseExact": true,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "value",
          "type": "string",
          "multiValued": false,
          "description": "The identifier value",
          "required": true,
          "caseExact": true,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        },
        {
          "name": "system",
          "type": "string",
          "multiValued": false,
          "description": "The system or domain for this identifier",
          "required": false,
          "caseExact": true,
          "mutability": "readWrite",
          "returned": "default",
          "uniqueness": "none"
        }
      ],
      "mutability": "readWrite",
      "returned": "default"
    }
  ],
  "meta": {
    "resourceType": "Schema",
    "location": "/v2/Schemas/urn:ietf:params:scim:schemas:core:2.0:AgenticApplication"
  }
}
```


# Security Considerations

This section outlines the security considerations specific to agent and agentic application management using SCIM.

## Authentication and Authorization

Implementations MUST follow the authentication and authorization requirements defined in {{RFC7644}}. Access to Agent and Agentic Application resources should be restricted based on the privileges of the requesting client.

## Handling of Sensitive Information

The Agents and Agentic Applications resources may contain sensitive information such as:
- OAuth client credentials in oAuthConfiguration
- System identifiers in the identifiers attribute
- URLs that may expose implementation details

Service providers should carefully consider which attributes are returned in different contexts and ensure proper authorization before disclosing sensitive information.

## Agent Subject Correlation

The subject attribute provides correlation between an agent and OpenID Connect identities. Implementation should take care to validate the authenticity of the token before establishing this correlation to prevent unauthorized access.

## Protocol Specifications

The protocolsSpecificationUrl attributes may link to documents containing sensitive details about how to communicate with an agent. Access to these specifications should be properly controlled to prevent unauthorized access to agent communication channels.

## Agentic Applications as OAuth Clients

When Agentic Applications are registered as OAuth clients, the normal security considerations for OAuth 2.0 [RFC6749] apply. In particular, implementations should validate redirect URIs, issue client secrets via secure channels, and implement proper token validation.

## Agent Privileges and Least Privilege

Agents should be provisioned with the minimum necessary entitlements required for their function. Implementers should regularly audit agent entitlements and roles, especially for agents with broad capabilities.

## Cross-domain Trust

When managing agents across different security domains, implementers must carefully consider the trust relationships established between domains. The presence of an agent in a SCIM server does not inherently convey trust - additional mechanisms should be employed to establish and validate cross-domain trust relationships.

## AI-specific Considerations

Due to the agentic nature of these resources, additional safeguards should be implemented:

1. Monitoring and auditing of agent actions to detect anomalous behavior
2. Mechanisms to suspend agent operation if unexpected behavior is detected
3. Clear boundaries on the resources and systems an agent can access
4. Regular review of agent permissions to detect privilege escalation

# IANA Considerations

This document has no IANA actions.

# Change Log

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
