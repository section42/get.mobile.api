

# GET mobile API v4

## Authorization
Basic Authorization

## Supported Protocols
Http/ Https

## Endpoints
***
### Menu endpoint
***
The menu endpoint provides information about the structure of the product menu, i.e. how items are nested in the menu.

URL: /api/menu/v4
Method: GET
Auth: Basic Authentication

Success Response:
Http code: 200

|Field|Data type|Description
|-|-|-|
|data|List\<MenuNode>| List containing the menu tree

### Product endpoint
***
The product endpoint provides information about the products available via the api. 

URL: /api/product/v4
Method: GET
Auth: Basic Authentication

Success Response:
Http code: 200

|Field|Data type|Description
|-|-|-|
|data|List<MenuProduct>| List containing the menu tree

### Configuration endpoint
****
The configuration endpoint provides information about the device and the current configuration.

URL: /api/config/v4
Method: GET
Auth: Basic Authentication

Success Response:
Http code: 200

|Field|Data type|Description
|-|-|-|
|data|DeviceConfig| Device status object

### Field endpoint
****
The field endpoint returns a list of fields confogured for the project.

URL: /api/field/v4
Method: GET
Auth: Basic Authentication

Success Response:
Http code: 200

|Field|Data type|Description
|-|-|-|
|data|List<FieldModel>| The list of available fields

### Image endpoint
****
This endpoint returns images associated with products in the devices menu. 

URL: /api/image/v4/{$image_id}
Method: GET
Auth: Basic Authentication

Success response:
Http code: 200
the respsective image

### TagInfo Endpoint
****
This endpoint returns the content of an nfc tag which is present at the reader when the request is received. If no nfc tag is present, an error code is returned.

URL: /api/tag_info/v4
Method: GET
Auth: Basic Authentication

Success response:
Http code: 200

|Field|Data type|Description
|-|-|-|
|data|NfcTag| The content of a nfc tag

### Events Endpoint
****
Allows a client to register/ unregister for push notifications of Api events. If the Api fails to make a call to the provided endpoint, the listener is dropped. At most 10 listeners can listen for events and therefore any new listener will replace the oldest listener (FIFO).

URL: /api/events/v4
Method: POST
Auth: Basic Authentication
Request Body:

|Field|Data type|Description
|-|-|-|
|url|String| The url where updates will be sent to, e.g.: http://192.168.0.1:8080/updates


Variants:

| Variant| Explanation  |
| :------------- |:-------------|
| /api/v4/events/register     | registers the listener associated with the url provided in the body |
| /api/v4/events/unregister   | unregisters the listener associated with the url provided in the body |

Success response:
Http code: 204

### Purchase Endpoint
****
This endpoint is used to start a purchase and get information about the status of purchase requests

URL: /api/purchase/v4/{$jobId}/async
Method: POST
Auth: Basic Authentication
Through this endpoint a shopping cart can be handed to the API which then tries to sell its content to the next nfc chip. Every request has to have a valid uuid in the header as jobId. Through this uuid the request can then be identified at a later point in time.

Request:

|Field|Data type|Description
|-|-|-|
|paymentType|String| the uuid of the payment type to use, can be found via the status endpoint
|cartItems|List<CartItem>| the items which are to be sold to a customer


Success response:
Http code: 200

|Field|Data type|Description
|-|-|-|
|data|JobInfo| the object containing information about where information about the job can be found

URL: /api/purchase/v4/{$jobId}/status
Method: GET
Auth: Basic Authentication
Returns the status of a job with the specified jobId

Success response:
Http code: 200

|Field|Data type|Description
|-|-|-|
|data|JobStatus| An object containing information about the specific job

URL: /api/purchase/v4/{$jobId}/cancel
Method: GET
Auth: Basic Authentication
Attempts to cancel the purchase job identified by the provided jobId

Success response:
Http code: 204

### Accreditation Endpoint
****
This endpoint is used to accredit nfc tags.

URL: /api/accredit/v4/{$jobId}/async
Method: POST
Auth: Basic Authentication
Accreditation information sent to this endpoint will be written on the next nfc tag available.

Request:

|Field|Data type|Description
|-|-|-|
|workerId|String| a random uuid to identify the worker
|workerType|String| one of <admin,manager,worker,unknown>, defines the access level of the nfc tag. If the value "unknown" is provided, the API does not change the nfc chips workerType. This can be of use if one wants to perform an upgrade giving out credits only.
|firstName|String| the first name of the person
|lastName|String| the last name of the person
|role|String| the role of the person, e.g. Barkeeper. this field is free text
|company|String| the company the person belongs to, this field is free text
|managerGroupNr|int|the manager group the person belongs to. The manager group is a access filter granting additional privileges
|normalCredits|String|the amount of normal credits to put on the nfc tag
|preGiftCredits|String|the amount of preGift credits to put on the nfc tag
|pureGiftCredits|String|the amount of pureGift credits to put on the nfc tag
|disableTagPawn|boolean| set true if no tag pawn should be charged
|disableActivationFee|boolean | set true if no activation fee should be charged
|fields|List<FieldModel>| A list of fields which the nfc tag should receive
|isUpgrade|boolean|defines of the accreditation is supposed to overwrite the existing accreditaiton on the tag (isUpgrade = false) or if the existing accreditation should be extended by the provided one (isUpgrade = true).

Success response:
Http code: 200

|Field|Data type|Description
|-|-|-|
|data|JobInfo| the object containing information about where information about the job can be found

URL: /api/accredit/v4/{$jobId}/status
Method: GET
Auth: Basic Authentication
Returns the status of a job with the specified jobId

Success response:
Http code: 200

|Field|Data type|Description
|-|-|-|
|data|JobStatus| An object containing information about the specific job

URL: /api/accredit/v4/{$jobId}/cancel
Method: GET
Auth: Basic Authentication
Attempts to cancel the accreditation job identified by the provided jobId

Success response:
Http code: 204

## Transaction Updates
***
Transaction updates are received when registering a callback url through the /events endpoint. The APi will attempt a POST request against the urls registered via /events endpoint containing an update object in its body 
The Api then tries to notify its listeners about the following event types:

1. Detected nfc tag: When a new nfc tag was read.
2. Job update: When a scheduled job has a status update.
3. 

The basic structure of any transaction update is as follows:

|Field|Data type|Description
|-|-|-|
|date|String| the date when the request was processed
|type|int| The type of the update (0 = New nfc tag, 1 = Job update)
|data|Object| An object containing information about the specific job. If the update is of type 0 (New nfc tag) this field will contain a **NfcTag** object. When the update is of type 1 (Job update) the object inside the data property will be of type **JobStatus**

The Api does not attempt to retry the update request on a failed connection, or anything similar. Updates enable a faster event propagation but there is no guarantee that a listener will receive an update.  It is therefore advised to additionally make use of the /status endpoints and the /tag_info endpoint to poll needed information as important events may be missed otherwise.

## Error Responses
***
If authorization fails, the API will respond with a simple 401 (Unauthorized) HTTP response code. If any error occurs after being authorized, error responses will contain additional information about the error in the following schema:

```
{
    "error_code": int,
    "error_message": String
}
```
The following list contains the possible error codes and short explanations.

| Code  | Message
|:-|:-
| **BadRequest** | The request is invalid,
| **ResourceNotFound** | The requested resource was not found (e.g. menu item with $id)
| **MethodNotAllowed** | The request method is not allowed (e.g. GET instead of POST
| **NoTagPresent** | There is no nfc tag on the reader currently - tag_info endpoint only
| **TagInvalid** | The nfc tag on the reader is invalid (e.g. broken) - tag_info endpoint only
| **PaymentTypeNotFound** | The requested payment type does not exists - purchase endpoint only
| **PaymentTypeNotAllowed** | The requested payment type cannot be used for a purchase - purchase endpoint only
| **CartInvalid** | The shopping cart received in the request contains errors - purchase endpoint only 
| **PriceMismatch** | An item in the shopping cart actually has a different price than what is assumed by the client - purchase endpoint only
| **DecimalPlacesMismatch** | The requested resource has already been created - purchase and accreditation only
| **ResourceAlreadyExists** | The resource cannot be created because it already exists
| **UnknownError** | An unknown error occured

# Models
***

## MenuNode
|Field|Data type|Description
|-|-|-|
|id|String| guid of this menu node
|parentId|String| guid of parent node in menu
|type|String| The node type, one of <Product, Category>
|name|String| The name of the product or category
|bgColor|String|hex color code of the background
|thumbnailRef|ImageRef| Image reference for the thumbnail picture associated with the menu node
|children|List<MenuNode>|List of child nodes

## MenuProduct
|Field|Data type|Description
|-|-|-|
|id|String| guid of this menu node
|type|String| The node type, one of <Product, CustomValueProduct, Other>
|name|String|The name of the product, e.g. "Soda"
|priceInCredits|String|The price of a product in system credits. Always formatted with a decimal point, e.g. "3.2"
|priceInCreditsFormatted|String|The price of a product in system credits with the credit symbol. For displaying the credit value. Decimal separator may be location dependent, e.g. "3,20T"
|priceInCurrency|String|The price of a product in real world currency. Always formatted with a decimal point, e.g. "6.4"
|priceInCurrencyFormatted|String|the price of a product in real world currency with the currency symbol. For displaying the currency value. Decimal separator may be location dependent, e.g. "6,40€"
|genericData|List<Generic>|List of generic functions associated with this menu node

## ImageRef
|Field|Data type|Description
|-|-|-|
|id|String| the guid of the image
|url|String| a relative url pointing to the image

## Generic
|Field|Data type|Description
|-|-|-|
|fullString|String| the complete generic as String, e.g. "simpleGeneric(Hello,World)"
|arg|List<String>| List containing all arguments of the generic, e.g. ["Hello","World"]
|functionName|String| the name of the generic function, e.g. "simlpeGeneric"
|paramString|String| the generic's parameters as a single String, e.g. "Hello,World"

## DeviceConfig
|Field|Data type|Description
|-|-|-|
|creditSymbol|String| The symbol used for representing credits
|currencySymbol|String| The symbol used for representing real world currency
|deviceId|String| The short id of the device,r epresented as a four digit String, e.g. "ACZK"
|paymentTypes| List<PaymentType>| A list of payment types enabled for this device
|projectName|String| the name of the current project
|siteName|String| the name of the active site
|unitName|String| the name of the unit the active site belongs to
|decimalSeparator|String| decimal separator used for displaying currency and credit values
|decimalPlacesCredits|int| the number of decimal places used for credits
|decimalPlacesCurrency|int| the number of decimal places used for currency

## FieldModel
|Field|Data type|Description
|-|-|-|
|key|String| the key of the field, e.g. "Adult"
|description|String| a short textual description about the field, e.g. "The owner of the nfc chip is over 18 years old"
|maxValue|int| the maximum value assignable to this field, if 1, effectively boolean

## PaymentType
|Field|Data type|Description
|-|-|-|
|id|String|The guid of the payment type
|name|String| The name of the payment type, e.g. "Cashless Card"
|type|String| one of <Cashless, Other>

## NfcTag
|Field|Data type|Description
|-|-|-|
|normalCredits|String| the available amount of normal credits on the nfc chip
|preGiftCredits|String| the available amount of preGift credits on the nfc chip
|pureGiftCredits|String| the available amount of pureGift credits on the nfc chip
|totalCredits|String| the total available amount of credits on the nfc chip (= normalCredits + preGiftCredits + pureGiftCredits)
|hasActivationFeeDisabled|boolean| true, if the activation fee is disabled for this nfc tag
|hasTagPawnDisabled|boolean| true, if the tag pawn is disablede for this nfc tag
|active|boolean| true, if the nfc tag has been activated. A nfc chip can only be used for payments if it is active
|hasTagPawnPaid|boolean|true, if the nfc tag has been charge with a tag pawn
|worker|boolean| true, if the nfc tag has worker access
|manager|boolean | true, if the nfc tag has manager access
|master|boolean|true, if the nfc tag has master access
|admin|boolean|true, if the nfc tag  has admin access
|fields|List<FieldModel>| a list of fields


## CartItem
|Field|Data type|Description
|-|-|-|
|count|int| the quantity of the item
|menuItemId|String| the uuid of the specific item, can be retrieved via the menu endpoint
|singlePrice|String| the price in credits of one unit of this item, can be retrieved via the product endpoint

## JobInfo
|Field|Data type|Description
|-|-|-|
|status_url|String| The url where status updates about the job can be found

## JobStatus
|Field|Data type|Description
|-|-|-|
|createdAt|String| The date when the request was received which created this job
|jobId|String| The uuid of the job
|result|NfcTag|After successfully charging an nfc tag, the result contains the nfc tags state after it was charged, it contains null otherwise
|status|int|The current status of the job, one of <Pending, Cancelled, Success>
|statusDetails|String| A textual description of the current status of the job to help with debugging. If for example a invalid nfc tag is used, the status of the current job will not change (it will stay in its pending state) but the statusDetails field will now contain a textual note stating that an invalid tag was last placed on the reader.
