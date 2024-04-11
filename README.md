Drafts of docs to present to AstroJS - https://docs.astro.build/en/guides/cms/

# Umbraco & Astro

[Umbraco CMS](https://umbraco.com/) is an open-source ASP.NET Core CMS. By default, Umbraco uses Razor pages for its front-end, but can be used as a headless CMS.

## Integrating with Astro

In this section you will use Umbraco's native [Content Delivery API](https://docs.umbraco.com/umbraco-cms/reference/content-delivery-api) to provide content to your Astro project.

## Prerequisites

To get started, you will need to have the following:

1. An Astro project - If you don’t have an Astro project yet, our [Installation guide](https://docs.astro.build/en/install/auto/) will get you up and running in no time.
2. An Umbraco (v12+) project - If you don’t have an Umbraco project yet, please see this [Installation guide](https://docs.umbraco.com/umbraco-cms/fundamentals/setup/install/).

## Setting up the Content Delivery API

To enable the Content Delivery API, update your Umbraco project's `appsettings.json` file:

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

While `Umbraco:CMS:DeliveryApi:Enabled` is the initial activator for the Content Delivery API, there are other options which can be set here. These options cover areas such as public access, API keys, allowed content types, membership authorisation and more. For a full explanation on usage of the Content Delivery API please consult the [Umbraco Documentation](https://docs.umbraco.com/umbraco-cms/reference/content-delivery-api)

## Fetching Data

Data can be fetched from the Umbraco Content Delivery API. Here's an example of how to display a list of articles, as described in the later section "Building a blog with Umbraco and Astro":

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
      <div set:html={article.properties.content?.markup}></div>
  ))
}
```

The `fetch` call returns all content from the site's Content Delivery API with content type `article`. The `article` content type must have a [Date](https://docs.umbraco.com/umbraco-cms/fundamentals/backoffice/property-editors/built-in-umbraco-property-editors/date) property called `articleDate`.

For more details on Data Fetching with Astro please read the [official documentation](https://docs.astro.build/en/guides/data-fetching/).

## Building a blog with Umbraco and Astro

### Prerequisites:

- Astro project, newly created with `npm create astro@latest` command with the following settings:
	- Where should we create your new project? `./astro.frontend`
	- How would you like to start your new project? `Empty`
 	- Do you plan to write TypeScript? `Yes`
  	- How strict should TypeScript be? `Strict (recommended)`
  	- Install dependencies? `Yes`
  	- Initialize a new git repository? `No`

- Umbraco project (v12+) - please follow this [Installation guide](https://docs.umbraco.com/umbraco-cms/fundamentals/setup/install/).
	- Enable the Content Delivery API. Instructions have been provided in the "Setting up the Content Delivery API" section above.

Once Astro has been setup and Umbraco installed, you are now able to create a blog that uses Umbraco as the CMS.

### Creating the blog posts in Umbraco

From the Umbraco backoffice, create a Document Type for a simple blog article called 'Article'.

To follow along below, your 'Article' Document Type should have the following properties:

- Title (DataType: Textstring)
- Article Date (DataType: Date Picker)
- Content (DataType: Richtext Editor)

For this simple example, on the 'Article' document type, set the permission "Allow as root" to `true`.

Please create and publish some test articles.

Once these articles are published you should be able to retrieve the article details for all published articles as a JSON object via a GET request to the following URL:

`http://localhost:44335/umbraco/delivery/api/v2/content?filter=contentType:article`

Please replace the port number with your local development environment's HTTP endpoint port number.

For information on creating Document Types, see [this guide](https://www.youtube.com/watch?v=otRuIkN80qM).

### Displaying a list of blog posts in Astro

Within our Astro project, this is the folder structure we will be using to create our blog:

```
src/
  │
  layouts/
    │   Layout.astro
  pages/
    │   index.astro
    │   [...slug].astro
```

If using the `Empty` project, please create the `layouts/` folder, and `Layout.astro` and `[...slug].astro` files.

For more information on Layouts see the [Astro documentation](https://docs.astro.build/en/basics/layouts/).

Add the following markup to the `Layout.astro` file:

```javascript
---
---
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>My Blog with Astro and Umbraco</title>
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

Replace the default contents of `index.astro` with the below:

`index.astro`
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

We will now create the file `[...slug].astro` file at the root of the `pages` directory.

The `[...slug]` convention here will be used to create the Dynamic Routes for our articles. Read more about Dynamic Routing [here](https://docs.astro.build/en/guides/routing/#dynamic-routes)

Here we use the `getStaticPaths()` function to return an an array of objects that represent out pages, in this case, our blog articles.

The `slug` property on `params` is used to generate the URL path of the page. This is defined by the `[...slug].astro` naming convention.

We are also passing a `props` object that will include the article returned from the API.

These properties can then be used within our markup by defining `const { article } = Astro.props;`

`[...slug].astro`
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

You should now be able to view your Umbraco-created content through the Astro front-end. Congratulations! 🚀


## Publishing your site

To deploy your site visit our [deployment guides](https://docs.astro.build/en/guides/deploy/) and follow the instructions for your preferred hosting provider.

## Local dev, HTTPS and self-signed SSL certificates

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

## Official Documentation
- Content Delivery API - Umbraco Documentation: https://docs.umbraco.com/umbraco-cms/reference/content-delivery-api

## Community Resources

- Astro-nomically Performant Websites using the Content Delivery API - Louis Richardson: https://24days.in/umbraco-cms/2023/sustainable-performant/astronomically-performant/
- Generating a TypeScript OpenAPI client from Umbraco's Content Delivery API - Rick Butterfield: https://rickbutterfield.dev/blog/typescript-openapi-umbraco-content-delivery/
- Jamstack For Free With Azure And CloudFlare - Kenn Jacobsen: https://kjac.dev/posts/jamstack-for-free-with-azure-and-cloudflare/
- Quick n' dirty blog with Astro and Umbraco - Kenn Jacobsen: https://kjac.dev/posts/quick-n-dirty-blog-with-astro-and-umbraco/
