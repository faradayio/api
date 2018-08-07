# Faraday API

* [Overview](#overview)
  * [Versions](#versions)
  * [Contributing](#contributing)
* [Basics](#basics)
  * [URLs](#urls)
  * [Authentication](#authentication)
  * [Special request value types](#special-request-value-types)
    * [Describing Audiences](#describing-audiences)
    * [FIG attributes](#fig-attributes)
  * [Responses](#responses)
* [Requests](#requests)
  * [Individuals](#individuals)

## Overview

Faraday exposes a REST-style API for retrieving and manipulating data.

### Versions

The current version of the API is **v3**.

#### History

* API version `v1` is currently **deprecated**
* API version `v2` is currently **deprecated**
* API version `v3` is currently **active**

### Contributing

* **Want a new resource exposed in the API?** Submit a pull request documenting the change you'd like to see.

* **Have a question or other feedback?** Open an issue.

## Basics

All API requests share some common components.

### URLs

API requests use the following base URL:

```
https://api.faraday.io/[VERSION]
```

The **current** base URL is:

```
https://api.faraday.io/v3
```

HTTPS is required for all requests.

### Authentication

Request an API token from Customer Success.

Authentication to the API occurs via the `api_key` request param.

### Special request value types

The request-level documentation below occasionally refers to a special type for a parameter.

#### Describing Audiences

Some endpoints allow you to specify one or more Audiences (e.g. segment membership in the individuals endpoint). Each element of the Audiences array can describe an Audience in one of two forms:

1. **ID** — Give the ID, as a UUID string, of an Audience which has been previously created on the Faraday platform in your Account (e.g. by using Explore interactively).
1. **Definition** — Express the from-scratch definition of the Audience in JSON using FIG attributes. For example:

```json
{
  "household_income": [0, 50000],
  "gardener": true
}
```

#### FIG attributes

Some endpoints allow you to indicate one or more FIG attributes (e.g. appending in the individuals endpoint). Please contact your Customer Success Manager for a data dictionary.

### Responses

All responses will arrive as JSON.

## Requests

The v3 API currently supports the following requests:

* [Individuals](#individuals)

### Individuals

The Individuals endpoint allows users to query Faraday regarding a single individual, including scoring that individual against one or more Outcomes, determining that individuals membership in one or more segments, and appending data for that individual.

* [Endpoint](#endpoint)
* [Response codes](#response-codes)
* [Request paramenters](#request-parameters)
  * [Auth](#auth)
  * [Identity](#identity)
  * [Matching settings](#matching-settings)
  * [Operations](#operations)
  * [Response settings](#response-settings)
  * [Response](#response)

#### Endpoint

`POST /individuals`

#### Response codes

* **200** OK
* **404** Individual could not be found

#### Request parameters

##### Auth

  * `api_key` _String_ **Required** — The API key for the Account within which this request should be made.

##### Identity

  * `person_first_name` _String_ — First name (if known).
  * `person_last_name` _String_ — Last name (if known).
  * `house_number_and_street` _String_ — Physical address including number and street.
  * `city` _String_ — City.
  * `state` _String_ — 2-letter postal abbreviation.
  * `postcode` _String_ — 5-digit zipcode. Send as string to preserve leading zeroes.
  * `phone` _String_ — E.123-compliant string representation.
  * `email` _String_ — E-mail address.

##### Matching settings

  * `allow_email_match` _"true" or omit_ — Allow Faraday to attempt an email match if matching by physical address fails. **You may incur charges.**
  * `allow_phone_match` _"true" or omit_ — Allow Faraday to attempt a phone match if matching by physical address fails. **You may incur charges.**
  * `match_tolerance` _"loose", "tight", or omit_ — By default, Faraday will match a given identity when lastname, normalized address, and postcode match. Tight mode, on the other hand, also requires a firstname match. Choose loose mode to ignore name and match on address only.

##### Operations

  * `audiences` _Array of Audiences_ — Check to see if the matched household falls within each of the specified Audiences. Each specified Audience must have been previously created with Explore.
  * `attributes` _Array of Strings_ — Append the specified FIG attributes, each identified by its handle.
  * `outcomes` _Array of UUID Strings_ — Use each specified Outcome's currently promoted Model to score the matching household.

##### Response settings
  * `response_prefix` _String_ — Prefix each standard response key with the specified string.
  * `postback_url` _String_ — In addition to the standard HTTP response, also POST the response to the specified URL.

#### Response

  * `person_first_name` _String_ — Passed through from request.
  * `person_last_name` _String_ — Passed through from request.
  * `house_number_and_street` _String_ — Normalized from request.
  * `city` _String_ — Normalized from request.
  * `state` _String_ — Normalized from request.
  * `postcode` _String_ — Normalized from request.
  * `latitude` _Float_ — Decimal geocoded latitude.
  * `longitude` _Float_ — Decimal geocoded longitude.
  * `match_code` _String_ — For internal troubleshooting purposes.
  * `attributes` _Hash_ — Each key is the handle of a requested FIG attribute. Each corresponding value is that attribute extracted from FIG.
  * `audiences` _Hash_ — Each key is the UUID of a requested Audience. Each corresponding value is a boolean indicating whether the household does or does not belong to that Audience.
  * `outcomes` _Hash_ — Each key is the ID of an Outcome this household was scored against. The corresponding value is the probability that the matched household will achieve this Outcome.
  * `warnings` _Array of Strings_ — Each warning is a human-interpretable message indicating an issue with the API request.
  * `error` _String_ — Error message.

## Copyright

Copyright 2018 Faraday
