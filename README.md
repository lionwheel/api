# API Documentation

## LionWheel enables developers and publishers to easily and efficiently interface through the API!

What is an API? An interface designed to connect different systems.

Our API is very easy to use and allows external software to connect and create deliveries automatically.

This technical documentation is intended for developers who want to create tasks and interact with [LionWheel](https://www.lionwheel.com).

**For any questions or problems you can contact help at support@lionwheel.com**

---

## Introduction

* All interface calls use the **https** protocol.
* The body of all requests and responses is **JSON** format (unless noted otherwise for file uploads).
* Dates are in **dd/mm/yyyy** format unless a field explicitly states otherwise.
* All interface calls are authenticated with an API key passed as the `key` query parameter.

### Organization API Key vs Company API Key

LionWheel provides two types of API keys. Both use the same `key` query parameter, but they identify different actors and grant different levels of access.

| | Organization API Key | Company API Key |
|---|---|---|
| **Who uses it** | The shipping company (organization) managing deliveries | A customer (company) of the shipping company |
| **Key format** | UUID (no prefix)<br>Sample: `a512cc96-2f02-491e-b1cf-6f2420ba956d` | Always starts with `c_key_`<br>Sample: `c_key_a512cc96-2f02-491e-b1cf-6f2420ba956d` |
| **Scope** | Full organization access — manage all companies, drivers, routes, and tasks | Scoped to a single company — tasks are automatically associated with that company |
| **`company_id` on create** | Required when the organization has companies enabled | Not required — the company is inferred from the key |

**Example request:**

```
https://members.lionwheel.com/api/v1/tasks/create?key=c_key_a512cc96-2f02-491e-b1cf-6f2420ba956d
```

### Allowed Key column

Throughout this document, the **Allowed Key** column indicates which API key type may use a given endpoint or parameter:

| Value | Meaning |
|---|---|
| **Org** | Only the Organization API key |
| **Company** | Only the Company API key |
| **Both** | Either key type |
| **Org only** | Endpoint accepts both key types, but this specific parameter is only available when using the Organization API key |

---

## Authentication

In all requests the API key must be passed in a query parameter named `key`:

```
https://members.lionwheel.com/api/v1/tasks/show/12345?key=YOUR_API_KEY
```

### Common authentication responses

| Status | Message | When |
|---|---|---|
| **401** | `API Key is missing` | `key` query parameter is blank |
| **401** | `API Key is wrong` | Key does not match any organization or company |
| **401** | `Account is inactive` | Organization or company account is inactive |
| **403** | `Authorization error` (or endpoint-specific message) | Authenticated key does not have permission to access the resource |
| **415** | `Content-Type must be application/json` | POST/PUT body endpoints called without `Content-Type: application/json` |

---

## List Tasks

### The request

Method: **GET**

URL: `https://members.lionwheel.com/api/v1/tasks?key=XXXXXX`

Returns the most recent unhandled tasks. When using a Company API key, results are scoped to that company. When using an Organization API key, results span the entire organization.

| Parameter | Type | Allowed Key | Description |
|---|---|---|---|
| **key** | string | Both | API key (query parameter) |
| limit | integer | Both | Maximum number of tasks to return. Default: `5` |

### The response

#### Success

Status: **200**

Payload — JSON

```json
{
  "tasks": [ ... ]
}
```

Each task object includes fields such as: `code`, `destination_city`, `destination_number`, `destination_floor`, `destination_apartment`, `destination_recipient_name`, `destination_phone`, `destination_phone2`, `destination_zip_code`, `source_zip_code`, `wp_order_id`, `status`, `is_roundtrip`, `photo_url`, `signature_url`, `is_photo_attached`, `age_verification`, `leave_next_to_door`, `failure_reason`, `order_items`, and organization-specific custom fields.

#### Failure

| Status | Message |
|---|---|
| **401** | Authentication errors (see [Authentication](#authentication)) |
| **403** | Authorization error |

---

## Create a Delivery

### The request

Method: **POST**

URL: `https://members.lionwheel.com/api/v1/tasks/create?key=XXXXXX`

Payload — JSON. Fields in **bold** are mandatory.

| Parameter | Type | Allowed Key | Description |
|---|---|---|---|
| **key** | string | Both | API key (query parameter) |
| original_order_id | string | Both | ID in external system. Should be unique per company, unless **Organization settings → Advanced → Allow import tasks with the same Order Id** is enabled |
| **destination_city** | string | Both | Destination city |
| destination_street | string | Both | Destination street |
| destination_number | string | Both | Destination house/building number |
| destination_recipient_name | string | Both | Recipient name at destination |
| destination_phone | string | Both | Recipient phone at destination |
| pickup_at | date | Both | Delivery date (defaults to current day) |
| company_id | integer | Org only | Company ID to associate the task with. **Required** when using an Organization API key and the organization has companies enabled |
| company | json | Org only | Create or match a company by external ID. Example: `{ "name": "New company", "external_id": "12345678", "recipient_name": "...", "phone": "...", "email": "..." }` |
| notes | string | Both | General delivery notes |
| order_at | date | Both | Original order date |
| source_name | string | Both | Source location name |
| source_city | string | Both | Source city |
| source_street | string | Both | Source street |
| source_number | string | Both | Source house/building number |
| source_zip_code | string | Both | Source zip or postal code |
| source_floor | string | Both | Source floor |
| source_apartment | string | Both | Source apartment |
| source_notes | string | Both | Source delivery notes |
| source_recipient_name | string | Both | Source recipient name |
| source_phone | string | Both | Source phone |
| source_email | string | Both | Source email |
| source_latitude | float | Both | Source latitude (overrides address geolocation) |
| source_longitude | float | Both | Source longitude (overrides address geolocation) |
| destination_name | string | Both | Destination location name |
| destination_zip_code | string | Both | Destination zip or postal code |
| destination_floor | string | Both | Destination floor |
| destination_apartment | string | Both | Destination apartment |
| destination_notes | string | Both | Destination notes |
| destination_entrance | string | Both | Destination entrance |
| destination_entrance_code | string | Both | Destination entrance code |
| destination_phone2 | string | Both | Secondary destination phone |
| destination_email | string | Both | Destination email |
| destination_latitude | float | Both | Destination latitude (overrides address geolocation) |
| destination_longitude | float | Both | Destination longitude (overrides address geolocation) |
| delivery_method | string | Both | Delivery method description |
| greeting | string | Both | Greeting message |
| gifter_name | string | Both | Gifter name |
| gifter_phone | string | Both | Gifter phone |
| is_roundtrip | boolean | Both | Whether this is a round-trip delivery |
| packages_quantity | integer | Both | Number of packages |
| pallets_quantity | integer | Both | Number of pallets (alias: `surfaces_quantity`) |
| surfaces_quantity | integer | Both | Number of surfaces/pallets |
| cartons_quantity | integer | Both | Number of cartons |
| money_collect | integer | Both | Amount to collect in cents (e.g. $3.00 = `300`). Whole numbers only |
| cod_type | integer | Both | Money collect type: Cash `0`, Cheque `1`, Card `2`, Bank transfer `3` |
| is_self_pickup | boolean | Both | Self-pickup delivery |
| age_verification | boolean | Both | Require age verification |
| leave_next_to_door | boolean | Both | Leave package next to door |
| earliest | time | Both | Earliest bound of delivery time window |
| latest | time | Both | Latest bound of delivery time window |
| wait_time | integer | Both | Service time at destination (minutes) |
| line_items | json/array | Both | Order line items. Supported fields per item: `name`, `quantity`, `sku`, `price`, `weight`, `volume`, `variant`, `notes`, `picked_quantity`. Example: `[{"name":"orange","quantity":"6","price":"11.99"}]` |
| orders | array | Both | Sub-orders with `code` and optional nested `order_items` |
| urgency | integer/string | Both | `REGULAR` (0), `URGENT` (1), `SUPER_URGENT` (2). Default: `REGULAR` |
| driver_id | integer | Org only | Assign a driver by ID |
| external_driver_id | string | Org only | Assign a driver by external ID (Company key: only available for specific company configurations) |
| driver | json | Org only | Find or create a driver. Fields: `external_id`, `name` |
| documents | array | Both | Attach documents at creation. Requires organization documents module. Each item: `{ "filename": "example.pdf", "file": "<Base64>" }`. Supported types: PDF, PNG, JPEG |
| self_pickup_point_uuid | string | Both | Self-pickup point UUID |
| self_pickup_point_external_id | string | Both | Self-pickup point external ID |
| agent | json | Org only | Agent assignment (requires `allow_agents`). Fields: `external_id`, `name` |
| price | number | Org only | Task price |
| sms_from_name | string | Org only | Custom SMS sender name |
| barcode | string | Both | Task barcode |
| packages | array | Both | Package barcodes. Each item: `{ "barcode": "..." }` |
| volume | integer | Both | Volume |
| weight | number | Both | Weight |
| pick_status | string | Org only | Pick status |
| payment_method | string | Both | Payment method |
| order_total | string | Both | Order total |
| document_number | string | Both | Document number |
| external_route_code | string | Org only | External route code |
| route_code | string | Org only | Route code |
| task_type | string | Both | Task type |
| other_user | string | Both | Other user reference |
| origin | string | Both | When set to `make`, sets creation origin to `make` |
| *custom fields* | varies | Both | Organization-specific custom fields (by field slug or name) |

> **Company key notes:**
> - Tasks are automatically associated with the company identified by the key.
> - If the organization enforces a company task limit and the limit is zero, the request is rejected.
> - `documents` require the organization documents module to be enabled.

> **Organization key notes:**
> - When companies are enabled, `company_id` (or `company` object) is required.
> - `driver_id` can be used to assign a driver at creation.

### The response

#### Success

Status: **200**

| Field | Type | Description |
|---|---|---|
| task_id | integer | LionWheel task ID |
| public_id | string | Public ID searchable by end users |
| original_order_id | string | ID in external system |
| destination_region_str | string | Destination region |
| label | string | Printable label URL |
| waybill | string | Waybill URL |
| barcode | string | Task barcode |
| tracking_link | string | Delivery tracking page URL |
| otp_code | string | One-time password (only when organization has OTP enabled) |

#### Failure

| Status | Message | When |
|---|---|---|
| **401** | Authentication errors | See [Authentication](#authentication) |
| **403** | `Error - company_id is missing` | Org key, companies enabled, no `company_id` |
| **403** | `Error - Please check company_id` | Invalid `company_id` |
| **403** | `Error - company is not active` | Company is inactive |
| **403** | `Documents module is not supported` | `documents` sent but module disabled |
| **403** | `Error - Please check driver_id` | Invalid driver |
| **403** | `Error - Original Order ID already exists` | Duplicate `original_order_id`. You can create deliveries with same order id when **Organization settings → Advanced → Allow import tasks with the same Order Id** is enabled |
| **403** | `Error - Self Pickup Point not found` | Invalid self-pickup point |
| **403** | Company tasks limit exceeded message | Company key, task limit is zero |
| **415** | `Content-Type must be application/json` | Missing JSON content type |
| **422** | `{ "errors": { ... } }` | Validation failure (`RecordInvalid`) |
| **422** | `{ "errors": "<message>" }` | Invalid `money_collect` or `cod_type` value |

---

## Get Task

### The request

Method: **GET**

URL: `https://members.lionwheel.com/api/v1/tasks/show/<task_id>?key=XXXXXX`

| Parameter | Type | Allowed Key | Description |
|---|---|---|---|
| **key** | string | Both | API key (query parameter) |
| task_id | integer | Both | Task ID (path parameter) |

### The response

#### Success

Status: **200**

Payload — JSON object with a `task` key containing the full task details, including:

| Field | Type | Description |
|---|---|---|
| task_id / id | integer | Task ID |
| pickup_at | DateTime | Delivery date |
| created_at | DateTime | Creation datetime |
| company_id | integer | Associated company ID |
| status | integer | Task status (see [Task Status Values](#task-status-values)) |
| driver_id | integer | Assigned driver ID |
| driver_str | string | Driver name (Organization key only) |
| visits | array | Visit records with driver details |
| photos | array | Photo URLs |
| order_items | array | Line items |
| failure_reason | string | Failure reason text |
| money_collect | integer | COD amount |
| cod | object | COD details (when money collect enabled) |
| documents | array | Attached documents (when documents module enabled) |
| pod_link | string | Proof of delivery link (when photos/signature exist) |
| return_document | object | Return document (when present) |
| *custom fields* | varies | Organization-specific custom fields |

#### Failure

| Status | Message | When |
|---|---|---|
| **401** | Authentication errors | See [Authentication](#authentication) |
| **403** | Authorization error | Task belongs to another organization/company |
| **404** | `Not found` | Task does not exist |

---

## Update Task

### The request

Method: **PUT**

URL: `https://members.lionwheel.com/api/v1/tasks/<task_id>/update?key=XXXXXX`

Payload — JSON

| Parameter | Type | Allowed Key | Description |
|---|---|---|---|
| **key** | string | Both | API key (query parameter) |
| task_id | integer | Both | Task ID (path parameter) |
| driver_id | integer | Org only | Assign driver |
| company_id | integer | Org only | Change associated company |
| user_id | integer | Org only | User ID |
| price | number | Org only | Task price |
| fee_cost | number | Org only | Fee cost |
| driver_note | string | Org only | Driver note |
| status | integer/string | Both | Task status. **Company key: only `CANCELED` is allowed** |
| pickup_at | date | Both | Delivery date |
| public_id | string | Both | Public ID |
| notes | string | Both | General notes |
| greeting | string | Both | Greeting |
| urgency | integer/string | Both | `REGULAR`, `URGENT`, `SUPER_URGENT` |
| is_roundtrip | boolean | Both | Round-trip flag |
| vehicle_kind | string | Both | Vehicle kind |
| packages_quantity | integer | Both | Package count |
| surfaces_quantity | integer | Both | Surface/pallet count |
| cartons_quantity | integer | Both | Carton count |
| stop_time | integer | Both | Stop time |
| gifter_name | string | Both | Gifter name |
| gifter_phone | string | Both | Gifter phone |
| earliest | time | Both | Earliest delivery window |
| latest | time | Both | Latest delivery window |
| wait_time | integer | Both | Wait time (minutes) |
| document_number | string | Both | Document number |
| wp_order_id | string | Both | External order ID |
| money_collect | integer | Both | COD amount (cents) |
| weight | number | Both | Weight |
| is_self_pickup | boolean | Both | Self-pickup flag |
| age_verification | boolean | Both | Age verification |
| leave_next_to_door | boolean | Both | Leave next to door |
| signee_name | string | Both | Signee name |
| destination_name | string | Both | Destination name. **Company key: read-only unless organization allows address editing** |
| destination_city | string | Both | Destination city. **Company key: read-only unless organization allows address editing** |
| destination_street | string | Both | Destination street. **Company key: read-only unless organization allows address editing** |
| destination_number | string | Both | Destination number |
| destination_zip_code | string | Both | Destination zip |
| destination_floor | string | Both | Destination floor |
| destination_apartment | string | Both | Destination apartment |
| destination_notes | string | Both | Destination notes |
| destination_entrance | string | Both | Destination entrance |
| destination_entrance_code | string | Both | Entrance code |
| destination_recipient_name | string | Both | Recipient name |
| destination_phone | string | Both | Recipient phone |
| destination_phone2 | string | Both | Secondary phone |
| destination_email | string | Both | Recipient email |
| source_name | string | Both | Source name. **Company key: read-only unless organization allows address editing** |
| source_city | string | Both | Source city. **Company key: read-only unless organization allows address editing** |
| source_street | string | Both | Source street. **Company key: read-only unless organization allows address editing** |
| source_number | string | Both | Source number |
| source_zip_code | string | Both | Source zip |
| source_floor | string | Both | Source floor |
| source_apartment | string | Both | Source apartment |
| source_notes | string | Both | Source notes |
| source_recipient_name | string | Both | Source recipient name |
| source_phone | string | Both | Source phone |
| source_phone2 | string | Both | Secondary source phone |
| source_email | string | Both | Source email |
| pick_status | string | Org only | Pick status |
| failure_reason_external_id | string | Both | Failure reason by external ID |
| line_items | json/array | Both | Updated line items (same structure as create) |
| packages | array | Both | Package barcodes. Replaces existing packages when provided |
| *custom fields* | varies | Both | Organization-specific custom fields |

> **Company key restrictions:**
> - Update is blocked if the task status does not allow company editing (`allow_company_to_edit`).
> - Only `CANCELED` status changes are permitted.
> - `driver_id`, `company_id`, `user_id`, `price`, `fee_cost`, `driver_note`, and `pick_status` cannot be changed.
> - Source and destination address fields cannot be changed unless the organization allows company address editing for the task's current status.

### The response

#### Success

Status: **200**

```json
{ "message": "Saved successfully" }
```

#### Failure

| Status | Message | When |
|---|---|---|
| **401** | Authentication errors | See [Authentication](#authentication) |
| **403** | `Not found` | Task does not exist |
| **403** | `Update not allowed for task status - company users cannot edit tasks in the current status` | Company key, task not editable by company for its current status |
| **403** | `Status change not allowed - company users can only change status to CANCELED` | Company key attempting a status change other than `CANCELED` |
| **403** | `Error - Please check driver_id` | Invalid driver |
| **403** | Authorization error | Task belongs to another organization/company |
| **409** | `Duplicated request` | Identical request sent within 1 second |
| **415** | `Content-Type must be application/json` | Missing JSON content type |
| **200** | `{ "errors": ... }` | Validation failure (returned with 200 status) |

---

## Get Tasks by Order ID

### The request

Method: **GET**

URL: `https://members.lionwheel.com/api/v1/tasks/by_order_id/<order_id>?key=XXXXXX`

| Parameter | Type | Allowed Key | Description |
|---|---|---|---|
| **key** | string | Both | API key (query parameter) |
| order_id | string | Both | External order ID (path parameter) |

### The response

#### Success

Status: **200**

Returns `{ "tasks": [ ... ] }` with full task info for each match. Returns `{ "tasks": [] }` when `order_id` is blank.

When using a Company API key, only tasks belonging to that company are returned.

#### Failure

| Status | Message |
|---|---|
| **401** | Authentication errors |

---

## Get Tasks by Phone

### The request

Method: **GET** or **POST**

URL: `https://members.lionwheel.com/api/v1/tasks/by_phone/<phone>?key=XXXXXX`

URL (POST): `https://members.lionwheel.com/api/v1/tasks/by_phone?key=XXXXXX`

| Parameter | Type | Allowed Key | Description |
|---|---|---|---|
| **key** | string | Both | API key (query parameter) |
| phone | string | Both | Recipient phone number (path parameter for GET, body/query for POST) |

### The response

#### Success

Status: **200**

Returns `{ "tasks": [ ... ] }` with full task info. Returns empty array when `phone` is blank.

#### Failure

| Status | Message |
|---|---|
| **401** | Authentication errors |

---

## Check Region

Checks whether a destination address falls within a delivery region and returns geocoding details.

### The request

Method: **POST**

URL: `https://members.lionwheel.com/api/v1/tasks/check_region?key=XXXXXX`

| Parameter | Type | Allowed Key | Description |
|---|---|---|---|
| **key** | string | Both | API key (query parameter) |
| city | string | Both | City (required) |
| street | string | Both | Street |
| number | string | Both | House/building number |

### The response

#### Success

Status: **200**

| Field | Type | Description |
|---|---|---|
| region_str | string | Matched region name |
| latitude | float | Geocoded latitude |
| longitude | float | Geocoded longitude |
| next_delivery_date | string | Next available delivery date |
| is_forbidden | boolean | Whether the region is forbidden |
| error | string | Error message when no region found |
| distribution_line | string | Distribution line (organization-specific) |
| price | number | Base price (organization-specific) |

#### Failure

| Status | Message | When |
|---|---|---|
| **401** | Authentication errors | See [Authentication](#authentication) |
| **422** | `City is missing` | `city` parameter is blank |
| **422** | `Couldn't find the address` | Geocoding failed |

---

## Add Document to Task

### The request

Method: **POST**

URL: `https://members.lionwheel.com/api/v1/tasks/<task_id>/add_document?key=XXXXXX`

Payload — JSON

| Parameter | Type | Allowed Key | Description |
|---|---|---|---|
| **key** | string | Both | API key (query parameter) |
| task_id | integer | Both | Task ID (path parameter) |
| file | string | Both | File encoded as Base64 |
| filename | string | Both | Filename (optional; auto-generated if omitted). Supported types: GIF, JPEG, PNG, PDF |

### The response

#### Success

Status: **200**

```json
{ "message": "Saved successfully" }
```

#### Failure

| Status | Message | When |
|---|---|---|
| **401** | Authentication errors | See [Authentication](#authentication) |
| **403** | Authorization error | Task belongs to another organization/company |
| **404** | `Not found` | Task not found or `file` is blank |
| **200** | `{ "errors": ... }` | File validation failure |

---

## Company Details

### The request

Method: **GET**

URL: `https://members.lionwheel.com/api/v1/companies/<company_id>?key=XXXXXX`

| Parameter | Type | Allowed Key | Description |
|---|---|---|---|
| **key** | string | Org | API key (query parameter) |
| company_id | string/integer | Org | Company ID or external ID (path parameter) |

### The response

#### Success

Status: **200**

| Field | Type | Description |
|---|---|---|
| id | integer | Company ID |
| name | string | Company name |
| primary_email | string | Primary email |
| office_phone | string | Office phone |
| cell_phone | string | Cell phone |
| default_recipient_name | string | Default contact name |
| default_phone | string | Default phone |
| default_email | string | Default email |
| is_active | boolean | Whether company is active |
| location | object | Default location (when present) |
| location.id | integer | Location ID |
| location.name | string | Location name |
| location.city | string | City |
| location.street | string | Street |
| location.number | string | Number |
| location.floor | string | Floor |
| location.apartment | string | Apartment |
| location.state | string | State |
| location.country | string | Country |
| location.zip_code | string | Zip code |
| location.latitude | float | Latitude |
| location.longitude | float | Longitude |
| location.default_recipient_name | string | Default recipient |
| location.default_phone | string | Default phone |
| location.default_email | string | Default email |

#### Failure

| Status | Message | When |
|---|---|---|
| **401** | Authentication errors | See [Authentication](#authentication) |
| **403** | `Unauthorized access` | Company API key used, or company not found |

---

## Create Company

### The request

Method: **POST**

URL: `https://members.lionwheel.com/api/v1/companies?key=XXXXXX`

Payload — JSON. The request body must be wrapped in a `company` object.

| Parameter | Type | Allowed Key | Description |
|---|---|---|---|
| **key** | string | Org | API key (query parameter) |
| company.name | string | Org | Company name |
| company.legal_id | string | Org | Legal ID |
| company.primary_email | string | Org | Primary email |
| company.fax | string | Org | Fax number |
| company.office_phone | string | Org | Office phone |
| company.cell_phone | string | Org | Cell phone |
| company.phone2 | string | Org | Alternate phone |
| company.default_earliest_at | time | Org | Default earliest time (e.g. `10:00`) |
| company.default_latest_at | time | Org | Default latest time (e.g. `18:00`) |
| company.is_active | boolean | Org | Active `true` or `false` |
| company.default_location_attributes | object | Org | Default location |
| company.default_location_attributes.name | string | Org | Location name |
| company.default_location_attributes.city | string | Org | City |
| company.default_location_attributes.street | string | Org | Street |
| company.default_location_attributes.number | string | Org | Number |
| company.default_location_attributes.floor | string | Org | Floor |
| company.default_location_attributes.apartment | string | Org | Apartment |
| company.default_location_attributes.zip_code | string | Org | Zip code |
| company.default_location_attributes.notes | string | Org | Notes |
| company.default_location_attributes.default_recipient_name | string | Org | Default recipient |
| company.default_location_attributes.default_phone | string | Org | Default phone |
| company.default_location_attributes.default_email | string | Org | Default email |

### The response

#### Success

Status: **200** — Returns the created company object (same structure as [Company Details](#company-details)).

#### Failure

| Status | Message | When |
|---|---|---|
| **401** | Authentication errors | See [Authentication](#authentication) |
| **403** | `Unauthorized access` | Company API key used |
| **422** | `{ "errors": [ ... ] }` | Validation failure |

---

## Update Company

### The request

Method: **PATCH** or **PUT**

URL: `https://members.lionwheel.com/api/v1/companies/<company_id>?key=XXXXXX`

Payload — JSON. The request body must be wrapped in a `company` object.

| Parameter | Type | Allowed Key | Description |
|---|---|---|---|
| **key** | string | Org | API key (query parameter) |
| company_id | string/integer | Org | Company ID or external ID (path parameter) |
| company.name | string | Org | Company name |
| company.legal_id | string | Org | Legal ID |
| company.primary_email | string | Org | Primary email |
| company.fax | string | Org | Fax |
| company.office_phone | string | Org | Office phone |
| company.cell_phone | string | Org | Cell phone |
| company.phone2 | string | Org | Alternate phone |
| company.default_earliest_at | time | Org | Default earliest time |
| company.default_latest_at | time | Org | Default latest time |
| company.is_active | boolean | Org | Active flag |
| company.default_location_attributes | object | Org | Location attributes |
| company.default_location_attributes.id | integer | Org | Existing location ID, or omit to create new |
| company.default_location_attributes.name | string | Org | Location name |
| company.default_location_attributes.city | string | Org | City |
| company.default_location_attributes.street | string | Org | Street |
| company.default_location_attributes.number | string | Org | Number |
| company.default_location_attributes.floor | string | Org | Floor |
| company.default_location_attributes.apartment | string | Org | Apartment |
| company.default_location_attributes.zip_code | string | Org | Zip code |
| company.default_location_attributes.notes | string | Org | Notes |
| company.default_location_attributes.default_recipient_name | string | Org | Default recipient |
| company.default_location_attributes.default_phone | string | Org | Default phone |
| company.default_location_attributes.default_email | string | Org | Default email |

### The response

#### Success

Status: **200** — Returns the updated company object.

#### Failure

| Status | Message | When |
|---|---|---|
| **401** | Authentication errors | See [Authentication](#authentication) |
| **403** | `Unauthorized access` | Company API key used, or company not found |
| **422** | `{ "errors": [ ... ] }` | Validation failure |

---

## List Drivers

### The request

Method: **GET**

URL: `https://members.lionwheel.com/api/v1/drivers?key=XXXXXX`

Alternate URL: `https://members.lionwheel.com/api/v1/drivers/index?key=XXXXXX`

| Parameter | Type | Allowed Key | Description |
|---|---|---|---|
| **key** | string | Org | API key (query parameter) |

### The response

#### Success

Status: **200** — JSON array of all drivers in the organization.

#### Failure

| Status | Message | When |
|---|---|---|
| **401** | Authentication errors | See [Authentication](#authentication) |
| **403** | `Unauthorized access` | Company API key used |

---

## Driver Daily Route

### The request

Method: **GET**

URL: `https://members.lionwheel.com/api/v1/drivers/<driver_id>/daily_route?key=XXXXXX`

| Parameter | Type | Allowed Key | Description |
|---|---|---|---|
| **key** | string | Org | API key (query parameter) |
| driver_id | integer | Org | Driver ID (path parameter) |
| date | date | Org | Route date (defaults to current day) |

### The response

#### Success

Status: **200** — JSON array of visits on the driver's daily route, including visit details and associated task information.

#### Failure

| Status | Message | When |
|---|---|---|
| **401** | Authentication errors | See [Authentication](#authentication) |
| **403** | `Unauthorized access` | Company API key used |
| **403** | `Unauthorized - please check the API Key or driver ID` | Driver not found or belongs to another organization |

---

## Optimize Daily Route

### The request

Method: **POST**

URL: `https://members.lionwheel.com/api/v1/drivers/<driver_id>/optimize_daily_route?key=XXXXXX`

| Parameter | Type | Allowed Key | Description |
|---|---|---|---|
| **key** | string | Org | API key (query parameter) |
| driver_id | integer | Org | Driver ID (path parameter) |
| date | date | Org | Route date (defaults to current day) |

### The response

#### Success

Status: **200**

| Message | When |
|---|---|
| `Please set default route start time in settings` | Default route start hour not configured |
| Route arranged successfully message | Optimization completed |
| Async optimization message | Optimization queued asynchronously |
| Various routing error messages | Optimization failed (invalid coordinates, unreachable time windows, etc.) |

#### Failure

| Status | Message | When |
|---|---|---|
| **401** | Authentication errors | See [Authentication](#authentication) |
| **403** | `Unauthorized access` | Company API key used |
| **403** | `Unauthorized - please check the API Key or driver ID` | Driver not found or belongs to another organization |

---

## Get Visit

### The request

Method: **GET**

URL: `https://members.lionwheel.com/api/v1/visits/<visit_id>?key=XXXXXX`

| Parameter | Type | Allowed Key | Description |
|---|---|---|---|
| **key** | string | Both | API key (query parameter) |
| visit_id | integer | Both | Visit ID (path parameter) |

### The response

#### Success

Status: **200**

```json
{ "visit": { ... } }
```

Returns the visit record with all visit attributes.

#### Failure

| Status | Message | When |
|---|---|---|
| **401** | Authentication errors | See [Authentication](#authentication) |
| **403** | Authorization error | Visit's task belongs to another organization/company |

---

## Update Visit

### The request

Method: **PUT**

URL: `https://members.lionwheel.com/api/v1/visits/<visit_id>?key=XXXXXX`

Payload — JSON

| Parameter | Type | Allowed Key | Description |
|---|---|---|---|
| **key** | string | Both | API key (query parameter) |
| visit_id | integer | Both | Visit ID (path parameter) |
| visit_at | date | Both | Visit date |
| driver_id | integer | Both | Driver to assign |

### The response

#### Success

Status: **200**

```json
{ "message": "Saved successfully" }
```

#### Failure

| Status | Message | When |
|---|---|---|
| **401** | Authentication errors | See [Authentication](#authentication) |
| **403** | `Error - Please check driver_id` | Invalid driver |
| **403** | Authorization error | Visit's task belongs to another organization/company |
| **415** | `Content-Type must be application/json` | Missing JSON content type |
| **200** | `{ "errors": ... }` | Validation failure |

---

## List Routes

### The request

Method: **GET**

URL: `https://members.lionwheel.com/api/v1/routes?key=XXXXXX`

| Parameter | Type | Allowed Key | Description |
|---|---|---|---|
| **key** | string | Org | API key (query parameter) |
| date | date | Org | Route date (defaults to today) |
| format | string | Org | Response format: `json` (default) or `xml` |

### The response

#### Success

Status: **200** — Array of route objects including driver, vehicle, visits, distance, weight, and scheduling details.

When `format=xml`, the response is XML instead of JSON.

#### Failure

| Status | Message | When |
|---|---|---|
| **401** | Authentication errors | See [Authentication](#authentication) |
| **403** | `Unauthorized access` | Company API key used |

---

## List Self Pickup Points

### The request

Method: **GET**

URL: `https://members.lionwheel.com/api/v1/self_pickup_points?key=XXXXXX`

| Parameter | Type | Allowed Key | Description |
|---|---|---|---|
| **key** | string | Org | API key (query parameter) |

### The response

#### Success

Status: **200**

Array of self-pickup point objects:

| Field | Type | Description |
|---|---|---|
| uuid | string | Point UUID |
| name | string | Point name |
| city | string | City |
| street | string | Street |
| number | string | Number |
| latitude | float | Latitude |
| longitude | float | Longitude |
| phone | string | Phone |
| notes | string | Notes |
| external_id | string | External ID |

#### Failure

| Status | Message |
|---|---|
| **401** | Authentication errors |
| **403** | Authorization error |

---

## Task Status Values

| Value | Name |
|---|---|
| 0 | UNASSIGNED |
| 1 | ASSIGNED |
| 2 | ACTIVE |
| 3 | COMPLETED |
| 4 | CANCELED |
| 5 | ROUNDTRIP_DELIVERED |
| 6 | IN_INVENTORY |
| 7 | OUT_INVENTORY |
| 8 | FAILED |
| 9 | FINAL_FAILED |
| 10 | IN_TRANSFER |
| 11 | READY_FOR_SELF_PICKUP |
| 12 | RETURN_FROM_SELF_PICKUP |

---

## Webhooks and Callbacks

You can set up a webhook to receive delivery status change notifications.

Configure webhooks in the system at:

`https://members.lionwheel.com/organization/edit?tab=api`

Settings required:

1. **Webhook URL** — the endpoint LionWheel will POST updates to
2. **Statuses** — which status changes to subscribe to

Method: **POST**

URL: As configured in settings

Payload: JSON — the structure matches the task creation response fields (task details, visits, order items, etc.).

> Webhooks are outbound from LionWheel to your server. They do not use the `key` query parameter — authentication is based on your webhook endpoint configuration.

---

## Tests and SandBox

A testing environment can be requested from support@lionwheel.com

---

## Endpoint Summary

| Method | Endpoint | Allowed Key |
|---|---|---|
| GET | `/api/v1/tasks` | Both |
| POST | `/api/v1/tasks/create` | Both |
| GET | `/api/v1/tasks/show/:id` | Both |
| PUT | `/api/v1/tasks/:id/update` | Both |
| POST | `/api/v1/tasks/:id/add_document` | Both |
| POST | `/api/v1/tasks/check_region` | Both |
| GET | `/api/v1/tasks/by_order_id/:order_id` | Both |
| GET | `/api/v1/tasks/by_phone/:phone` | Both |
| POST | `/api/v1/tasks/by_phone` | Both |
| GET | `/api/v1/companies/:id` | Org |
| POST | `/api/v1/companies` | Org |
| PATCH/PUT | `/api/v1/companies/:id` | Org |
| GET | `/api/v1/drivers` | Both |
| GET | `/api/v1/drivers/index` | Both |
| GET | `/api/v1/drivers/:id/daily_route` | Both |
| POST | `/api/v1/drivers/:id/optimize_daily_route` | Both |
| GET | `/api/v1/visits/:id` | Both |
| PUT | `/api/v1/visits/:id` | Both |
| GET | `/api/v1/routes` | Org |
| GET | `/api/v1/self_pickup_points` | Both |
