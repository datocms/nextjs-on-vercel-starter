<!--datocms-autoinclude-header start-->

<a href="https://www.datocms.com/"><img src="https://www.datocms.com/images/full_logo.svg" height="60"></a>

👉 [Visit the DatoCMS homepage](https://www.datocms.com) or see [What is DatoCMS?](#what-is-datocms)

<!--datocms-autoinclude-header end-->

# Next.js + DatoCMS Cache Tags

Everything you need to know to build a Next.js project powered by DatoCMS Cache Tags to achieve the perfect balance of performance, efficiency, and real-time updates.

<img src="https://github.com/datocms/nextjs-with-cache-tags-starter/raw/main/images/browser.png" />

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Getting Started](#getting-started)
  - [Step 1: Clone the DatoCMS project](#step-1-clone-the-datocms-project)
  - [Step 2: Environment variables](#step-2-environment-variables)
      - [`WEBHOOK_TOKEN`](#webhook_token)
      - [`PUBLIC_DATOCMS_API_TOKEN`](#public_datocms_api_token)
      - [`TURSO_DATABASE_URL`, `TURSO_AUTH_TOKEN`](#turso_database_url-turso_auth_token)
  - [Step 3: Install dependencies and download the DatoCMS GraphQL Schema](#step-3-install-dependencies-and-download-the-datocms-graphql-schema)
  - [Step 4: Run development server](#step-4-run-development-server)
- [Deployment](#deployment)
- [Useful resources to navigate the code](#useful-resources-to-navigate-the-code)
  - [Execution of GraphQL queries](#execution-of-graphql-queries)
  - ["Cache Tags Invalidation" webhook](#cache-tags-invalidation-webhook)

## Getting Started

### Step 1: Clone the DatoCMS project

Start by pressing this button to create a new project on DatoCMS containing the data expected by this project:

[![Clone DatoCMS project](https://dashboard.datocms.com/clone/button.svg)](https://dashboard.datocms.com/clone?projectId=23796&name=Next.js+%2B+Cache+Tags)

### Step 2: Environment variables

Use `.env.local.example` file as a starting point:

```bash
cp .env.local.example .env.local
```

Now open `.env.local` and start populating the variables:

##### `WEBHOOK_TOKEN`

Any secure random string. It will be used to authenticate the webhook requests that come from DatoCMS.

##### `PUBLIC_DATOCMS_API_TOKEN`

An API token from the DatoCMS project you just created. [Learn to create an API token](https://www.datocms.com/docs/content-delivery-api/authentication).

##### `TURSO_DATABASE_URL`, `TURSO_AUTH_TOKEN`

Create a database and initialize it with the expected schema, just pressing following button: after signing up or logging in, it's a single click process.

[![Create a database with Turso](https://sqlite.new/button)](https://sqlite.new?dump=https%3A//raw.githubusercontent.com/datocms/nextjs-with-cache-tags-starter/main/schema.sql)

Copy the database URL and use it to fill the `TURSO_DATABASE_URL` environment variable.

Also create a token for the database, with no expiration and read/write permissions: copy and use it for the `TURSO_AUTH_TOKEN` variable.

We selected Turso because it's an incredibly cost-effective solution and is compatible with any hosting service. However, any other storage solution would be just as effective due to the simplicity of the saved data. 

### Step 3: Install dependencies and download the DatoCMS GraphQL Schema

Simply run `npm install` (or the equivalent command for your package manager of choice): a `schema.graphql` will be generated.

### Step 4: Run development server

Now you can run the development server:

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

## Deployment

The project works well on both Vercel and Netlify. Just make sure to set the environment variables as previously described.

**Important:** Once the site is deployed and you know the final domain, you need to set up a ["Invalidate cache tag" webhook](https://www.datocms.com/docs/content-delivery-api/cache-tags#step-3-implement-the-invalidate-cache-tag-webhook) on your DatoCMS project, configured like this:

<img src="https://github.com/datocms/nextjs-with-cache-tags-starter/raw/main/images/webhook.png" width="500" />

## Useful resources to navigate the code

This repository aims to demonstrate how you can selectively invalidate the Next.js cache when receiving DatoCMS Cache Tags invalidation events. Before exploring this repository, it's very useful to understand the general concepts behind Cache Tags:

- DatoCMS Cache Tags: [Announcement](https://www.datocms.com/blog/introducing-datocms-cache-tags), [Documentation](https://www.datocms.com/docs/content-delivery-api/cache-tags)
- Cache Tags with Next.js: [Guide](https://www.datocms.com/docs/next-js/using-cache-tags)

The code essentially consists of two parts that are strongly interdependent:

### Execution of GraphQL queries

The [`executeQuery()` function](https://github.com/datocms/nextjs-with-cache-tags-starter/blob/main/lib/fetch-content.ts#L20) is responsible for executing a GraphQL query using the DatoCMS Content Delivery API and caching the result. To support cache invalidation, the request is tagged with a unique identifier in the Next.js Data Cache.

The mapping between the unique identifier of the query, and the DatoCMS Cache Tags returned in the response is stored in Turso. We use a [simple table](https://github.com/datocms/nextjs-with-cache-tags-starter/blob/main/schema.sql) made up of just two columns:

* `query_id` (TEXT): A unique identifier for the query, used to tag the request;
* `cache_tag` (TEXT): The actual cache tag returned by the query.

You can switch the storage option from Turso to other options by adjusting the code in [`lib/database.ts`](https://github.com/datocms/nextjs-with-cache-tags-starter/blob/main/lib/database.ts).

### "Cache Tags Invalidation" webhook

The [route handler `/api/invalidate-cache-tags`](https://github.com/datocms/nextjs-with-cache-tags-starter/blob/main/app/api/invalidate-cache-tags/route.ts) receives ["Cache Tag Invalidation" events from a DatoCMS webhook](https://www.datocms.com/docs/content-delivery-api/cache-tags#step-3-implement-the-invalidate-cache-tag-webhook) and is responsible for invalidating every cached GraphQL query that is linked to those tags.

Since the `executeQuery()`:

- Tags each GraphQL request with a unique ID in the Next.js Data Cache, and
- Saves the "Query ID <-> Cache Tags" mapping on a Turso database...

The endpoint can find in the database the query IDs associated with the received tags, and use `revalidateTag()` to invalidate the relevant requests.

<!--datocms-autoinclude-footer start-->

# What is DatoCMS?

<a href="https://www.datocms.com/"><img src="https://www.datocms.com/images/full_logo.svg" height="60"></a>

[DatoCMS](https://www.datocms.com/) is the REST & GraphQL Headless CMS for the modern web.

Trusted by over 25,000 enterprise businesses, agency partners, and individuals across the world, DatoCMS users create online content at scale from a central hub and distribute it via API. We ❤️ our [developers](https://www.datocms.com/team/best-cms-for-developers), [content editors](https://www.datocms.com/team/content-creators) and [marketers](https://www.datocms.com/team/cms-digital-marketing)!

**Quick links:**

- ⚡️ Get started with a [free DatoCMS account](https://dashboard.datocms.com/signup)
- 🔖 Go through the [docs](https://www.datocms.com/docs)
- ⚙️ Get [support from us and the community](https://community.datocms.com/)
- 🆕 Stay up to date on new features and fixes on the [changelog](https://www.datocms.com/product-updates)

**Our featured repos:**

- [datocms/react-datocms](https://github.com/datocms/react-datocms): React helper components for images, Structured Text rendering, and more
- [datocms/js-rest-api-clients](https://github.com/datocms/js-rest-api-clients): Node and browser JavaScript clients for updating and administering your content. For frontend fetches, we recommend using our [GraphQL Content Delivery API](https://www.datocms.com/docs/content-delivery-api) instead.
- [datocms/cli](https://github.com/datocms/cli): Command-line interface that includes our [Contentful importer](https://github.com/datocms/cli/tree/main/packages/cli-plugin-contentful) and [Wordpress importer](https://github.com/datocms/cli/tree/main/packages/cli-plugin-wordpress)
- [datocms/plugins](https://github.com/datocms/plugins): Example plugins we've made that extend the editor/admin dashboard
- [datocms/gatsby-source-datocms](https://github.com/datocms/gatsby-source-datocms): Our Gatsby source plugin to pull data from DatoCMS
- Frontend examples in different frameworks: [Next.js](https://github.com/datocms/nextjs-demo), [Vue](https://github.com/datocms/vue-datocms) and [Nuxt](https://github.com/datocms/nuxtjs-demo), [Svelte](https://github.com/datocms/datocms-svelte) and [SvelteKit](https://github.com/datocms/sveltekit-demo), [Astro](https://github.com/datocms/datocms-astro-blog-demo), [Remix](https://github.com/datocms/remix-example). See [all our starter templates](https://www.datocms.com/marketplace/starters).

Or see [all our public repos](https://github.com/orgs/datocms/repositories?q=&type=public&language=&sort=stargazers)

<!--datocms-autoinclude-footer end-->
