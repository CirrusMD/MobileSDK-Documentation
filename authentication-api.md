## Authentication API

In order to provide a secured and verified access token to the Mobile SDK it is necessary to implement a server side integration with the CirrusMD Customer API. There are two primary endpoints that are required. The first is an SSO like endpoint which allows a customer to find or create a member within our platform while at the same time retrieving a newly created access token for consumption by the Mobile SDK. The second is an endpoint for "logging out" a member session.

#### Request Signing

Requests are signed and authenticated using JWTs. Each customer is given an _client\_id_ and a _shared\_secret_ that is used for signing.

Below is the json claim used to generate the JWT.

```
{
	"client_id": <uuid>,
	"exp": 1512057262 # unix timestamp 2 minutes from now
	"iat": 1512057262 # unix timestamp when claim created
}
```

To create a JWT you need to define a header that specifies the agreed upon signing mechanism. For the API requests we are requiring signing via HMAC SHA256

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

The JWT is then created by base 64 encoding the header and the payload and signing it.

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  shared_secret)
```

This will generate a JWT formed token that looks something like

```
NTlhMzE1MWQtMDdjZS00M2NhLTljZGEtZGMyMjU1NTc2ZDBj.LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUFxNjJ5SDBBd1ptVWthU3p6Mjh1Ugp6Tml4Zk9KQjNja3NZbmVHcUxTTjFQMHIvRk5HWHM1WUtFTXVKSlpsby8zZmdxcjA2V2xTYlIwL1FNaklMUnVHCmdzTGNlcWpaZVFUVjBWMW4yYzVhNHluWGJJZVdYZFN2RzFaTGVsNXdDblpHazVlRjYxeGNwZm1VUWduY0RTc1UKSEd3Z2tTT0l5Q2NpNnNlZkw1eVYzRXVQNTcrSkQxdE1qNzczNE14TjZiQU9FcmRjUmd3VXozWVNwdTE4Z29YZApvQjNZUHZzTW1SVzRURG91elJuTTgwcGUwcWNyc0QxUG9HdE8rVW9JYncxN2tQaUpQMzRxUU9lMENOVllzVnY5CkdvbzZ6SCtFcjZVTE1WTTdyRFhKK2R3R2UyWFpCbFZqTkQ4d2xkRFI2bUlZN3R5SjVuaHhiYkhDZ2hVNkVBdEUKb1FJREFRQUIKLS0tLS1FTkQgUFVCTElDIEtFWS0tLS0tCg==
```

#### 

#### Create Session Endpoint

##### Parameters

|  | Description |
| :--- | :--- |
| email | patient email \(REQUIRED\) |
| first\_name | patient first name \(REQUIRED\) |
| last\_name | patient last name \(REQUIRED\) |
| dob | The date of birth \(ISO8601\) \(REQUIRED\) |
| gender | patient gender \(REQUIRED\) |
| zipcode | patient residential zipcode \(optional\) |
| member\_id | member\_id \(REQUIRED\) |
| metadata | additional membership data \(as object\) \(optional\) |

##### Request

```
POST https://api.cirrusmd.com/sdk/v1/sessions
Content-Type: application/json
Authorization: Bearer <SIGNED_JWT_ACCESS_TOKEN_FOR_API>

{
    "email": "jane@jones.com",
    "first_name":"Jane",
    "last_name":"Jones",
    "dob":"1977-01-11T00:00:00Z",
    "gender":"female",
    "zipcode":null,
    "member_id":"<Customer_Specific_Member_Identifier>"
}
```

##### Response

```
HTTP/1.1 200 OK
Status: 200 OK
Content-Type: application/json

{
    access_token: <SIGNED_JWT_ACCESS_TOKEN_FOR_SDK>
}
```



#### Destroy Session Endpoint

```
DELETE https://api.cirrusmd.com/sdk/v1/sessions
Content-Type: application/json
Authorization: Bearer <SIGNED_JWT_ACCESS_TOKEN_FOR_API>

{
    access_token: <SIGNED_JWT_ACCESS_TOKEN_FOR_SDK>
}
```



