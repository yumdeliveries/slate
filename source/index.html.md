---
title: YUM Logistis API

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='#'>We shall </a>
  - <a href='#'> &copy;2018 Yum Deliveries LTD</a>

includes:
  - errors

search: true
---

# Introduction

Yum Logistics simplifies your deliveries by providing optimal riders on request and enabling your customers to live track their orders location.

The Yum Logistics API enables you to integate your ordering platform to Yum Logistics for dispatch management.  The full integration is summarised in the table below.

Service | Integration | Description
--------- | ------- | -----------
Order Processing | API | Enable Client to Create, Update and Retrieve orders to the logistics platform.
Rider Availability | API | Enable  Client to query riders availability relative to a specific location
Order Status Notifications | Web Hook | Notify the client when an order status changes in real time.
Master Data Mapping | Configuration | Match organisations and branches for metrics tracking.
 
# Master Data

For reporting purposes and information visibility organisations and branches from the client's platform have to be matched with those on the logistics platform. Unique IDs for branches maintained by the client must be posted as part of the order information explained in the order API section.

After organisation and branches information is configured, clients are issued with a  username and password for consuming the API. Any information posted using these credentials, is associated to the respective organisations and branches.
 
The branches are saved together with their addresses which are used when requesting for a rider to the branch.

## Organisations object data fields

Name | Description
--------- | -----------
name | The name of the organisation eg KFC
mapping_id | the client ID of the organisation e.g 1 . Incase where we have one organisation a default can be agreed on
description | an optional description of the organisation

## Branch object data fields

Name | Description
--------- | -----------
name | The name of the organisation eg KFC
mapping_id | the client ID of the organisation e.g 1 . Incase where we have one organisation a default can be agreed on
description | an optional description of the organisation
organisation | The branch the organisation belongs to
phone_number | An optional contact number to the branch
location | The location object of the branch
location.lat | The map latitude of the branchs location
location.log | The map longitude of the branchs location
location.address | The address info with format of House Number, Street Direction, Street Name, Street Suffix, City, State, Zip, Country

<aside class="notice">
  The client will share the master data items in a an excel/csv document for the automated population to the database.
</aside>


# API 

We offer a RESTful api to interact with the logistics platform, that speaks exclusively in JSON. Any request to the API endpoints need to be authenticated.
A Sandbox will be provided for testing the API and in order to facilitate the integration.

<aside class="notice">
  Since the API is exclusively JSON, you  should always set the <code>Content-Type</code> header to <code>application/json</code> to ensure that your requests are properly accepted and processed by the API
</aside>

## Authentication
> Every request requiring authentication should have the headers set as below

```shell
# With shell, you can just pass the correct header with each request
curl "/api/v1/auth/login/"
  -H "Authorization: Bearer __token__"
```

> To get a token, request on with the code below:


```shell
curl -X POST --header 'Content-Type: application/json' -d '{ \ 
   "username": "testuser", \ 
   "password": "passtest" \ 
 }' '/api/v1/auth/login/'

```

> A succesfull authentication returns JSON structured like this:

```json
{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6IkFkbWluIiwiZXhwIjoxNTE5MzgyMzE3LCJlbWFpbCI6InRlY2hAeXVtLmNvLmtlIn0.-2-P6mP7JO-dIiBt5-HyWC41HT9KqtTjTrsaKsbpFLc"
}

```

> Example token referesh

```shell
curl -X POST --header 'Content-Type: application/json' -d '{ \ 
   "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6IkFkbWluIiwiZXhwIjoxNTE5MzgyMzE3LCJlbWFpbCI6InRlY2hAeXVtLmNvLmtlIn0.-2-P6mP7JO-dIiBt5-HyWC41HT9KqtTjTrsaKsbpFLc" \ 
 }' '/api/v1/auth/token/refresh/'
 ```

We use a token based authentication based on JWT (Javascript Web tokens). After a username and password is given, the endpoints described in the table below are used for the authentication. The token has a an expiry period of 12 hours though this can be changed to suite the integration. To call any other endpoint, a valid bearer token  must be sent on the authorization headers in order for the request to succeed as shown in example below using curl:

<aside class="notice">
<code>curl -X GET --header 'Authorization: Bearer token ' 'http://somegetapiendpoint.com/api/v1'</code>
</aside>

EndPoint | Method | Response | Description
--------- | ------- | ------- | -----------
api/v1/auth/login | POST | ``` {"token":<token>} ``` | Returns a valid token after a username and password is posted.
api/v1/auth/token/refresh/ | POST | ``` {"token":<token>} ``` | Returns a refreshed token (with new expiration) based on posted token
api/v1/auth/token/verify | POST | ``` {"token":<token>} ``` | Checks the veracity of a token, returning the token if it is valid

<aside class="success">
A succesfull call to any of the endpoints above returns a token e.g
<code>{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6IkFkbWluIiwiZXhwIjoxNTE5MzgyMzE3LCJlbWFpbCI6InRlY2hAeXVtLmNvLmtlIn0.-2-P6mP7JO-dIiBt5-HyWC41HT9KqtTjTrsaKsbpFLc"
}</code>
</aside>

## Orders

The REST API provides endpoints for creating, updating, querying and cancelling orders. Once the dispatch process begins, changes on the orders are send back as webhooks to callback configured by the client.

> Get orders

```shell
curl -X GET --header 'Accept: application/json' --header 'Authorization: Bearer token ' '/api/v1/orders/?limit=50&offset=50'
```



## <code> GET /api/v1/orders/ </code>

Get a list of all orders at the resource <code>/api/v1/orders/</code>. Requires a valid authorization token to be sent in the headers.

#### Query Parameters
Name | Type | Description
--------- | ------- | -----------
limit  | integer | Number of results to return per page.
offset | integer | The initial index from which to return the results
today_only | string | Filters orders for today only. Takes the value <code>'True'</code> when used
state | integer | An order state value to filter for. The various order state numbers are listed in the table for order states
ordering | string | Which field to use when ordering the results
search | string | A search term.

> Response Status-Code: 200 OK

```shell
{
  "count": 6426,
  "next": "/api/v1/orders/?limit=50&offset=100",
  "previous": "/api/v1/orders/?limit=50",
  "results": [
      {
        // Order object (omitted for clarity)
      },
     // ... Order objects
    ]
}

```

### Response


Name | Type | Description
--------- | ------- | -----------
count  | integer | Total number of items available.
next | string | Link to the next set of items
previous | string |Link to the previous set of items
results | array | List of order objects, sorted in reverse chronological order by time of creation.


## <code> POST /api/v1/orders/ </code>

Creates an order object with a POST to the resource <code>/api/v1/orders/</code>. Requires a valid authorization token to be sent in the headers.

> Create An Order

```shell
curl -X POST --header 'Content-Type: application/json' --header 'Authorization: Bearer token' -d '{ \ 
   "order_ref": "1589274", \ 
   "state": 1, \ 
   "order_total": "4600.00", \ 
   "branch": 208, \ 
   "items": [ \ 
     { \ 
       "quantity": 3, \ 
       "title": "Streetwise Two", \ 
       "instructions": "Zinger", \ 
       "price": "1450.00" \ 
     } \ 
   ], \ 
   "customer": { \ 
     "first_name": "Leon", \ 
     "last_name": "Bareham", \ 
     "phone": "+254719619017" \ 
   }, \ 
   "source": 1, \ 
   "extra_information": "", \ 
   "payment_method": 1, \ 
   "payment_status": 1, \ 
   "address": { \ 
     "lat": "-1.279533800000000", \ 
     "log": "36.789020100000000", \ 
     "address": "Street Name: Kenton College, Gichugu Road, Cross Street: kieni rd, Apartment: Leon%27s House, Other Instructions: Please ask security at the Main Gate to direct you to Leon%27s house. thank you." \ 
        \ 
   }  \ 
 }' '/api/v1/orders/'

```



#### POST parameters


Name | Type | Description
--------- | ------- | -----------
order_ref | string |  A client internal order id. Must be unique for each order. This field is required.
state | integer | A number representing an order state . The various states meanings are in the order states table. This field is required.
source | integer | A special integer assigned to the to client for each API connecting to the logistics. This is required.
branch | integer | A branch id as configured as part masterdata configuration. Is the client ID for branch the order has been placed
order_total | float | The order total cost paid or to be paid by the customer.
extra_information | string | Any Additional instructions for the order. This field is optional
payment_method | integer | A number representing the payment method used by the customer for the order as documented in the payment methods table. This field is optional
payment_status | integer | A number representing the payment status of the order as documented in the payment methods table.
items | object | The items being delivered
items.quantity | integer | The number of this item
items.title | string | The title of the item. Limited to 128 characters.
items.price | float | The price of the Item
items.instructions | string | Additional information specific to this iye
customer | object | The order's Customer
customer.first_name | string | customer first name 
customer.last_name | string | Customer last name
customer.phone | string | customer phone number in international format
address | object | The delivery address
address.lat | float | The map latitude of the delivery address
address.log | float | The map longitude of the delivery address
address.address | string | The address with the preferred format of House Number, Street Direction, Street Name, Street Suffix, City, State, Zip, Country

> Response status code 200

```json
    {
      "order_ref": "1591444",
      "state": 2,
      "state_display": "Waiting order preparation",
      "order_id": "020f5d58-aa9b-4b2c-8625-742525ce98fa",
      "order_total": "1070.00",
      "branch": 1,
      "items": [
        {
          "title": "Snacks - Crunch Burger",
          "price": "300.00",
          "instructions": "colonel, sweet chilli or zinger dressing - zinger dressing.",
          "quantity": 1
        },
        {
          "title": "Snacks - Crispy Fillets.",
          "price": "390.00",
          "instructions": "3 pieces",
          "quantity": 1
        },
        {
          "title": "Krushers - Oreo Krusher",
          "price": "280.00",
          "instructions": null,
          "quantity": 1
        }
      ],
      "customer": {
        "first_name": "JOSE",
        "last_name": ".Handoyo",
        "phone": "+254719619017"
      },
      "source": 1,
      "extra_information": "#M",
      "rider": null,
      "created_date": "2018-01-31 13:09:30.928587+00:00",
      "branch_name": "KFC Waiyaki Way",
      "branch_address": {
        "lat": "-1.258591700000000",
        "log": "36.781353300000000",
        "address": "Waiyaki Way",
        "address_json": [
          "Waiyaki Way"
        ]
      },
      "payment_status": 1,
      "payment_status_display": "PAID",
      "address": {
        "lat": "-1.258695600000000",
        "log": "36.781334100000000",
        "address": "Street Name: Waiyaki Way, Cross Street: MUTHANGARI DRIVE, Apartment: MUTHANGARI RESIDENCE, Apartment #: UNIT 5, Other Instructions: RIGHT BEHIND WEST END TOWERS.",
        "address_json": {
          "apartment#": " UNIT 5",
          "apartment": " MUTHANGARI RESIDENCE",
          "streetname": " Waiyaki Way",
          "crossstreet": " MUTHANGARI DRIVE",
          "otherinstructions": " RIGHT BEHIND WEST END TOWERS."
        }
      },
      "updated_date": "2018-01-31T13:10:24.615336Z"
  }
```

### Order States Description

States | Description
--------- | -------
1 | SUBMITTED. An order just submitted by a customer
2 | ACCEPTED_BY_RESTAURANT . Order accepted by restaurant awaiting preparation
3 | READY_FOR_PICKUP. An order has been prepared an is awaiting pick up
4 | ASSIGNED_TO_RIDER . Order assigned to a rider pending acceptance
5 | ACCEPTED_BY_RIDER . Order accepted by the assigned rider
6 | PICKED_UP. Order has been picked up and on the way to the customer.
7 | DELIVERED. Order has been delivered 
8 | CANCELLED. Order cancelled

<aside class="warning">
  Some states here may not be available depending on the the scope of integration. In such a case webhook notifications will only be sent for the applicable states.
</aside>

### Payment states Description

States | Description
--------- | -------
1 | PAID. The order has already paid by client.
2 | UNPAID. The order hasn't been paid for yet, rider to collect payment upon delivery

### Payment methods Description

States | Description
--------- | -------
1 | CASH. Order paid by cash.
2 | MPESA. Mobile money payments. (Many not be applicable in some markets)
3 | CARD. Card payments
4 | UNAPPLICABLE. This is for unpaid orders where the driver updates the payment type upon receiving payment.

### Extra fields of order object in response

Name | Type | Description
--------- | ------- | -----------
state_display | string | The description of the order state integer representation
order_id | string(uuid) | Yum logistics order id
source | id | The identity of the API that posted the order
rider | object | Rider object: null if no rider assigned rider object if assigned
branch_name | string | The name of the branch for the order
created_date | string | Order creation date with timezone
updated_date | string | Order update date with timezone
payment_status_display | string | Order payment status description
tracking_url | string | Driver live location tracking url. Only available if order is in state 6 (PICKED_UP)

## <code>GET api/v1/orders/{order_id}/</code>

Gets a specific order. This order_id here refers to the Yum Logistics order id which is a uuid.

> Get specific order object

```shell
curl -X GET --header 'Accept: application/json' --header 'Authorization: Bearer token ' '/api/v1/orders/b0899fba-aa03-4e1f-8985-75bfc470c355/'
```
> Response status code 200

```json
{
  // Order object (omitted for clarity)
}
```

## <code>PATCH api/v1/orders/{order_id}/</code>

Updates any of the allowable fields for an order. This order_id here refers to the Yum Logistics order id which is a uuid.

> Update an order object

```shell
curl -X PATCH --header 'Content-Type: application/json' --header 'Authorization: Bearer <token>' -d '{"state":2}' '/api/v1/orders/b0899fba-aa03-4e1f-8985-75bfc470c355/'
```
> Response status code 200

```json
{
  // Order object (omitted for clarity)
}
```

## Webhooks

> Example POST request

```json
{
  "order_ref":"78399303",
  "state":6,
  "tracking_url":"https://trck.at/7BWnF9JX"

}
```

Your application receives a webhook notification whenever the status of an order changes. Webhooks notifications are sent as an HTTP POST request to your server.Once you implement and share the webhook endpoint, we configure it to receive notifications for all the orders created with your api details.

<aside class="notice">
  When updating an order state on your server the steps are:
  1. Consume the nofication via the webhook for the order change
  2. Call the GET order api for the full details of the order object
</aside>

### JSON Body parameters

The format of the JSON has POST the parameters outlined below:

Name | Type | Description 
--------- | ------- | -----------
order_ref | string | The clients order id
state | integer | The latest order order state
tracking_url | string | Tracking url. This is only available when the order has been picked up by driver.


## Riders

> Check if rider is available for a checkout about to compeleted by client

```shell

curl -X GET --header 'Accept: application/json' --header 'Content-Type: application/json' '/api/v1/available-riders/{branch_id}/'

```

To check a rider availability for a client checking out, we need to send the information of the  branch the client is checking out.

<code>GET /api/v1/available-riders/{branch_id}/</code>

#### Response


> Response status_code: 200 OK

```json

{
  "count": 1,
  "next": null,
  "previous": null,
  "results": [
    {
      "user": {
        "first_name": "Harrison",
        "last_name": "Kimani",
        "phone_number": "0742392231",
        "username": "0742392231",
        "role": "riders"
      },
      "state": 4,
      "state_display": "Free for assignment",
      "distance":100.00,
    }
  ]
}

```

Returns a list of available riders ordered by the proximity to the store in an ascending order. The rider object in the response has the following fields.

Name | Type | Description 
--------- | ------- | -----------
user | object| User object for this rider
user.first_name | string | Riders first name
user.last_name | string | Rider last name
user.phone_number | string | Rider phone numbe
user.role | string | The role of "rider"
state| integer | An integer representation of the riders state
state_display | string | An explanation of the rider state
distance | float | The distance of the rider from the store
