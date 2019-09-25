# PoS Reservation Methods: PoS -> MobilePay Backend

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->
- [ReservationStart](#ReservationStart)
- [GetReservationStatus](#GetReservationStatus)
- [ReservationCancel](#ReservationCancel)
- [GetCurrentReservation](#GetCurrentReservation)
- [ReservationCapture](#ReservationCapture)
<!-- /TOC -->

## ReservationStart
### Purpose:
This method is called when the Point of Sale (cash register / terminal) wishes to start a reservation (The customer does not have to be checked-in in advance). 
ReservationStart is only possible if no active MobilePay reservation entity exists for current PoS.
A ReservationStart will delete an earlier finished reservation entity – if it is   in status 'Done', 'Cancel' or 'Error'.
It is expected that the PoS system keeps track of reservations internally in order to be able to capture them late

### Request:
```json
{
"MerchantId":"POSDK99999",
"LocationId":"88888",
"PoSId":"a123456-b123-c123-d123-e12345678901",
"OrderId":"123A124321",
"Amount":"1023.43",
"BulkRef":"MP Bulk Reference",
“CaptureType”:”Full”
}
```
|Parameter    |Type        |Required  |Description                                                      |
|-------------|------------|----------|-----------------------------------------------------------------|
|MerchantId   |String      | required | Merchant ID related to current PoS ID. |
|LocationId   |String      | required | Location ID related to current merchant ID and PoS ID.|
|PoSId        |String      | required | Current Point of Sale ID (cash register/terminal). |
|OrderId      |String      | required | The OrderId is a unique id that identifies the payment. The OrderId is issued by the merchant and is attached to the reservation inside MobilePay system. <br>The order ID must be unique for the merchant/location combination. This means that there should be only one completed reservation with any given order ID for the same merchant and location (store) during the lifetime of the merchant/location.<br>CASE SENSITIVE|
|Amount       |String      | required | The amount for the payment. <br>Always with 2 decimals and no thousand separators.<br>Note: Decimal point is "." |
|BulkRef      |String      | required | An option for grouping the payments – a text or ID. The field has a maximum length of 18 characters.<br>If the field remains empty and the merchant does not have a Bulkpost agreement, the merchant will receive all mobile payments from any connected shops as individual postings in the reconciliation file.<br>If the field remains empty and the merchant does have a Bulkpost agreement, the merchant will receive all mobile payments bulked with a default bulkref of the MP Enterprise Serialnumber value in the reconciliation file.<br>It must be a merchant decision whether they want all individual postings or a bulk posting per store or the entire group as one posting.|
|CaptureType  |String      | required | Full = Fixed to always capture full (initial) amount when ReservationCapture is called<br>Partial = Amounts equal to or lower than initial amount sent with ReservationStart is accepted|

### Response
HTTP 200 – Ok
```json
{
"CustomerToken":null,
}
```
|Parameter    |Type        |Description                                                      |
|-------------|------------|-----------------------------------------------------------------|
|CustomerToken       |String      |Contains loyalty card number, if user is part of a loyalty program with the current Merchant, and has enrolled a loyalty card with MobilePay, the loyalty card <br>Null if not part of loyalty program. |

HTTP 400 – See ReservationStart error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### ReservationStart Error Codes
|Error code   |Error text       |
|-------------|-----------------|
|10           |Missing or invalid parameters. |
|30           |The key “MerchantId, LocationId and PoSId” does not exist. |
|40           |Reservation already exists and has been accepted. |
|50           |Reservation already in progress. |

## GetReservationStatus
### Purpose:
Get a Reservation status for current PoS ID.
Used for polling for status. Polling has to be done every 1 second until the ReservationStatus is 100 ('Done') or if the reservation request has been rejected (ReservationStatus 40 ('Cancel') or 50 ('Error')).

### Request:
```json
{
"MerchantId":"POSDK99999",
"LocationId":"88888",
"PoSId":"a123456-b123-c123-d123-e12345678901",
"OrderId":"123A124321"
}
```
|Parameter    |Type        |Required  |Description                                                      |
|-------------|------------|----------|-----------------------------------------------------------------|
|MerchantId   |String      | required | Merchant ID related to current PoS ID. |
|LocationId   |String      | required | Location ID related to current merchant ID and PoS ID.|
|PoSId        |String      | required | Current Point of Sale ID (cash register/terminal). |
|OrderId      |String      | required | Order ID related to current reservation status request. |

### Response
HTTP 200 – Ok
```json
{
"PoSId":"a123456-b123-c123-d123-e12345678901",
"ReservationStatus":100,
"OrderId":"123A124321",
"Amount":1023.43,
"CustomerId":" B52456-9123-A153-D923-721345678901",
"CustomerToken":"",
}
```
|Parameter    |Type        |Description                                                      |
|---------------------|------------|-----------------------------------------------------------------|
|PoSId                |String      |Unique ID that identifies the PoS that has initiated current reservation request.|
|ReservationStatus    |Integer     |See status values below <br>- **10 ('Idle')** No reservation request in the queue<br>- **20 ('Issued')** - Reservation request is sent to customer.<br>- **30 ('AwaitCheckIn')** - Await customer check-in.<br>- **40 ('Cancel')** - Customer has cancelled/rejected reservation request.<br>- **50 ('Error')** - MobilePay is not able to handle the reservation – the PoS should cancel the MobilePay reservation request.<br>- **80 ('ReservationAccepted')** - The reservation request is accepted by the customer – await reservation confirmation from the reservation transaction system.<br>- **100 ('Done')** - "reservation Confirmed" and TransactionId, Amount, CustomerId will contain a value. |
|OrderId              |String      |The OrderId assigned to current reservation. |
|TransactionId        |String      |Unique ID that identifies the payment (transaction ID). ID is generated by Danske Bank and is shown on the receipt inside the MobilePay app. |
|Amount               |Number      |The amount for the reservation. <br>Note: Decimal point is “.” |
|CustomerId           |String      |Unique ID of the customer. The ID is generated by MobilePay. |
|CustomerToken        |String      |Contains customer token if customer has checked-In with a merchant token ID related to this merchant’s loyalty program. |
|CustomerReceiptToken |String      |Used for customer receipt token (In DK: Service agreement with Storebox implies that Storebox user Id is provided) <br>Max 32 char. |
|PaymentSignature     |null        |null (deprecated). |


HTTP 400 – See GetReservationStatus error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### GetReservationStatus Error Codes
|Error code   |Error text       |
|-------------|-----------------|
|10           |Missing or invalid parameters. |
|30           |The key "MerchantId, LocationId and PoSId" does not exist. |

## ReservationCancel
### Purpose:
Cancel Reservation request for current PoS ID.
Cancel is principal possible as long as earlier request for reservation hasn't been finalized.
A ReservationCancel will delete current reservation entity active or not unless earlier finished reservation ended in status Done (status code 100) which will remains until a new reservation starts.

### Request:
```json
{
"MerchantId":"POSDK99999",
"LocationId":"88888",
"PoSId":"a123456-b123-c123-d123-e12345678901",
"OrderId":"123A124321"
}
```
|Parameter    |Type        |Required  |Description                                                      |
|-------------|------------|----------|-----------------------------------------------------------------|
|MerchantId   |String      | required | Merchant ID related to current PoS ID. |
|LocationId   |String      | required | Location ID related to current merchant ID and PoS ID.|
|PoSId   |String        | required | Current Point of Sale ID (cash register/terminal). |
|OrderId         |String      | required | The OrderId assigned to current reservation. |

### Response
HTTP 200 – Ok
```json
{
}
```

HTTP 400 – See ReservationCancel error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### ReservationCancel Error Codes
|Error code   |Error text       |
|-------------|-----------------|
|10           |Missing or invalid parameters. |
|30           |The key "MerchantId, LocationId and PoSId" does not exist. |
|50           |Action not possible. |

## GetCurrentReservation
### Purpose:
Get Information about the current reservation. 

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
|PoSId   |String        | required | Current Point of Sale ID (cash register/terminal). |

### Response
HTTP 200 – Ok
```json
{
  "PosId": " a123456-b123-c123-d123-e12345678901",
  "PosUnitId": "123456789012345",
  "PaymentStatus": 50,
  "OrderId": "123A124321",
  "TransactionId": null,
  "Amount": 0.01,
  "CustomerId": " B52456-9123-A153-D923-721345678901",
  "CustomerToken": null,
  "CustomerReceiptToken": null,
  "LastestUpdate": "2017-05-22T10:00:02.977"
}
```

HTTP 400 – See GetCurrentReservation error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```

## ReservationCapture
### Purpose:
This method is called when the Point of Sale (cash register / terminal) wishes to Capture the Reservation
ReservationCapture is only possible when a Reservation exists with the provided order ID. Reservations made as Full Capture reservations should always be captured with 0.00 in amount, and Partial Capture with the amount to capture

### Request:
```json
{
"MerchantId":"POSDK99999",
"LocationId":"88888",
"PoSId":"a123456-b123-c123-d123-e12345678901",
"OrderId":"123A124321",
"Amount":"1023.43",
"BulkRef":"MP Bulk Reference"
}
```
|Parameter    |Type        |Required  |Description                                                      |
|-------------|------------|----------|-----------------------------------------------------------------|
|MerchantId   |String      | required | Merchant ID related to current PoS ID. |
|LocationId   |String      | required | Location ID related to current merchant ID and PoS ID.|
|PoSId   |String        | required | Current Point of Sale ID (cash register/terminal). |
|OrderId         |String      | required | The OrderId is a unique id that identifies the payment. The OrderId is issued by the merchant and is attached to the reservation inside MobilePay system. <br>The order ID must be unique for the merchant/location combination. This means that there should be only one completed reservation with any given order ID for the same merchant and location (store) during the lifetime of the merchant/location.<br>CASE SENSITIVE |
|Amount         |String      | required | The amount for the payment. <br>Always with 2 decimals and no thousand separators.<br>For capture of reservations made with CaptureType = Full – amount is to be set to 0.00<br>Note: Decimal point is "." |
|BulkRef         |String      | required | An option for grouping the payments – a text or ID. The field has a maximum length of 18 characters.<br>If the field remains empty and the merchant does not have a Bulkpost agreement, the merchant will receive all mobile payments from any connected shops as individual postings in the reconciliation file.<br>If the field remains empty and the merchant does have a Bulkpost agreement, the merchant will receive all mobile payments bulked with a default bulkref of the MP Enterprise Serialnumber value in the reconciliation file.<br>It must be a merchant decision whether they want all individual postings or a bulk posting per store or the entire group as one posting.<br>The field is mandatory in the request even though it might be an empty string.|

### Response
HTTP 200 – Ok
```json
{
}
```
|Parameter    |Type        |Description                                                      |
|-------------|------------|-----------------------------------------------------------------|
|CustomerReceiptToken       |String      |Used for customer receipt token (In DK: Service agreement with Storebox implies that Storebox user Id is provided).<br>May be null.<br>Max 32 characters.|

HTTP 400 – See ReservationCapture error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### ReservationCapture Error Codes
|Error code   |Error text       |
|-------------|-----------------|
|10           |Missing or invalid parameters. |
|30           |The key “MerchantId, LocationId and PoSId” does not exist. |
|50           |Error during card capture process. |


