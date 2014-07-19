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

