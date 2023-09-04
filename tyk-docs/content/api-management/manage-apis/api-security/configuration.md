---
title: "Resource consumption"
date: 2023-09-04
tags: ["API Security", "security configuration"]
description: "General security configuration"
---

Modern APIs are often backed by large technology stacks composed of numerous components and libraries. Each of these is a potential weak link in the security chain, so efforts must be made to ensure that security measures are implemented throughout. The API gateway plays a critical part in an overall security strategy, by utilising its ability to process requests in a secure manner.

Secure Connections: Use transport layer security where possible. Most importantly, on inbound connections to the gateway and outbound connection from the gateway to the upstream API and other services. TLS can also be used as a form of authentication, using Mutual TLS.

Limit Functionality: Use security policies to specify which paths, methods and schemas are accessible, whilst blocking all others.

Mitigate Server-Side Request Forgery: Restrict any URL-based input data to specific schemas, hosts and paths by using schema validation. When data is fetched server-side, it should be validated and not returned to the client in raw format.

Protect Secrets: Prevent sensitive data, such as usernames, passwords, licence keys and other secrets, from being stored as plain text in application configuration files. Use key value secret storage to dynamically load sensitive data from a secure secret manager.

Sanitise Responses: Modify or remove sensitive data from responses by using transforms to alter the response headers and body.
