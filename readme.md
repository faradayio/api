## API documentation

**Base URL:** `https://api.faraday.io/v3`

### Households: data append and segment membership

#### Endpoint

`POST /households`

#### Response codes

* **200** OK
* **404** Household could not be found

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

  * `allow_reverse_email` _"true" or omit_ — Allow Faraday to attempt an email match if matching by physical address fails. **You may incur charges.**
  * `allow_reverse_phone` _"true" or omit_ — Allow Faraday to attempt a phone match if matching by physical address fails. **You may incur charges.**
  * `match_algorithm` _"loose", "tight", or omit_ — By default, Faraday will match a given identity when lastname, normalized address, and postcode match. Tight mode, on the other hand, also requires a firstname match. Choose loose mode to ignore name and match on address only.

##### Operations

  * `audiences` _Array of UUID Strings_ — Check to see if the matched household falls within each of the specified Audiences. Each specified Audience must have been previously created with Explore.
  * `attributes` _Array of Strings_ — Append the specified FIG attributes, each identified by its handle.

##### Response settings
  * `prefix` _String_ — Prefix each standard response key with the specified string.
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
  * `match_algorithm` _"loose", "tight", or omit_ — By default, Faraday will match a given identity when normalized address and postcode match exactly and lastname matches closely. Tight mode, on the other hand, requires an exact lastname match. Choose loose mode to ignore name and match on address only.
  * `attributes` _Hash_ — Each key is the handle of a requested FIG attribute. Each corresponding value is that attribute extracted from FIG.
  * `audiences` _Hash_ — Each key is the UUID of a requested Audience. Each corresponding value is a boolean indicating whether the household does or does not belong to that Audience.
  * `warnings` _Array of Strings_ — Each warning is a human-interpretable message indicating an issue with the API request.
  * `error` _String_ — Error message.

### Scores

#### Endpoint

`POST /scores`

#### Response codes

* **200** OK
* **422** Household could not be found

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

  * `allow_reverse_email` _"true" or omit_ — Allow Faraday to attempt an email match if matching by physical address fails. **You may incur charges.**
  * `allow_reverse_phone` _"true" or omit_ — Allow Faraday to attempt a phone match if matching by physical address fails. **You may incur charges.**
  * `match_algorithm` _"loose", "tight", or omit_ — By default, Faraday will match a given identity when lastname, normalized address, and postcode match. Tight mode, on the other hand, also requires a firstname match. Choose loose mode to ignore name and match on address only.

##### Operations

  * `outcome_id` _UUID String_ — Use the specified Outcome's currently promoted Model to score the matching household.
  * `campaign_id` _UUID String_ **Deprecated** — Use the specified Campaign's currently promoted Model to score the matching household.

##### Response settings

  * `prefix` _String_ — Prefix each standard response key with the specified string.
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
  * `match_algorithm` _String_ — For internal use. Please report this value if you are having matching problems.
  * `score` _Float_ — The probability that the matched household will achieve the indicated Outcome/Campaign.
  * `warnings` _Array of Strings_ — Each warning is a human-interpretable message indicating an issue with the API request.
  * `error` _String_ — Error message.

## Copyright

Copyright 2018 Faraday
