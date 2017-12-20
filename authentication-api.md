## Authentication API

In order to provide a secured and verified access token to the Mobile SDK it is necessary to implement a server side integration with the CirrusMD Customer API. There are two primary endpoints that are required. The first is an SSO like endpoint which allows a customer to find or create a member within our platform while at the same time retrieving a newly created access token for consumption by the Mobile SDK. The second is an endpoint for "logging out" a member session.

##### Request

```
POST https://api.cirrusmd.com/sdk/v1/sessions
Content-Type: application/json
Authorization: Bearer <Signed_JWT_ACCESS_TOKEN>

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

| Parameter | Description |
| :--- | :--- |
| email | patient email \(REQUIRED\) |
| first\_name | patient first name \(REQUIRED\) |
| last\_name | patient last name \(REQUIRED\) |
| dob | The date of birth \(ISO8601\) \(REQUIRED\) |
| gender | patient gender \(REQUIRED\) |
| zipcode | patient residential zipcode \(optional\)  |
| member\_id | member\_id \(REQUIRED\)  |
| metadata | additional membership data \(as object\) \(optional\) |



##### Response

```
Status Code: 200
Content-Type: application/json

{
    token: <Signed_JWT_ACCESS_TOKEN>,
    expires_in: 7200
}
```



