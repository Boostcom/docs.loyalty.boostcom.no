# Api V2 reference (experimental)

You can use Api V2 fully, only when your schema is ready to support it (*coming soon*).

Please don't use it before contacting with us.

## Get loyalty club's schema

```shell
curl "https://connect.bstcm.no/api/v2/loyalty_clubs/:loyalty_club_slug/member_schema"
  -H "X-Customer-Public-Token: alphanumeric_string"
  -H "X-Product-Name: custom-product-name"
```

> When successful, the above command returns JSON structured like this:

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type": "object",
  "languages": ["en", "no"],
  "default_language": "no",
  "version": "v2",
  "properties": {
    "first_name": {
      "type": "string"
    },
    "last_name": {
      "type": "string"
    },
    "birthday": {
      "type": "string",
      "format": "date"
    },
    "interests": {
      "type": "array",
      "items": {
        "type": "string",
        "enum": [
          "bikes_and_cars",
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

Keys explanation:

* `languages` - array of languages set up in loyalty club
* `default_language` - default language used in loyalty club and also used for mappings to Api V1
* `version` - version of schema, currently the newest is `v2`

### HTTP Request

**GET** `https://connect.bstcm.no/api/v2/loyalty_clubs/:loyalty_club_slug/member_schema`

Parameter | Description
--------- | -------
loyalty_club_slug | unique slugified name of the loyalty club. Example: `boosters`.

<aside class="notice">
Authentication only with <code>X-Customer-Public-Token</code>.
</aside>

## Check if member exists

```shell
curl -I "https://connect.bstcm.no/api/v2/loyalty_clubs/:loyalty_club_slug/members/:msisdn"
  -H "X-Customer-Public-Token: alphanumeric_string"
  -H "X-Product-Name: custom-product-name"
```

> Response will be 200 or 404 code.

It can be used to check if member is in loyalty club or not.

### HTTP Request

**HEAD** `https://connect.bstcm.no/api/v1/loyalty_clubs/:loyalty_club_slug/members/:msisdn`

### URL Parameters

Parameter | Description
--------- | -----------
loyalty_club_slug | unique slugified name of the loyalty club. Example: `boosters`.
msisdn | unique member's msisdn as defined by E.164 (described above) Example: `4740485124`.

### Response codes

* **200** - member exists
* **404 Not found** - member with `msisdn` not found or it is invalid

<aside class="notice">
Authentication with <code>X-Customer-Public-Token</code>.
</aside>

## Get member

```shell
curl "https://connect.bstcm.no/api/v2/loyalty_clubs/:loyalty_club_slug/members/:msisdn"
  -H "X-Customer-Private-Token: alphanumeric_string"
  -H "X-Product-Name: custom-product-name"
```

> When successful, the above command returns JSON structured like this:

```json
{
  "properties": {
    "first_name": "Ola",
    "last_name": "Nordmann",
    "birthday": "1990-10-23",
    "interests": [
      "bikes_and_cars",
      "sportwear"
    ],
    "child_birth_years": [
      2010,
      2011,
      2011
    ]
  }
}
```

Fetches member's properties.

### HTTP Request

**GET** `https://connect.bstcm.no/api/v1/loyalty_clubs/:loyalty_club_slug/members/:msisdn`

### URL Parameters

Parameter | Description
--------- | -----------
loyalty_club_slug | unique slugified name of the loyalty club. Example: `boosters`.
msisdn | unique member's msisdn as defined by E.164 (described above) Example: `4740485124`.

### Responses

* **200** - success with member's properties in response body
* **404 Not found** - member with `msisdn` not found or it is invalid

<aside class="notice">
Authentication with <code>X-Member-Token</code> and <code>X-Customer-Private-Token</code>.
</aside>

## Create member

Create member with given properties.

When invalid authentication token is provided response `401 Unauthorized` is returned disregarding whether member exists or not.

### HTTP Request

**PUT** `https://connect.bstcm.no/api/v1/loyalty_clubs/:loyalty_club_slug/members/:msisdn`

### URL Parameters

Parameter | Description
--------- | -----------
loyalty_club_slug | unique slugified name of the loyalty club. Example: `boosters`.
msisdn | unique member's msisdn as defined by E.164 (described above) Example: `4740485124`.

### PUT (JSON) Parameters

Parameter | Description | Type
--------- | ----------- | ---------
properties | JSON with properties for member | JSON Object
send_welcome_message | If true, SMS welcome message will be send to member | Boolean
send_email_welcome_message | If true and emails configured in loyalty club, email welcome (verification) message will be send to member | Boolean

### Responses

* **200** - success with member's properties in response body
* **400 Bad request** - `msisdn` is invalid
* **409 Conflict** - member with `msisdn` already exists in loyalty club

<aside class="notice">
Authentication with <code>X-Member-Token</code> and <code>X-Customer-Private-Token</code>.
</aside>

## Update member

Update member's properties with given ones.

It is intended for partial updates - not given properties are neither deleted nor overwritten.

### HTTP Request

**PATCH** `https://connect.bstcm.no/api/v1/loyalty_clubs/:loyalty_club_slug/members/:msisdn`

### URL Parameters

Parameter | Description
--------- | -----------
loyalty_club_slug | unique slugified name of the loyalty club. Example: `boosters`.
msisdn | unique member's msisdn as defined by E.164 (described above) Example: `4740485124`.

### PATCH (JSON) Parameters

Parameter | Description | Type
--------- | ----------- | ---------
properties | JSON with properties for member | JSON Object

### Responses

* **200** - success with member's properties in response body
* **404 Not found** - member with `msisdn` not found or it is invalid


<aside class="notice">
Authentication with <code>X-Member-Token</code> and <code>X-Customer-Private-Token</code>.
</aside>

## Remove member

Removes member from loyalty club.

It is possible to trigger optout message by setting `send_unsubscribe_message=true` query string parameter.

### HTTP Request

**DELETE** `https://connect.bstcm.no/api/v1/loyalty_clubs/:loyalty_club_slug/members/:msisdn`

### URL Parameters

Parameter | Description
--------- | -----------
loyalty_club_slug | unique slugified name of the loyalty club. Example: `boosters`.
msisdn | unique member's msisdn as defined by E.164 (described above) Example: `4740485124`.

### Query Parameters

Parameter | Description | Type
--------- | ----------- | ---------
send_unsubscribe_message | If true, optout message will be send to member. Example: `true`. | Boolean

### Responses

* **200** - success with member's properties in response body
* **404 Not found** - member with `msisdn` not found or it is invalid

<aside class="notice">
Authentication with <code>X-Member-Token</code> and <code>X-Customer-Private-Token</code>.
</aside>