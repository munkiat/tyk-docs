---
title: Tyk Enterprise Developer Portal v1.6.0
description: Release notes documenting updates, enhancements and changes for Tyk Enterprise Developer Portal v1.6.0
tags: ["Developer Portal", "Release notes", "changelog", "v1.6.0"]
menu:
main:
parent: "Release Notes"
weight: 6
---

# Release Highlights
## OAuth2.0 flow now supports multiple identity providers
Now the Tyk Enterprise Developer portal can use multiple identity providers (IdPs) for OAuth2.0 via the Dynamic Client Registration flow. If your company has multiple oAuth2.0 providers now you can utilize them all for oAuth2.0 authentication. For instance, if your company uses different IdPs for different products (e.g. one for the U.S. and another for the EU) you can now achieve that with Tyk.

Just create multiple IdPs in the App registration menu:
{{< img src="/img/dashboard/portal-management/enterprise-portal/1.6.0-multiple-idps-index.png" width=500px alt="OAuth2.0 providers page">}}

And then use them to enable OAuth2.0 authentication for API Products:
{{< img src="/img/dashboard/portal-management/enterprise-portal/1.6.0-multiple-idps-edit.png" width=500px alt="OAuth2.0 provider overview">}}

## New Admin API for all content-blocks 
You can download all CMS content with just one API call with the brand new API endpoint GET /pages/all/content-blocks that returns all content blocks for all pages. Now migration between environments and deployment is much easier.

## Support for Mutual TLS
For customers who need extra security for their APIs such financial institutions and payment providers we introduced an ability for the portal to surface Mutual TLS APIs. Now you can configure API Key and oAuth2.0 API to support Mutual TLS. Just create an API that supports multiple authentication mechanisms in the Dashboard and publish it to the portal:
{{< img src="/img/dashboard/portal-management/enterprise-portal/1.6.0-mtl-support-dashboard.png" width=500px alt="Mutual TLS auth API in the Tyk Dashboard">}}

Now your developers can discover and request access to them in the portal:
{{< img src="/img/dashboard/portal-management/enterprise-portal/1.6.0-mtl-support-published.png" width=500px alt="Mutual TLS auth API Product is published in the portal">}}

{{< img src="/img/dashboard/portal-management/enterprise-portal/1.6.0-mtl-support-checkout.png" width=500px alt="Mutual TLS auth API Product in the checkout flow">}}

## Display-only support for API Products with the custom authentication 
This new capability you can display on the portal APIs use your own custom authentication mechanisms. We appreciate that many customer use their own auth mechanisms and even though at the moment we cannot create credentials for custom authentication schemas, we still want to support customers using these.

To display API Products that support custom authentication, you need simply to create an API Product that include APIs with custom authentication and synchronize it to the portal:
{{< img src="/img/dashboard/portal-management/enterprise-portal/1.6.0-display-custom-auth-apis.png" width=500px alt="Custom auth API Product is published in the portal">}}


# Changelog

## Added
- Added support for multiple IdPs of the OAuth2.0 flow.
- Added new admins APIs for querying all content-blocks. 
- Added support for API Products that usr Mutual TLS.
- Added display-only support for API Products with the custom authentication. 

## Changed
- Simplified the connection settings to the portal assets storage (where all images, themes, and other CMS files are stored) to help our customers get up to speed quicker. We are well aware that installing and configuring on-premise software can be tricky, especially when it comes to infrastructure, storage, and databases. Hence, we have decided to ease this burden for you:
  - By default, the portal uses the `db` [storage type]({{<ref "/tyk-stack/tyk-developer-portal/enterprise-developer-portal/install-tyk-enterprise-portal/configuration#portal_storage">}}) for storing its themes and other CMS assets and it doesn’t require any additional configuration. This means, you can start the portal right away without specifying any additional setting for the assets storage.
  - We also simplified setting for the s3 storage: now you need only to configure connection settings to the bucket and the portal will handle the rest.


## Fixed
- In 1.6.0 multiple important security bugs are fixed:
  - Added the ability to disable the theme upload capability. Since we don’t validate the theme content it might have viruses and other malicious software. So, super secure environments, we added a setting to disable the theme upload via the UI and API:
    {{< img src="/img/dashboard/portal-management/enterprise-portal/1.6.0-theme-upload-is-disabled.png" width=500px alt="Mutual TLS auth API Product in the checkout flow">}}
  - Fixed the bug where the session is not invalidated after a user logs out.
  - Fixed the role permission issue when a provider-admin can deactivate and delete a super-admin.
  - Fixed the Users API resource which allowed to enter any value into the Provider and Role fields.
- In addition to the security fixes, several bugs related to the theme management are fixed:
  - The list of available templates is now automatically updated when a new theme is loaded.
  - Fixed the bug where theme unpacking required unnecessary write permission to the /tmp folder..
  - Fixed icon alignment in the UI on the main page of the default theme.

