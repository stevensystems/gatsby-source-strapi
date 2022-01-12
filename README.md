<p align="center">
  <a href="https://www.gatsbyjs.com">
    <img alt="Gatsby" src="https://www.gatsbyjs.com/Gatsby-Monogram.svg" width="60" />
  </a>
</p>
<h1 align="center">
  Strapi Source Plugin
</h1>

Source plugin for pulling documents into Gatsby-v4 from the Strapi-v4 graphql API.

- [x] Deep nesting and dynamic zones
- [x] Relationships with other collection types
- [x] Gatsby Cloud CMS previews & incremental builds
- [x] Build caching and only fetching changes since last build
- [x] Support for images in markdown fields

## 🚀 Installing the plugin

> This version of gatsby-source-strapi is only compatible with Strapi v4 and uses the graphql api of Strapi.


```shell
# Using Yarn
yarn add gatsby-source-strapi@relate-app/gatsby-source-strapi

# Or using NPM
npm install --save gatsby-source-strapi@relate-app/gatsby-source-strapi
```

## 🔥 Setting up the plugin

You can enable and configure this plugin in your gatsby-config.js file.

### Basic usage

```js
// In your gatsby-config.js
plugins: [
  {
    resolve: 'gatsby-source-strapi',
    options: {
      apiURL: 'http://localhost:1337',
      collectionTypes: ['Article', 'User'],
      singleTypes: ['Home Page', 'Contact'],
      // Extract images from markdown fields.
      markdownImages: {
        typesToParse: {
          Article: ['body'],
          ComponentBlockBody: ['text'],
        },
      },
      // Only include specific locale.
      locale: 'en', // default to all
      // Include drafts in build.
      preview: true, // defaults to false
      // Use application token.
      token: 'xxx',
      // Add additional headers.
      headers: {},
    },
  },
];
```

### Internationalization support

Strapi now supports internationalization. But by default, this plugin will only fetch data in the default locale of your Strapi app. If your content types are available in different locales, you can also pass an entity definition object to specify the locale you want to fetch for a content type. Use the all value to get all available locales on a collection type.

### Relationship support

Relationships to other collections both in collection fields and components are automatically connected to each other and included as a field if the node being linked to is included in the collectionTypes in your gatsby-config.

### Draft content

Strapi now supports Draft and publish, which allows you to save your content as a draft and publish it later. By default, this plugin will only fetch the published content.

But you may want to fetch unpublished content in Gatsby as well. To do so, find a content type that has draft & publish enabled, and add an entity definition object to your config. Then, set preview to true in gatsby-config.

### Authenticated requests

Strapi's Roles & Permissions plugin allows you to protect your API actions. If you need to access a route that is only available to a logged in user, you can provide your credentials (token) so that this plugin can access to the protected data.

### Refresh content locally

Set ENABLE_GATSBY_REFRESH_ENDPOINT=true in your node env for gatsby, then you can add the a webhook in Strapi pointed towards http://localhost:8080/__refresh and see content update when saving it locally.

See more about this feature here:
https://www.gatsbyjs.com/docs/refreshing-content/

## Querying data

You can query Document nodes created from your Strapi API like the following:

```graphql
{
  allStrapiArticle {
    edges {
      node {
        id
        title
        content
      }
    }
  }
}
```

You can query Document nodes in a chosen language:

```graphql
{
  allStrapiArticle(filter: { locale: { eq: "en" } }) {
    edges {
      node {
        id
        title
        content
      }
    }
  }
}
```

To query images you can do the following:

```graphql
{
  allStrapiArticle {
    edges {
      node {
        id
        singleImage {
          file {
            publicURL
          }
        }
        multipleImages {
          file {
            publicURL
          }
        }
      }
    }
  }
}
```

To query markdown images for a markdown field named "text" you can do the following:

> To replace in markdown use the base returned from the file and create a custom renderer to match the url on the image with the base.

```graphql
{
  allStrapiArticle {
    edges {
      node {
        id
        text
        text_images {
          base
          publicURL
        }
      }
    }
  }
}
```

This version also adds support for union types (most commonly used on dynamic component fields).
All types in the gatsby schema gets prepended with the Strapi namespace.
To query for dynamic field types you can do the following:

```graphql
{
  allStrapiArticle {
    edges {
      node {
        id
        blocks {
          ... on StrapiComponentBody {
            body
          }
          ... on StrapiComponentImage {
            image {
              file {
                publicURL
              }
            }
          }
        }
      }
    }
  }
}
```