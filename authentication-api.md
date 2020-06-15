## Authentication API

In order to provide a secured and verified access token to the Mobile SDK it is necessary to implement a server side integration with the CirrusMD SDK API. There is only one endpoint required. That endpoint allows a customer to find or create a member within our platform. It also creates and returns an access token for consumption by Mobile SDK clients.

#### Request Signing

Requests are signed and authenticated using JWTs. Each customer is given an _client\_id_ and a _shared\_secret_ that is used for request signing. More details on the JWT specification can be found at [jwt.io](https://jwt.io/introduction/). Please talk to your account manager for details on how to acquire your _client\_id_  and _shared\_secret._

Below is the JSON claim used to generate the JWT.

```
{
    "sub": "<client_id>",
    "scope": "sdk",
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

The create session endpoint is the primary endpoint for creating a session for the Mobile SDK. The platform requires a minimal set of member data so that we only create a single record in the database for a member. It utilizes a find then update or create methodology to setup the member and then it returns an authentication token in the form of a JWT that will be passed back to the Mobile SDK.

##### Parameters

|  | Description |
| :--- | :--- |
| external\_user\_id | a globally unique identifier that maps back to your system \(REQUIRED\) |
| email | patient email \(REQUIRED\) |
| first\_name | patient first name \(REQUIRED\) |
| last\_name | patient last name \(REQUIRED\) |
| dob | The date of birth \(ISO8601\) \(REQUIRED\) |
| gender | patient gender, must be one of male, female, or other \(REQUIRED\) |
| zipcode | patient residential zipcode \(optional\) |
| member\_id | member id \(REQUIRED\) |
| metadata | additional membership data \(as object\) \(optional\) |

**Note on `external_user_id` parameter:**

The `external_user_id` is a unique identifier that you define and ideally maps back to an id in your database. It may often be the same as the member id. However, we've found the member id may not be globally unique across different regions. The `external_user_id` is an opportunity to further distinguish users, such as, by region. 

Example: 
```
external_user_id = "<region>" + "-" + "<member_id>"
```

##### Request

```
POST https://staging.cirrusmd.com/sdk/v1/public/sessions
Content-Type: application/json
Authorization: Bearer <SIGNED_JWT_ACCESS_TOKEN_FOR_API>

{
	"external_user_id": "<Customer_Specific_Unique_Identifier>",
    "email": "jane@jones.com",
    "first_name": "Jane",
    "last_name": "Jones",
    "dob": "1977-01-11T00:00:00Z",
    "gender": "female",
    "zipcode": null,
    "member_id": "<Customer_Specific_Member_Identifier>"
}
```

##### Successful Response

```
HTTP/1.1 201 OK
Status: 201 OK
Content-Type: application/json

{
    access_token: "<SIGNED_JWT_ACCESS_TOKEN_FOR_SDK>"
}
```

##### Unauthorized Response

A 401 indicates there is a problem with the jwt in the `Authorization` header. Ensure you are creating the jwt as specified in this document. The `exp` claim must NOT exceed 2 minutes from the current time.

```
HTTP/1.1 401 OK
Status: 401 OK
Content-Type: application/json

{
    error_message: "<detailed error message>"
}
```

##### Unprocessable Entity Response

A 422 indicates there is a problem with the parameters in the post body. Therefore, the API could not create a valid patient record. Most likely, you are missing required parameters. 

```
HTTP/1.1 422 OK
Status: 422 OK
Content-Type: application/json

{
    error_message: "<detailed error message>"
}
```


