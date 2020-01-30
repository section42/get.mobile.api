

# GET mobile API v4

## Index
1. [Change Log](#changeLog)
2. [Authentication](#authorization)
3. [Supported Protocols](#supportedProtocols)
4. [Endpoints](#endpoints)
	* [Menu](#menuEndpoint)

## <a name="changeLog">Change Log</a>

### Device version 49.48

Events (previously referred to as transaction updates): An event (can be received when a callback is registered) now consists of a type <JobUpdate, NewTag> providing information about which type of event ocurred. Depending on the type, either the "newTag" property is set, indicating a [NfcTag](#nfcTag) has been detected while no job is currently running, or the "jobUpdate" property is set, indicating that new information is available related to the currently running [job](#jobStatus).

```json
{
	"type": "JobUpdate",
	"newTag":null,
	"jobUpdate":{
		"jobId":"414badd8-dff0-4a08-9a8d-a5f964a63d29",
		"status":"Pending",
		"result":null,
		"createdAt":1580380077351
	}
}

```

JobStatus: the createdAt property's dataType has changed from String to long. The long value represents the current timestamp as milliseconds since unix epoch.

[FieldModel](#fieldModel): The properties description and maxValue have been removed. Instead, a field now has the properties key, value and type. The value is a String representation of the fields value and the type gives information about the fields data type <TypeBoolean, TypeInteger>. A boolean field can have the values <true,false> in String representation whereas an integer field's value containes the string representation of an integer value.

```json
[
	{
		"key": "some_flag",
		"value":"false",
		"type":"TypeBoolean"
	},
	{
		"key": "some_other_flag",
		"value":"5",
		"type":"TypeInteger"
	}
]

```

Accreditation Endpoint: The functionality to accredit fields has changed. The "fields" property is now named "fieldChanges". Field changes can now have different data types. For the moment, these data types are limited to Boolean and Integer. In addition, different data types have different change types.
Example of all field changes currently available:
```json
	
"fieldChanges":[
	{
		"key": "some_flag",
		"value":"true",
		"type":"BooleanSet"
	},
	{
		"key": "some_other_flag",
		"value":"5",
		"type":"IntegerAbsolute"
	},
	{
		"key": "third_flag",
		"value":"-3",
		"type":"IntegerRelative"
	},
	{
		"key": "another_flag",
		"value":"8",
		"type":"IntegerRelative"
	}
]

```

The value field is now a String field which holds the desired change value in String representation. For integer fields this is the String representation of any integer. For boolean fields it is either "true" or "false".


### Device version 49.22, released on 19.06.2019
Accreditation Endpoint: Added functionality for accrediting fields. An accreditation request is now <b>required to specify an operation per field</b> sent in the request. Any existing fields not defined in the request remain untouched. An operation is one of <Set,Increment,Decrement>. Previous versions only allowed to set a flag to a specified value. From now on, also incrementing and decrementing a fields value are supported.

previous fields data for accreditation request:
```json
	
"fields":[
	{
		"key": "some_flag",
		"value":1
	},
	{
		"key": "some_other_flag",
		"value":5
	}
]

```

new fiels data for accreditation request:
```json
	
"fields":[
	{
		"key": "some_flag",
		"value":1,
		"operation":"Set"
	},
	{
		"key": "some_other_flag",
		"value":5,
		"operation":"Increment"
	},
	{
		"key": "third_flag",
		"value":2,
		"operation":"Decrement"
	}
]

```


## <a name="authorization">Authentication</a>
[Basic Authentication](https://swagger.io/docs/specification/authentication/basic-authentication/)

The test authentication parameters are:
username: api
password: test

## <a name="supportedProtocols">Supported Protocols</a>
Http/ Https. If the API is configured to run via http, the api has to be addressed via port 8080, if it is in https mode, its port 9443. Therefore, to check the config of the device the address would be for example: https://<span></span>10.10.10.10:9443/api/config/v4

## <a name="endpoints">Endpoints</a>
***
### <a name="menuEndpoint">Menu</a>
***
The menu endpoint provides information about the structure of the product menu, i.e. how items are nested in the menu.

URL: /api/menu/v4  
Method: GET  
Auth: [Basic Authentication](#authorization)

Success Response:  
Http code: 200  
Response Body:  

|Field|Data type|Description
|-|-|-|
|data|List\<[MenuNode](#menuNode)>| List containing the menu tree

### <a name="productEndpoint">Product</a>
***
The product endpoint provides information about the products available via the api. 

URL: /api/product/v4  
Method: GET  
Auth: [Basic Authentication](#authorization)

Success Response:  
Http code: 200  
Response Body:  

|Field|Data type|Description
|-|-|-|
|data|List\<[MenuProduct](#menuProduct)>| List containing all products

### <a name="configurationEndpoint">Configuration</a>
****
The configuration endpoint provides information about the device and the current configuration.

URL: /api/config/v4  
Method: GET  
Auth: [Basic Authentication](#authorization) 

Success Response:  
Http code: 200  
Response Body:  

|Field|Data type|Description
|-|-|-|
|data|[DeviceConfig](#deviceConfig)| Device status object

### <a name="fieldsEndpoint">Fields</a>
****
The field endpoint returns a list of fields confogured for the project.

URL: /api/field/v4  
Method: GET  
Auth: [Basic Authentication](#authorization)

Success Response:  
Http code: 200  
Response Body:  

|Field|Data type|Description
|-|-|-|
|data|List\<[FieldModel](#fieldModel)>| The list of available fields

### <a name="imageEndpoint">Images</a>
****
This endpoint returns images associated with products in the devices menu. 

URL: /api/image/v4/{$image_id}  
Method: GET  
Auth: [Basic Authentication](#authorization) 

Success response:  
Http code: 200  
the respsective image  

### <a name="tagInfoEndpoint">Nfc Tag Info</a>
****
This endpoint returns the content of an nfc tag which is present at the reader when the request is received. If no nfc tag is present, an error code is returned.

URL: /api/tag-info/v4  
Method: GET  
Auth: [Basic Authentication](#authorization)

Success response:  
Http code: 200  
Response Body:  

|Field|Data type|Description
|-|-|-|
|data|[NfcTag](#nfcTag)| The content of a nfc tag

### <a name="eventsEndpoint">Events</a>
****
Allows a client to register/ unregister for push notifications of Api events. At most 10 listeners can listen for events and therefore any new listener will replace the oldest listener (FIFO).

URL: /api/events/v4  
Method: POST  
Auth: [Basic Authentication](#authorization)  
Request Body:  

|Field|Data type|Description
|-|-|-|
|url|String| The url where updates will be sent to, e.g.: http://<span></span>192.168.0.1:8080/updates


Example:

```json
{
	"url":"http://10.64.5.226/callback"
}
```

Variants:  

| Variant| Explanation  |
| :------------- |:-------------|
| /api/v4/events/register     | registers the listener associated with the url provided in the body |
| /api/v4/events/unregister   | unregisters the listener associated with the url provided in the body |

Success response:  
Http code: 204  

### <a name="purchaseEndpoint">Purchase</a>
****
This endpoint is used to start a purchase and get information about the status of purchase requests

URL: /api/purchase/v4/{$jobId}/async  
Method: POST  
Auth: [Basic Authentication](#authorization)  
Through this endpoint a shopping cart can be handed to the API which then tries to sell its content to the next nfc chip. Every request has to have a valid uuid in the header as jobId. Through this uuid the request can then be identified at a later point in time.  

Request:  

|Field|Data type|Description|
|-|-|-|
|paymentType|String| the uuid of the payment type to use, can be found via the status endpoint
|cartItems|List\<[CartItem](#cartItem)>| the items which are to be sold to a customer
|requiredChip|String|Optional. If provided, only the chip with the defined Id can be used

Example:

```json
{
	"paymentType":"f3fd29bd-d5b9-4fa3-a68c-c8cfdfc0a482",
	"cartItems":[
		{
			"count":1,
			"menuItemId": "3cfed0c0-2658-4a4f-9bcd-bb43f3f1535a",
			"singlePrice": "2.1"
		},
		{
			"count":5,
			"menuItemId": "123ed0c0-2658-4a4f-9bcd-bb43f3f1535a",
			"singlePrice": "8"
		}
	]
}
```

Success response:  
Http code: 200  
Response Body:  

|Field|Data type|Description
|-|-|-|
|data|[JobInfo](#jobInfo)| the object containing information about where information about the job can be found


URL: /api/purchase/v4/{$jobId}/status  
Method: GET  
Auth: Basic Authentication  
Returns the status of a job with the specified jobId  

Success response:  
Http code: 200  
Response Body:  

|Field|Data type|Description
|-|-|-|
|data|[JobStatus](#jobStatus)| An object containing information about the specific job

URL: /api/purchase/v4/{$jobId}/cancel  
Method: GET  
Auth: [Basic Authentication](#authorization)
Attempts to cancel the purchase job identified by the provided jobId. It is required to check the status of the job even after invoking cancellation, as the job might still be completed. Only if the status of the job is one of <Cancelled, Success> the job has completed.

Success response:  
Http code: 204  

### <a name="accreditationEndpoint">Accreditation</a>
****
This endpoint is used to accredit nfc tags.  

URL: /api/accredit/v4/{$jobId}/async  
Method: POST  
Auth: [Basic Authentication](#authorization) 
Accreditation information sent to this endpoint will be written on the next nfc tag available.  

Request:  

|Field|Data type|Description
|-|-|-|
|workerId|String| a random uuid to identify the worker
|workerType|String| one of <unknown, guest, worker, powerworker, serviceuser, manager, master, admin>, defines the access level of the nfc tag. If the value "unknown" is provided, the API does not change the nfc chips workerType. This can be of use if one wants to perform an upgrade giving out credits only.
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
|fieldChanges|List\<[FieldChangeRequest](#fieldChangeRequest)>| A list of fields which the nfc tag should receive
|isUpgrade|boolean|defines of the accreditation is supposed to overwrite the existing accreditaiton on the tag (isUpgrade = false) or if the existing accreditation should be extended by the provided one (isUpgrade = true). Note that a value of false will reset all fields on the nfc chip before applying any provided field changes.
|requiredChip|String|Optional. If provided, only the chip with the defined Id can be used

Example:
```json
{
	"externalWorkerId":"123ed0c0-2658-4a4f-9bcd-bb43f3f1535a",
	"workerType":"worker",
	"firstName": "John",
	"lastName": "Doe",
	"fields":[
		{
			"key": "AAA",
			"value":1,
			"operation":"Set"
		},
		{
			"key": "FoodCoupon",
			"value":5,
			"operation":"Increment"
		}
	],
	"normalCredits":"1",
	"preGiftCredits":"0",
	"pureGiftCredits":"0",
	"isUpgrade":false,
	"disableTagPawn":true,
	"disableActivationFee":true
}
```

Success response:  
Http code: 200  
Response Body:  

|Field|Data type|Description
|-|-|-|
|data|JobInfo| the object containing information about where information about the job can be found

URL: /api/accredit/v4/{$jobId}/status  
Method: GET  
Auth: [Basic Authentication](#authorization) 
Returns the status of a job with the specified jobId  

Success response:  
Http code: 200  
Response Body:  

|Field|Data type|Description
|-|-|-|
|data|[JobStatus](#jobStatus)| An object containing information about the specific job

URL: /api/accredit/v4/{$jobId}/cancel  
Method: GET  
Auth: [Basic Authentication](#authorization) 
Attempts to cancel the accreditation job identified by the provided jobId. It is required to check the status of the job even after invoking cancellation, as the job might still be completed. Only if the status of the job is one of <Cancelled, Success> the job has completed.

Success response:  
Http code: 204  

## <a name="errorResponses">Error Responses</a>
***
If authorization fails, the API will respond with a simple 401 (Unauthorized) HTTP response code. If any error occurs after being authorized, error responses will contain additional information about the error in the following schema:

```
{
    "error_code": String,
    "error_message": String
}
```
The following list contains the possible error codes and short explanations.

| Code  | Message
|:-|:-
| **BadRequest** | The request is invalid,
| **ResourceNotFound** | The requested resource was not found (e.g. menu item with $id)
| **MethodNotAllowed** | The request method is not allowed (e.g. GET instead of POST
| **ApiNotActive**| The API is currently deactivated. Requests are only processed when API is activated
| **NoTagPresent** | There is no nfc tag on the reader currently - tag_info endpoint only
| **TagInvalid** | The nfc tag on the reader is invalid (e.g. broken) - tag_info endpoint only
| **PaymentTypeNotFound** | The requested payment type does not exists - purchase endpoint only
| **PaymentTypeNotAllowed** | The requested payment type cannot be used for a purchase - purchase endpoint only
| **CartInvalid** | The shopping cart received in the request contains errors - purchase endpoint only 
| **PriceMismatch** | An item in the shopping cart actually has a different price than what is assumed by the client - purchase endpoint only
| **DecimalPlacesMismatch** | The number of decimal places provided exceeds the allowed number of decimal places
| **ResourceAlreadyExists** | The resource cannot be created because it already exists
| **UnknownError** | An unknown error occured

# <a name="models">Models</a>
***

## <a name="events">Events</a>
***
Events are received when registering a callback url through the /events endpoint. The APi will attempt a POST request against the urls registered via /events endpoint containing an update object in its body 
The Api then tries to notify its listeners about the following event types:

1. Detected nfc tag: When a new nfc tag was read.
2. Job update: When a scheduled job has a status update.

The basic structure of any transaction update is as follows:

|Field|Data type|Description
|-|-|-|
|type|String| the type of the event. one of <JobUpdate, NewTag>
|newTag|[NfcTag](#nfcTag)| Set if the event is of type "NewTag"
|jobUpdate|[JobStatus](#jobStatus)| Set if the event is of type "JobUpdate"

```json
{
	"type": "JobUpdate",
	"newTag":null,
	"jobUpdate":{
		"jobId":"414badd8-dff0-4a08-9a8d-a5f964a63d29",
		"status":"Pending",
		"result":null,
		"createdAt":1580380077351
	}
}

```

```json
{
	"type": "NewTag",
	"newTag":{
		
	},
	"jobUpdate":null
}

```

The Api does not attempt to retry the update request on a failed connection, or anything similar. Updates enable a faster event propagation but there is no guarantee that a listener will receive an update.  It is therefore advised to additionally make use of the /status endpoints and the /tag_info endpoint to poll needed information as important events may be missed otherwise.

## <a name="menuNode">MenuNode</a>
|Field|Data type|Description
|-|-|-|
|id|String| guid of this menu node
|parentId|String| guid of parent node in menu
|type|String| The node type, one of <Product, Category>
|name|String| The name of the product or category
|bgColor|String|hex color code of the background
|thumbnailRef|[ImageRef](#imageRef)| Image reference for the thumbnail picture associated with the menu node
|children|List\<MenuNode>|List of child nodes

## <a name="menuProduct">MenuProduct</a>
|Field|Data type|Description
|-|-|-|
|id|String| guid of this menu node
|type|String| The node type, one of <Product, CustomValueProduct, Other>
|name|String|The name of the product, e.g. "Soda"
|priceInCredits|String|The price of a product in system credits. Always formatted with a decimal point, e.g. "3.2"
|priceInCreditsFormatted|String|The price of a product in system credits with the credit symbol. For displaying the credit value. Decimal separator may be location dependent, e.g. "3,20T"
|priceInCurrency|String|The price of a product in real world currency. Always formatted with a decimal point, e.g. "6.4"
|priceInCurrencyFormatted|String|the price of a product in real world currency with the currency symbol. For displaying the currency value. Decimal separator may be location dependent, e.g. "6,40€"
|genericData|List\<[Generic](#generic)>|List of generic functions associated with this menu node

## <a name="imageRef">ImageRef</a>
|Field|Data type|Description
|-|-|-|
|id|String| the guid of the image
|url|String| a relative url pointing to the image

## <a name="generic">Generic</a>
|Field|Data type|Description
|-|-|-|
|fullString|String| the complete generic as String, e.g. "simpleGeneric(Hello,World)"
|arg|List\<String>| List containing all arguments of the generic, e.g. ["Hello","World"]
|functionName|String| the name of the generic function, e.g. "simlpeGeneric"
|paramString|String| the generic's parameters as a single String, e.g. "Hello,World"

## <a name="deviceConfig">DeviceConfig</a>
|Field|Data type|Description
|-|-|-|
|creditSymbol|String| The symbol used for representing credits
|currencySymbol|String| The symbol used for representing real world currency
|deviceId|String| The short id of the device,r epresented as a four digit String, e.g. "ACZK"
|paymentTypes| List\<[PaymentType](#paymentType)>| A list of payment types enabled for this device
|projectName|String| the name of the current project
|siteName|String| the name of the active site
|unitName|String| the name of the unit the active site belongs to
|decimalSeparator|String| decimal separator used for displaying currency and credit values
|decimalPlacesCredits|int| the number of decimal places used for credits
|decimalPlacesCurrency|int| the number of decimal places used for currency

## <a name="fieldModel">FieldModel</a>
|Field|Data type|Description
|-|-|-|
|fieldKey|String| the key of the field, e.g. "Adult"
|filedType|String| one of <TypeBoolean, TypeInteger>
|integerField|[IntegerFieldModel](#integerFieldModel)| Set if fieldType is set to "TypeInteger"


## <a name="integerFieldModel">IntegerFieldModel</a>
|Field|Data type|Description
|-|-|-|
|minIntegerValue|int| the minimum value of the field
|maxIntegerValue|int| the maximum value of the field

## <a name="fieldChangeRequest">FieldChangeRequest</a>
|Field|Data type|Description
|-|-|-|
|key|String| the key of the field, e.g. "free_drink"
|value|String| String representation of the value. For Boolean fields this is one of <true,false>, for Integer fields the Integer value as String
|type|String| one of <BooleanSet, IntegerAbsolute, IntegerRelative>. The operation which should be applied onto the field. Dependent on field's data type

Example of different field changes

```
[
	{
		"key": "some_flag",
		"value":"true",
		"type":"BooleanSet"
	},
	{
		"key": "some_other_flag",
		"value":"5",
		"type":"IntegerAbsolute"
	},
	{
		"key": "third_flag",
		"value":"-3",
		"type":"IntegerRelative"
	},
	{
		"key": "another_flag",
		"value":"8",
		"type":"IntegerRelative"
	}
]
```

## <a name="paymentType">PaymentType</a>
|Field|Data type|Description
|-|-|-|
|id|String|The guid of the payment type
|name|String| The name of the payment type, e.g. "Cashless Card"
|type|String| one of <Cashless, Other>

## <a name="nfcTag">NfcTag</a>
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
|fields|List\<[FieldModel](#fieldModel)>| a list of fields


## <a name="cartItem">CartItem</a>
|Field|Data type|Description
|-|-|-|
|count|int| the quantity of the item
|menuItemId|String| the uuid of the specific item, can be retrieved via the menu endpoint
|singlePrice|String| the price in credits of one unit of this item, can be retrieved via the product endpoint

## <a name="jobInfo">JobInfo</a>
|Field|Data type|Description
|-|-|-|
|status_url|String| The url where status updates about the job can be found

## <a name="jobStatus">JobStatus</a>
|Field|Data type|Description
|-|-|-|
|createdAt|long| The date when the request was received which created this job. Milliseconds since unix epoch
|jobId|String| The uuid of the job
|result|[NfcTag](#nfcTag)|After successfully charging an nfc tag, the result contains the nfc tags state after it was charged, it contains null otherwise
|status|int|The current status of the job, one of <Pending, Cancelled, Success>
|statusDetails|String| A textual description of the current status of the job to help with debugging. If for example a invalid nfc tag is used, the status of the current job will not change (it will stay in its pending state) but the statusDetails field will now contain a textual note stating that an invalid tag was last placed on the reader.
