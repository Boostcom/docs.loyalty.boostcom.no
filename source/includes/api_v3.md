# Api V3 reference (draft, work in progress)

## General info

**This is only draft!**

You can use Api V3 only when your customer is migrated to new backend service for data.

### Product

> Example header: `X-Product-Name: android-app`

Each system that is communicating with us should uniquely identify itself so it is possible to distinguish optin/update channels.
That will allow further targeting members by communication channel.

For that we use product name. It should be passed as **required** header `X-Product-Name` that is intended to provide the necessary granularity.

If you miss your product name, please [let us know](http://boostcom.no).

### Client Authorization

> Example header: `X-Client-Authorization: B7t9U9tsoWsGhrv2ouUoSqpM`

All of the endpoints **require** a client authorization header - `X-Client-Authorization`.

It should be used only on backend and never exposed in frontend code.

Each token has are permits assigned to it. Depending what permits are assigned, access to some of the endpoints
may be restricted.

If you miss your authentication token, please [let us know](http://boostcom.no).

### X-User-Agent

> Example header: `X-User-Agent: Infinity Mall Android App `
  
`X-User-Agent` is **required** to distinguish specific client for information and debugging purposes so we better know who uses the service.

It should be arbitrarily chosen to represent client specifics (e.g. 'Infinity Mall Android App v1.2' or 'Infinity Mall Backend Service')

### <a name="v3-oauth2"></a> OAuth2

> Example header: `Authorization: Bearer 8433d608645345a45ce5a0f5ba1225e57546e86ac49e5fec842159dc82218522`

Actions related to specific member (the one that is using your application) require to have member authenticated and we implement
OAuth2 flow for this.

Thos are specific actions that require OAuth2 authentication:

* [Me > Get](#v3-me-get)
* [Me > Update](#v3-me-update)
* [Me > Destroy](#v3-me-destroy)

To authorize them, we **require** `Authorization` header that should contain: `Bearer :access_token`. 

Look at [OAuth Token &bull; Create](#v3-token-create) to see how to obtain the :access_token.

### Common URL Parameter - `:loyalty_club_slug`

> Example slug: 'infinity-mall'

As all of the API endpoints work in the context of specific Loyalty Club, they're scoped within it's identifier - :loyalty_club_slug.

It's an unique slugified name of the Loyalty Club.

### Common HTTP error codes

Status | Reason
-------|-----|-------
`400` | Some of required header is missing
`401` | `X-Client-Authorization` is invalid (or doesn't match provided loyalty club or product)
`403` | Not authorized to perform this action (provided `X-Authorization-Token` doesn't have required permit)
`404` | The requested resource doesn't exist
`422` | Invalid parameters are provided (e.g. incorrect properties on member creation)
`460` | OAuth token required for the action is invalid (applies only to OAuth-related actions - see [OAuth2](#v3-oauth2))

Also, most of handled errors have JSON response body like this:

`{"error": "Message describing what went wrong"}`

<!--- ############################################################################################################# --->

## <a name="v3-member-model"></a> Member JSON model

> Example:

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

Standard member JSON model returned from many API endpoints.

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

<!--- ############################################################################################################# --->

## <a name="v3-token-create"></a> OAuth Token &bull; Create

> Create token example:

```shell
curl -X POST "https://connect.bstcm.no/api/v3/loyalty_clubs/infinity-mall/members/oauth/token" \
  -H 'Content-Type: application/json' \
  -H 'X-Client-Authorization: B7t9U9tsoWsGhrv2ouUoSqpM' \
  -H 'X-Product-Name: default' \
  -H 'X-User-Agent: CURL manual test' \
  -d '{
      "grant_type": "password",
      "identifier_type": "id",
      "identifier": 42,
      "password": "123"
    }'
```

> Refresh token example:

```shell
curl -X POST "https://connect.bstcm.no/api/v3/loyalty_clubs/infinity-mall/members/oauth/token" \
  -H 'Content-Type: application/json' \
  -H 'X-Client-Authorization: B7t9U9tsoWsGhrv2ouUoSqpM' \
  -H 'X-Product-Name: default' \
  -H 'X-User-Agent: CURL manual test' \
  -d '{
  	  "grant_type": "refresh_token",
  	  "refresh_token": "36c636e4290d28488a13691afce351397bec21b1246c2c7896a8262d9bfbc4c4"
    }'
```

> When successful, both commands return JSON structured like this:

```json
{
  "access_token": "af9e5361cd7e083dfa4132df3ea7ab82fac21496991632a9994a8c2a9f33884f",
  "token_type": "bearer",
  "expires_in": 86400,
  "refresh_token": "36c636e4290d28488a13691afce351397bec21b1246c2c7896a8262d9bfbc4c4",
  "created_at": 1506523094,
  "resource_owner_id": 42
}
```

**POST** `api/v3/loyalty_clubs/:loyalty_club_slug/members/oauth/token`

Creates a new access token by one of two methods (grant types):

* `password` - token is issued by providing user credentials
* `refresh_token` - token is issued by providing refresh token obtained from `password` grant type 

### Creating token (`password` grant)

When creating new access token, `"grant_type": "password"` should be given along with member credentials (see example on the right).

In response, two tokens are returned:

* `access_token` that is valid for **24 hours** - may be used to authenticate member in member-related actions (see: [OAuth2](#v3-oauth2)).
* `refresh_token` which is valid for **1 year** - may be used to get a new access token (with `refresh_token` grant)

Also, `resource_owner_id` is returned, is an ID of member that the token has been issued for. 

### Refreshing token (`refresh_token` grant)

You can obtain a new token after (or before) it's expiration time, by using `refresh_token` grant.

Param `grant_type: "refresh_token"` must be provided along with `refresh_token: ":refresh_token"`  (see example on the right).

It returns a token response, same as for `password` grant, but with new tokens.

### POST Parameters (JSON object)

Parameter | Description | Type | For grant type
--------- | ------- | ----- | ----- 
grant_type | type of grant | enum: `password`, `refresh_token` | -
identifier_type | identifier type that user should be retrieved by | enum: `id`, `email`, `msisdn` | `password` 
identifier | value of member identifier | mixed (e.g. `134123123`, `+47123456789` or `alice@example.com`) | `password`
password | member password | string | `password`
refresh_token | refresh token | string | `access_token`

### Response (JSON object)

Key | Description | Type
--------- | ----------- | ---------
access_token | Token that member can be authenticated with | string
token_type | Always "bearer" | string
expires_in | Seconds for how long token will be valid | integer (seconds)
refresh_token | Token that may be used to issue a new :access_token | string
created_at | When the token has been created | integer (timestamp)
resource_owner_id | ID of member that the token has been for | integer

### Error responses

Status | Reason
--------- | ----------- 
`421` | Invalid member credentials provided for `password` grant. Either member could not be found or password is wrong
`422` | Invalid refresh_token provided for `refresh_token` grant (may be expired)

<aside class="notice">
Requires <code>BL:Api:Members:OAuth</code> permit
</aside>

<!--- ############################################################################################################# --->

## <a name="v3-token-revoke"></a> OAuth Token &bull; Revoke

> Example:

```shell
curl -X POST "https://connect.bstcm.no/api/v3/loyalty_clubs/infinity-mall/members/oauth/revoke" \
  -H 'Content-Type: application/json' \
  -H 'X-Client-Authorization: B7t9U9tsoWsGhrv2ouUoSqpM' \
  -H 'X-Product-Name: default' \
  -H 'X-User-Agent: CURL manual test' \
  -d '{
      "token": "36c636e4290d28488a13691afce351397bec21b1246c2c7896a8262d9bfbc4c4" 
    }'
```

> Always returns an empty JSON object

```json
{
  // Empty object
}
```

**POST** `api/v3/loyalty_clubs/:loyalty_club_slug/members/oauth/revoke`

Revokes a token (access or refresh). 

Always returns an empty response (even if given token is invalid).

### POST Parameters (JSON)

Parameter | Description | Type
--------- | ------- | -------
token | access or refresh token obtained from [OAuth Token &bull; Create](#v3-token-create) | string

<aside class="notice">
Requires <code>BL:Api:Members:OAuth</code> permit
</aside>

<!--- ############################################################################################################# --->

## <a name="v3-token-info"></a> OAuth Token &bull; Info

> Example:

```shell
curl -X POST "https://connect.bstcm.no/api/v3/loyalty_clubs/infinity-mall/members/oauth/token/info" \
  -H 'Content-Type: application/json' \
  -H 'X-Client-Authorization: B7t9U9tsoWsGhrv2ouUoSqpM' \
  -H 'X-Product-Name: default' \
  -H 'X-User-Agent: CURL manual test' \
  -H 'Authorization: Bearer 8433d608645345a45ce5a0f5ba1225e57546e86ac49e5fec842159dc82218522'

```

> When successful, return JSON structured like this:

```json
{
    "resource_owner_id": 42, // ID of member that the token has been issued for
    "scopes": [],
    "expires_in_seconds": 73614, // Seconds until expiration
    "application": {
        "uid": null
    },
    "created_at": 1506516784
}
```

**GET** `api/v3/loyalty_clubs/:loyalty_club_slug/members/oauth/token/info`

Returns info of given token (not refresh token) from `Authorization` header.

### Response (JSON object)

Key | Description | Type
--------- | ----------- | ---------
resource_owner_id | ID of member that the token has been for | integer
scopes | Not implemented | Array<string>
expires_in_seconds | Seconds for how long token will be valid | integer (seconds)
application | Not implemented | Object
created_at | When the token has been created | integer (timestamp)

### Error responses

Status | Reason
--------- | ----------- 
`460` | Token from `Authorization` is invalid (or expired)

<aside class="notice">
Requires <code>BL:Api:Members:OAuth</code> permit
</aside>

<!--- ############################################################################################################# --->

## <a name="v3-loyalty-clubs-schema"></a> Loyalty Clubs &bull; Get schema

> Example

```shell
curl "https://connect.bstcm.no/api/v3/loyalty_clubs/infinity-mall/member_schema" \
  -H 'Content-Type: application/json' \
  -H 'X-Client-Authorization: B7t9U9tsoWsGhrv2ouUoSqpM' \
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

**GET** `api/v3/loyalty_clubs/:loyalty_club_slug/member_schema`

Properties for members are defined as part of the loyalty club they belong to.

To describe member properties we use [JSON Schema](http://json-schema.org/documentation.html) definition.
Properties of each member must conform to the defined schema.

We support **JSON schema Draft V4** with format extension for `date` (YYYY-MM-DD).

### Response (JSON object)

Key | Description | Type
--------- | ----------- | ---------
identifiers | Fields that are set to identify member | Array<string>
languages | Languages set up in loyalty club | Array<string>
default_language | Default language used in loyalty club and also used for mappings to Api v2 | string
version | Version of schema, currently the newest is `v2` | string
products | Properties scoping and ordering by product name, default one is `default` | object
required | Required properties for member | integer (timestamp)
properties | Properties describing member | object

<aside class="notice">
Requires <code>BL:Api:Schema:Get</code> permit
</aside>

<!--- ############################################################################################################# --->

## <a name="v3-members-check"></a> Members &bull; Check if exists

> Example:

```shell
curl -I "https://connect.bstcm.no/api/v3/loyalty_clubs/infinity-mall/members/:id" \
  -H 'Content-Type: application/json' \
  -H 'X-Client-Authorization: B7t9U9tsoWsGhrv2ouUoSqpM' \
  -H 'X-Product-Name: default' \
  -H 'X-User-Agent: CURL manual test'
```

> Returns hash consisting of just one property: :exists

```json
{
  "exists": true 
}
```

**GET** `api/v3/loyalty_clubs/:loyalty_club_slug/members/:id/check_existence`

**GET** `api/v3/loyalty_clubs/:loyalty_club_slug/members/by_msisdn/:msisdn/check_existence`

**GET** `api/v3/loyalty_clubs/:loyalty_club_slug/members/by_email/:email/check_existence`

It can be used to check whether member exists in loyalty club or not.

### URL Parameters

Parameter | Description | Type
--------- | ----------- | ------
id | Member's ID | integer
msisdn | Member's msisdn | string (format as defined [here](#msisdn-member-identifier) - example: `4740485124`)
email | Member's email | string (email)

### Response (JSON object)

Key | Type
--------- | ---------
exists | boolean

<aside class="notice">
Requires <code>BL:Api:Members:Check</code> permit
</aside>

<!--- ############################################################################################################# --->

## <a name="v3-members-get"></a> Members &bull; Get

> Example:

```shell
curl "https://connect.bstcm.no/api/v3/loyalty_clubs/infinity-mall/members/:id" \
  -H 'Content-Type: application/json' \
  -H 'X-Client-Authorization: B7t9U9tsoWsGhrv2ouUoSqpM' \
  -H 'X-Product-Name: default' \
  -H 'X-User-Agent: CURL manual test'
```

> When successful, the above command returns member object as depicted [here](#v3-member-model)

**GET** `api/v3/loyalty_clubs/:loyalty_club_slug/members/:id`

**GET** `api/v3/loyalty_clubs/:loyalty_club_slug/members/by_msisdn/:msisdn`

**GET** `api/v3/loyalty_clubs/:loyalty_club_slug/members/by_email/:email`

Returns member by one of three identifier types: `id`, `member` or `email`

### URL Parameters

Parameter | Description | Type
--------- | ----------- | ------
id | Member's ID | integer
msisdn | Member's msisdn | string (format as defined [here](#msisdn-member-identifier) - example: `4740485124`)
email | Member's email | string (email)

### Response (JSON object)

See: [Member model](#v3-member-model)

### Error responses

Status | Reason
--------- | ----------- 
`404` | Member with given identifier could not found 

<aside class="notice">
Requires <code>BL:Api:Members:Get</code> permit
</aside>

<!--- ############################################################################################################# --->

## <a name="v3-members-create"></a> Members &bull; Create

> Example:

```shell

curl -X POST \
  https://connect.bstcm.no/api/v3/loyalty_clubs/infinity-mall/members \
  -H 'Content-Type: application/json' \
  -H 'X-Client-Authorization: B7t9U9tsoWsGhrv2ouUoSqpM' \
  -H 'X-Product-Name: default' \
  -H 'X-User-Agent: CURL manual test' \
  -d '{
	"properties": {
		"email": "dev+6@test.com",
		"msisdn": "4740485124",
		"first_name": "The",
		"last_name": "Doge"
	},
	"send_welcome_message": false
}'

```

> When successful, the above command returns created member object as depicted [here](#v3-member-model)

> When payload is invalid, validation errors like this are returned:

```json
{
    "email": [
        {
        
            "property": "email",
            "error": "duplicated_email_in_community"
        }
    ]
}
```

**POST** `api/v3/loyalty_clubs/:loyalty_club_slug/members`

Create member with given properties.

Available properties and their validation rules are defined by Loyalty Club schema (see: [here](#v3-loyalty-clubs-schema)).

Actual welcome messages sending depends on Loyalty Club and Product configuration.
For example, even if send_email_welcome_message:true param is provided, message may not be sent because either Product 
or Loyalty has disabled welcome messages or Loyalty Club has no e-mails configured at all.

There is also a possibility to have multiple SMS welcome messages sent. The one that matches Product or default one will be sent.

### POST Parameters (JSON)

Parameter | Required? | Default | Description | Type
--------- | ----------- | ----------- | --------- | -----------
properties | **yes** | none | JSON with properties for member | JSON Object
properties\['language'\] | no | "default_language" from schema | Language used by user | string
properties\['msisdn'\] | yes* | none | Unique member's msisdn as defined [here](#msisdn-member-identifier)) Example: `4740485124`.| string
properties\['email'\] | yes* | none | Member's email | string
password | no | none | Member's password. Not required, but user won't be able to log in without this | string
sms_enabled | no | true | Should SMS channel be enabled for member? | Boolean
email_enabled | no | true | Should email channel be enabled for member? | Boolean
push_enabled | no | true | Should push channel will be enabled for member? | Boolean
send_sms_welcome_message | no | true | Should SMS welcome message be sent to member? | Boolean
send_email_welcome_message | no | true | Should email welcome be sent to member | Boolean

&ast; At least one of those properties must be provided

### Response (JSON object)

Created member properties - see: [Member model](#v3-member-model)

### Error responses

Status | Reason
--------- | ----------- 
`422` | [validation errors](#validation-on-members) JSON object.

<aside class="notice">
Requires <code>BL:Api:Members:Create</code> permit
</aside>

<!--- ############################################################################################################# --->

## <a name="v3-members-update"></a> Members &bull; Update

> Example:

```shell

curl -X PUT \
  https://connect.bstcm.no/api/v3/loyalty_clubs/infinity-mall/members/:id \
  -H 'Content-Type: application/json' \
  -H 'X-Client-Authorization: B7t9U9tsoWsGhrv2ouUoSqpM' \
  -H 'X-Product-Name: default' \
  -H 'X-User-Agent: CURL manual test' \
  -d '{
	"properties": {
		"last_name": "Doge"
	},
	"password": "new_password"
}'

```

> When successful, the above command returns updated member object as depicted [here](#v3-member-model)

> When payload is invalid, validation errors as depicted [here](#v3-members-create)

**PUT** `api/v3/loyalty_clubs/:loyalty_club_slug/members/:id`

Update member's properties with given ones.

It is intended for partial updates - not given properties are neither deleted nor overwritten.

If deleting attribute is intended, it's value should be sent as `null`.

### URL Parameters

Parameter | Description | Type
--------- | ----------- | ------
id | Member's ID | integer

### PUT Parameters (JSON)

Parameter | Description | Type
--------- | --------- | -----------
properties | JSON with properties for member | JSON Object
password | Member's password | string
sms_enabled | Should SMS channel be enabled for member? | Boolean
email_enabled | Should email channel be enabled for member? | Boolean
push_enabled | Should push channel will be enabled for member? | Boolean

### Response (JSON object)

Member properties after update - see: [Member model](#v3-member-model)

### Error responses

Status | Reason
--------- | ----------- 
`404` | Member could not be found
`422` | [validation errors](#validation-on-members) JSON object.

<aside class="notice">
Requires <code>BL:Api:Members:Update</code> permit
</aside>

<!--- ############################################################################################################# --->

## <a name="v3-members-destroy"></a> Members &bull; Destroy

**DELETE** `api/v3/loyalty_clubs/:loyalty_club_slug/members/:id`

Permanently removes member.

It is possible to trigger optout message by setting `send_unsubscribe_message=true` query string parameter.
Same as welcome messages, optout messages sending also depends on Loyalty Club and Product configuration.

> Example:

```shell
curl -X DELETE \
    "https://connect.bstcm.no/api/v3/loyalty_clubs/infinity-mall/members/:id" \
    -H 'Content-Type: application/json' \
    -H 'X-Client-Authorization: B7t9U9tsoWsGhrv2ouUoSqpM' \
    -H 'X-Product-Name: default' \
    -H 'X-User-Agent: CURL manual test'
```

> When successful, the above command returns destroyed member object as depicted [here](#v3-member-model)

### URL Parameters

Parameter | Description | Type
--------- | ----------- | ------
id | Member's ID | integer

### Query Parameters

Parameter | Required? | Default | Description | Type
--------- | ----------- | ----------- | --------- | -----------
send_unsubscribe_message | no | true | Should optout message be sent to member? | Boolean

### Response (JSON object)

Properties of member that has been destroyed - see: [Member model](#v3-member-model)

### Error responses

Status | Reason
--------- | ----------- 
`404` | Member could not be found
`422` | [validation errors](#validation-on-members) JSON object.

<aside class="notice">
Requires <code>BL:Api:Members:Destroy</code> permit
</aside>

<!--- ############################################################################################################# --->

## <a name="v3-members-send-token"></a> Members &bull; Send Token

<aside class="warn">
Draft  - Not implemented yet
</aside>

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
msisdn | unique member's msisdn as defined [here](#msisdn-member-identifier)) Example: `4740485124`.

<aside class="notice">
Authentication with <code>X-Client-Authorization</code> or <code>X-Customer-Private-Token</code>.
</aside>

<!--- ############################################################################################################# --->

## <a name="v3-me-get"></a> Me &bull; Get

> Example:

```shell
curl "https://connect.bstcm.no/api/v3/loyalty_clubs/infinity-mall/members/me" \
  -H 'Content-Type: application/json' \
  -H 'X-Client-Authorization: B7t9U9tsoWsGhrv2ouUoSqpM' \
  -H 'X-Product-Name: default' \
  -H 'X-User-Agent: CURL manual test' \
  -H 'Authorization: Bearer 8433d608645345a45ce5a0f5ba1225e57546e86ac49e5fec842159dc82218522'
```

> Responses as same as in [Members &bull; Get](#v3-members-get)

**GET** `api/v3/loyalty_clubs/:loyalty_club_slug/members/me`

Returns "current" member - the one that given Authorization token has been issued for.

As a member-related action, it requires member authorization. See [OAuth](#v3-oauth2).

### Response (JSON object)

See: [Member model](#v3-member-model)

### Error responses

Status | Reason
--------- | ----------- 
`404` | Member associated with given Authorization token does not exist
`460` | Member not authorized (Invalid or expired OAuth token)

<aside class="notice">
Requires <code>BL:Api:Members:OAuth:Get</code> permit.
</aside>

<!--- ############################################################################################################# --->

## <a name="v3-me-update"></a> Me &bull; Update

> Example:

```shell
curl -X PUT \
  https://connect.bstcm.no/api/v3/loyalty_clubs/infinity-mall/members/me \
  -H 'Content-Type: application/json' \
  -H 'X-Client-Authorization: B7t9U9tsoWsGhrv2ouUoSqpM' \
  -H 'X-Product-Name: default' \
  -H 'X-User-Agent: CURL manual test' \
  -H 'Authorization: Bearer 8433d608645345a45ce5a0f5ba1225e57546e86ac49e5fec842159dc82218522' \
  -d '{
	"properties": {
		"last_name": "Doge"
	},
	"password": "new_password"
}'
```

> Responses are same as in [Members &bull; Update](#v3-members-update)

**PUT** `api/v3/loyalty_clubs/:loyalty_club_slug/members/me`

Updates ""current" member - the one that given Authorization token has been issued for.

As a member-related action, it requires member authorization. See [OAuth](#v3-oauth2).

### Parameters

It accepts same PUT and URL parameters as in [Members &bull; Update](#v3-members-update).

### Response (JSON object)

Same as in [Members &bull; Update](#v3-members-update).

### Error responses

Status | Reason
--------- | -----------
`404` | Member associated with given Authorization token does not exist
`422` | [Validation errors](#validation-on-members) JSON object.
`460` | Member not authorized (Invalid or expired OAuth token)

<aside class="notice">
Requires <code>BL:Api:Members:OAuth:Update</code> permit.
</aside>

<!--- ############################################################################################################# --->

## <a name="v3-me-destroy"></a> Me &bull; Destroy

> Example:

```shell
curl -X DELETE \
    "https://connect.bstcm.no/api/v3/loyalty_clubs/infinity-mall/members/me" \
    -H 'Content-Type: application/json' \
    -H 'X-Client-Authorization: B7t9U9tsoWsGhrv2ouUoSqpM' \
    -H 'X-Product-Name: default' \
    -H 'X-User-Agent: CURL manual test' \
    -H 'Authorization: Bearer 8433d608645345a45ce5a0f5ba1225e57546e86ac49e5fec842159dc82218522' \
```

> Response as same as in [Members &bull; Destroy](#v3-members-destroy)

**DELETE** `api/v3/loyalty_clubs/:loyalty_club_slug/members/me`

Permanently removes "current" member - the one that given Authorization token has been issued for.

As a member-related action, it requires member authorization. See [OAuth](#v3-oauth2).

### Parameters

It accepts same URL & query parameters as in [Members &bull; Destroy](#v3-members-destroy).

### Response (JSON object)

Same as in [Members &bull; Destroy](#v3-members-destroy).

### Error responses

Status | Reason
--------- | -----------
`404` | Member associated with given Authorization token does not exist
`460` | Member not authorized (Invalid or expired OAuth token)

<aside class="notice">
Requires <code>BL:Api:Members:OAuth:Destroy</code> permit.
</aside>

<!--- ############################################################################################################# --->
