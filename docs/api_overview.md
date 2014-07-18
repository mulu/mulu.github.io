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

Name   | Required? | Description
----   | --------- | -----------
apiKey | Yes       | Api Key provided to you as part of your account setup
