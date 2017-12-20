## Authentication API

In order to provide a secured and verified access token to the Mobile SDK it is necessary to implement a server side integration with the CirrusMD Customer API. There are two primary endpoints that are required. The first is an SSO like endpoint which allows a customer to find or create a member within our platform while at the same time retrieving a newly created access token for consumption by the Mobile SDK. The second is an endpoint for "logging out" a member session.

#### Request Signing

Requests are signed and authenticated using JWTs. Each customer is given an _client\_id_ and a _shared\_secret_ that is used for request signing. More details on the JWT specification can be found at [https://jwt.io/introduction/](https://jwt.io/introduction/). Please talk to your account manager for details on how to acquire your _client\_id _ and _shared\_secret._

Below is the JSON claim used to generate the JWT.

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

A signature then needs to be created using the _shared\_secret_.

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  shared_secret)
```

Once you have a signed claim you will construct the JWT by base 64 encoding the header and payload and joining with the signature separated by a "."

```
base64UrlEncode(header) + "." + base64UrlEncode(payload) + "." + signature
```

Which will result in a JWT that looks something like

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbGllbnRfaWQiOiJhYWFhZGtmajEyMzEyYXFrdmFzZGZkczIzNDQyMzE0MXZhc2FkYiIsImV4cCI6MTUxMjA1NzI2MiwiaWF0IjoxNTEyMDU3MjYyfQ.aCNXGvoiDz2s6Pu0Yt4fRFRTCGt0FjwUIARarT68YN8
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



