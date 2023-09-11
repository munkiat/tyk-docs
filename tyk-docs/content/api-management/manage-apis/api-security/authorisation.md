---
title: "Authorisation"
date: 2023-09-04
tags: ["API Security", "Authorisation"]
description: "Authorisation best practices"
---

Authorisation is the process of validating API client requests against the access rights they have been granted, ensuring that the requests comply with any imposed limitations. It’s the most prevalent topic on the OWASP list, with three entries covering different levels of authorisation.

Almost any part of a request can be scrutinised as part of authorisation, but choosing the best approach depends on the type of API. For example, with REST APIs, the requested method and path are good candidates, but they aren’t relevant for GraphQL APIs, which should focus on the GraphQL query instead.

Authorisation can be a complex process that occurs at multiple locations throughout the request lifecycle. For example, a gateway can use access control policies to determine whether a required path is acceptable. But for decisions based on object data, such as when a client requests a particular record from the database, it’s the API that’s best positioned, as only it has access to the necessary data. For more information about the authorisation process, see Authorisation Levels in the appendix.

### Split Authorisation

Implement authorisation in the best locations across the stack. Use the gateway to handle general API authorisation related to hosts, methods, paths and properties. This leaves the API to handle the finer details of object-level authorisation. In terms of OWASPs authorisation categories, it can be split as follows:

#### Object Level Authorisation

Handle with the API. It can access and understand the data needed to make authorisation decisions on individual objects within its database.

#### Object Property Level Authorisation

Handle with both the API and the gateway. The approach depends on the type of API:

For REST APIs, it’s the API that’s primarily responsible for returning the correct data. To complement this, the gateway can use [body transforms]({{< ref "advanced-configuration/transform-traffic/response-body" >}}) to remove sensitive data from responses if the API is unable to do so itself. The gateway can also enforce object property-level restrictions using [JSON validation]({{< ref "advanced-configuration/transform-traffic/validate-json" >}}), for scenarios where the client is sending data to the API.

For GraphQL APIs, use the gateway to define [GraphQL schemas]({{< ref "graphql-proxy-only#managing-gql-schema" >}}) to limit which properties are queryable, then optionally use [field-based permissions]({{< ref "graphql-proxy-only#field-based-permission" >}}) to also specify access rights to those properties. 

#### Function Level Authorisation

Handle with the gateway. Use [security policies]({{< ref "basic-config-and-security/security/security-policies" >}}), [path-based permissions]({{< ref "security/security-policies/secure-apis-method-path" >}}), [allow lists]({{< ref "advanced-configuration/transform-traffic/endpoint-designer#allowlist" >}}) and [block lists]({{< ref "advanced-configuration/transform-traffic/endpoint-designer#blocklist" >}}) to manage authorisation of hosts and paths.

### Assign Least Privileges

Design [security policies]({{< ref "getting-started/key-concepts/what-is-a-security-policy" >}}) that contain the least privileges necessary for users to achieve the workflows supported by the API. By favouring specific, granular access over broad access, this enables user groups and use cases to be addressed directly, as opposed to broad policies that cover multiple use cases and expose functionality unnecessarily.

### Deny by Default

Favour use of [allow lists]({{< ref "advanced-configuration/transform-traffic/endpoint-designer#allowlist" >}}) to explicitly allow endpoints access, rather than [block lists]({{< ref "advanced-configuration/transform-traffic/endpoint-designer#blocklist" >}}) to explicitly deny. This approach prevents new API endpoints from being accessible by default, as the presence of other, allowed endpoints means that access to them is implicitly denied.

### Validate and Control All User Input

Protect APIs from erroneous or malicious data by validating all input before it’s processed by the API. Bad data, whether malicious or not, can cause many problems for APIs, from basic errors and bad user experience, to data leaks and downtime. The standard mitigation approach is to validate all user input, for which there are various solutions depending on the type of API:

For REST APIs, use [schema validation]({{< ref "graphql/validation#schema-validation" >}}) to control acceptable input data values.

For GraphQL APIs, use [GraphQL schema]({{< ref "graphql-proxy-only#managing-gql-schema" >}}) definitions to limit what data can be queried and mutated. Additionally, [complexity limiting]({{< ref "graphql/complexity-limiting" >}}) can be used to block resource-intensive queries.

### Track Anomalies

Use [log aggregation]({{< ref "log-data#integration-with-3rd-party-aggregated-log-and-error-tools" >}}) and [event triggers]({{< ref "basic-config-and-security/report-monitor-trigger-events" >}}) to push data generated by application logs and events into centralised monitoring and reporting systems. This real-time data stream can be used to highlight application issues and security-related events, such as authentication and authorisation failures.

### Understand System State

Perform application performance monitoring by capturing gateway [instrumentation data]({{< ref "basic-config-and-security/report-monitor-trigger-events/instrumentation" >}}). This enables the current system state, such as requests per second and response time, to be monitored and alerted upon.

### Manage Cross-Origin Resource Sharing

Use [CORS filtering]({{< ref "tyk-apis/tyk-gateway-api/api-definition-objects/cors" >}}) to control the resources accessible by browser-based clients. This is a necessity for APIs that expect to be consumed by external websites.


### Appendix: Authorisation Levels

This section provides basic examples of where different authorisation levels occur in the API management stack. The accompanying diagrams use colour-coding to show links between request element and the associated authorisation locations and methods.

This is how OWASP describe the attack vectors for the three authorisation levels:

**Object Level Authorisation**: “Attackers can exploit API endpoints that are vulnerable to broken object-level authorization by manipulating the ID of an object that is sent within the request. Object IDs can be anything from sequential integers, UUIDs, or generic strings. Regardless of the data type, they are easy to identify in the request target (path or query string parameters), request headers, or even as part of the request payload.” (source: [OWASP Github](https://github.com/OWASP/API-Security/blob/9c9a808215fcbebda9f657c12f3e572371697eb2/editions/2023/en/0xa1-broken-object-level-authorization.md))

**Object Property Level Authorisation**: “APIs tend to expose endpoints that return all object’s properties. This is particularly valid for REST APIs. For other protocols such as GraphQL, it may require crafted requests to specify which properties should be returned. Identifying these additional properties that can be manipulated requires more effort, but there are a few automated tools available to assist in this task.” (source: [OWASP Github](https://github.com/OWASP/API-Security/blob/9c9a808215fcbebda9f657c12f3e572371697eb2/editions/2023/en/0xa3-broken-object-property-level-authorization.md))

**Function Level Authorisation**: “Exploitation requires the attacker to send legitimate API calls to an API endpoint that they should not have access to as anonymous users or regular, non-privileged users. Exposed endpoints will be easily exploited.” (source: [OWASP Github](https://github.com/OWASP/API-Security/blob/9c9a808215fcbebda9f657c12f3e572371697eb2/editions/2023/en/0xa3-broken-object-property-level-authorization.md))


##### REST API - Reading Data

{{< img src="/img/api-management/security/rest-api-read-data.jpeg" alt="Rest API - Read Data" width="150px" >}}

The client sends a `GET` request using the path `/profile/1`. This path has two parts:

1. `/profile/`: The resource type, which is static for all requests related to profile objects. This requires function level authorisation.

2. `1`: The resource reference, which is dynamic and depends on the profile is being requested. This requires object level authorisation.

Next, the gateway handles function level authorisation by checking that the static part of the path, in this case `/profile/`, is authorised for access. It does this by cross referencing the security policies connected to the API key provided in the `authorization` header.

The gateway ignores the dynamic part of the part of the path, in this case `1`, as it doesn't have access to the necessary object-level data to make an authorisation decision for this.

Lastly, the API handles object level authorisation by using custom logic. This typically involves using the value of the `authorization` header in combination with the ownership and authorisation model specific to the API to determine if the client is authorised to read is requested record.

##### REST API - Writing Data

{{< img src="/img/api-management/security/rest-api-write-data.jpeg" alt="Rest API - Write Data" width="150px" >}}

The client sends a `POST` request using the path `/profile` and body data containing the object to write. The path `/profile` is static and requires function level authorisation. The body data contains a JSON object that has two fields:

1. `name`: A standard object field. This requires object property authorisation.

2. `id`: An object identifier field that refers to the identity of an object, so needs to be treated differently. As such, it requires both object property authorisation, like name, and also object authorisation.

Next, the gateway handles function level authorisation, by checking that the path, in the case `/profile`, is authorised for access. It does this by cross referencing the security policies connected to the API key provided in the `authorization` header.

The gateway can also perform object property level authorisation, by validating that the values of the body data fields, `name` and `id`, conform to a schema.

Lastly, the API handles object level authorisation by using custom logic. This typically involves using the value of the `authorization` header in combination with the ownership and authorisation model specific to the API to determine if the client is authorised to write the requested data.

##### GraphQL API - Querying Data


{{< img src="/img/api-management/security/graphql-query-data.jpeg" alt="Rest API - Write Data" width="150px" >}}

The client sends a _POST_ request using the path `/graphql` and body data containing a GraphQL query. The path `/graphql` is static and requires function level authorisation. The GraphQL query contains several elements:

- `profile`: An object type, referring to the type of object being requested. This requires object property authorisation.
- `id`: An object identifier field that refers to the identity of an object, so needs to be treated differently. As such, it requires both object property authorisation, like name, and also object authorisation.
- `name`: A standard object field, referring to a property of the profile object type. This requires object property authorisation.

Next, the Gateway handles function level authorisation, by checking that the path, in the case `/graphql`, is authorised for access. It does this by cross referencing the security policies connected to the API key provided in the `authorization` header. Due to the nature of GraphQL using just a single endpoint, there is no need for additional path-based authorisation features, only a basic security policy is required.

Another difference between this and the REST examples is in the way that the body data is authorised:

All object types and fields contained in the query are checked against the API’s GraphQL schema, to ensure they are valid. In this case, the object type is `profile`, and the fields are `id` and `name`. The schema defined in the gateway configuration can differ from that in the upstream API, which enables fields to be restricted by default.

Field-based permissions can also be used, to authorise client access of individual fields available in the schema. In this case, `id` and `name`.

Lastly, the API handles object level authorisation by using custom logic. This typically involves using the value of the `authorization` header in combination with the ownership and authorisation model specific to the API to determine if the client is authorised to access the requested data. This can be more complicated for GraphQL APIs, as the data presented by the schema may actually come from several different data sources.