---
layout: docs
title: Api Overview
permalink: /docs/match_search_api.html
---

# Match Search

## Overview

This api allows searching for matches based on a set of provided criteria. The search critera are provided as query
string parameters to an [HTTP GET](http://tools.ietf.org/html/rfc2616#section-9.3) request to the `/v1/matches`
resource. note that since many of the parameter values will contain characters not permitted in query string parameter
values, they must be [url encoded](http://tools.ietf.org/html/rfc3986#section-2.1). Results are returned as a
[JSON](http://tools.ietf.org/html/rfc7159) document containing an Array of Objects each representing the presence of a
match between an Offer and a published piece of content.

## Request

Submit an HTTP GET request to `/v1/matches` on the domain provided as part of your account setup. Use the following
query string parameters to specify the criteria for which you wish to search. Note that the values for query string
parameters **MUST** be urlencoded.

### Request Parameters

| Name | Required? | Description |
|:---+:---+:---|
| apiKey | Yes | Api Key provided to you as part of your account setup |
| urlPrefix | Yes &#91;[1](#reqParam1)&#93; | A string prefix which the urls of matches must match. The prefix must be on a directory boundary<br>(i.e. `http://magazine.com/category/` not `http://magazine.com/categ`) |
| productName | Yes &#91;[1](#reqParam1)&#93; | A String of words which the product name from matches must be similar to.<br>**(Note: this is being deprecated in favor of offer IDs)** |
| offer | Yes &#91;[1](#reqParam1)&#93; | A String representing the source's identifier for this product. Can be provided multiple times to search for multiple products. |
| crawledAfter | No &#91;[2](#reqParam2)&#93; | A date/time in [ISO8601](http://en.wikipedia.org/wiki/ISO_8601) format which the match must have been found on or after<br>(i.e. `2014-02-19T00:00:00.0+00:00`) |
| crawledBefore | No | A date/time in [ISO8601](http://en.wikipedia.org/wiki/ISO_8601) format which the match must have been found on or before<br>(i.e. `2014-02-19T00:00:00.0+00:00`) |
| minimumConfidence | No &#91;[3](#reqParam3)&#93; | A minimum confidence score the matches must meet |
| source | No &#91;[4](#reqParam4)&#93; | A source from which the products are selected. (use multiple times to support multiple sources) |
| category | No | A category to which the product matched must belong as defined by the product's source |

1. At least one of UrlPrefix or ProductName must be provided<a name="reqParam1"> </a>
2. crawledAfter is required in order for crawledBefore to function<a name="reqParam2"> </a>
3. minimumConfidence is set to a default value if not provided<a name="reqParam3"> </a>
4. sources are set to the sources to which the apiKey is configured to have access by default<a name="reqParam4"> </a>

## Response

### Example Response


    [
      {
        "id": "urn:x-mulu-match:bmV3IGJlc3QgZnJpZW5",
        "product": {
          "id": "",
          "productName": "Name of the product",
          "merchantName": "SOME MERCHANT",
          "productPrice": "$99.99",
          "priceUnit": "USD",
          "productSource": "SOME_PRODUCT_SOURCE",
          "imageUrl": "http://some.domain.com/where/an/image/of/the/product/is/available.png",
          "category": "Widgets",
          "upc": "",
          "productUrl": "http://some.domain.com/where/you/can/buy/a/widget",
          "description": "A description of the product",
          "manufacturer": "The manufacturer or brand of the product",
          "offerId": "The ID for the product from the product source",
          "type": "PRODUCT"
        },
        "document": {
          "url": "http://the.publisher.com/the/url/where/the/product/is/mentioned",
          "publisher": "publisher's ID"
        },
        "percentTermsMatched": "50.0",
        "percentProductWordsMatched": "50.0",
        "excerpt": "An excerpt from the publication where the match is located, consists of several sentences, both before and after the text of the match",
        "ngram": "The words used to match the product, with extraneous words removed and the words stemmed",
        "diceIndex": "0.50",
        "dateCrawled": "2014-01-11T00:00:00.000-07:00",
        "sentence": "The specific sentence from the publication where the product is mentioned.",
        "categoryConfidence": "0.0",
        "aggregateConfidence": "0.50",
        "brandConfidence": "0.0",
        "originalMatchText": "The specific portion of the sentence where the product is mentioned"
      },
      ...
    ]

#### Match Field definitions

| Field | Description |
|:---+:---|
| id | Internal unique identifier for a match in the mulu system |
| [product](#prodFields) | Json object representation of the product matched |
| [document](#docFields) | Json object representation of the document where the product was matched |
| percentTermsMatched | the percentage of the pertinent words from the document found in the product name |
| percentProductWordsMatched | the percentage of the pertinent words from the product name found in the document |
| excerpt | An excerpt from the publication where the match is located, consists of several sentences, both before and after the text of the match |
| ngram | The words used to match the product, with extraneous words removed and the words [stemmed](http://en.wikipedia.org/wiki/Stemming) |
| diceIndex | A measure of how lexically similar the Product Name and ngram are |
| dateCrawled | The date on which this match was last detected in [ISO8601](http://en.wikipedia.org/wiki/ISO_8601) format |
| sentence | The specific sentence from the publication where the product is mentioned |
| categoryConfidence | A measure of confidence based on the ratios of the categories of products found in the document |
| aggregateConfidence | An aggregation of other confidence measures |
| brandConfidence | A measure of confidence based on the presence of the brand name for the product in the sentence and the likelihood that the brand would appear in normal language |
| originalMatchText | The specific portion of the sentence where the product is mentioned |

#### Product Field definitions<a name="prodFields"> </a>

| Field | Description |
|:---+:---|
| id | Internal unique identifier for a product |
| productName | Name of the product |
| merchantName | Name of the merchant offering the product |
| productPrice | Price of the product (may or may not include currency indicator such as $) |
| priceUnit | Currency and Unit for the price listing |
| productSource | Identifier for the source of the product information |
| imageUrl | Url at which an image of the product can be found |
| category | The category the product is in, as provided by the product source |
| upc | Universal Price Code, as provided by the product source |
| productUrl | A URL at which the product can be purchased |
| productUrl | A text or HTML description of the product, as provided by the product source |
| manufacturer | The product’s manufacturer or brand, as provided by the product source |
| offerId | The ID for the product from the product source |
| type | Internal high level categorization of the product |

#### Document Field definitions<a name="docFields"> </a>

| Field | Description |
|:---+:---|
| url | Url of the document where the match was found |
| publisher | Identifier for the publisher of the document |


## Example Requests

The following are some example requests using this API  (example url parameters remain un-encoded for legibility)

#### Fetch all matches for a given article with a confidence greater than the default

    GET v1/matches?apiKey=AAAA-9999-AAAA-9999&urlPrefix=http://www.magazine.com/article

#### Fetch all matches for a given article with a confidence greater than 0.5

    GET v1/matches?apiKey=AAAA-9999-AAAA-9999&urlPrefix=http://www.magazine.com/article&minimumConfidence=0.5

#### Fetch all matches for a given article where the products are in the Cosmetics category

    GET v1/matches?apiKey=AAAA-9999-AAAA-9999&urlPrefix=http://www.magazine.com/article&category=Cosmetics

#### Fetch all matches for a given article where the products are in the Cosmetics or Personal Care categories

    GET v1/matches?apiKey=AAAA-9999-AAAA-9999&urlPrefix=http://www.magazine.com/article&category=Cosmetics&category=Personal Care

#### Fetch all matches for the product identified by it's source as PRODUCTID004

    GET v1/matches?apiKey=AAAA-9999-AAAA-9999&offer=PRODUCTID004

#### Fetch all matches for products identified by it's source as either PRODUCTID004 of PRODUCTID005

    GET v1/matches?apiKey=AAAA-9999-AAAA-9999&offer=PRODUCTID004&offer=PRODUCTID005

#### Fetch all matches or the product identified by it's source as PRODUCTID004 from the magazine.com domain

    GET v1/matches?apiKey=AAAA-9999-AAAA-9999&offer=PRODUCTID004&urlPrefix=http://www.magazine.com/

