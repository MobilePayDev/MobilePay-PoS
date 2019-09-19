# MobilePay-PoS
Our MobilePay PoS REST api  is intended for software developers implementing MobilePay payments in a PoS system

## RegisterPoS
### Purpose:
Register a new Point of Sale terminal in the MobilePay PoS system. This must be done before PoSUnit ID is associated with a PoS.
Data validation:
Call used MerchantId and LocationId must be validated as known and related in addition to used PoSId must be unique before registration will succeed. 
### Request:
```json
{
"MerchantId":"POSDK99999",
"LocationId":"88888",
"PoSId":"a123456-b123-c123-d123-e12345678901",
"Name":"PoS Name"
}
```
|Parameter    |Type        |Required  |Description                                                      |
|-------------|------------|----------|-----------------------------------------------------------------|
|MerchantId   |String      | required | The merchant identification number provided by MobilePay. |
|LocationId   |String      | required | Location ID related to current merchant ID provided by MobilePay. |
|PoSId   |String        | optional | MobilePay system unique Point of Sale ID (cash register / terminal) – provided by either merchant (256 characters) or MobilePay PoS number generator (36 characters). If the request contains an empty PoSId value, the response will contain current auto-generated PoS assigned ID. CASE SENTITIVE |
|Name         |String      | required | PoS name to be shown in MobilePay App when the customer has checked-in. Example: “Cash register 1” |

### Response
HTTP 200 – Ok
```json
{
"PoSId":"a123456-b123-c123-d123-e12345678901"
}
```
|Parameter    |Type        |Description                                                      |
|-------------|------------|-----------------------------------------------------------------|
|PoSId        |String      | MobilePay system unique PoS ID  – either the PoSId value contained in the RegisterPoS request or supplied by merchant. |

HTTP 400 – See RegisterPoS error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### RegisterPoS Error Codes
|Error code   |Error text       |
|-------------|-----------------|
|10           |Missing or invalid parameters. | 

## UpdateRegisteredPoSName
### Purpose:
Update registered Point of Sale name in the MobilePay PoS system.
Data validation:
Call used MerchantId, LocationId and PoSId must be validated as known and related before name update will succeed. 
### Request:
```json
{
"MerchantId":"POSDK99999",
"LocationId":"88888",
"PoSId":"a123456-b123-c123-d123-e12345678901",
"Name":" PoS Name"
}

```
|Parameter    |Type        |Required  |Description                                                      |
|-------------|------------|----------|-----------------------------------------------------------------|
|MerchantId   |String      | required | Merchant ID related to current PoS ID. |
|LocationId   |String      | required | Location ID related to current merchant ID and PoS ID. |
|PoSId   |String        | required |Current Point of Sale ID (cash register/terminal) |
|Name         |String      | required | Specified PoS name to be shown in MobilePay App when customer has checked-in |

### Response
HTTP 200 – Ok
```json
{
}
```

HTTP 400 – See UpdateRegisteredPoSName error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### UpdateRegisteredPoSName  Error Codes
|Error code   |Error text       |
|-------------|-----------------|
|10           |Missing or invalid parameters. | 

