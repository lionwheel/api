# API Documentation
## LionWheel enables developers and publishers to easily and efficiently interface through the API!

What is an API? An interface designed to connect different systems.

Our API is very easy to use and allows external software to connect and create deliveries automatically.

This technical documentation is intended for developers who want to create tasks and interact with [LionWheel](https://www.lionwheel.com).

**For any questions or problems you can contact help at support@lionwheel.com**

## Introduction

* All interface calls are in the https protocol.
* The body of all requests and responses are json format.
* The dates are in dd/mm/yyyy format.
* All interface calls are authenticated with that will is used to identify and associate the request to the correct customer.

It is important to note that there are two types of calls (with two different types of tokens):
Interface of the shipping company - will be used by the shipping company to create deliveries for their customers.
Token - of the shipping company. The customer must indicate the shipping affiliation (company_id) in the request body
Customer interface of the shipping company - will be used by the customer to create deliveries (the delivery will be automatically associated with the customer and the shipping company).

## Authentication

In all requests the token must be passed in a query parameter named key, for example:

```
https://members.lionwheel.com/api/v1/tasks/create?key=my_cool_token
```

How do I know which token I should use?

As mentioned, there are two types of tokens:
Shipping Company - The token is in the settings under the API definition - KEY API key.
In addition to the token, it is mandatory to request the customer ID to which the shipment will be associated. The customer ID is at the top of the customer settings page.
Customer of a delivery company - ask for the token from the shipping company.
The shipping company will bring the ID from the customer definition page under the API key.
It is important to note and make sure that a token of this type starts with c_key.

## Create a delivery
### The request
Method: POST

Url: https://members.lionwheel.com/api/v1/tasks/create?key=XXXXXX

Payload - json

Fields in bold are mandatory

Field name | type | description
------------ | ------------- | -------------
**pickup_at** | Date | delivery date (defaults to the current day)
company_id | integer | the company id to which the task should be associated
notes | string | general delivery notes
**original_order_id** | string | id in external system, should be unique
source_city | string | 
source_street | string | 
source_number | string | 
source_zip_code | string |  zip or postal code
source_floor | string | 
source_apartment | string | 
source_notes | string | 
source_recipient_name | string | 
source_phone | string | 
source_email | string |
source_latitude | float | providing it will override address geolocation
source_longitude | float | providing it will override address geolocation
**destination_city** | string |
**destination_street** | string |
**destination_number** | string |
destination_zip_code | string | zip or postal code
destination_floor | string |
destination_apartment | string |
destination_notes | string |
destination_entrance | string |
destination_entrance_code | string |
**destination_recipient_name** | string |
**destination_phone** | string |
destination_phone2 | string |
destination_email | string | 
destination_latitude | float | providing it will override address geolocation
destination_longitude | float | providing it will override address geolocation
delivery_method | string | 
greeting | string | 
gifter_name | string | 
gifter_phone | string | 
is_roundtrip | boolean | 
packages_quantity | integer | 
pallets_quantity | integer |
money_collect | integer | amount in cents, $3 should be 300
is_self_pickup | boolean
age_verification | boolean
leave_next_to_door | boolean
earliest | Time | earliest bound of time window
latest | Time | latest bound of time window
line_items | json | Json of the content of the task <br> Supported fields: <br> name - string <br> quantity - string <br> sku - string <br> price - string <br> weight - string <br> variant - string <br> notes - string <br> `[{"name":"orange","quantity":"6", "price": "11.99"}, {"name":"apple","quantity":"8", "price": "10.99"}]`
urgency | integer | REGULAR: 0 <br> URGENT: 1 <br> SUPER_URGENT: 2
driver_id | integer | assigned driver
wait_time | integer | Service time at the destination (minutes)


### The response
#### Success
Status: 200

Payload - json 

Field name | type | description
------------ | ------------- | -------------
task_id | integer | LionWheel's task id
public_id | string | task's public id that can be searched by the end user
original_order_id | string | id in external system
destination_region_str | string | task's destination region
label | string | task's printable label
barcode | string | task's barcode
tracking_link | string | url of delivery's tracking page

#### Failure
Status: 401 - authentication error

Status: 403 - data mismatch error

## Receiving delivery
### The request
Method: GET

Url: https://members.lionwheel.com/api/v1/tasks/show/<task_id>?key=XXXXXX

Payload - json

### The response 
#### Success
Status: 200

Payload - json

Field name | type | description
------------ | ------------- | -------------
task_id | integer |
pickup_at | DateTime | delivery_date (defaults to the current day)
created_at | DateTime | creation datetime
company_id | integer | the company id to which the task should be associated 
status | integer | UNASSIGNED: 0 <br> ASSIGNED: 1 <br> ACTIVE: 2 <br> COMPLETED: 3 <br> CANCELED: 4 <br> ROUNDTRIP_DELIVERED: 5 <br> IN_INVENTORY: 6<br> OUT_INVENTORY: 7 <br> FAILED: 8 <br> FINAL_FAILED: 9 <br> IN_TRANSFER: 10
driver_id | integer | assigned driver

## Update Task
### The request
Method: PUT

Url: https://members.lionwheel.com/api/v1/tasks/<task_id>/update?key=XXXXXX

Payload - json

Field name | type | description
------------ | ------------- | -------------
driver_id | integer | driver that is assigned on the task
pickup_at | date | date of the task
status | integer | UNASSIGNED: 0 <br> ASSIGNED: 1 <br> ACTIVE: 2 <br> COMPLETED: 3 <br> CANCELED: 4 <br> ROUNDTRIP_DELIVERED: 5 <br> FAILED: 8

## Daily route
### The request
Method: GET

Url: https://members.lionwheel.com/api/v1/drivers/<driver_id>/daily_route?key=XXXXXX

Payload - json

Field name | type | description
------------ | ------------- | -------------
date | DateTime | route date (defaults to the current day)

## Optimize daily route
### The request
Method: POST

Url: https://members.lionwheel.com/api/v1/drivers/<driver_id>/optimize_daily_route?key=XXXXXX

Payload - json

Field name | type | description
------------ | ------------- | -------------
date | DateTime | route date (defaults to the current day)

## Receiving visit
### The request
Method: GET

Url: https://members.lionwheel.com/api/v1/visits/<visit_id>?key=XXXXXX

Payload - json

### The response 
#### Success
Status: 200

## Update Visit
### The request
Method: PUT

Url: https://members.lionwheel.com/api/v1/visits/<visit_id>?key=XXXXXX

Payload - json

Field name | type | description
------------ | ------------- | -------------
driver_id | integer | driver that is assigned on the visit
visit_at | date | date of the visit

## List Routes
### The request
Method: GET

Url: https://members.lionwheel.com/api/v1/routes?key=XXXXXX

Payload - query string

Field name | type | description
------------ | ------------- | -------------
date | date | could be any date, if blank it will consider today's date
format | string | could be json OR xml, default would be json

## Webhooks and callbacks
You can set up a webhook to get updates of deliveries status changes.
In order to so, please login to the system and go to:
https://members.lionwheel.com/organization/edit?tab=api
There you will need to provide:
1. Webhook Url - to which LionWheel will be sending the updates
2. Statuses - the statuses to which you would like to subscribe

Please note that the structure of the sent request payload will be exactly as the in the delivery creation call above.

## Tests and SandBox

A testing environment can be requested from support@lionwheel.com
