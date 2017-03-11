---
title: Boostcom Loyalty API Reference

toc_footers:
  - <a href='https://loyalty.boostcom.no'>Boostcom Loyalty</a>
  - <a href='http://boostcom.com/'>Boostcom</a>
  - <hr/>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - api_v1
  - api_v2

search: true
---

# Introduction

It's an API to manage loyalty club's members.
It can be accessed from both Boostcom's internal server applications as well as from native mobile applications as well as external client's server applications.

# Global settings

## Host

API host is https://connect.bstcm.no/

## Content-Type

This API supports `application/json` only. 

Both payload and response body are supposed to be a valid JSON so when making a request send a `Content-Type: application/json` header.

If payload is not a valid JSON, then `406 Not Acceptable` HTTP code and empty response body are returned.

## Authentication

All of the endpoints require a single authentication header.

* `X-Member-Token` represents SMS pass code sent to the member and allows to manage his own profile only. To trigger that SMS use `/send_token` call.
* `X-Customer-Public-Token` is used for endpoints that are not sensitive (not destructive and do not leak any personal data). Currently:
    + Get schema
    + Send token
    + Check if member exists
* `X-Customer-Private-Token` can be used for batch operations on any member of the customer's loyalty club.
It could be used for all operations that allow authentication with `X-Customer-Public-Token`.
It should be used only on backend and never exposed in frontend code.

If you miss your authentication tokens, please [let us know](http://boostcom.no).

<aside class="notice">
You must use only one header in each request.
</aside>

## Product name

Each system that is communicating with us should uniquely identify itself so it is possible to distinguish optin/update channels.
That will allow further targeting members by communication channel.
For that we use product name and header `X-Product-Name` is intended to provide the necessary granularity.

If you miss your product name, please [let us know](http://boostcom.no).

<aside class="warning">
Product Name header is required in each request!
<br>
When it is missing or it has incorrect value, then <code>401 Unauthorized</code> HTTP code and empty response body are returned.
</aside>

## Msisdn - member identifier

Members in our systems are identified with their phone numbers (msisdns) according to [E.164](https://en.wikipedia.org/wiki/E.164)
We use format without leading `00` or `+` and without spaces, so that it contains only digits, i.e. `4740485124`.

There is no special handling in case it has wrong format, except for creating a member and sending a token.
`404 Not Found` is returned and empty response body unless stated otherwise.

## Registration options

New members could be created with or without password confirmation depending on which token is being used on *Create member* action.

*Send token* action will trigger sending member token/password to provided msisdn no matter whether msisdn is already a member or not.

# Validation on members

All members' endpoints have properties validation. Properties are validated to conform loyalty club's schema.

If something is wrong, you will receive explanation of errors in response.

<aside class="warning">
Errors format and strings are not yet stable and may change without further notice.
</aside>

## Schema validation errors

```shell
curl -X PUT -H "X-Product-Name: custom-product-name" \
    -H "X-Customer-Private-Token: token" \
    -H "Content-Type: application/json" -d '{
	"properties": {
		"first_name": "",
		"last_name": "err",
		"gender": "man",
		"birthday": "201X-01-01"
	}
}' "https://tbp.bstcm.no/api/v2/loyalty_clubs/:loyalty_club_slug/members/:msisdn"
```

> Example response (with code 400):

```json
{
  "errors": {
    "properties": {
      "birthday": [
        {
          "error": "invalid_date_format",
          "property": "birthday"
        }
      ],
      "gender": [
        {
          "error": "value_not_match",
          "property": "gender",
          "value": "man",
          "values": "Mann, Kvinne"
        }
      ],
      "first_name": [
        {
          "error": "not_contain_required_property",
          "property": "first_name"
        }
      ]
    }
  }
}
```

Error | Description
-------|------------
invalid_date_format | Invalid date format
invalid_time_format | Invalid time format
invalid_date_time_format | Invalid date with time format
must_be_valid_RFC3339_date_time_string | Format beyond the RFC3339 standard
invalid_URI | Invalid URI
additional_array_elements | Additional elements in the array
additional_properties | Additional properties
property_not_match_all_of | The property did not match all of the required schemas
property_not_match_any_of | The property did not match any of the required schema
property_matched_more_than_one | The property matched more than one of the required schemas
depends_on_a_missing_property | The validated property has a property that depends on the other missing property
value_not_match | The property value did not match one of the given values.
schema_cannot_be_found | The extended schema cannot be found
not_a_valid_schema | The property was not a valid schema
minimum_string_length | The property was not of a minimum string length of minimum limit.
maximum_string_length | The property was not of a maximum string length of maximum limit.
less_item_than_minimum | The property did not contain a minimum number of minimum items limit 
more_item_than_maximum | The property had more items than the allowed items limit
less_properties_than_minimum | The property did not contain a minimum number of properties
more_properties_than_maximum | The property had more properties than allowed
not_have_value_of_exclusively | The property did not have value of exclusively
not_have_value_of_inclusively | The property did not have value of inclusively
more_decimal_places_than_maximum | The property had more decimal places than the allowed maximum
matched_the_disallowed_schema | The property matched the disallowed schema
the_regex_not_match | The property did not match the regex 
not_contain_required_property | The validated property did not contain a required property
contained_undefined_properties | The validated property contained undefined properties
referenced_schema_cannot_be_found | The referenced schema cannot be found
invalid_schema | The property was not a valid schema
matched_one_or_more_types | The property matched one or more of the given types
one_or_more_types_not_match | The property did not match one or more of the given types
type_not_match | The property did not match the given type
contained_duplicated_array_values | The property contained duplicated array values
invalid_email | Invalid email format
disposable_email | The email was [disposable](https://github.com/lisinge/valid_email2/blob/master/vendor/disposable_emails.yml)
duplicated_email | The email was duplicated in community
