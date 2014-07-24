---
layout: docs
title: Api Overview
permalink: /docs/match_search_api.html
---
 
## Overview

Mulu analyzes text on the Internet to find references to products sold by the largest online retailers such as Amazon, EBay, Overstock, Popsugar, and iTunes. The Mulu API finds and returns Mulu Product Matches that identify product mentions on review sites, magazines, and other high-value content. 

For example, suppose a magazine reviews a product. Mulu detects the product reference and determines which retailers sell the product. Online retailers can use the Mulu API to add links to any relevant product reviews. Advertising networks can use the Mulu API to improve targeted advertising.

This API reference uses the following definitions:

*  _offer_ - a product's unique product or offer ID. 
* _product source_ - an online retailer 
* _publisher_ - a magazine or other content publisher that generates Mulu Product Matches
* _published content_ - one web page (or other content with a unique URL) from a publisher
* _product match_ - a mention of a product in published content

The Mulu API is a REST API, which means that it is a language-neutral API that you can integrate with any server. Specify your search criteria as query string parameters on a standard HTTP GET request URL. You can test your queries easily in any web browser as standard HTTP URLs.

The Mulu API returns Mulu Product Matches as a [JSON](http://tools.ietf.org/html/rfc7159) array. Each array element is a JSON object that represents a Mulu Product Match. If multiple product sources sell the same product, there are multiple items in the results for each match between the published

####<a name="confidencelevels"></a>Confidence Levels

The Mulu linguistic engine generates an associated confidence level from 0% to 100% for each Product Match. A 100% confidence match represents a strong confidence that the product reference really represents the product represented by the offer ID from that product source. In the Mulu API, the confidence level is a floating point number from 0 to 1 (not 100).

When searching for product matches, optionally specify a _minimum confidence level_ for results. Depending on your situation, you might want to search for Mulu Product Matches with a high minimum confidence level or a low minimum confidence level. For more results, set the minimum confidence level close to 0.

As you process search results, parse each Product Match to examine its confidence level. Use that value to treat results differently based on the confidence level. Unlike in the search criteria, there are three confidence levels in results, all of which are defined in the API as a floating point number from 0 to 1 (not 100):

| Result Field | Description |
|:---+:---|
| _aggregate&nbsp;confidence_ | The overall confidence of the match. This field corresponds most directly to the _minimum&nbsp;confidence&nbsp;level_ in your search query. This is the main confidence level result field. All other confidence fields are for advanced use.
| _Dice&nbsp;index_ |  Confidence of similarity between the set of words in the Product Name and the set of words detected in on the published page, as determined by the [Dice index](https://en.wikipedia.org/wiki/S%C3%B8rensen%E2%80%93Dice_coefficient) algorithm. Think of this is a lexical confidence of the product name. 
| _category&nbsp;confidence_ | The confidence of the match based on the product category similarity with other matches on the same page of published content determined by URL. For example, suppose Mulu detects that a web page mentions 10 bicycles and 1 song name. The Product Matches for the bicycles have very high category confidence. The Product Matches for the song name would be a low category confidence.
| _brand&nbsp;confidence_ | Confidence of the match based on (1) the presence of the brand name for the product in the sentence and (2) the likelihood that the brand would appear in normal language.

## Searching for Product Matches

To search for product matches, submit an [HTTP&nbsp;GET](http://tools.ietf.org/html/rfc2616#section-9.3) request to the `api.mulu.me` domain for the `/v1/matches` resource. Add query string parameters for your account API key and search criteria. 

A simple Mulu API search using a URL prefix looks like the following:
`http://api.mulu.me/v1/matches?apiKey=YOUR_API_KEY&urlPrefix=http://www.magazine.com/`

Parameter values typically contain characters not permitted in query string parameters, such as ampersand and spaces. You MUST use [URL encoding](http://tools.ietf.org/html/rfc3986#section-2.1) for query string parameters. 

### Specifying Request Parameters

Provide the `apiKey` parameter and at least one other search parameter. Provide at least one of the following: `urlPrefix` or `offer`.


| Name | Required? | Description |
|:---+:---+:---|
| `apiKey` | Yes | Your Mulu Account API key. To set up a new Mulu account, [contact us](http://mulu.me/contact). |
| `urlPrefix` | Yes &#91;[1](#reqParam1)&#93; | String. The prefix (first characters) of publisher URL that you want to match. For HTTP URLs, the URL prefix text must begin with "http://". The prefix must be on a directory (slash) boundary. If a slash character is not the last character of the urlPrefix, a final slash is implicit. For example, to match the URL `http://magazine.com/category/mypage`, pass the urlPrefix value `http://magazine.com/category` or `http://magazine.com/category/`. In contrast, the URL prefix `http://magazine.com/categ` does not match that URL.) |
| `offer` | Yes &#91;[1](#reqParam1)&#93; | String. A unique ID for one product or offer as specified by its product source. For example, specify an Amazon Standard Identification Number (ASIN). To search for multiple products, provide the `offer` parameter multiple times in your search query string. |
| `crawledAfter` | No &#91;[2](#reqParam2)&#93; | Date/time in [ISO8601](http://en.wikipedia.org/wiki/ISO_8601) format. Specify this parameter to limit results only to matches found on or after this date. You must include the time zone information. For example, `2014-02-19T00:00:00.0+00:00`. If you use the crawledBefore parameter, you must also specify crawledAfter.|
| `crawledBefore` | No | Date/time in [ISO8601](http://en.wikipedia.org/wiki/ISO_8601) format. Specify this parameter to limit results only to matches found on or before this date. You must include the time zone information. For example, `2014-02-19T00:00:00.0+00:00`. If you use the crawledBefore parameter, you must also specify crawledAfter. |
| `minimumConfidence` | No | A minimum confidence value for any matches, as a floating point number from 0 to 1. To get more results from the Mulu API, specify a lower `minimumConfidence` value. If unspecified, the default is 0.8, which represents 80% confident of a match.  |
| `source` | No | Specify `source` to limit results only a specific product sources. To search multiple sources, provide the `source` parameter multiple times in your search query string. If `source` is unspecified, Mulu API searches the default set of sources for your Mulu Account API key. If you provide multiple sources and also provide the `offer` parameter, the `offer` ID could potentially match results from multiple sources. However, that edge case is uncommon in practice due to different offer ID styles from product sources.|
| `category` | No | String. Specify a required product category as defined by the product source. For example, Amazon includes categories such as `Clothing` and `Beauty`. If unspecified, will match all product categories. |

1. Provide at least one of the following: `urlPrefix` or `offer`.<a name="reqParam1"> </a>
2. crawledAfter is required in order for crawledBefore to function<a name="reqParam2"> </a>

## Understanding the JSON Response

### Example Response

The following JSON describes an example match for a red stained glass lamp, sold by online vendor LightingOnline, reviewed by a magazine called LampReviews.com.

    [
      {
        "id": "urn:x-mulu-match:bmV3IGJlc3QgZnJpZW5",
        "product": {
          "id": "",
          "productName": "LampCo Red Lamp",
          "merchantName": "LightingOnline",
          "productPrice": "$99.99",
          "priceUnit": "USD",
          "productSource": "lightingonline",
          "imageUrl": "http://lightingonline.com/images/red_lampco_stainedglass.png",
          "category": "Widgets",
          "upc": "",
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
        "ngram": "red lamp ",
        "diceIndex": "0.50",
        "dateCrawled": "2014-01-11T00:00:00.000-07:00",
        "sentence": "I love my red LampCo stained glass lamp!",
        "categoryConfidence": "0.0",
        "aggregateConfidence": "0.50",
        "brandConfidence": "0.0",
        "originalMatchText": "LampCo stained glass lamp"
      },
      ...
    ]

#### Match Field definitions

| Field | Description |
|:---+:---+:---|
| `id` | Internal unique identifier for a match in the Mulu system |
| 'product' | JSON object representation of the product matched. See [the Product Field section]#prodFields |
| `document` [document](#docFields)) | JSON object representation of the document where the product was matched |
| percentTermsMatched | the percentage of the pertinent words from the document found in the product name |
| percentProductWordsMatched | the percentage of the pertinent words from the product name found in the document |
| excerpt | The published content excerpt where Mulu found a Product Match. The excerpt consists of several sentences, both before and after the text of the match. This is the longest of data fields with data from the published content |
| ngram | The words used to match the product, with extraneous words removed and the words [stemmed](http://en.wikipedia.org/wiki/Stemming) |
| dateCrawled | The date on which this match was last detected in [ISO8601](http://en.wikipedia.org/wiki/ISO_8601) format |
| originalMatchText | The part of the published content sentence that mentions the product |
| sentence | The published content sentence that mentions the product |
| aggregateConfidence | Overall confidence based on other confidence values, , as a floating point number from 0 to 1. See [Confidence Levels](#confidencelevels)|
| diceIndex | Dice index value, the lexical similarity of the product name and matched words on the published site. See [Confidence Levels](#confidencelevels)|
| categoryConfidence | Category confidence value, as a floating point number from 0 to 1. See [Confidence Levels](#confidencelevels) |
| brandConfidence | Brand confidence value, as a floating point number from 0 to 1. See [Confidence Levels](#confidencelevels)|

#### Product Field definitions<a name="prodFields"> </a>

| Field | Description |
|:---+:---|
| id | Internal unique identifier for a product |
| productName | Name of the product |
| merchantName | Name of the merchant offering the product. The merchant name might not be the same as the product source. For example, Amazon sells some products directly, but the Amazon product source also provides product offers from partners and affiliates. Note that the product affiliates and |
| productPrice | Price of the product (may or may not include currency indicator such as $) |
| priceUnit | Currency and Unit for the price listing |
| productSource | Identifier for the source of the product information |
| imageUrl | Url at which an image of the product can be found |
| category | The category the product is in, as provided by the product source |
| upc | Universal Price Code, as provided by the product source |
| productUrl | A URL at which the product can be purchased |
| description | A text or HTML description of the product, as provided by the product source |
| manufacturer | The product’s manufacturer or brand, as provided by the product source |
| offerId | The ID for the product from the product source |
| type | The product's Mulu Category Code. Current values are `PRODUCT` (non-music product), `SONG` (a song), `ALBUM` (an album), `ARTIST` (a music artist). |

#### Document Field definitions<a name="docFields"> </a>

| Field | Description |
|:---+:---|
| url | Url of the document where the match was found |
| publisher | Identifier for the publisher of the document |


## Example Requests

The following are some example requests using this API  (example URL parameters remain un-encoded for legibility)

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

