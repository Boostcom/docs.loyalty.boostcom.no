---
title: Boostcom Loyalty API Reference

toc_footers:
  - <a href='http://boostcom.com/'>Boostcom</a>
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
