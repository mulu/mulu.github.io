---
layout: docs
title: API Overview
permalink: /
---

## Overview

The Mulu API identifies mentions of products sold by online retailers such as Amazon, EBay,
Overstock, Popsugar, and iTunes in a published document such as a web site or social media post.

The Mulu API is an HTTP-based API, which means that it is a language-neutral API that you can
integrate with your application. You can also test your queries easily in any web browser as
standard HTTP URLs.

The Mulu API returns product mentions as a [JSON](http://tools.ietf.org/html/rfc7159) array.  Each
array element is a JSON object that represents a product mention. If multiple offer feeds sell
the same product, the results include an array element for every product mention. In other words,
every product mention element pairs one instance of published document with an offer from an offer
feed. If a query identifies no product mentions, the API returns a JSON array with zero items.

This API reference uses the following definitions:

* _offer_ - an opportunity to purchase a product from a retailer
* _offer feed_ - a source of offers from a retailer
* _publisher_ - a web site, social media site, or anyone who publishes text
* _published document_ - a web page or other document with a unique URL
* _product mention_ - a mention of a product in published document


#### Scores <a name="scores"></a>

The Mulu API returns several associated scores for each product mention. A higher score
represents evidence that the product reference really represents the product
represented by a specific offer from a specific offer feed. In the Mulu API, all scores
are floating point numbers from 0 to 1 with 1 being the highest possible score.

When searching for product mentions, optionally specify a _minimum lexical similarity score_ for
results. Depending on your situation, you might want to search for product mentions with a high
minimum score or a low minimum score. For more results, set the minimum score close to 0.

Product mention results also provide additional score fields for advanced use. The following table
compares the score fields in results.

| Result Field | Description |
|:---+:---|
| _lexical&nbsp;similarity_ |  Similarity between the words in the product name and the words detected in the published document, as determined by the [Dice coefficient](https://en.wikipedia.org/wiki/S%C3%B8rensen%E2%80%93Dice_coefficient) algorithm. Think of this as the lexical closeness of the product name.
| _product&nbsp;brand&nbsp;confidence_ |  Score based on (1) the presence of the brand name for the product in the sentence and (2) the likelihood that the brand would appear in normal language.
| _product&nbsp;category&nbsp;score_ |  Score based on the product category similarity with other product mentions in the same published document. For example, suppose Mulu detects that a web page mentions 10 bicycles and 1 song name. The product mentions for the bicycles have very high category scores because that is the most common product category on the page. The product mention for the song name would have a low category score, which could indicate a false positive for the mention.
| _product&nbsp;type&nbsp;score_ |  Score based on the generic product type appearing in the published document. The generic product type is the product name with brand names removed.


## Searching for Product Mentions

To search for product mentions, submit an
[HTTP&nbsp;GET](http://tools.ietf.org/html/rfc2616#section-9.3) request to the server `api.mulu.me`
at the path `/2/mention/search`. Add query parameters to specify your account API key and one or
more search criteria.

For basic testing in a browser, format Mulu API searches as standard HTTP URLs:

    https://api.mulu.me/2/mention/search?apiKey=APIKEY&documentUrlPrefix=http://www.example.com/

Parameter values typically contain characters not permitted in query parameters, such as
ampersand, spaces, and other special characters. You must
[URL&nbsp;encode](http://tools.ietf.org/html/rfc3986#section-2.1) all query parameters.


### Query Parameters

Provide the `apiKey` parameter and at least one other search parameter. All field types are text
values unless otherwise specified.


| Name | Required?&nbsp;&nbsp; | Description |
|:---+:---+:---|
| `apiKey` | Required | Your Mulu Account API key. To set up a new Mulu account, [contact us](http://mulu.me/contact). |
| `documentUrlPrefix` | Provide at least one of `documentUrlPrefix` or `offer` | The prefix (first characters) of published document URLs that you want to select. For HTTP URLs, the URL prefix must begin with `http://`. |
| `offer` | Provide at least one of `documentUrlPrefix` or `offer` | A unique ID for an offer as specified by its offer feed. For example, if the offer came from Amazon, specify an Amazon Standard Identification Number ([ASIN](https://en.wikipedia.org/wiki/Amazon_Standard_Identification_Number)). To search for multiple products, provide the `offer` query parameter multiple times. |
| `minModifiedDate` | Required if you specify `maxModifiedDate` | Date/time in [ISO8601](http://en.wikipedia.org/wiki/ISO_8601) format. Specify this parameter to limit results to only product mentions found on or after this date. You must include the time zone information. For example, `2014-02-19T00:00:00.0+00:00`. If you set the `maxModifiedDate` parameter, you must also set `minModifiedDate`.|
| `maxModifiedDate` | Optional | Date/time in [ISO8601](http://en.wikipedia.org/wiki/ISO_8601) format. Specify this parameter to limit results only to product mentions found on or before this date. You must include the time zone information. For example, `2014-02-19T00:00:00.0+00:00`. If you set the `maxModifiedDate` parameter, you must also set `minModifiedDate`. |
| `minLexicalSimilarityScore` | Optional | Floating point number from 0 to 1. Specify a minimum lexical similarity score value. To get more results from the Mulu API, specify a lower minimum value. If unspecified, the default is 0.8. |
| `minProductBrandScore` | Optional | Floating point number from 0 to 1. Specify a minimum product brand score value. To get more results from the Mulu API, specify a lower minimum value. If unspecified, the default is 0. |
| `minProductCategoryScore` | Optional | Floating point number from 0 to 1. Specify a minimum product category score value. To get more results from the Mulu API, specify a lower minimum value. If unspecified, the default is 0. |
| `minProductTypeScore` | Optional | Floating point number from 0 to 1. Specify a minimum product type score value. To get more results from the Mulu API, specify a lower minimum value. If unspecified, the default is 0. |
| `offerFeed` | Optional | Specify an offer feed ID to limit results to only a specific offer feed. Offer feed IDs are case-sensitive. For example, for offers from eBay, set to `EBAY`. To search multiple offer feeds, provide this query parameter multiple times. If `offerFeed` is unspecified, Mulu API searches the default set of offer feeds for your Mulu Account API key.  If you provide multiple offer feeds and also provide the `offer` parameter, the same `offer` ID could potentially be found in multiple offer feeds. In practice, duplicates are unlikely because offer feeds use different syntax for offer&nbsp;IDs. |
| `productCategory` | Optional | Specify a required product category as defined by the offer feed. For example, `Clothing` or `Beauty`. If unspecified, the results include all product categories. |


## Example Search Requests

The following are some example Mulu API requests. To make it easier to read, the example queries
are shown with newlines and without parameter URL encoding. For real queries you must use
[URL&nbsp;encoding](http://tools.ietf.org/html/rfc3986#section-2.1) for all query parameters.


#### URL Prefixes

Return product mentions for a specific publisher:

    GET /2/mention/search?apiKey=ExampleAPIKey
                         &documentUrlPrefix=http://www.example.com/

Return product mentions for a specific publisher but only for URLs under the `article` path:

    GET /2/mention/search?apiKey=ExampleAPIKey
                         &documentUrlPrefix=http://www.example.com/article/


#### Scores

Return product mentions for a specific publisher using default minimum lexical similarity score (0.8):

    GET /2/mention/search?apiKey=ExampleAPIKey
                         &documentUrlPrefix=http://www.example.com/

Return product mentions for a specific publisher using custom minimum lexical similarity score:

    GET /2/mention/search?apiKey=ExampleAPIKey
                         &documentUrlPrefix=http://www.example.com/
                         &minLexicalSimilarityScore=0.5


#### Product Categories

Return product mentions for a specific publisher where the products are in the `Cosmetics` category

    GET /2/mention/search?apiKey=ExampleAPIKey
                         &documentUrlPrefix=http://www.example.com/
                         &productCategory=Cosmetics

Return product mentions for a specific publisher for products in categories `Cosmetics` or `Clothing`

    GET /2/mention/search?apiKey=ExampleAPIKey
                         &documentUrlPrefix=http://www.example.com/
                         &productCategory=Cosmetics
                         &productCategory=Clothing


#### Offer IDs and Offer Feeds

To search for an offer, provide the offer ID as specified by its offer feed.

Return product mentions for the offer `SKU123` from _any_ product source:

    GET /2/mention/search?apiKey=ExampleAPIKey
                         &offer=SKU123

Note that in practice, the major retailers use different style offer IDs, so an offer ID tends to
appear in at most one offer feed. However, you can specify an offer feed.

Return product mentions for the offer `SKU123` from offer feed `MYSTORE`:

    GET /2/mention/search?apiKey=ExampleAPIKey
                         &offer=SKU123
                         &offerFeed=MYSTORE

Return product mentions for offers `SKU123` or `SKU456` from offer feed `MYSTORE`:

    GET /2/mention/search?apiKey=ExampleAPIKey
                         &offer=SKU123
                         &offer=SKU456
                         &offerFeed=MYSTORE

Return product mentions from a specific publisher for offer `SKU123` from offer feed `MYSTORE`

    GET /2/mention/search?apiKey=ExampleAPIKey
                         &documentUrlPrefix=http://www.example.com/
                         &offer=SKU123
                         &offerFeed=MYSTORE


#### Date Searches

Return product mentions for the offer `SKU123` from offer feed `MYSTORE` only if found
between midnight January 1, 2014 and July 1, 2014 in a specific time zone:

    GET /2/mention/search?apiKey=ExampleAPIKey
                         &offer=SKU123
                         &offerFeed=MYSTORE
                         &minModifiedDate=2014-01-01T00:00:00.000-07:00
                         &maxModifiedDate=2014-07-01T00:00:00.000-07:00


## Understanding the JSON Response

### Example Response

The following JSON describes an example product mention for a red stained glass lamp, sold by online
vendor LightingOnline, reviewed by magazine LampReviews.com.

    [
      {
        "sentence": "I love my red LampCo stained glass lamp!",
        "excerpt": "My place looks great! I love my red LampCo stained glass lamp! yay!",
        "lexicalSimilarityScore": 0.5,
        "productBrandScore": 1.0,
        "productCategoryScore": 0.6,
        "productTypeScore": 0.6,
        "document": {
          "documentUrl": "http://lampreviews.com/articles/best_red_lamps_of_2014",
          "title": "Best Red Lamps of 2014",
          "modifiedDate": "2014-01-11T00:00:00.000-07:00"
        },
        "offer": {
          "offerUrl": "http://lightingonline.com/products/red_stained_glass_lampco",
          "offerFeed": "lightingonline",
          "offerId": "lightingonline-124455",
          "currency": "USD",
          "price": 100,
          "productBrand": "LampCo",
          "productCategory": "Housewares",
          "productDescription": "A red stained glass lamp from LampCo.",
          "productImageUrl": "http://lightingonline.com/images/red_lampco_stainedglass.png",
          "productName": "LampCo Red Lamp"
        },
      },
      ...
    ]


#### Product Mention JSON

The following table describes the format of a product mention JSON object.  The result of a query is
a JSON array of product mention JSON objects.  All field types are strings unless otherwise
specified. All of the fields in a product mention JSON object are always populated.

| Field | Description |
|:---+:---+:---|
| `sentence` | The published document sentence that mentions the product. |
| `excerpt` | The published document excerpt that mentions the product. The excerpt consists of several sentences before and after the product mention. This is the longest data field with text from the published document. |
| `lexicalSimilarityScore` | _Floating point number from 0 to 1._  Similarity of (1) the product name and (2) text in the published document. See [Scores](#scores).|
| `productCategoryScore` | _Floating point number from 0 to 1._  Product category score. See [Scores](#scores). |
| `productBrandScore` | _Floating point number from 0 to 1._  Product brand score. See [Scores](#scores). |
| `productTypeScore` | _Floating point number from 0 to 1._  Product type score. See [Scores](#scores). |
| `document` | _JSON object_. The JSON representation of the document that mentions the product. See [Document&nbsp;JSON](#documentfields). |
| `offer` | _JSON object_. The JSON representation of the product offer. See [Offer&nbsp;JSON](#offerfields). |


#### Document JSON<a name="documentfields"></a>

The following table describes the format of a document JSON object, which is included in every
product mention JSON object. All field types are text values.

| Field | Description |
|:---+:---|
| `documentUrl` | The complete URL of the published document where Mulu found the product mention, including the URI scheme. |
| `title` | Title of the published document. |
| `modifiedDate` | _Date in [ISO8601](http://en.wikipedia.org/wiki/ISO_8601) format_. The most recent date Mulu found this product mention. |

For example:

        {
          "documentUrl": "http://lampreviews.com/articles/best_red_lamps_of_2014",
          "title": "Best Red Lamps of 2014",
          "modifiedDate": "2014-01-11T00:00:00.000-07:00"
        },


#### Offer JSON<a name="offerfields"> </a>

The following table describes the format of an offer JSON object, which is included in every product
mention JSON object. All field types are text values unless otherwise specified. All values are provided
and formatted by the offer feed data unless otherwise specified as provided by Mulu. All fields
always exist as JSON fields but some may be unpopulated (see second column for details).

| Field | Always populated | Description |
|:---+:---+:---|
| `offerUrl` | yes | A URL to purchase the product. This URL is not necessarily on the same domain as the offer feed. |
| `offerFeed` | yes |  Offer feed ID, as determined by Mulu. For example, for eBay, this is `EBAY`. |
| `offerId` | typically, but not always (see&nbsp;note) | An offer's unique identifier assigned by the offer feed. For example, an Amazon Standard Identification Number ([ASIN](https://en.wikipedia.org/wiki/Amazon_Standard_Identification_Number)). Note: For nearly all cases, `offerId` is populated but in rare cases a retailer includes an `offerUrl` but no `offerId`. |
| `currency` | no | Currency code in [ISO&nbsp;4217](https://en.wikipedia.org/wiki/ISO_4217) format for the value in the `price` field. For example, for U.S. Dollars the value is `USD`. |
| `price` | no | An integer calculated by rounding up the price to the unit currency denomination. For example, if the offer feed provided a price of "$49.99", this field contains the value 50. |
| `productBrand` | no | The product’s brand or manufacturer. For example, `Sony`. |
| `productCategory` | no |  Product category, for example `Clothing`.  |
| `productDescription` | no | A description of the product, which may be in plain text or in HTML. |
| `productImageUrl` | no |  URL for the product image. |
| `productName` | yes | Name of the product. |

For example:

        {
          "offerUrl": "http://lightingonline.com/products/red_stained_glass_lampco",
          "offerFeed": "lightingonline",
          "offerId": "lightingonline-124455",
          "currency": "USD",
          "price": 100,
          "productBrand": "LampCo",
          "productCategory": "Housewares",
          "productDescription": "A red stained glass lamp from LampCo.",
          "productImageUrl": "http://lightingonline.com/images/red_lampco_stainedglass.png",
          "productName": "LampCo Red Lamp"
        }
