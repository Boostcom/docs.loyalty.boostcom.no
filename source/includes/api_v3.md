# Api V3 reference (draft, work in progress)

**This is only draft!**

You can use Api V3 only when your customer is migrated to new backend service for data.

## Product name

Each system that is communicating with us should uniquely identify itself so it is possible to distinguish optin/update channels.
That will allow further targeting members by communication channel.
For that we use product name and header `X-Product-Name` is intended to provide the necessary granularity.

If you miss your product name, please [let us know](http://boostcom.no).

## Client Authentication and Authorization (new)

All of the endpoints require a client authorization header - `X-Authorization-Token`.

It should be used only on backend and never exposed in frontend code.

There are permits assigned to the token. Depending what permits are assigned, access to some of the endpoints
may be restricted.

If you miss your authentication tokens, please [let us know](http://boostcom.no).

## X-User-Agent header
 
As single `X-Authorization-Token` may be common for multiple clients, `X-User-Agent` is required to identify client
for information purposes and debugging so we better know who uses the service.

It should be arbitrary chosen to represent client specifics (e.g. 'SuperMall Android App' or 'SuperMall backend users service')

## Common HTTP errors

Error | Reason
-------|------------
`400 (Bad Request)` | Some of required headers is missing
`401 (Unauthorized)` | `X-Authorization-Token` is invalid (or doesn't match provided loyalty club or product)
`403 (Forbidden)` | Not authorized to perform this action (no permit)
`404 (Not Found)` | The requested resource doesn't exist
`422 (Unprocessable Entity)` | Invalid params are provided (e.g. incorrect properties on member creation)

## **OAuth2 for members**

### Testing and implementation

You can use e.g. this oauth2 client for Ruby: https://github.com/intridea/oauth2

### Getting Auth Token

#### HTTP request

```shell
curl -F grant_type=password \
-F msisdn=4740012345 \
-F password=reallyhardpassword \
-X POST http://localhost:3000/oauth/token
```

```shell
curl -F grant_type=refresh_token \
-F client_id=9b36d8c0db59eff5038aea7a417d73e69aea75b41aac771816d2ef1b3109cc2f \
-F client_secret=d6ea27703957b69939b8104ed4524595e210cd2e79af587744a7eb6e58f5b3d2 \
-F refresh_token=c65b265611713028344a2c285dfdc4e28f9ce2dbc36b9f7e12f626a3d106a304 \
-X POST http://localhost:3000/oauth/token
```

> When successful, the above commands returns JSON structured like this:

```json
{
  "access_token":"0ddb922452c983a70566e30dce16e2017db335103e35d783874c448862a78168",
  "token_type":"bearer",
  "expires_in":7200,
  "refresh_token":"f2188c4165d912524e04c6496d10f06803cc08ed50271a0b0a73061e3ac1c06c",
  "scope":"public"
}
```

**POST** `api/v3/loyalty_clubs/:loyalty_club_slug/members/oauth/token`

#### URL Parameters

Parameter | Description
--------- | -------
loyalty_club_slug | unique slugified name of the loyalty club. Example: `boosters`.

#### POST (JSON) Parameters

Parameter | Description
--------- | -------
grant_type | type of grant (e.g. `password` or `refresh_token`)
'identifier' (e.g. `email` or `msisdn`) | identifier of member 
password | member password while using `password` grant_type
client_id | client id while using `refresh_token` grant_type
client_secret | client secret while using `refresh_token` grant_type
refresh_token | refresh token while using `refresh_token` grant_type

### Authorizing with access_token

Add to your request `Authorization` header with value: `"Bearer "` + `access_token`
or add it to query params, e.g. `http://something.pl/anyurl?access_token=fe087c17dd15a84b3c939fbbbd1bbfd196d7ea28cfafbf1d6f15a6c74822ef30`.

## Get loyalty club's schema

```shell
curl "https://connect.bstcm.no/api/v3/loyalty_clubs/:loyalty_club_slug/member_schema" \
  -H 'Content-Type: application/json' \
  -H 'X-Authorization-Token: B7t9U9tsoWsGhrv2ouUoSqpM' \
  -H 'X-Product-Name: default' \
  -H 'X-User-Agent: CURL manual test'
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
Authentication only with <code>X-Authorization-Token</code>.
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
msisdn | unique member's msisdn as defined [here](#msisdn-member-identifier)) Example: `4740485124`.

<aside class="notice">
Authentication with <code>X-Authorization-Token</code> or <code>X-Customer-Private-Token</code>.
</aside>

## Check if member exists

```shell
curl -I "https://connect.bstcm.no/api/v3/loyalty_clubs/:loyalty_club_slug/members/:id" \
  -H 'Content-Type: application/json' \
  -H 'X-Authorization-Token: B7t9U9tsoWsGhrv2ouUoSqpM' \
  -H 'X-Product-Name: default' \
  -H 'X-User-Agent: CURL manual test'
```

> Returns hash consisting of just one property: :exists

```json
{
  "exists": true // If exists, true. If not, false.
}
```

It can be used to check if member is in loyalty club or not.

### HTTP Request

**GET** `api/v3/loyalty_clubs/:loyalty_club_slug/members/:id/check_existence`

**GET** `api/v3/loyalty_clubs/:loyalty_club_slug/members/by_msisdn/:msisdn/check_existence`

**GET** `api/v3/loyalty_clubs/:loyalty_club_slug/members/by_email/:email/check_existence`

### URL Parameters

Parameter | Description
--------- | -----------
loyalty_club_slug | unique slugified name of the loyalty club. Example: `boosters`.
id | member's ID.
msisdn | unique member's msisdn as defined [here](#msisdn-member-identifier)) Example: `4740485124`.
email | unique member's email.

### Response code

* **200** 

<aside class="notice">
Requires 'BL:Api:Members:Check' permit
</aside>

## Get member

```shell
curl "https://connect.bstcm.no/api/v3/loyalty_clubs/:loyalty_club_slug/members/:id" \
  -H 'Content-Type: application/json' \
  -H 'X-Authorization-Token: B7t9U9tsoWsGhrv2ouUoSqpM' \
  -H 'X-Product-Name: default' \
  -H 'X-User-Agent: CURL manual test'
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
msisdn | unique member's msisdn as defined [here](#msisdn-member-identifier)) Example: `4740485124`.
email | unique member's email.

### Responses

* **200** - success with member's properties in response body
* **404 Not found** - member with given identifier not found 

<aside class="notice">
Requires 'BL:Api:Members:Get' permit
</aside>

## Create member

```shell

curl -X POST \
  https://connect.bstcm.no/api/v3/loyalty_clubs/:loyalty_club_slug/members \
  -H 'Content-Type: application/json' \
  -H 'X-Authorization-Token: B7t9U9tsoWsGhrv2ouUoSqpM' \
  -H 'X-Product-Name: default' \
  -H 'X-User-Agent: CURL manual test' \
  -d '{
	"properties": {
		"email": "dev+6@test.com",
		"msisdn": "4740485124",
		"first_name": "THe",
		"last_name": "Doge"
	},
	"send_welcome_message": false
}'

```

> When successful, the above command returns JSON as depicted in "Get member" section

> When payload is invalid, validation errors like this are returned:

```json
{
    "email": [
        {
        
            "property": "email",
            "error": "duplicated_email_in_community",
        }
    ]
}
```

Create member with given properties.

Available properties and their validation rules are defined by Loyalty Club schema (see: [Get loyalty's club schema](#get-loyalty-clubs-schema34)).

Actual welcome messages sending depends on Loyalty Club and Product configuration.
For example, even if send_email_welcome_message:true param is provided, message may not be sent because either Product 
or Loyalty has disabled welcome messages or Loyalty Club has no e-mails configured at all.

There is also a possibility to have multiple SMS welcome messages sent. The one that matches Product or default one will be sent.

### HTTP Request

**POST** `api/v3/loyalty_clubs/:loyalty_club_slug/members`

### URL Parameters

Parameter | Description
--------- | -----------
loyalty_club_slug | unique slugified name of the loyalty club. Example: `boosters`.

### POST (JSON) Parameters

Parameter | Required? | Default | Description | Type
--------- | ----------- | ----------- | --------- | -----------
properties | **yes** | none | JSON with properties for member | JSON Object
properties\['language'\] | no | "default_language" from schema | Language used by user | string
properties\['msisdn'\] | yes* | none | Unique member's msisdn as defined [here](#msisdn-member-identifier)) Example: `4740485124`.| string
properties\['email'\] | yes* | none | Member's email | string
sms_enabled | no | true | Should SMS channel be enabled for member? | Boolean
email_enabled | no | true | Should email channel be enabled for member? | Boolean
push_enabled | no | true | Should push channel will be enabled for member? | Boolean
send_welcome_message | no | true | Should SMS welcome message be sent to member? | Boolean
send_email_welcome_message | no | true | Should email welcome be sent to member | Boolean

&ast; At least one of those properties must be provided

### Responses

* **200** - Created member JSON object
* **422 Bad request** - [validation errors](#validation-on-members) JSON object.

<aside class="notice">
Requires 'BL:Api:Members:Create' permit
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
curl -X DELETE \
    "https://connect.bstcm.no/api/v3/loyalty_clubs/:loyalty_club_slug/members/:id" \
    -H 'Content-Type: application/json' \
    -H 'X-Authorization-Token: B7t9U9tsoWsGhrv2ouUoSqpM' \
    -H 'X-Product-Name: default' \
    -H 'X-User-Agent: CURL manual test'
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
