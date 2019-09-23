# MobilePay-PoS
Our MobilePay PoS REST api  is intended for software developers implementing MobilePay payments in a PoS system

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->
- [RegisterPoS](#RegisterPoS)
- [UpdateRegisteredPoSName](#UpdateRegisteredPoSName)
<!-- /TOC -->


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

## UnRegisterPoS
### Purpose:
Unregister current Point of Sale terminal ID in the MobilePay PoS system.
Data validation:
Call used MerchantId, LocationId and PoSId must be validated as known and related before PoS unregister session will succeed.
### Request:
```json
{
"MerchantId":"POSDK99999",
"LocationId":"88888",
"PoSId":"a123456-b123-c123-d123-e12345678901"

}
```
|Parameter    |Type        |Required  |Description                                                      |
|-------------|------------|----------|-----------------------------------------------------------------|
|MerchantId   |String      | required | Merchant ID related to current PoS ID. |
|LocationId   |String      | required | Location ID related to current merchant ID and PoS ID. |
|PoSId   |String        | required | Current Point of Sale ID (cash register / terminal)|

### Response
HTTP 200 – Ok
```json
{
}
```

HTTP 400 – See UnRegisterPoS  error codes
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
|30           |The Key “MerchantId, LocationId and PoSId” does not exist. | 

## AssignPoSUnitIdToPos
### Purpose:
Assign a PoSUnit ID to current PoS (cash register / terminal) in the MobilePay PoS system.
Data validation:
*	Call used MerchantId , LocationId and PoSId must be validated as known and related before registration will succeed. 
*	Assigned PoSUnitId must be validated as system unique before registration will succeed.

When calling AssignPoSUnitIdToPos, the normal situation is that a new PoS unit is assigned to a cash register which previously had no assigned PoS unit. Note, however: 
*	The method works even though the PoS unit is already assigned to another cash register
*	The method works even though the PoS already has another assigned PoS unit

If there is an existing payment for the PoS unit, this will be cancelled when AssignPoSUnitIdToPos is called. If PoS unit A is already assigned to the cash register, and AssignPoSUnitIdToPos is called for another PoS unit B, then any payment belonging to either PoS unit A or PoS unit B are cancelled.

### Request:
```json
{
"MerchantId":"POSDK99999",
"LocationId":"88888",
"PoSId":"a123456-b123-c123-d123-e12345678901",
"PoSUnitId":"123456789012345"
}
```
|Parameter    |Type        |Required  |Description                                                      |
|-------------|------------|----------|-----------------------------------------------------------------|
|MerchantId   |String      | required | Merchant ID related to current PoS ID. |
|LocationId   |String      | required | Location ID related to current merchant ID and PoS ID.|
|PoSId   |String        | required | Current Point of Sale ID (cash register/terminal). |
|PoSUnitId         |String      | required | Current PoSUnitId to be assigned |

### Response
HTTP 200 – Ok
```json
{
}
```

HTTP 400 – See AssignPoSUnitIdToPos error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### AssignPoSUnitIdToPos Error Codes
|Error code   |Error text       |
|-------------|-----------------|
|10           |Missing or invalid parameters. |

## UnAssignPoSUnitIdToPoS
### Purpose:
Unassign current PoSUnitId from current PoS cash register / terminal.
Data validation:
Call used MerchantId, LocationId, PoSId and PoSUnitId must be validated as known and related before unlinking will succeed. 

### Request:
```json
{
"MerchantId":"POSDK99999",
"LocationId":"88888",
"PoSId":"a123456-b123-c123-d123-e12345678901",
"PoSUnitId":"123456789012345"
}
```
|Parameter    |Type        |Required  |Description                                                      |
|-------------|------------|----------|-----------------------------------------------------------------|
|MerchantId   |String      | required | Merchant ID related to current PoS ID. |
|LocationId   |String      | required | Location ID related to current merchant ID and PoS ID.|
|PoSId   |String        | required | Current Point of Sale ID (cash register/terminal). |
|PoSUnitId         |String      | required | Current PoSUnitId to be unassigned |

### Response
HTTP 200 – Ok
```json
{
}
```

HTTP 400 – See UnAssignPoSUnitIdToPoS error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### UnAssignPoSUnitIdToPoS Error Codes
|Error code   |Error text       |
|-------------|-----------------|
|10           |Missing or invalid parameters. |
|30           |The Key: MerchantId, LocationId, PoSId and PoSUnitId does not exist. | 

## ReadPoSAssignPoSUnitId
### Purpose:
Read PoSUnitId assigned to current PoS terminal.
Data validation:
Call used MerchantId, LocationId and PoSId must be validated as known and related before PoSUnitId value is returned. 

### Request:
```json
{
"MerchantId":"POSDK99999",
"LocationId":"88888",
"PoSId":"a123456-b123-c123-d123-e12345678901"
}
```
|Parameter    |Type        |Required  |Description                                                      |
|-------------|------------|----------|-----------------------------------------------------------------|
|MerchantId   |String      | required | Merchant ID related to current PoS ID. |
|LocationId   |String      | required | Location ID related to current merchant ID and PoS ID.|
|PoSId   |String        | required | Point of Sale ID (cash register / terminal) that is assigned to current PoSUnitId. |

### Response
HTTP 200 – Ok
```json
{
"PoSUnitId":"123456789012345"
}
```
|Parameter    |Type       |Description                                                      |
|-------------|-----------|-----------------------------------------------------------------|
|PoSUnitId   |String      | Current assigned PoSUnitId. Empty string if none assigned. |

HTTP 400 – See ReadPoSAssignPoSUnitId error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### ReadPoSAssignPoSUnitId Error Codes
|Error code   |Error text       |
|-------------|-----------------|
|10           |Missing or invalid parameters. |
|30           |The Key: MerchantId, LocationId and PoSId does not exist. | 

## ReadPoSUnitAssignedPoSId
### Purpose:
Read PoS (cash register / terminal) ID that is assigned to current PoSUnitId
Data validation:
Call used MerchantId, LocationId and PoSUnitId must be validated as known and related before PoSUnit assigned PoS ID value is returned. 

### Request:
```json
{
"MerchantId":"POSDK99999",
"LocationId":"88888",
"PoSUnitId":"123456789012345"
}
```
|Parameter    |Type        |Required  |Description                                                      |
|-------------|------------|----------|-----------------------------------------------------------------|
|MerchantId   |String      | required | Merchant ID related to current PoS ID. |
|LocationId   |String      | required | Location ID related to current merchant ID and PoS ID.|
|PoSUnitId         |String      | required | Current PoSUnitId |

### Response
HTTP 200 – Ok
```json
{
"PoSId":"a123456-b123-c123-d123-e12345678901"
}
```
|Parameter    |Type       |Description                                                      |
|-------------|-----------|-----------------------------------------------------------------|
|PoSId   |String      | Point of Sale ID (cash register / terminal) that is assigned to current PoSUnitId. |

HTTP 400 – See ReadPoSUnitAssignedPoSId error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### ReadPoSUnitAssignedPoSId Error Codes
|Error code   |Error text       |
|-------------|-----------------|
|10           |Missing or invalid parameters. |
|30           |The Key: MerchantId, LocationId and PoSUnitId does not exist. | 

## GetUniquePoSId
### Purpose:
Get a global unique PoSId to use for registering a new PoS in MobilePay Backend system.
A Globally Unique IDentifier (GUID) is a unique reference number used as an identifier in computer software. The term GUID typically refers to various implementations of the universally unique identifier (UUID) standard.
GUIDs are usually stored as 128-bit values, and are commonly displayed as 32 hexadecimal digits with groups separated by hyphens, such as {21EC2020-3AEA-4069-A2DD-08002B30309D}.

### Request:
```json
{
"MerchantId":"POSDK99999"
}
```
|Parameter    |Type        |Required  |Description                                                      |
|-------------|------------|----------|-----------------------------------------------------------------|
|MerchantId   |String      | required | Merchant ID must be known in the system. |

### Response
HTTP 200 – Ok
```json
{
"PoSId":"a123456-b123-c123-d123-e12345678901"
}
```
|Parameter    |Type       |Description                                                      |
|-------------|-----------|-----------------------------------------------------------------|
|PoSId   |String      | GUID – global unique ID to POS. |

HTTP 400 – See GetUniquePoSId error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### GetUniquePoSId Error Codes
|Error code   |Error text       |
|-------------|-----------------|
|10           |Missing or invalid parameters. |
|20           |The Key: MerchantId does not exist. |
