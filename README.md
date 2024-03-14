# astrojs-with-umbraco-cms-docs
Drafts of docs to present to AstroJS - https://docs.astro.build/en/guides/cms/

[Umbraco CMS](https://umbraco.com/) is an open-source ASP.NET Core CMS.

Umbraco serves as both a CMS with a built-in frontend and a headless CMS via its Content Delivery API that can be used to provide content to your Astro project.

## Integrating with Astro

Umbraco offers a built-in Content Delivery API for connecting your Umbraco content to any application.

### Prerequisites

To get started, you will need to have the following:

1. An Astro project - If you don’t have an Astro project yet, our [Installation guide](https://docs.astro.build/en/install/auto/) will get you up and running in no time.
2. An Umbraco (v12+) project - If you don’t have an Umbraco project yet, please see this [Installation guide](https://docs.umbraco.com/umbraco-cms/fundamentals/setup/install/).

### Setting up the Content Delivery API

To enable the Content Delivery API, you must modify your Umbraco project's `appsettings.json` file, adding the following `key:value` pair:

```json
{
  "Umbraco": {
    "CMS": {
      "DeliveryApi": {
        "Enabled": true
      }
    }
  }
}
```

While `Umbraco:CMS:DeliveryApi:Enabled` is the initial activator for the Content Delivery API, there are numerous other options which can be defined here. These options cover areas such as public access, API keys, allowed content types, membership authorisation and more.

For a full explanation on usage of the Content Delivery API please consult the [Umbraco Documentation](https://docs.umbraco.com/umbraco-cms/reference/content-delivery-api)

### Fetching Data

TODO: Add explanation of simple data fetch for a page

TODO: Clarify - should local dev always have Astro calling the HTTP endpoint, not the HTTPS endpoint? I've had issues on my local dev, so I think we should mention something either way. 

## Building a blog with Umbraco and Astro

TODO: Add section explaining how to create a simple blog

May include:
1. Creating Blog Document Types & Content
2. Displaying a list of Umbraco posts
3. Using the Content Delivery API to generate pages

## Publishing your site

To deploy your site visit our [deployment guides](https://docs.astro.build/en/guides/deploy/) and follow the instructions for your preferred hosting provider.

### Rebuild on Umbraco changes

TODO: Add section explaining webhooks

## Official Documentation
- Content Delivery API - Umbraco Documentation: https://docs.umbraco.com/umbraco-cms/reference/content-delivery-api

## Community Resources

- Astro-nomically Performant Websites using the Content Delivery API - Louis Richardson: https://24days.in/umbraco-cms/2023/sustainable-performant/astronomically-performant/
- Generating a TypeScript OpenAPI client from Umbraco's Content Delivery API - Rick Butterfield: https://rickbutterfield.dev/blog/typescript-openapi-umbraco-content-delivery/
- Jamstack For Free With Azure And CloudFlare - Kenn Jacobsen: https://kjac.dev/posts/jamstack-for-free-with-azure-and-cloudflare/

### Notes

- Guide for writing Astro docs: https://contribute.docs.astro.build/
- Explore running Astro locally w HTTPS: https://medium.com/geekculture/ssl-cert-https-for-local-development-using-astrojs-a58f4ae39bf1 <= [Unsuccessful, currently believe best local dev approach is to use HTTP endpoints only - happy to be corrected! - Rich J]
