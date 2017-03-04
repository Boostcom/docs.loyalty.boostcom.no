---
title: Boostcom Loyalty API Reference

language_tabs:
  - shell

toc_footers:
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

It's an API to manage loyalty club's members. It can be accessed from both Boostcom's internal server applications as well as from native mobile applications as well as external client's server applications.

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
* `X-Customer-Private-Token` can be used for batch operations on any member of the customer's loyalty club. It could be used for all operations that allow authentication with `X-Customer-Public-Token`. It should be used only on backend and never exposed in frontend code.

If you miss your authentication tokens, please [let us know](http://boostcom.no).

<aside class="notice">
You must use only one header in each request.
</aside>

## Product name

Each system that is communicating with us should uniqly identify itself so it is possible to distinguish optin/update channels. That will allow further targeting members by communication channel. For that we use product name and header `X-Product-Name` is intended to provide the necessary granularity.

If you miss your product name, please [let us know](http://boostcom.no).

<aside class="warning">
Product Name header is required in each request!
<br>
When it is missing or it has incorrect value, then <code>401 Unauthorized</code> HTTP code and empty response body are returned.
</aside>

## Msisdn - member identifier

Members in our systems are identified with their phone numbers (msisdns) according to [E.164](https://en.wikipedia.org/wiki/E.164) We use format without leading `00` or `+` and without spaces, so that it contains only digits, i.e. `4740485124`.

There is no special handling in case it has wrong format, except for creating a member and sending a token. `404 Not Found` is returned and empty response body unless stated otherwise.

## Registration options

New members could be created with or without password confirmation depending on which token is being used on *Create member* action.

*Send token* action will trigger sending member token/password to provided msisdn no matter whether msisdn is already a member or not.

# Api V1 reference

## Notice

Api V1 endpoints that are in Api V2 will be soon deprecated. It will always work, but it's not advised to use it.

Main goal of Api V2 is to support internationalized member properties.

You can use Api V2 only when your schema is ready to support it (*coming soon*).

## Schema

```shell
curl "https://connect.bstcm.no/api/v1/loyalty_clubs/:loyalty_club_slug/member_schema"
  -H "X-Customer-Public-Token: alphanumeric_string"
  -H "X-Product-Name: custom-product-name"
```

> The above command returns JSON structured like this:

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type": "object",
  "properties": {
    "first_name": {
      "title": "Fornavn",
      "type": "string"
    },
    "last_name": {
      "title": "Etternavn",
      "type": "string"
    },
    "birthday": {
      "title": "Fødselsdato",
      "type": "string",
      "format": "date"
    },
    "interests": {
      "title": "Interesser",
      "type": "array",
      "items": {
        "type": "string",
        "enum": [
          "bikes & cars",
          "sportwear"
        ]
      }
    }
  },
  "required": [
    "first_name",
    "last_name",
    "birthday"
  ]
}
```

Properties for members are defined as part of the loyalty club they belong to.
To describe member properties we use [JSON Schema](http://json-schema.org/documentation.html) definition.
Properties of each member must conform to the defined schema.
We support **JSON schema Draft V4** with format extension for `date` (YYYY-MM-DD).

### HTTP Request

`GET https://connect.bstcm.no/api/v1/loyalty_clubs/:loyalty_club_slug/member_schema`

Parameter | Description
--------- | -------
:loyalty_club_slug | unique slugified name of the loyalty club. Example: `boosters`.

<aside class="notice">
Authentication only with <code>X-Customer-Public-Token</code>.
</aside>

## Member tokens

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

**Member tokens** are used to authenticate actions on particular members.
**Member token** could be issued even before registering enduser as a member in community.
In that case we create temporary token that is valid till end of day.
That **member token** could be used to authenticate *Create Member* action described below.

### HTTP Request

`POST https://connect.bstcm.no/api/v1/loyalty_clubs/:loyalty_club_slug/members/:msisdn/send_token`

### URL Parameters

Parameter | Description
--------- | -----------
:loyalty_club_slug | unique slugified name of the loyalty club. Example: `boosters`.
:msisdn | unique member's msisdn as defined by E.164 (described above) Example: `4740485124`.

<aside class="notice">
Authentication with <code>X-Customer-Public-Token</code> or <code>X-Customer-Private-Token</code>.
</aside>