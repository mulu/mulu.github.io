---
layout: docs
title: Api Overview
permalink: /
---
 
## Overview

The Mulu API identifies mentions of products sold by online retailers such as Amazon, EBay,
Overstock, Popsugar, and iTunes in a published document such as a web site or social media post.

The Mulu API is an HTTP-based API, which means that it is a language-neutral API that you can
integrate with your application. You can also test your queries easily in any web browser as
standard HTTP URLs.

The Mulu API returns product matches as a [JSON](http://tools.ietf.org/html/rfc7159) array.  Each
array element is a JSON object that represents a product match. If multiple product sources sell
the same product, the results include an array element for every product match. In other words,
every product match element pairs one instance of published document with an offer from a product
source. If a query identifies no product matches, the API returns a JSON array with zero items.

This API reference uses the following definitions:

* _product source_ - an online retailer 
* _offer_ - a product's unique product or offer ID as specified by a product source 
* _publisher_ - a web site, social media site, or anyone who publishes text
* _published document_ - one web page or other document with a unique URL
* _product match_ - a mention of a product in published document

####<a name="confidencelevels"></a>Confidence Levels

The Mulu API returns an associated confidence level for each product match. A higher confidence
level represents strong confidence that the product reference really represents the product
represented by a specific offer ID from a specific product source. In the Mulu API, all confidence
levels are floating point numbers from 0 to 1 with 1 being the highest possible score.

When searching for product matches, optionally specify a _minimum confidence level_ for results.
Depending on your situation, you might want to search for product matches with a high minimum
confidence level or a low minimum confidence level. For more results, set the minimum confidence
level close to 0.

As you process search results, parse each product match to examine its _aggregate confidence
level_, which is the overall confidence of the match. Product Match results also provide additional
confidence level fields for advanced use. The following table compares the confidence level fields
in results.

| Result Field | Description |
|:---+:---|
| _aggregate&nbsp;confidence_ | Overall confidence of the match. This field corresponds to the _minimum&nbsp;confidence&nbsp;level_ in your search query. This is the main confidence level result field. All other confidence fields are for advanced use. Mulu uses the other confidence fields to calculate the overall confidence of the match.
| _Dice&nbsp;index_ |  Confidence of similarity between the set of words in the Product Name and the set of words detected in on the published page, as determined by the [Dice index](https://en.wikipedia.org/wiki/S%C3%B8rensen%E2%80%93Dice_coefficient) algorithm. Think of this as the lexical confidence of the product name. 
| _category&nbsp;confidence_ | Confidence of the match based on the product category similarity with other matches on the same page of published document determined by URL. For example, suppose Mulu detects that a web page mentions 10 bicycles and 1 song name. The Product Matches for all bicycles have very high category confidence because that is the most common product category on the page. The Product Matches for the song name would be a low category confidence, which could indicate a false positive for the match.
| _brand&nbsp;confidence_ | Confidence of the match based on (1) the presence of the brand name for the product in the sentence and (2) the likelihood that the brand would appear in normal language.


## Searching for Product Matches

To search for product matches, submit an
[HTTP&nbsp;GET](http://tools.ietf.org/html/rfc2616#section-9.3) request to the server `api.mulu.me`
and request the `/v1/matches` resource. The operation `/v1/matches` is the only API operation in
the Mulu&nbsp;API&nbsp;v1. Add query string parameters for your account API key and one or more
search criteria.

For basic testing in a browser, format Mulu API searches as standard HTTP URLs:

     http://api.mulu.me/v1/matches?apiKey=APIKEY&urlPrefix=http://www.magazine.com/

Parameter values typically contain characters not permitted in query string parameters, such as
ampersand, spaces, and other special characters. It is a critical requirement that you must
[URL&nbsp;encode](http://tools.ietf.org/html/rfc3986#section-2.1) all query string parameters. 

###<a name="requestParams"></a>Request Parameters

Provide the `apiKey` parameter and at least one other search parameter. All field types are text
values unless otherwise specified.


| Name | Required?&nbsp;&nbsp; | Description |
|:---+:---+:---|
| `apiKey` | Required | Your Mulu Account API key. To set up a new Mulu account, [contact us](http://mulu.me/contact). |
| `urlPrefix` | Provide at least one of `urlPrefix` or `offer` | The prefix (first characters) of publisher URLs that you want to match. For HTTP URLs, the URL prefix text must begin with `http://`. The prefix must be on a directory (slash) boundary. If a slash character is not the last character of the `urlPrefix`, a final slash is implicit. For example, to match the URL `http://magazine.com/category/mypage`, pass the `urlPrefix` value `http://magazine.com/category` or `http://magazine.com/category/`. In contrast, the URL prefix `http://magazine.com/categ` does _not_ match that URL. |
| `offer` | Provide at least one of `urlPrefix` or `offer` | A unique ID for one product or offer as specified by its product source. For example, specify an Amazon Standard Identification Number ([ASIN](https://en.wikipedia.org/wiki/Amazon_Standard_Identification_Number)) . To search for multiple products, provide the `offer` parameter multiple times in your search query string. |
| `crawledAfter` | Required if you specify `crawledBefore` | Date/time in [ISO8601](http://en.wikipedia.org/wiki/ISO_8601) format. Specify this parameter to limit results only to matches found on or after this date. You must include the time zone information. For example, `2014-02-19T00:00:00.0+00:00`. If you set the `crawledBefore` parameter, you must also set `crawledAfter`.|
| `crawledBefore` | Optional | Date/time in [ISO8601](http://en.wikipedia.org/wiki/ISO_8601) format. Specify this parameter to limit results only to matches found on or before this date. You must include the time zone information. For example, `2014-02-19T00:00:00.0+00:00`. If you set the `crawledBefore` parameter, you must also set `crawledAfter`. |
| `minimumBrandConfidence` | Optional | Floating point number from 0 to 1. Specifiy a minimum brand confidence value for any matches. To get more results from the Mulu API, specify a lower `minimumBrandConfidence` value. If unspecified, the default is 0, which means 0% confidence of a match. |
| `minimumTypeConfidence` | Optional | Floating point number from 0 to 1. Specifiy a minimum product type confidence value for any matches. To get more results from the Mulu API, specify a lower `minimumTypeConfidence` value. If unspecified, the default is 0, which means 0% confidence of a match. |
| `minimumConfidence` | Optional | Floating point number from 0 to 1. Specifiy a minimum confidence value for any matches. To get more results from the Mulu API, specify a lower `minimumConfidence` value. If unspecified, the default is 0.8, which means 80% confidence of a match. This corresponds to the _aggregate confidence level_ field in Match JSON. See [Confidence&nbsp;Levels](#confidencelevels). |
| `source` | Optional | Specify a Mulu product source ID to limit results only a specific product source name. Source names are case-sensitive. For example, for Amazon results set to `AMAZON`. To search multiple sources, provide the `source` parameter multiple times in your search query string. If `source` is unspecified, Mulu API searches the default set of product sources for your Mulu Account API key.  If you provide multiple sources and also provide the `offer` parameter, the `offer` ID could potentially match results from multiple sources. In practice, duplicates are unlikely because product sources use different syntax for offer&nbsp;IDs. |
| `category` | Optional | Specify a required product category as defined by the product source. For example, `Clothing` or `Beauty`. If unspecified, matches all product categories. |


## Example Search Requests

The following are some example Mulu API requests. To make it easier to read examples, demo queries
are shown with newlines and without parameter URL encoding. For real queries you must use
[URL&nbsp;encoding](http://tools.ietf.org/html/rfc3986#section-2.1) for all query string
parameters.


#### URL Prefixes

Return matches for a specific publisher:

    GET v1/matches?apiKey=DemoAPIKey
                  &urlPrefix=http://www.magazine.com

Return matches for a specific publisher but only for URLs in the `article` folder:

    GET v1/matches?apiKey=DemoAPIKey
                  &urlPrefix=http://www.magazine.com/article
                  
Important: The URL prefix must be a directory (slash) boundary. See [Request&nbsp;Parameters](#requestParams).
                  
#### Confidence Levels

Return matches for a specific publisher using default minimum confidence level (80%):

    GET v1/matches?apiKey=DemoAPIKey
                  &urlPrefix=http://www.magazine.com/article

Return matches for a specific publisher using custom minimum confidence level (50%):

    GET v1/matches?apiKey=DemoAPIKey
                  &urlPrefix=http://www.magazine.com/article
                  &minimumConfidence=0.5

#### Categories

Return matches for a specific publisher where the products are in the `Cosmetics` category

    GET v1/matches?apiKey=DemoAPIKey
                  &urlPrefix=http://www.magazine.com/article
                  &category=Cosmetics

Return matches for a specific publisher for products in categories `Cosmetics` or `Clothing`

    GET v1/matches?apiKey=DemoAPIKey
                  &urlPrefix=http://www.magazine.com/article
                  &category=Cosmetics
                  &category=Clothing

#### Offer IDs and Product Sources

To search for a product, provide the offer ID as specified by its product source. 

Return matches for the product offer `SKU123` from _any_ product source:

    GET v1/matches?apiKey=DemoAPIKey
                  &offer=SKU123
                  
Note that in practice, the major retailers use different style offer IDs, so one offer ID tends to
only match one store. However, you can specify a specific product source.

Return matches for the product offer `SKU123` from product source `MYSTORE`:

    GET v1/matches?apiKey=DemoAPIKey
                  &offer=SKU123
                  &source=MYSTORE

Return matches for products `SKU123` or `SKU456` from product source `MYSTORE`:

    GET v1/matches?apiKey=DemoAPIKey
                  &offer=SKU123
                  &offer=SKU456
                  &source=MYSTORE

Return matches from a specific publisher for product `SKU123` from product source `MYSTORE`

    GET v1/matches?apiKey=DemoAPIKey
                  &offer=SKU123
                  &urlPrefix=http://www.magazine.com/
                  &source=MYSTORE

#### Date Searches

Return matches for the product offer `SKU123` from product source `MYSTORE` only if confirmed
between midnight January 1, 2014 and July 1, 2014 in a specific time zone:

    GET v1/matches?apiKey=DemoAPIKey
                  &offer=SKU123
                  &source=MYSTORE
                  &crawledAfter=2014-01-01T00:00:00.000-07:00
                  &crawledBefore=2014-07-01T00:00:00.000-07:00

Remember that for real queries you must use
[URL&nbsp;encoding](http://tools.ietf.org/html/rfc3986#section-2.1) for query string parameters.

## Understanding the JSON Response

### Example Response

The following JSON describes an example match for a red stained glass lamp, sold by online vendor
LightingOnline, reviewed by magazine LampReviews.com.

    [
      {
        "id": "urn:x-mulu-match:bmV3IGJlc3QgZnJpZW5",
        "product": {
          "id": "urn:x-mulu-match:ZmFuY2kgamVzc2ljYSB",
          "productName": "LampCo Red Lamp",
          "merchantName": "LightingOnline",
          "productPrice": "$99.99",
          "priceUnit": "USD",
          "productSource": "lightingonline",
          "imageUrl": "http://lightingonline.com/images/red_lampco_stainedglass.png",
          "category": "Housewares",
          "upc": "123456789999",
          "productUrl": "http://lightingonline.com/products/red_stained_glass_lampco",
          "description": "A red stained glass lamp from MyLampCo.",
          "manufacturer": "MyLampCo",
          "offerId": "lightingonline-124455",
          "type": "PRODUCT"
        },
        "document": {
          "url": "http://lampreviews.com/articles/best_red_lamps_of_2014",
          "publisher": "lampreviews"
        },
        "percentTermsMatched": "50.0",
        "percentProductWordsMatched": "50.0",
        "excerpt": "My place looks great! I love my red LampCo stained glass lamp! yay!",
        "ngram": "red LampCo stained glass lamp ",
        "diceIndex": "0.50",
        "dateCrawled": "2014-01-11T00:00:00.000-07:00",
        "sentence": "I love my red LampCo stained glass lamp!",
        "categoryConfidence": "0.6",
        "aggregateConfidence": "0.5",
        "brandConfidence": "0.6",
        "originalMatchText": "LampCo stained glass lamp"
      },
      ...
    ]

#### Match JSON

The following table describes the format of a Match JSON object. The result of a query is a JSON
array of Match JSON objects, each describing one Mulu Product Match.  All field types are text
values unless otherwise specified. All of the fields in a Match JSON object are always populated.

| Field | Description |
|:---+:---+:---|
| `id` | _Internal Mulu use only. Never store or rely on this value._ |
| `product` | _JSON object_. The JSON representation of the product matched. See [Product&nbsp;JSON](#prodFields)|
| `document` | _JSON object_. The JSON representation of the document where Mulu matched the product. See  [Document&nbsp;JSON](#docFields). |
| `excerpt` | The published document excerpt where Mulu found a Product Match. The excerpt consists of several sentences before and after the product name. This is the longest data field with text from the published document. |
| `originalMatchText` | The part of the published document sentence that mentions the product. |
| `ngram` | The subset of the `originalMatchText` words from the document that Mulu thinks refers to the product. Mulu removes extraneous words then converts the words to their root forms (a process called [stemming](http://en.wikipedia.org/wiki/Stemming)). Not all words may match the product name -- see `percentTermsMatched` and `percentProductWordsMatched`. |
| `percentTermsMatched` | _Floating point number from 0 to 100._ Percentage indicating ratio between two numbers: (1) number of matched words (2) number of words in the `ngram` field. |
| `percentProductWordsMatched` | _Floating point number from 0 to 100._ Percentage indicating ratio between two numbers: (1) number of matched words (2) number of words in the product name after removing extraneous words.  |
| `dateCrawled` | _Date in [ISO8601](http://en.wikipedia.org/wiki/ISO_8601) format_. The most recent date Mulu confirmed this match. |
| `sentence` | The published document sentence that mentions the product. |
| `aggregateConfidence` |  _Floating point number from 0 to 1._ Overall confidence based on other confidence values. This is the main confidence value, and it corresponds to the search query parameter `minimumConfidence`. See [Confidence&nbsp;Levels](#confidencelevels).|
| `diceIndex` | _Floating point number from 0 to 1._  Dice index for the match, which is the lexical similarity of (1) the product name and (2) matched words returned in the `ngram` field . See [Confidence&nbsp;Levels](#confidencelevels).|
| `categoryConfidence` | _Floating point number from 0 to 1._ Category confidence value. See [Confidence Levels](#confidencelevels). |
| `brandConfidence` | _Floating point number from 0 to 1._ Brand confidence value. See [Confidence&nbsp;Levels](#confidencelevels).|

#### Product JSON<a name="prodFields"> </a>

The following table describes the format of a Product JSON object, which is included in every Match
JSON object. All field types are text values unless otherwise specified. All fields are provided
and formatted by the product source data unless otherwise specified as provided by Mulu. All fields
always exist as JSON fields but some may be unpopulated (see second column for details).

| Field | Always populated | Description |
|:---+:---+:---|
| `id` | yes | _Internal Mulu use only. Never store or rely on this value._ |
| `productName` | yes | Name of the product. |
| `productUrl` | yes | A URL to purchase the product. This URL is not necessarily on the same domain as the data source or the merchant (see `merchantName`). |
| `offerId` | typically, but not always (see&nbsp;note)| A product's unique product or offer ID specific to this product source. For example, an Amazon Standard Identification Number ([ASIN](https://en.wikipedia.org/wiki/Amazon_Standard_Identification_Number)). Note: For nearly all cases, `offerId` is populated but for rare cases a retailer includes a `productUrl` but no `offerID`. |
| `type` | yes | The product's Mulu Category Code as determined by Mulu. Possible values are `SONG` (a song), `ALBUM` (an album), `ARTIST` (a music artist), and `PRODUCT` (non-music product). |
| `merchantName` | no | Name of the merchant that offers the product. The merchant name is not always the same as the name of the product source. For example, Amazon sells some products directly, but Amazon also provides product offers from partners and affiliates.|
| `productPrice` | no | Price of the product, as a text value as formatted by the product source. The price may or may not include currency indicator such as $, before or after the number. For example, "$45.06". Also see `priceUnit`. |
| `priceUnit` | no | Currency code in [ISO&nbsp;4217](https://en.wikipedia.org/wiki/ISO_4217) format for the value in the `productPrice` field. For example, for U.S. dollars the value is `USD`. Note that the currency indicator in the `productPrice` field (such as `$`) can be different from the `priceUnit` (such as `USD`). |
| `productSource` | no |  Mulu product source ID for this Product Match, as determined by Mulu. For example, for Amazon the `productSource` is `AMAZON`. |
| `imageUrl` | no |  URL for the product image.  |
| `category` | no |  Product category, for example `Clothing`.  |
| `upc` | no | The product's [Universal Product Code](https://en.wikipedia.org/wiki/Universal_Product_Code).  |
| `description` | no | A description of the product, which may be in plain text or in HTML. |
| `manufacturer` | no | The product’s manufacturer or brand. For example, `Sony`. |

For example:

        {
          "id": "urn:x-mulu-match:ZmFuY2kgamVzc2ljYSB",
          "productName": "LampCo Red Lamp",
          "merchantName": "LightingOnline",
          "productPrice": "$99.99",
          "priceUnit": "USD",
          "productSource": "lightingonline",
          "imageUrl": "http://lightingonline.com/images/red_lampco_stainedglass.png",
          "category": "Widgets",
          "upc": "123456789999",
          "productUrl": "http://lightingonline.com/products/red_stained_glass_lampco",
          "description": "A red stained glass lamp from MyLampCo.",
          "manufacturer": "MyLampCo",
          "offerId": "lightingonline-124455",
          "type": "PRODUCT"
        }

#### Document JSON<a name="docFields"></a>

The following table describes the format of a Document JSON object, which is included in every
Match JSON object. All field types are text values.

| Field | Description |
|:---+:---|
| `url` | The complete URL of the published document where Mulu found the match, including the URI scheme. |
| `publisher` | The publisher ID for the matched document. The publisher ID is the domain name not including the top level domain or subdomains. For example, if the matched document is on domain `www1.cdn.mycompany.com`, the publisher ID is `mycompany`. |

For example:

        {
          "url": "http://lampreviews.com/articles/best_red_lamps_of_2014",
          "publisher": "lampreviews"
        },
