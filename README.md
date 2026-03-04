# **Learning API testing with POSTMAN on [restful-booker](https://restful-booker.herokuapp.com/)**
In this repo I will upload all the tests that will be created by me to test [restful-booker](https://restful-booker.herokuapp.com/) in json format exported from Postman.
Also, I will document all the bugs that will be found in this API both on Jira and in this file.



# **Found bugs section**
## Design issue: Incosistent handling of errors in authorization module
API uses different formats of handling bad requests on one method
In **POST Auth method** if client sent request with body
`{ "username": "admin" }`
and did not send field password handling of error is within REST standarts and server responses with **400 - Bad request**

However, if client sent request with body
`{ "password" : "password123" }`
handling of error is inconsistent: server responds with **200 - OK** status and returns body:
`{ "reason" : "Bad credentials" }`
### Right handling
In both scenarios server obligied to return a standardized error object with a **4xx** status code

## Bug: Invalid date format - empty strings
Sending empty strings in fields "checkin" and "checkout" via **PATCH, POST or PUT** methods that includes object:
Same issue persists for null value
### Payload sent
`"bookingdates":{"checkin":"","checkout":""}`
`"bookingdates":{"checkin": null,"checkout": null}`
### Expected result
Expect server to return status code **400 - Bad request**.
### Actual result
However, server returns **200 - OK** and saves corrupt data with NaNs:
`"bookingdates": { "checkin": "0NaN-aN-aN", "checkout": "0NaN-aN-aN" }`


## Bug: Checkout before checkin
Sending request via **POST, PATCH, PUT** where checkout date is before checkin date returns 200 and saves corrupt data
### Payload sent
`{  "checkin_date": "2026-06-20", "checkout_date": "2026-06-10"  }`
### Expected result
Expect server to return status code **400 - Bad request**
### Actual result
However, server returns **200 - OK** and saves corrupt booking:
```
    "bookingdates": {
                "checkin": "2026-06-20",
                "checkout": "2026-06-10"
            },
```

## Bug: Invalid date format - not YYYY-MM-DD
Sending request via **POST, PATCH, PUT** where dates are strings that are not having format "YYYY-MM-DD" returns 200 and saves corrupt data.
Moreover, server attempts to convert date in YYYY-MM-DD format that leads to high risks of data corruption due to date ambiguity.
### Payload sent
```
"bookingdates": {
        "checkin": "06-20-2026",
        "checkout": "30-06-2026"
    },
```
### Expected result
Expect server to return status code **400 - Bad request**
### Actual result
However, server returns **200 - OK** and saves corrupt booking:
```
    "bookingdates": {
        "checkin": "2026-06-20",
        "checkout": "0NaN-aN-aN"
    },
```

## Bug: Invalid date format - numbers instead of strings
Sending request via **POST, PATCH, PUT** where dates are random numbers returns 200 and saves corrupt data.
### Payload sent
```
    "bookingdates": {
        "checkin": "20260501",
        "checkout": "20260515"
    },
```
### Expected result
Expect server to return status code **400 - Bad request**
### Actual result
However, server returns **200 - OK** and saves corrupt booking:
```
    "bookingdates": {
        "checkin": "0NaN-aN-aN",
        "checkout": "0NaN-aN-aN"
    },
```

## Bug: Invalid date format - random strings
Sending request via **POST, PATCH, PUT** where dates are random strings returns 200 and saves corrupt data.
### Payload sent
```
    "bookingdates": {
        "checkin": "not-a-date",
        "checkout": "not-a-date"
    },
```
### Expected result
Expect server to return status code **400 - Bad request**
### Actual result
However, server returns **200 - OK** and saves corrupt booking:
```
    "bookingdates": {
        "checkin": "0NaN-aN-aN",
        "checkout": "0NaN-aN-aN"
    },
```

## Desing issue: Invalid date format - non-existent date on non-leap year
Sending request via **POST, PATCH, PUT** where checkin date is 2026-02-29 (non-leap year) returns 200 and moves checkin date to 2026-03-01
**!!!** Same problem persists for checkout date too
### Payload sent
```
    "bookingdates": {
        "checkin": "2026-02-29",
        "checkout": "2026-03-05"
    },
```
### Expected result 
Expect server to return status code **400 - Bad request**
### Actual result
Server returns **200 - OK** and moves checkin date
```
    "bookingdates": {
        "checkin": "2026-03-01",
        "checkout": "2026-03-05"
    },
```