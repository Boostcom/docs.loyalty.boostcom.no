# Api V3 reference (draft, work in progress)

**This is only draft!**

You can use Api V3 only when your customer is migrated to new backend service for data.

## Get loyalty club's schema

```shell
curl "https://connect.bstcm.no/api/v3/loyalty_clubs/:loyalty_club_slug/member_schema" \
  -H "X-Customer-Public-Token: alphanumeric_string" \
  -H "X-Product-Name: custom-product-name"
```

> When successful, the above command returns JSON structured like this:

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type": "object",
  "identifiers": ["email"],
  "languages": ["en", "no"],
  "default_language": "no",
  "version": "v2",
  "properties": {
    "email": {
      "type": "string",
      "format": "email"
    },
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

* `identifiers` [Array of strings] - fields that are set to identify member
* `languages` [Array of strings] - array of languages set up in loyalty club
* `default_language` [string] - default language used in loyalty club and also used for mappings to Api v2
* `version` [string] - version of schema, currently the newest is `v2`
* `products` [Object] - properties scoping and ordering by product name, default one is `default`
* `required` [Array of strings] - required properties for member

### HTTP Request

**GET** `api/v3/loyalty_clubs/:loyalty_club_slug/member_schema`

Parameter | Description
--------- | -------
loyalty_club_slug | unique slugified name of the loyalty club. Example: `boosters`.

<aside class="notice">
Authentication only with <code>X-Customer-Public-Token</code>.
</aside>

## Send member token

**Member tokens** are used to authenticate actions on particular members.
**Member token** could be issued even before registering enduser as a member in community.
In that case we create temporary token that is valid till end of day.
That **member token** could be used to authenticate *Create Member* action described below.

It sends member token to the user via SMS.

It can be used multiple times.

If msisdn is not valid, then `400 Bad Request` is returned.

### HTTP Request

**POST** `api/v3/loyalty_clubs/:loyalty_club_slug/members/:msisdn/send_token`

### URL Parameters

Parameter | Description
--------- | -----------
loyalty_club_slug | unique slugified name of the loyalty club. Example: `boosters`.
msisdn | unique member's msisdn as defined by E.164 (described above) Example: `4740485124`.

<aside class="notice">
Authentication with <code>X-Customer-Public-Token</code> or <code>X-Customer-Private-Token</code>.
</aside>

## Check if member exists

```shell
curl -I "https://connect.bstcm.no/api/v3/loyalty_clubs/:loyalty_club_slug/members/:id" \
  -H "X-Customer-Public-Token: alphanumeric_string" \
  -H "X-Product-Name: custom-product-name"
```

> Response will be 200 or 404 code.

It can be used to check if member is in loyalty club or not.

### HTTP Request

**HEAD** `api/v3/loyalty_clubs/:loyalty_club_slug/members/:id`

**HEAD** `api/v3/loyalty_clubs/:loyalty_club_slug/members/by_msisdn/:msisdn`

**HEAD** `api/v3/loyalty_clubs/:loyalty_club_slug/members/by_email/:email`

### URL Parameters

Parameter | Description
--------- | -----------
loyalty_club_slug | unique slugified name of the loyalty club. Example: `boosters`.
id | member's ID.
msisdn | unique member's msisdn as defined by E.164 (described above). Example: `4740485124`.
email | unique member's email.

### Response codes

* **200** - member exists
* **404 Not found** - member with given identifier not found

<aside class="notice">
Authentication with <code>X-Customer-Public-Token</code>.
</aside>

## Get member

```shell
curl "https://connect.bstcm.no/api/v3/loyalty_clubs/:loyalty_club_slug/members/:id" \
  -H "X-Customer-Private-Token: alphanumeric_string" \
  -H "X-Product-Name: custom-product-name"
```

> When successful, the above command returns JSON structured like this:

```json
{
  "id": 42,
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
    ],
    "language": "no"
  },
  "sms_status": "enabled",
  "email_status": "hard_bounced",
  "push_status": "disabled",
  "created_at": "2017-01-19T10:07:08.336+01:00",
  "updated_at": "2017-04-03T09:35:19.313+02:00"
}
```

Fetches member's properties.

Key | Description | Type
--------- | ----------- | ---------
id | Member ID | integer
properties | Object with member's properties | JSON Object
properties\['language'\] | Language used by user | string
consents | [incoming] | JSON Object
sms_status | Status of sms channel | string
email_status | Status of email channel | string
push_status | Status of push channel | string
created_at | Time when the user was firstly created | string
updated_at | Time when the user was last updated | string

### HTTP Request

**GET** `api/v3/loyalty_clubs/:loyalty_club_slug/members/:id`

**GET** `api/v3/loyalty_clubs/:loyalty_club_slug/members/by_msisdn/:msisdn`

**GET** `api/v3/loyalty_clubs/:loyalty_club_slug/members/by_email/:email`

### URL Parameters

Parameter | Description
--------- | -----------
loyalty_club_slug | unique slugified name of the loyalty club. Example: `boosters`.
id | member's ID.
msisdn | unique member's msisdn as defined by E.164 (described above). Example: `4740485124`.
email | unique member's email.

### Responses

* **200** - success with member's properties in response body
* **404 Not found** - member with given identifier not found 

<aside class="notice">
Authentication with <code>X-Member-Token</code> and <code>X-Customer-Private-Token</code>.
</aside>

## Create member

Create member with given properties.

When invalid authentication token is provided response `401 Unauthorized` is returned disregarding whether member exists or not.

### HTTP Request

**POST** `api/v3/loyalty_clubs/:loyalty_club_slug/members`

### URL Parameters

Parameter | Description
--------- | -----------
loyalty_club_slug | unique slugified name of the loyalty club. Example: `boosters`.
id | member's ID

### POST (JSON) Parameters

Parameter | Description | Type
--------- | ----------- | ---------
properties | JSON with properties for member | JSON Object
properties\['language'\] | Language used by user, if not set then "default_language" is taken from schema | string
send_welcome_message | If true, SMS welcome message will be send to member | Boolean
send_email_welcome_message | If true and emails configured in loyalty club, email welcome (verification) message will be send to member | Boolean
sms_enabled | If true SMS channel will be enabled for member | Boolean
email_enabled | If true email channel will be enabled for member | Boolean
push_enabled | If true push channel will be enabled for member | Boolean

<aside class="notice">
There is possibility to have multiple SMS welcome messages. Sent will be the one that matches Product or default one.
</aside>

### Responses

* **200** - success with member's properties in response body
* **400 Bad request** - there are [validation errors](#validation-on-members).

<aside class="notice">
Authentication with <code>X-Member-Token</code> and <code>X-Customer-Private-Token</code>.
</aside>

## Update member

Update member's properties with given ones.

It is intended for partial updates - not given properties are neither deleted nor overwritten.

### HTTP Request

**PATCH** `api/v3/loyalty_clubs/:loyalty_club_slug/members/:id`

### URL Parameters

Parameter | Description
--------- | -----------
loyalty_club_slug | unique slugified name of the loyalty club. Example: `boosters`.
id | member's ID.

### PATCH (JSON) Parameters

Parameter | Description | Type
--------- | ----------- | ---------
properties | JSON with properties for member | JSON Object
sms_enabled | If true SMS channel will be enabled for member | Boolean
email_enabled | If true email channel will be enabled for member | Boolean
push_enabled | If true push channel will be enabled for member | Boolean

### Responses

* **200** - success with member's properties in response body
* **400 Bad request** - there are [validation errors](#validation-on-members).
* **404 Not found** - member with ID was not found


<aside class="notice">
Authentication with <code>X-Member-Token</code> and <code>X-Customer-Private-Token</code>.
</aside>

## Remove member

Removes member from loyalty club.

It is possible to trigger optout message by setting `send_unsubscribe_message=true` query string parameter.

### HTTP Request

```shell
curl -X DELETE -H "X-Customer-Private-Token: alphanumeric_string" \
     -H "X-Product-Name: custom-product-name" \
     "https://connect.bstcm.no/api/v3/loyalty_clubs/:loyalty_club_slug/members/:id"
```

> Successful removal is indicated by response code 200

**DELETE** `api/v3/loyalty_clubs/:loyalty_club_slug/members/:id`

### URL Parameters

Parameter | Description
--------- | -----------
loyalty_club_slug | unique slugified name of the loyalty club. Example: `boosters`.
id | member's ID.

### Query Parameters

Parameter | Description | Type
--------- | ----------- | ---------
send_unsubscribe_message | If true, optout message will be send to member. Example: `true`. | Boolean

### Responses

* **200** - success with member's properties in response body
* **404 Not found** - member not found

<aside class="notice">
Authentication with <code>X-Member-Token</code> and <code>X-Customer-Private-Token</code>.
</aside>