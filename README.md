<div align="center">
  <img style="width: 160px; height: auto;" src="public/logo.png" alt="Logo for Strapi preview button plugin" />
  <h1>Strapi Preview Button</h1>
  <p>A plugin for Strapi CMS that adds a preview button and live view button to the content manager edit view.</p>
  <img style="width: 960px; height: auto;" src="public/screenshot.png" alt="Screenshot for Strapi preview button plugin" />
</div>

## Get Started

- [Features](#features)
- [Installation](#installation)
- [Requirements](#requirements)
- [Configuration](#configuration)
- [User Guide](#user-guide)

## <a id="features"></a>✨ Features

- Adds a new button in content manager sidebar which links the user to a preview or live view of a frontend app view.
- Customize which content types should use the preview button.
- Customize endpoints for draft and published URLs.
- Supports collection and single types.

## <a id="installation"></a>💎 Installation

```bash
yarn add strapi-plugin-qrcode-generator@latest
```

## <a id="requirements"></a>🚨 Requirements

Include the following variables in your application's `.env` file.

```
STRAPI_PREVIEW_SECRET=YOURSECRET
STRAPI_PREVIEW_DRAFT_URL=https://example.com/api/preview
STRAPI_BASE_FRONTEND=https://example.com
```

You must generate your own secret key to use for `STRAPI_PREVIEW_SECRET` which will also be given to the frontend app later.

#### Draft and publish mode

With `draftAndPublish` mode enabled for a content type, a preview button will render when the entry is in a draft state while a live view button will render when it is in a published state.

It is **not** required to enable `draftAndPublish` for content types using this plugin. The live view button will still display to conveniently redirect a user to the live version of the page.

## <a id="configuration"></a>🔧 Configuration

| property     | type (default) | description                                                                       |
| ------------ | -------------- | --------------------------------------------------------------------------------- |
| contentTypes | array (`[]`)   | An array of objects describing which content types should use the preview button. |

### `contentTypes`

An array of objects describing which content types should use the preview button.

Each object in the array requires a `uid` prop at minimum. If you are configuring a collection type, a `targetField` is also required but it is optional with single types. The field name "slug" is recommended for the `targetField` value because it represents the unique part of the URL path, but it is not required.

#### Example for Collection type

Consider we have `Page` and `Post` content types, where each has a `uid` field named `slug` and entries created for each, with the slug values set to `my-page` and `my-post`. Here is the minimum required config.

```js
module.exports = {
  'qrcode-generator': {
    config: {
      contentTypes: [
        {
          uid: 'api::page.page',
          targetField: 'slug',
        },
        {
          uid: 'api::post.post',
          targetField: 'slug',
        },
      ],
    },
  },
};
```

In this example, our pages and posts will be routed differently in our frontend app. To help with this, props like `basePath` and `query` may be configured for variations between `draft` and `published` URLs.

```js
module.exports = {
  'qrcode-generator': {
    config: {
      contentTypes: [
        {
          uid: 'api::page.page',
          targetField: 'slug',
        },
        {
          uid: 'api::post.post',
          targetField: 'slug',
          draft: {
            query: {
              type: 'post',
            },
          },
          published: {
            basePath: 'blog',
          },
        },
      ],
    },
  },
};
```

This configuration will result in the following preview URLs for `Page` and `Post`. Notice in the URLs below how env vars and config settings work together to build the final URLs.

```
// Draft URLs
https://example.com/api/preview?slug=my-page&secret=YOURSECRET
https://example.com/api/preview?slug=my-post&type=post&secret=YOURSECRET

// Published URLs
https://example.com/my-page
https://example.com/blog/my-post
```

> The `query` prop can actually take any params you want to pass to your frontend app.

#### Example with different `targetField` values

Depending on how your preview method works in your frontend app, you may not always want to use the same `targetField` for both draft and published URLs.

In the example below you can see how the configuration for `Post` moved the `targetField` prop to the `draft` and `published` props, which will use the `id` instead of the `slug` for draft URLs.

```js
module.exports = {
  'qrcode-generator': {
    config: {
      contentTypes: [
        {
          uid: 'api::page.page',
          targetField: 'slug',
        },
        {
          uid: 'api::post.post',
          draft: {
            query: {
              type: 'post',
            },
            targetField: 'id',
          },
          published: {
            basePath: 'blog',
            targetField: 'slug',
          },
        },
      ],
    },
  },
};
```

```
// Draft URLs
https://example.com/api/preview?slug=my-page&secret=YOURSECRET
https://example.com/api/preview?id=my-post&type=post&secret=YOURSECRET

// Published URLs
https://example.com/my-page
https://example.com/blog/my-post
```

#### Example for Single type

Consider we have an `About` single type to serve as our "About" page. Unlike collection types, we don't necessarily need an editable `targetField` with our model if we know it will always be `/about`. You can still provide the `targetField` if you want to keep this value editable in Strapi.

```js
module.exports = {
  'qrcode-generator': {
    config: {
      contentTypes: [
        {
          uid: 'api::about.about',
          draft: {
            query: {
              slug: 'about',
            },
          },
          published: {
            basePath: 'about',
          },
        },
      ],
    },
  },
};
```

This configuration will result in the following preview URLs for `About`.

```
// Draft URL
https://example.com/api/preview?slug=aboute&secret=YOURSECRET

// Published URL
https://example.com/about
```

## <a id="user-guide"></a>📘 User Guide

### How does this work with my frontend app?

The **Open live view** button will lead directly to the live page URL.

The **Open draft preview** button should lead to an endpoint that redirects to the appropriate preview page based on the query parameters passed to it.

Before granting access to the preview, the values for `STRAPI_PREVIEW_SECRET` should be compared and validated between both Strapi and the frontend app.

For in-depth examples and instructions, please reference the links below to learn how this can be accomplished with Next.js and Strapi.

- [Next.js Preview Mode](https://nextjs.org/docs/advanced-features/preview-mode)
- [Next.js Preview Mode example with Strapi](https://github.com/vercel/next.js/tree/canary/examples/cms-strapi)
# strapi-plugin-qrcode-generator
