[![Build Status](https://travis-ci.org/lotas/contentful-graph.svg?branch=master)](https://travis-ci.org/lotas/contentful-graph)

# contentful-graph
Visual representation of contentful content models in form of graphs

Can be exported with fields:

![alt](./samples/model.png)

or without:

![alt](./samples/model-nofields.png)

# Requirements

## graphviz

If you plan to render the graph as image or PDF, make sure you have `graphviz` installed on your system.

## node.js

The library is using async/await functions, so you need the latest `7` or `8` versions.

# Installation

Can be installed globally:

`npm i -g contentful-graph`

Or locally

`npm i --save-dev contentful-graph`

# Running import

There are several ways of running the import:
1. running CLI `bin/contentful-graph` command
2. import API and invoking functions programmatically

For this purpose, you'll need to know your contentful `spaceId` and `token`

Token may come in two flavors: Delivery token or Management token

**Management token** has an advantage of showing one-to-one relationships for models (as pointed by @jelz in #1)

## Command-line version

When running the command line version, make sure you export following variables:

```
CONTENTFUL_SPACE_ID= spaceId
CONTENTFUL_TOKEN= either deliveryToken
CONTENTFUL_MANAGEMENT_TOKEN= or management token
```

or simply in command-line:

`CONTENTFUL_SPACE_ID=123 CONTENTFUL_MANAGEMENT_TOKEN=token ./bin/contentful-graph`


## Api version

Package exposes following functions:

`getContentTypesFromManagementApi(spaceId: string, managementToken: string): Promise<Object>`

Will import content types from contentful using management api

`getContentTypesFromDistributionApi(spaceId: string, apiToken: string): Promise<Object>`

Will import content types from contentful using distributions api

`contentTypesToModelMap(contentTypes: Object): Object`

Will enrich existing content types with mapping information (one-to-one, one-to-many)

`modelsMapToDot(modelsMap: Object, showEntityFields: Boolean): String`

Will create dot graph definition out of the model map


You can use them as follows:

```
const convertApi = require('contentful-graph');

// either with managementToken
const contentTypes = await convertApi.getContentTypesFromManagementApi(spaceId, managementToken);
// or with apiToken, you just need one of the calls
const contentTypes = await convertApi.getContentTypesFromDistributionApi(spaceId, apiToken);

// enrich content types with relationship definitions
const modelsMap = convertApi.contentTypesToModelMap(contentTypes);

// generate dot string that can be passed to graphviz
const dotStr = convertApi.modelsMapToDot(modelsMap, true);
```

# Generating graphs

After you would run the import, you should have something like this:

```
digraph obj {
  node[shape=record];

  Image [label="{Image |        | <title> Title|<photo> Photo|<imageCaption> Image caption|<imageCredits> Image credits|<categoryId> CategoryId}" shape=Mrecord];
  Asset
  Author [label="{Author |        | <name> Name|<twitterHandle> Twitter handle|<profilePhoto> Profile photo|<biography> Biography|<createdEntries> Created entries}" shape=Mrecord];
  Category [label="{Category |        | <categoryId> Category Id|<categoryName> Category name}" shape=Mrecord];
  PhotoGallery [label="{Photo Gallery |        | <title> Title|<slug> Slug|<author> Author|<coverImage> Cover Image|<description> Description|<images> Images|<tags> Tags|<date> Date|<location> Location}" shape=Mrecord];

  Image:photo -> Asset [dir=forward];
  Image:categoryId -> Category [dir=forward];
  Author:profilePhoto -> Asset [dir=forward];
  Author:createdEntries -> Image [dir=forward,label="1..*"];
  PhotoGallery:author -> Author [dir=forward];
  PhotoGallery:coverImage -> Asset [dir=forward];
  PhotoGallery:images -> Image [dir=forward,label="1..*"];
}
```

To render this to image or PDF, pipe it to the `dot` command:

```
./import.js | dot -Tpng > model.png
./import.js | dot -Tpdf > model.pdf
```

Where `./import` is the script that produces `dot` output


## Links

[Contentful Management API](https://www.contentful.com/developers/docs/references/content-management-api/#/reference/content-types)

[Contentful Delivery API](https://www.contentful.com/developers/docs/references/content-delivery-api/#/reference/content-types)

[GraphViz Fiddle](http://stamm-wilbrandt.de/GraphvizFiddle/)
