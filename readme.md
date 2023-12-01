## Faraday Inform API (legacy)

**Base URL:** `https://api.faraday.ai/v3`

## Difference between Faraday Inform API (v3) and Faraday API (v1)

This document refers to the Faraday Inform API (knowable by prefix `v3`), a read-only legacy product with a maximum rate of 5 calls per second per API token.

The [Faraday API](https://faraday.ai/developers/reference) (knowable by prefix `v1` even though it is more modern) is a different product with different capabilities.

<table>
<tr>
<th>Feature</th>
<th>Faraday API (v1)</th>
<th>Legacy Faraday Inform API (v3) - this document</th>
</tr>
<tr>
<td>Configure Faraday resources</td>
<td>✅</td>
<td></td>
</tr>
<tr>
<td>Read predictions up to 100/second</td>
<td>✅</td>
<td></td>
</tr>
<tr>
<td>Read predictions up to 5/second</td>
<td></td>
<td>✅</td>
</tr>
<tr>
<td>Get probability in addition to percentile</td>
<td>✅</td>
<td></td>
</tr>
<tr>
<td><a href="https://docs.google.com/document/d/1nThkUeqJROPjJEs8E9zy4vnzRajLQIV66SiHmujT-z0/edit#heading=h.7k6rycn881a">Faraday Salesforce Lightning Connector</a></td>
<td></td>
<td>✅</td>
</tr>
<tr>
<td>Return average conversion rates per Outcome</td>
<td></td>
<td>✅</td>
</tr>
</table>

### Households: scoring, persona assignment, data append, and segment membership

#### Endpoint

`POST /v3/households` or `GET /v3/households`

#### Response codes

- **200** OK
- **404** Household could not be found

#### Request parameters

##### Auth

HTTP Basic Authentication is the preferred method.

- `username` — empty
- `password` — Your account's API key

You can also put the API key in the parameters as `api_key` if that's easier.

##### Identity

- `person_first_name` _String_ — First name (if known).
- `person_last_name` _String_ — Last name (if known).
- `house_number_and_street` _String_ — Physical address including number and street.
- `city` _String_ — City.
- `state` _String_ — 2-letter postal abbreviation.
- `postcode` _String_ — 5-digit zipcode. Send as string to preserve leading zeroes.
- `phone` _String_ — E.123-compliant string representation.
- `email` _String_ — E-mail address.

##### Matching settings

- `match_algorithm` _"loose", "tight", or omit_ — By default, Faraday will match a given identity when lastname, normalized address, and postcode match. Tight mode, on the other hand, also requires a firstname match. Choose loose mode to ignore name and match on address only.
- `allow_reverse_email` _"true" or omit_ **Deprecated** — Ignored. Reverse email is always attempted if email provided.
- `allow_reverse_phone` _"true" or omit_ **Deprecated** — Ignored. Reverse phone is an account-level setting that cannot be changed for individual lookups.

##### Operations

- `outcome_ids` _Array of UUIDs_ — Use the specified Outcomes to score the matching household.
- `persona_set_ids` _Array of UUIDs_ — Use the specified Persona Sets to assess the matching household.
- `outcome_id` _UUID String_ **Deprecated**— Use the specified Outcome to score the matching household. Use `outcome_ids` instead.
- `persona_set_id` _UUID String_ **Deprecated**— Use the specified Persona Set to score the matching household. Use `persona_set_ids` instead.
- `campaign_id` _UUID String_ **Deprecated** — Use the specified Campaigns to score the matching household.
- `audiences` _Array of UUID Strings_ **Deprecated** — Check to see if the matched household falls within each of the specified Audiences. Each specified Audience must have been previously created with Explore.
- `attributes` _Array of Strings_ **Deprecated** — Append the specified FIG attributes, each identified by its handle.

##### Response settings

Callers can specify a `prefix` and/or `postback_url`, _or_ a configuration for posting to Hubspot. In order to post to Hubspot, we require both a `vid` and a configuration of fields to post.

- `include_average_conversion_rates` **Boolean** — Enable returning average conversion rates.
- `prefix` _String_ — Prefix each standard response key with the specified string.
- `postback_url` _String_ — In addition to the standard HTTP response, also POST the response to the specified URL.
- `hubspot` _Object_ — A mapping of `fdy_field_name` to `hubspot_field_name`. For example:
  ```js
    {
      'persona_name': 'hb_persona_name',
      'persona_id': 'hb_persona_id',
      'house_number_and_street': 'hb_house_num'
    }
  ```
- `vid` _String_ — ID of the hubspot customer to update with fields in `hubspot` object. The Hubspot webhook provides this automatically.

#### Response

##### Elements

- `attributes` _Object_ — Each key is the handle of a requested FIG attribute. Each corresponding value is that attribute extracted from FIG.
- `audiences` _Object_ — Each key is the UUID of a requested Audience. Each corresponding value is a boolean indicating whether the household does or does not belong to that Audience.
- `city` _String_ — Normalized from request.
- `email` _String_ — Passed through from request.
- `error` _String_ — Error message.
- `house_number_and_street` _String_ — Normalized from request.
- `latitude` _Float_ — Decimal geocoded latitude.
- `longitude` _Float_ — Decimal geocoded longitude.
- `match_algorithm` _"loose", "tight", or omit_ — Passed through from request.
- `match_code` _String_ — Match code.
- `person_first_name` _String_ — Passed through from request.
- `person_last_name` _String_ — Passed through from request.
- `postcode` _String_ — Normalized from request.
- `state` _String_ — Normalized from request.
- `persona_sets` _Object_ - Each key is a Persona Set ID. Each corresponding value is an Object containing a Persona ID and a Persona Name.
- `scores` _Object_ — Each key is an Outcome ID. Each corresponding value is the score.
- `score_percentiles` _Object_ — Each key is an Outcome ID. Each corresponding value is the score percentile (if available).
- `warnings` _Array of Strings_ — Each warning is a human-interpretable message indicating an issue with the API request.
- `average_conversion_rates` _Object_ — Each key is an Outcome ID. Each corresponding value is its average conversion rate (if available). `include_average_conversion_rates` must be set to true.

#### Deprecated elements

Still supported.

- `persona_id` _String_ — ID of the persona that individual belongs to. Requires personas. Talk to your CSM if this is not in the response.
- `persona_name` _String_ — Name of the persona that individual belongs to. Requires personas. Talk to your CSM if this is not in the response.
- `score` _Float_ — The probability that the matched household will achieve the indicated Outcome/Campaign.
- `score_percentile` _Float_ — Score percentile within the cross-validation dataset (if available).

### Scores

#### Endpoint

`POST /v3/scores` or `GET /v3/scores`

Deprecated. Use `/v3/households` instead, with the same inputs and outputs.

### Match codes

All endpoints return a `match_code` of the form `oFLX`. Each letter stands for something.

- `F` — first name used
- `L` — last name used
- `P` — full name used
- `N` — nickname used (e.g. Bill matching to William)
- `E` — exact address used
- `X` — address prefix used (e.g., 123 N Blount St matching to 123 N Blount St Apt 403... it's a prefix)

The letters `i` (tight), `o` (default), and `a` (loose) refer to the match algorithm, but this can be seen more easily from the `match_algorithm` return value.

Examples:

- `oP-E` — Default mode full name exact address match. "Seamus Abshere 1038 E Dayton St" matched "Shamus Abshere 1038 E Dayton St".
- `oFLX` — Default mode first and last name address prefix match. "Devin/Abshere 123 N Blount St" matched to "Devon/Abshere 123 N Blount Apt 403".
- `a-LX` — Loose mode last-name only prefix match. "Seamus Abshere 123 N Blount St" matched to "Devin Abshere 123 N Blount Apt 403".

## Copyright

Copyright 2023 Faraday
