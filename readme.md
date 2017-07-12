# Faraday API

* [Overview](#overview)
  * [Versions](#versions)
  * [Contributing](#contributing)
* [Basics](#basics)
  * [Environments](#environments)
  * [URLs](#urls)
  * [Authentication](#authentication)
  * [Headers](#headers)
  * [Parameters](#parameters)
  * [Special request value types](#special-request-value-types)
    * [Segment specification](#segment-specification)
    * [Geography](#geography)
    * [Attribute condition](#attribute-condition)
  * [Responses](#responses)
* [Requests](#requests)
  * [Record a customer](#record-a-customer)
  * [Record a lead](#record-a-lead)
  * [Record a prospect](#record-a-prospect)
  * [Upload a file](#upload-a-file)
  * [List household attributes](#list-household-attributes)
  * [List products](#list-products)

## Overview

Faraday exposes a REST-style API for retrieving and manipulating data.

### Versions

* API version `v1` is currently in **alpha**

### Contributing

* **Want a new resource exposed in the API?** Submit a pull request documenting the change you'd like to see.

* **Have a question or other feedback?** Open an issue.

## Basics

All API requests share some common components.

### Environments

Two environments are offered for developer convenience:

* The **production** environment performs all requests against production data, returns complete responses, and incurs charges where applicable. Use of your **production API key** will perform API operations on the production environment.

* The **test** environment uses fake random data, will not make any changes, and does not incur charges. Use of your **test API key** will perform API operations on the test environment.

<!--
### Rate limiting

API package | Rate limit
------------|------------------
Free        | 100 requests/hour
Standard    | 1,000 requests/hour
Extended    | Unlimited

#### After you hit your limit

Faraday will return [status 429](http://httpstatus.es/429).

#### Checking rate limit status

All responses include a `X-Ratelimit-Remaining` header. To check your quota without performing a real request, use the `HEAD` action instad of `GET`, `POST`, etc.
-->

### URLs

API requests use the following base URL:

```
https://api.faraday.io/v1
```

HTTPS is required for all requests.

### Authentication

Request an API token from Customer Success.

Authentication to the API occurs via HTTP Basic Auth. Provide your API key as the basic auth username, leaving the password section blank.

```shell
$ curl https://api.faraday.io/v1/audiences \
  -u sk_test_E6V2sxvyYD5PX8a1tqqNE3eG:
```

### Headers

Required headers are in **bold**.

Header | Description | Example
-------|-------------|--------
**`Accept`** | Indicates desired response format. Currently only JSON is supported. | `Accept: application/json`
`Content-Type` | **Required for POST requests.** Indicates the request format. Currently only JSON is supported. | `Content-Type: application/json`
`X-Request-Id` | If this header is set with a UUID in the request, it will be returned as-is in the response. Potentially useful for handling parallelized API consumption. | `X-Request-Id: 3fe4594e-10f5-46e8-8ab3-28b812a3fc47`

### Parameters

The Faraday API accepts parameters in the query string (for GET-style requests) and as JSON in the request body (for POST-style requests).

<!--

### Accepting charges

Many requests will cause your account to be charged. In order to avoid confusion, Faraday requires that you indicate your acknowledgement of this using the `Faraday-Accept-Charges` header:

Value | Intent
------|-------
`false`     | **I do not accept charges for this request.** If the request would cause your account to be charged, Faraday will instead return [status 402](http://httpstatus.es/402).
`true`    | **I accept charges for this request**.

-->

### Special request value types

The request-level documentation below occasionally refers to a special type for a parameter.

#### Segment specification

A *segment* describes the group of all known Faraday households in a given *geography* that meet certain *criteria*. A *segment specification* serializes this description into a JSON object:

Top-level key | Description | Example
--------------|-------------|--------
`geography` | A [geography](#geography) or array of geographies within which the households must be located to be included in the segment. When the array form is used, households must belong to *at least one* element to be considered in-segment. | `[{ "type": "place", "id": 1234 }, { "type": "place", "id": 5678 }]`
`criteria` | An object whose key-value pairs are [attribute conditions](#attribute-condition) which households must meet to be included in the segment. A household must meet *all* criteria to be considered in-segment. | `[{ "household_income": [80000, 120000], "credit_rating": [700, "Infinity"]}]`

#### Geography

A *geography* is used to describe a geographical area. There are currently five types of geographic areas that Faraday supports: **named places**, **zip codes**, **radius around current customers**, **radius around points of interest**, and **drawn areas**. Indicate geography type with the `type` field and use additional fields described below to identify the area itself.

The most common geography type is a **named place**. Faraday maintains a [list of named places](https://github.com/faradayio/places).

The format for each of the supported geography types:

* **Named place**: `{ "type": "place", "id": "01234" }`. `id` must be a string to allow for leading zeroes.
* **Zip code**: `{ "type": "zipcode", "id": "05401" }`
* **Custom shape** `{ "type": "custom_shape", "shape": {...} }`. The provided `shape must be valid [GeoJSON](http://geojson.org/).
* **Radius around customers** (must be combined with a non-radius geography type) `{ "type": "customer_proximity", "distance": 1000 }`. `distance` is radius in meters.
* **Radius around points of interest** (must be combined with a non-radius geography type) `{ "type": "poi_proximity", "distance": 1000, "points_of_interest": [123, 456]}`

Note that radiuses only work when combined with a named place, zip code, or custom shape.

#### Attribute condition

An *attribute condition* is generally one element of a JSON object listing several household criteria. Each such element must conform to the format appropriate for the [household attribute](#list-household-attributes) in question:

* **Continuously variabled attributes** are specified with a range: `"household_income": [80000, 120000]`. For unbounded ranges, use `"-Infinity"` (no minimum) and `"Infinity"` (no maximum). To specifically look for null values (no coverage), use `"NULL"`.

* **Boolean attributes**: `"gardener": false`. To specifically look for null values (no coverage), use `"NULL"`; if non-truthy, `"FALSE_OR_NULL"`.

* **Categorical attributes** are specified with an array of valid desired categories: `"house_style": ["Ranch", "Townhouse"]`. Households must belong to *one* of the listed categories to meet the condition. To specifically look for null values (no coverage), use `"NULL"`.

### Responses

All responses will arrive as JSON. Your requests should include the `Accept: application/json` header to acknowledge this.

## Requests

* [Record a customer](#record-a-customer)
* [Record a lead](#record-a-lead)
* [Record a prospect](#record-a-prospect)
* [Upload a file](#upload-a-file)
* [List household attributes](#list-household-attributes)
* [List products](#list-products)

### Record a customer

**Status:** Live

A *customer* is a household that has purchased your product or service, whether or not as a result of Faraday-facilitated outreach. Reporting your customers to Faraday improves predictions and allows you to visualize and target your customers with the platform.

#### Request

```http
POST https://api.faraday.io/v1/customers
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
Content-Type: application/json
Accept: application/json
```

#### Parameters

Required parameters in **bold**.

Parameter | Type | Description | Example
----------|------|-------------|--------
`person` | *String* | The combined first and last name of the customer. | `Michael Faraday`
**`house_number_and_street`** | *String* | The physical address of the customer. | `123 Main St.`
`city` | *String* | The customer's city. | `Burlington`
`state` | *String* | The 2-letter postal abbreviation of the customer's state. | `VT`
**`postcode`** | *String* | The customer's 5-digit zip code. (Use a string to avoid issues with leading zeroes.) | `05402`
**`product_id`** | *String* | The UUID of a [recognized Faraday product](#list-products) indicating the purchase that was made by the customer. | `b9dfbba4-bdc3-47dd-8006-2ee84b4861d4`
`became_customer_at` | *String* | A [ISO8601](https://en.wikipedia.org/wiki/ISO_8601) date/time string indicating when the purchase was made. Uses the current time if missing. | `20151025T223451Z`
`attributes` | *String* or *Array of strings* | Instructs Faraday to include (append) one or more attributes of the customer—if a match to a known household can be made. Each attribute should be a [valid Faraday household attribute](#list-household-attributes); unrecognized attributes will be ignored and added to the `Faraday-Unrecognized-Attributes` response header. | `["household_income", "credit_rating"]`
`qualify` | [*Segment specification*](#segment-specification) | Instruct Faraday to qualify this customer by determining its inclusion in the specified segment, if the customer can be matched with high confidence to a known Faraday household. | `{ "geography": [ { "type": "place", "id": 1234 } ], "criteria": { "household_income": [80000, "Infinity"]} }`

#### Response

Top-level key | Value description | Example
--------------|-------------------|--------
`id` | The internal Faraday ID (UUID) for your newly submitted customer. | `4a991134-2677-46c5-b01e-298582982fa0`
`customer_id` | (DEPRECATED; see `id`) The internal Faraday ID (UUID) for your newly submitted customer. | `4a991134-2677-46c5-b01e-298582982fa0`
`household_id` | The internal Faraday ID (UUID) for the known household that the customer matched to (if a match could be made). Note that Faraday household IDs are ephemeral and should be neither persisted nor relied upon; we include them for debugging purposes. | `2e231a5a-e3f7-4a09-b4fc-21289f7debcf`
`person` | Null unless provided in request. The name of record for the known household your customer matched to, if such a match could be made. | `Michael Faraday`
`attributes` | Requested household-level attributes, if any were provided and a match could be made. | `{ "household_income": 110000, "credit_rating": 690 }`
`qualify` | A boolean indicating whether the matched household (if any) is included in the segment specified in your request (if provided). | `true`
`disqualifications` | The reason for a false value in the `qualify` response value. Only included when `qualify` is false. | `[{"household_income": "expected between 50000.0 and Infinity, but outside"}]`

### Record a lead

**Status:** Live

In Faraday parlance, a *lead* is a household that has expressed interest in your product or service, often by *responding* to marketing—whether or not the campaign was facilitated by Faraday.

#### Request

```http
POST https://api.faraday.io/v1/leads
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
Content-Type: application/json
Accept: application/json
```

#### Parameters

Required parameters in **bold**.

Parameter | Type | Description | Example
----------|------|-------------|--------
`person` | *String* | The combined first and last name of the lead. | `Michael Faraday`
**`house_number_and_street`** | *String* | The physical address of the lead. | `123 Main St.`
`city` | *String* | The lead's city. | `Burlington`
`state` | *String* | The 2-letter postal abbreviation of the lead's state. | `VT`
**`postcode`** | *String* | The lead's 5-digit zip code. (Use a string to avoid issues with leading zeroes.) | `05402`
**`product_id`** | *String* | The UUID of a [recognized Faraday product](#list-products) indicating the category of interest for the lead. | `b9dfbba4-bdc3-47dd-8006-2ee84b4861d4`
**`became_lead_at`** | *String* | A [ISO8601](https://en.wikipedia.org/wiki/ISO_8601) date/time string indicating when the lead emerged. Uses the current time if missing. | `20151025T223451Z`
`attributes` | *String* or *Array of strings* | Instructs Faraday to include (append) one or more attributes of the lead—if a match to a known household can be made. Each attribute should be a [valid Faraday household attribute](#list-household-attributes); unrecognized attributes will be ignored and added to the `Faraday-Unrecognized-Attributes` response header. | `["household_income", "credit_rating"]`
`qualify` | [*Segment specification*](#segment-specification) | Instruct Faraday to qualify this lead by determining its inclusion in the specified segment, if the lead can be matched with high confidence to a known Faraday household. | `{ "geography": [ { "type": "place", "id": 1234 } ], "criteria": { "household_income": [80000, "Infinity"]} }`
`predict` | *Boolean* | Instruct Faraday to predict the likelihood that the lead will purchase the specified [product](#list-products), if a high-confidence match can be made to a known Faraday household. | `true`

#### Response

Top-level key | Value description | Example
--------------|-------------------|--------
`id` | The internal Faraday ID (UUID) for your newly submitted lead. | `4a991134-2677-46c5-b01e-298582982fa0`
`lead_id` | (DEPRECATED; see `id`) The internal Faraday ID (UUID) for your newly submitted lead. | `4a991134-2677-46c5-b01e-298582982fa0`
`household_id` | The internal Faraday ID (UUID) for the known household that the lead matched to (if a match could be made). Note that Faraday household IDs are ephemeral and should be neither persisted nor relied upon; we include them for debugging purposes. | `2e231a5a-e3f7-4a09-b4fc-21289f7debcf`
`person` | Null unless provided in request. The name of record for the known household your lead matched to, if such a match could be made. | `Michael Faraday`
`attributes` | Requested household-level attributes, if any were provided and a match could be made. | `{ "household_income": 110000, "credit_rating": 690 }`
`predict` | A number between -1 and 1 indicating the likelihood that the matched household (if such a match was possible) will purchase the product specified in the request. Positive values indicate that the household is predicted to purchase; negative values indicate the the household is predicted to reject. The absolute value of the number indicates Faraday's confidence in its prediction. A null response value indicates that a prediction was impossible. | `0.89`
`qualify` | A boolean indicating whether the matched household (if any) is included in the segment specified in your request (if provided). | `true`
`disqualifications` | The reason for a false value in the `qualify` response value. Only included when `qualify` is false. | `"household_income: expected between 50000 and Infinity, but outside"`

### Record a prospect

**Status:** Live

In Faraday parlance, a *prospect* is a household that has been contacted (or will shortly be contacted) but where an outcome is not yet known.

#### Request

```http
POST https://api.faraday.io/v1/prospects
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
Content-Type: application/json
Accept: application/json
```

#### Parameters

Required parameters in **bold**.

Parameter | Type | Description | Example
----------|------|-------------|--------
`person` | *String* | The combined first and last name of the prospect. | `Michael Faraday`
**`house_number_and_street`** | *String* | The physical address of the prospect. | `123 Main St.`
`city` | *String* | The prospect's city. | `Burlington`
`state` | *String* | The 2-letter postal abbreviation of the prospect's state. | `VT`
**`postcode`** | *String* | The prospect's 5-digit zip code. (Use a string to avoid issues with leading zeroes.) | `05402`
**`product_id`** | *String* | The UUID of a [recognized Faraday product](#list-products) indicating the category of interest for the prospect. | `b9dfbba4-bdc3-47dd-8006-2ee84b4861d4`
`became_prospect_at` | *String* | A [ISO8601](https://en.wikipedia.org/wiki/ISO_8601) date/time string indicating when the prospect emerged. Uses the current time if missing. | `20151025T223451Z`
`attributes` | *String* or *Array of strings* | Instructs Faraday to include (append) one or more attributes of the prospect—if a match to a known household can be made. Each attribute should be a [valid Faraday household attribute](#options-households); unrecognized attributes will be ignored and added to the `Faraday-Unrecognized-Attributes` response header. | `["household_income", "credit_rating"]`
`qualify` | [*Segment specification*](#segment-specification) | Instruct Faraday to qualify this prospect by determining its inclusion in the specified segment, if the prospect can be matched with high confidence to a known Faraday household. | `{ "geography": [ { "type": "place", "id": 1234 } ], "criteria": { "household_income": [80000, "Infinity"]} }`
`predict` | *Boolean* | Instruct Faraday to predict the likelihood that the lead will purchase the specified [product](#list-products), if a high-confidence match can be made to a known Faraday household. | `true`

#### Response

Top-level key | Value description | Example
--------------|-------------------|--------
`id` | The internal Faraday ID (UUID) for your newly submitted prospect. | `4a991134-2677-46c5-b01e-298582982fa0`
`prospect_id` | (DEPRECATED; see `id`) The internal Faraday ID (UUID) for your newly submitted prospect. | `4a991134-2677-46c5-b01e-298582982fa0`
`household_id` | The internal Faraday ID (UUID) for the known household that the prospect matched to (if a match could be made). Note that Faraday household IDs are ephemeral and should be neither persisted nor relied upon; we include them for debugging purposes. | `2e231a5a-e3f7-4a09-b4fc-21289f7debcf`
`person` | Null unless provided in request. The name of record for the known household your prospect matched to, if such a match could be made. | `Michael Faraday`
`attributes` | Requested household-level attributes, if any were provided and a match could be made. | `{ "household_income": 110000, "credit_rating": 690 }`
`predict` | A number between -1 and 1 indicating the likelihood that the matched household (if such a match was possible) will purchase the product specified in the request. Positive values indicate that the household is predicted to purchase; negative values indicate the the household is predicted to reject. The absolute value of the number indicates Faraday's confidence in its prediction. A missing response value indicates that a prediction was impossible. | `0.89`
`qualify` | A boolean indicating whether the matched household (if any) is included in the segment specified in your request (if provided). | `true`
`disqualifications` | The reason for a false value in the `qualify` response value. Only included when `qualify` is false. | `"household_income: expected between 50000 and Infinity, but outside"`

### Upload a file

This endpoint can be used to submit bulk CSV data for managed ingestion, such as customer data, marketing lists, and points of interest. A Faraday analyst will inspect your file and may contact you for clarification.

#### Request

```http
POST https://api.faraday.io/v1/files
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
Content-Type: application/json
Accept: application/json
```

#### Parameters

Required parameters in **bold**.

Parameter | Type | Description | Example
----------|------|-------------|--------
**`url`** | *String* | Publicly accessible URL to the CSV file | `https://example.com/mydata.csv`
**`email`** | *String* | An email address of a valid Faraday user for use in clarification after ingestion | `some.user@your_company.com`

#### Response

Top-level key | Value description | Example
--------------|-------------------|--------
`id` | The internal Faraday ID (UUID) for your newly submitted file. | `4a991134-2677-46c5-b01e-298582982fa0`

### List household attributes

**Status:** Live

Faraday maintains data on each U.S. household, as many as hundreds of attributes depending on local coverage. This list of attributes changes frequently as new data sources are added.

#### Request

```http
GET https://api.faraday.io/v1/household_attributes
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
Accept: application/json
```

#### Parameters

None.

#### Response

The response is a JSON array, each element of which is an object with the following structure:

Top-level key | Description | Example
--------------|-------------|--------
`name`        | The `snake_case` identifier for the attribute | `household_income`
`description` | A short human-readable description of the attribute | `Household income`
`type`        | Either `continuous` (numeric), `boolean`, or an array of valid categories (each a string) for categorical attributes | `continuous`

### List products

**Status:** Live

Faraday maintains an internal database of products (e.g. "Solar," "Life insurance") to facilitate predictive model creation. Some API requests require the ID of a product; use this request to find that ID.

#### Request

```http
GET https://api.faraday.io/v1/products
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
Accept: application/json
```

#### Parameters

None.

#### Response

The response is a JSON array, each element of which is an object with the following structure:

Top-level key | Description | Example
--------------|-------------|--------
`id`  | The internal ID of the product. | `07224389-ae7a-4b0e-b3c8-49e110e9c285`
`name`        | The `Sentence case` name of the product. | `Life insurance`
