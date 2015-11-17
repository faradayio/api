# Faraday API

* [Overview](#overview)
  * [Versions](#versions)
  * [Contributing](#contributing)
* [Basics](#basics)
  * [Environments](#environments)
  * [URLs](#urls)
  * [Authentication](#authentication)
  * [Parameters](#parameters)
  * [Special request value types](#special-request-value-types)
    * [Segment specification](#segment-specification)
    * [Geography](#geography)
    * [Attribute condition](#attribute-condition)
  * [Responses](#responses)
* [Requests](#requests)
  * [Retrieve a campaign](#retrieve-a-campaign)
  * [Create a campaign](#create-a-campaign)
  * [Retrieve an audience](#retrieve-an-audience)
  * [Create an audience](#create-an-audience)
  * [Record a customer](#record-a-customer)
  * [Record a lead](#record-a-lead)
  * [Record a prospect](#record-a-prospect)
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

* The **test** environment also uses production data, but will not make any changes, returns partially elided responses, and does not incur charges. Use of your **test API key** will perform API operations on the test environment.

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

In your Faraday settings, go to the [Integrations section](https://app.faraday.io/#/settings/integrations). You will see two API tokens provided, one for the production environment, and one for test. See the section on Environments for more information.

Authentication to the API occurs via HTTP Basic Auth. Provide your API key as the basic auth username. You do not need to provide a password.

```shell
$ curl https://api.faraday.io/v1/campaigns \
  -u sk_test_E6V2sxvyYD5PX8a1tqqNE3eG:
```

### Parameters

The Faraday API accepts parameters in the query string (for GET-style requests) and as JSON in the request body (for POST-style requests). When sending JSON, don't forget to include `Content-Type: application/json` in your headers.

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

The format for each of the supported geography types:

* **Named place**: `{ "type": "place", "id": "01234" }`. `id` must be a string to allow for leading zeroes.
* **Zip code**: `{ "type": "zipcode", "id": "05401" }`
* **Radius around customers** `{ "type": "customer_proximity", "distance": 1000 }`. `distance` is radius in meters.
* **Radius around points of interest** `{ "type": "poi_proximity", "distance": 1000, "points_of_interest": [123, 456]}`
* **Custom shape** `{ "type": "custom_shape", "shape": {...} }`. The provided `shape must be valid [GeoJSON](http://geojson.org/).

#### Attribute condition

An *attribute condition* is generally one element of a JSON object listing several household criteria. Each such element must conform to the format appropriate for the [household attribute](#list-household-attributes) in question:

* **Continuously variabled attributes** are specified with a range: `"household_income": [80000, 120000]`. For unbounded ranges, use `"-Infinity"` (no minimum) and `"Infinity"` (no maximum).

* **Boolean attributes**: `"gardener": false`.

* **Categorical attributes** are specified with an array of valid desired categories: `"house_style": ["Ranch", "Townhouse"]`. Households must belong to *one* of the listed categories to meet the condition.

### Responses

All responses will arrive as JSON. Your requests should include the `Accept: application/json` header to acknowledge this.

## Requests

* [Retrieve a campaign](#retrieve-a-campaign)
* [Create a campaign](#create-a-campaign)
* [Retrieve an audience](#retrieve-an-audience)
* [Create an audience](#create-an-audience)
* [Record a lead](#record-a-lead)
* [Record a prospect](#record-a-prospect)
* [List household attributes](#list-household-attributes)
* [List products](#list-products)

### Retrieve a campaign

A Faraday *campaign* is a method for *contacting* some or all members of an *audience* using a specific *channel*. Depending on the channel, your campaign can be *pushed* to an *execution vendor* or *downloaded* as a list, or both.

#### Request

The first type of request retrieves the campaign's details:

```http
GET https://api.faraday.io/v1/campaigns/57a257d0-752a-4a02-b039-93e56aeac77e
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
Accept: application/json
```

#### Parameters

None

#### Response

Top-level key | Value description | Example
--------------|-------------------|--------
`campaign_id`  | The internal Faraday ID (UUID) for this campaign. | `105b15a0-d539-4b13-861f-ccdb35b0b6ea`
`audience_id` | The internal Faraday ID (UUID) for the audience the campaign belongs to. | `84d64b50-4610-4b7c-ba27-76af665480a8`
`status` | The current state of the campaign as it builds: `pending`, `building`, or `ready` | `ready`
`channel` | The channel of the campaign (`postal`, `telephone`, `canvassing`, `email`, or `social`). | `social`
`size` | The size of the campaign. Only included for `ready` campaigns. | `500`
`list_url` | The URL to a downloadable list of this campaign's recipients, if available. Only included for `ready` campaigns. | `http://example.com/list.csv`
`response_endpoints` | A summary of response endpoints, if response tracking is enabled for this campaign. | `{ "phone": "+18024580441", "email": "sales@example.com", "web": "http://example.com/landingpage" }`
`tracking_domain` | The tracking domain selected for this campaign, if response tracking is enabled. | `acme.replyspot.com`
`variants` | A summary of A/B-style variants, if configured for this campaign. | `{ "A": "Funny", "B": "Serious" }`

Note that your code may need to poll this resource until it achieves `ready` status when attempting to retrieve the campaign's size or list URL.

### Create a campaign

#### Request

```http
POST https://api.faraday.io/v1/audiences/18b68e74-1953-4784-a37b-58e0c7d54961/campaigns
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
Content-Type: application/json
Accept: application/json
```

#### Parameters

Required parameters in **bold**.

Parameter | Type | Description | Example
----------|------|-------------|--------
`request_id` | *String* | An optional unique identifier which will be returned with the API response. (Consider generating a UUID.)
**`channel`** | *String* | The channel the campaign will be launched on: `postal`, `telephone`, `canvassing`, `email`, or `social` | `social`
**`maximum_size`** | *Integer* | The maximum size desired for the campaign (allows for unpredictable reachability). | `500`
**`response_endpoints`** | *Object* | How campaign recipients would respond. Faraday uses this information to create unique tracking contact information that redirects to these endpoints. | `{ "phone": "+18024580441", "email": "sales@example.com", "web": "http://example.com/landingpage" }`
**`tracking_domain`** | *String* | A domain to use for branding tracking URLs and email addresses. Specify a built-in Replyspot domain like `anything.replyspot.com` (the default), or use a custom domain like `mycampaign.mycompany.com` and add the corresponding CNAME record to your DNS (`mycampaign.mycompany.com. CNAME campaigns.faraday.io`) | `acme.replyspot.com`
`variants` | *Array of strings* | A/B-style variants. If included, each recipient will be assigned randomly to one of these variants. | `["Funny", "Serious", "Casual"]`

#### Response

Top-level key | Value description | Example
--------------|-------------------|--------
`campaign_id`  | The internal Faraday ID (UUID) for this campaign. | `105b15a0-d539-4b13-861f-ccdb35b0b6ea`
`audience_id` | The internal Faraday ID (UUID) for the audience the campaign belongs to. | `84d64b50-4610-4b7c-ba27-76af665480a8`
`status` | The current state of the campaign as it builds: `pending`, `building`, or `ready` | `building`
`channel` | The channel of the campaign (`postal`, `telephone`, `canvassing`, `email`, or `social`). | `social`
`size` | The size of the campaign. Only included for `ready` campaigns. | `500`
`list_url` | The URL to a downloadable list of this campaign's recipients, if available. Only included for `ready` campaigns. | `http://example.com/list.csv`
`response_endpoints` | A summary of response endpoints, if response tracking is enabled for this campaign. | `{ "phone": "+18024580441", "email": "sales@example.com", "web": "http://example.com/landingpage" }`
`tracking_domain` | The tracking domain selected for this campaign, if response tracking is enabled. | `acme.replyspot.com`
`variants` | A summary of A/B-style variants, if configured for this campaign. | `{ "A": "Funny", "B": "Serious" }`

### Retrieve an audience

A Faraday *audience* is a fixed group of specific households within a geography that meet a set of criteria. Audiences are the basis for *campaigns* and must be created first.

#### Request

```http
GET https://api.faraday.io/v1/audiences/3bdbf545-d5d8-4ddb-b813-f954a1882cc2
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
Accept: application/json
```

#### Parameters

None.

#### Response

Top-level key | Value description | Example
--------------|-------------------|--------
`audience_id` | The internal Faraday ID (UUID) of the audience | `3bdbf545-d5d8-4ddb-b813-f954a1882cc2`
`status` | The audience's current state (`pending`, `building`, or `ready`) | `ready`
`segment` | The geography and criteria used to construct this audience, delivered as a [segment specficiation](#segment-specification) | `{ "geography": [ { "type": "place", "id": 1234 } ], "criteria": { "household_income": [80000, "Infinity"]} }`
`sizing_method` | A summary of the approach used to arrive at the audience's size | `{ "maximum_size": 10000, "quality_tolerance": 0 }`
`size` | The size of the audience—only available when `ready` | `9434`
`campaign_count` | The number of campaigns launched against this audience | `3`
`reachability` | A summary of the audience's reachability by channel | `{ "postal": 9434, "telephone": 4993, "email": 3542, "social": 4853 }`

### Create an audience

#### Request

```http
POST https://api.faraday.io/v1/audiences
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
Content-Type: application/json
Accept: application/json
```

#### Parameters

Required parameters in **bold**.

Parameter | Type | Description | Example
----------|------|-------------|--------
`request_id` | *String* | An optional unique identifier which will be returned with the API response. (Consider generating a UUID.)
**`segment`** | *Segment specification* | Describes the population segment from which to draw households when building this audience | `{ "geography": [ { "type": "place", "id": 1234 } ], "criteria": { "household_income": [80000, "Infinity"]} }`
**`name`** | *String* | A name for this audience | `Wealthy Brooklynites`
**`product`** | *String* | The ID of a Faraday product | `15c13b63-250b-4c24-96a5-955ccd7f3ae0`
`predict` | *Boolean* | Enables predictive targeting, which will select households in your segment with the highest likelihood of purchasing your product first | `true`
`maximum_size` | *Integer* | Faraday will add households from your segment to the audience (according to quality tolerance, if predictive targeting is enabled) until this size is reached | `10000`
`quality_tolerance` | *String* | With predictive targeting enabled, this controls the heuristic for adding households from your segment to this audience before stopping — with the default of `0`, the system will stop adding households once its confidence in the next-best household purchasing your product falls below 50%; with `1` the system will accept any households predicted to purchase, regardless of confidence, and with `2` the system will accept any household from the specified segment, regardless of predicted outcome | `1`
`minimum_size` | *Integer* | Ensures the audience will contain at least this many households, constrained only by the size of the segment itself and overriding the quality tolerance if necessary | `5000`


#### Response

Top-level key | Value description | Example
--------------|-------------------|--------
`request_id`  | The request ID passed through from your request, if given. | `105b15a0-d539-4b13-861f-ccdb35b0b6ea`
`audience_id` | The internal Faraday ID (UUID) for your newly created audience | `8c52b329-33b5-4d3a-b0ab-1aed32db633a`
`status` | The state of the audience as it undergoes its build: `pending`, `building`, `ready` | `building`
`segment` | The geography and criteria used to construct this audience, delivered as a [segment specficiation](#segment-specification) | `{ "geography": [ { "type": "place", "id": 1234 } ], "criteria": { "household_income": [80000, "Infinity"]} }`
`sizing_method` | A summary of the approach used to arrive at the audience's size | `{ "maximum_size": 10000, "quality_tolerance": 0 }`
`size` | The ultimate size of the audience—only available when `ready` | `9434`
`campaign_count` | The number of campaigns launched against this audience | `0`
`reachability` | A summary of the audience's reachability by channel | `{ "postal": 9434, "telephone": 4993, "email": 3542, "social": 4853 }`

Note that your code should poll the newly created audience until it reaches `ready` status before continuing with further actions on this audience, such as building campaigns.

### Record a customer

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
`request_id` | *String* | An optional unique identifier which will be returned with the API response. (Consider generating a UUID.)
**`person`** | *String* | The combined first and last name of the customer. | `Michael Faraday`
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
`request_id`  | The request ID passed through from your request, if given. | `105b15a0-d539-4b13-861f-ccdb35b0b6ea`
`customer_id` | The internal Faraday ID (UUID) for your newly submitted customer. | `4a991134-2677-46c5-b01e-298582982fa0`
`household_id` | The internal Faraday ID (UUID) for the known household that the customer matched to (if a match could be made). Note that Faraday household IDs are ephemeral and should be neither persisted nor relied upon; we include them for debugging purposes. | `2e231a5a-e3f7-4a09-b4fc-21289f7debcf`
`person` | The name of record for the known household your customer matched to, if such a match could be made. | `Michael Faraday`
`attributes` | Requested household-level attributes, if any were provided and a match could be made. | `{ "household_income": 110000, "credit_rating": 690 }`
`qualify` | A boolean indicating whether the matched household (if any) is included in the segment specified in your request (if provided). | `true`

### Record a lead

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
`request_id` | *String* | An optional unique identifier which will be returned with the API response. (Consider generating a UUID.)
**`person`** | *String* | The combined first and last name of the lead. | `Michael Faraday`
**`house_number_and_street`** | *String* | The physical address of the lead. | `123 Main St.`
`city` | *String* | The lead's city. | `Burlington`
`state` | *String* | The 2-letter postal abbreviation of the lead's state. | `VT`
**`postcode`** | *String* | The lead's 5-digit zip code. (Use a string to avoid issues with leading zeroes.) | `05402`
`became_lead_at` | *String* | A [ISO8601](https://en.wikipedia.org/wiki/ISO_8601) date/time string indicating when the lead emerged. Uses the current time if missing. | `20151025T223451Z`
`attributes` | *String* or *Array of strings* | Instructs Faraday to include (append) one or more attributes of the lead—if a match to a known household can be made. Each attribute should be a [valid Faraday household attribute](#list-household-attributes); unrecognized attributes will be ignored and added to the `Faraday-Unrecognized-Attributes` response header. | `["household_income", "credit_rating"]`
`qualify` | [*Segment specification*](#segment-specification) | Instruct Faraday to qualify this lead by determining its inclusion in the specified segment, if the lead can be matched with high confidence to a known Faraday household. | `{ "geography": [ { "type": "place", "id": 1234 } ], "criteria": { "household_income": [80000, "Infinity"]} }`

#### Response

Top-level key | Value description | Example
--------------|-------------------|--------
`request_id`  | The request ID passed through from your request, if given. | `105b15a0-d539-4b13-861f-ccdb35b0b6ea`
`lead_id` | The internal Faraday ID (UUID) for your newly submitted lead. | `4a991134-2677-46c5-b01e-298582982fa0`
`household_id` | The internal Faraday ID (UUID) for the known household that the lead matched to (if a match could be made). Note that Faraday household IDs are ephemeral and should be neither persisted nor relied upon; we include them for debugging purposes. | `2e231a5a-e3f7-4a09-b4fc-21289f7debcf`
`person` | The name of record for the known household your lead matched to, if such a match could be made. | `Michael Faraday`
`attributes` | Requested household-level attributes, if any were provided and a match could be made. | `{ "household_income": 110000, "credit_rating": 690 }`
`qualify` | A boolean indicating whether the matched household (if any) is included in the segment specified in your request (if provided). | `true`

### Record a prospect

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
`request_id` | *String* | An optional unique identifier which will be returned with the API response. (Consider generating a UUID.)
**`person`** | *String* | The combined first and last name of the prospect. | `Michael Faraday`
**`house_number_and_street`** | *String* | The physical address of the prospect. | `123 Main St.`
`city` | *String* | The prospect's city. | `Burlington`
`state` | *String* | The 2-letter postal abbreviation of the prospect's state. | `VT`
**`postcode`** | *String* | The prospect's 5-digit zip code. (Use a string to avoid issues with leading zeroes.) | `05402`
`became_prospect_at` | *String* | A [ISO8601](https://en.wikipedia.org/wiki/ISO_8601) date/time string indicating when the lead emerged. Uses the current time if missing. | `20151025T223451Z`
`attributes` | *String* or *Array of strings* | Instructs Faraday to include (append) one or more attributes of the prospect—if a match to a known household can be made. Each attribute should be a [valid Faraday household attribute](#options-households); unrecognized attributes will be ignored and added to the `Faraday-Unrecognized-Attributes` response header. | `["household_income", "credit_rating"]`
`qualify` | [*Segment specification*](#segment-specification) | Instruct Faraday to qualify this prospect by determining its inclusion in the specified segment, if the prospect can be matched with high confidence to a known Faraday household. | `{ "geography": [ { "type": "place", "id": 1234 } ], "criteria": { "household_income": [80000, "Infinity"]} }`

#### Response

Top-level key | Value description | Example
--------------|-------------------|--------
`request_id`  | The request ID passed through from your request, if given. | `105b15a0-d539-4b13-861f-ccdb35b0b6ea`
`prospect_id` | The internal Faraday ID (UUID) for your newly submitted prospect. | `4a991134-2677-46c5-b01e-298582982fa0`
`household_id` | The internal Faraday ID (UUID) for the known household that the prospect matched to (if a match could be made). Note that Faraday household IDs are ephemeral and should be neither persisted nor relied upon; we include them for debugging purposes. | `2e231a5a-e3f7-4a09-b4fc-21289f7debcf`
`person` | The name of record for the known household your prospect matched to, if such a match could be made. | `Michael Faraday`
`attributes` | Requested household-level attributes, if any were provided and a match could be made. | `{ "household_income": 110000, "credit_rating": 690 }`
`qualify` | A boolean indicating whether the matched household (if any) is included in the segment specified in your request (if provided). | `true`

### List household attributes

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
`product_id`  | The internal ID of the product. | `07224389-ae7a-4b0e-b3c8-49e110e9c285`
`name`        | The `Sentence case` name of the product. | `Life insurance`
