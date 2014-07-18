---
layout: docs
title: Api Overview
permalink: /docs/match_search_api.html
---

# Match Search

This api allows searching for matches based on a set of provided criteria. The search critera are provided as
query string parameters to an HTTP GET request. note that since many of the parameter values will contain characters
not permitted in query string parameter values, they must be
[url encoded](http://tools.ietf.org/html/rfc3986#section-2.1).

