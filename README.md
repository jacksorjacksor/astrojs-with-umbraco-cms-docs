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
        "Enabled": true,
        "PublicAccess": true
      }
    }
  }
}
```

While `Umbraco:CMS:DeliveryApi:Enabled` is the initial activator for the Content Delivery API, there are numerous other options which can be defined here. These options cover areas such as public access, API keys, allowed content types, membership authorisation and more.

For a full explanation on usage of the Content Delivery API please consult the [Umbraco Documentation](https://docs.umbraco.com/umbraco-cms/reference/content-delivery-api)

### Fetching Data

Data can be fetched from the Umbraco Content Delivery API.

For example, to display a list of blog articles:

```javascript
---
const res = await fetch('http://localhost/umbraco/delivery/api/v2/content?filter=contentType:article');
const articles = await res.json();
---
<h1>Astro + Umbraco 🚀</h1>
{
  articles.items.map((article) => (
      <h1>{article.name}</h1>
      <p>{article.properties.articleDate}</p>
  ))
}
```

## Building a blog with Umbraco and Astro

Once Astro has been setup and Umbraco installed, you are now able to create a blog that uses Umbraco as the CMS.

Firstly, create a Document Type in Umbraco for a simple blog 'Article'.

For information on creating Document Types, see [this guide](https://www.youtube.com/watch?v=otRuIkN80qM)

To follow along below, your 'Article' Document Type should have the following properties:

- Title (DataType: Text String)
- Article Date (DataType: Date Picker)
- Content (DataType: Rich Text Editor)

### Displaying a list of blog posts

This is the folder structure we will be using to create our blog.

```
src/
  │
  layouts/
    │   Layout.astro
  pages/
    │   index.astro
    │   [...slug].astro
```

If you don't have an existing Layout.astro template, create a `Layout.astro` file (for more information on Layouts see [here](https://docs.astro.build/en/basics/layouts/))

Then add the following markup to the `Layout.astro` file:

```javascript
---
---
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title></title>
</head>
<body>
    <main>
        <slot />
    </main>
</body>
</html>
```

We will use the homepage `index.astro` file at the root to list out the blog articles.

For this we will use a `fetch` to call the Content Delivery API `content` endpoint and filter by contentType of 'article'.

The article objects will include the properties and content set in the CMS.

We can then loop through the articles and display a desired listing with a link to the article.

```javascript
---
import Layout from '../layouts/Layout.astro';
const res = await fetch('http://localhost/umbraco/delivery/api/v2/content?filter=contentType:article');
const articles = await res.json();
---

<Layout>
	<h2>Blog Articles</h2>
	{
        articles.items.map((article: any) => (
            <div>
                <h3>{article.properties.title}</h3>
                <p>{article.properties.articleDate}</p>
                <a href={article.route.path}>Read more</a>
            </div>
        ))
    }
</Layout>
```

### Generating individual blog posts

We will now create the file `[...slug].astro` file at the root of the pages directory.

The `[...slug]` convention here will be used to create the Dynamic Routes for our articles. Read more about Dynamic Routing [here](https://docs.astro.build/en/guides/routing/#dynamic-routes)

Here we use the `getStaticPaths()` function to return an an array of objects that represent out pages, in this case, out blog articles.

the `slug` property on `params`, is used to generate the URL path of the page.

This is defined by the `[...slug].astro` naming convention.

We are also passing a `props` object that will include the article returned from the API.

These properties can then be used within our markup by defining `const { article } = Astro.props;`

```javascript
---
import Layout from '../layouts/Layout.astro';

export async function getStaticPaths() {
    let data = await fetch("http://localhost/umbraco/delivery/api/v2/content?filter=contentType:article");
    let articles = await data.json();
    
    return articles.items.map((article: any) => ({
        params: { slug: article.route.path },
        props: { article: article },
    }));
}

const { article } = Astro.props;
---

<Layout>
  <h1>{article.properties.title}</h1>
  <p>{article.properties.articleDate}</p>
  <div set:html={article.properties.content?.markup}></div>
</Layout>
```

## Publishing your site

To deploy your site visit our [deployment guides](https://docs.astro.build/en/guides/deploy/) and follow the instructions for your preferred hosting provider.

### Rebuild on Umbraco changes

TODO: Add section explaining webhooks


## Advanced Content Delivery API

#### Swagger endpoint

The Content Delivery API provides a Swagger endpoint at `/umbraco/swagger`.

![image](https://github.com/jacksorjacksor/astrojs-with-umbraco-cms-docs/assets/5838388/3dc156ad-b3f9-472c-98aa-35452172b2db)

## Misc, Support, Gotchas, Other notes

(This stuff may work it's way in, but may be too much information for the basic setup)

### Local dev, HTTPS and self-signed SSL certificates

When developing locally, be aware that Node (upon which Astro is built) won't accept self-signed certificates by default. This is a problem if using the Umbraco HTTPS endpoint locally, as any `fetch` queries will result in `fetch failed` with code `DEPTH_ZERO_SELF_SIGNED_CERT`.

To avoid this, either:

1. Use the Umbraco HTTP endpoint for local dev
2. Set `NODE_TLS_REJECT_UNAUTHORIZED=0` within `astro.config.js`. An approach for this involves two files:

.env.development
```
NODE_TLS_REJECT_UNAUTHORIZED=0
```

astro.config.js
```
import { defineConfig } from 'astro/config';
import { loadEnv } from "vite";

const { NODE_TLS_REJECT_UNAUTHORIZED } = loadEnv(process.env.NODE_ENV, process.cwd(), "");
process.env.NODE_TLS_REJECT_UNAUTHORIZED = NODE_TLS_REJECT_UNAUTHORIZED;
// https://astro.build/config
export default defineConfig({});
```

This method is described in [this blog post](https://kjac.dev/posts/jamstack-for-free-with-azure-and-cloudflare/), with an [accompanying repo](https://github.com/kjac/UmbracoAzureCloudflare).

#### No client / Basic `fetch`

In Astro you are now able to make `fetch` calls to the Umbraco Content Delivery API endpoint as defined in the [Data Fetching](https://docs.astro.build/en/guides/data-fetching/) area of the Astro documentation.

Example:
```javascript
const res = await fetch(
  "http://localhost:9858/umbraco/delivery/api/v2/content"
).then((res) => res.json());
console.log(res);
```

You can then tailor the query, and your handling of the response, accordingly.

This [blog](https://kjac.dev/posts/jamstack-for-free-with-azure-and-cloudflare/) article features an example of this.

#### With client

TODO: add Typescript client version, and add Content Delivery API typing via Delivery Api Extensions


## Official Documentation
- Content Delivery API - Umbraco Documentation: https://docs.umbraco.com/umbraco-cms/reference/content-delivery-api

## Community Resources

- Astro-nomically Performant Websites using the Content Delivery API - Louis Richardson: https://24days.in/umbraco-cms/2023/sustainable-performant/astronomically-performant/
- Generating a TypeScript OpenAPI client from Umbraco's Content Delivery API - Rick Butterfield: https://rickbutterfield.dev/blog/typescript-openapi-umbraco-content-delivery/
- Jamstack For Free With Azure And CloudFlare - Kenn Jacobsen: https://kjac.dev/posts/jamstack-for-free-with-azure-and-cloudflare/
- Quick n' dirty blog with Astro and Umbraco - Kenn Jacobsen: https://kjac.dev/posts/quick-n-dirty-blog-with-astro-and-umbraco/

### Notes

- Guide for writing Astro docs: https://contribute.docs.astro.build/
