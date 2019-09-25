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
"CaptureType”:”Full"
}
```
<table>
  <tr>
    <th>Parameter</th>
    <th>Type</th>
    <th>Required</th>
    <th>Description</th>	
  </tr>
  <tr>
    <td>MerchantId</td>
    <td>String</td>
    <td>required</td>
    <td>Id of the merchant.</td>
  </tr>
  <tr>
    <td>LocationId</td>
    <td>String</td>
    <td>required</td>
    <td>Location ID related to current merchant ID and PoS ID.</td>
  </tr>
  <tr>
    <td>PoSId</td>
    <td>String</td>
    <td>required</td>
    <td>Current Point of Sale ID (cash register/terminal).</td>
  </tr>
  <tr>
    <td>OrderId</td>
    <td>String</td>
    <td>required</td>
    <td>Order ID that identifies the payment to refund. </td>
  </tr>
  <tr>
    <td>Amount</td>
    <td>String</td>
    <td>required</td>
    <td>Amount to refund. <br>If  0.00 or blank, the whole transaction will be refunded.<br>Note: Decimal point is “.”<br>Examples: "Amount": "0.00"  -  "Amount": "100.00"</td>
  </tr>
  <tr>
    <td>BulkRef</td>
    <td>String</td>
    <td>required</td>
    <td>Note: this parameter is required but currently only a placeholder for future use. For now, passed values will not be used, but we recommend preparing your solution for this feature. We expect to implement this in an upcoming update of the API.<br> An option for grouping the refunds – a text or ID. The field has a maximum length of 18 characters.<br> If the field remains empty and the merchant does not have a Bulkpost agreement, the merchant will receive all mobile refunds from any connected shops as individual postings in the reconciliation file.<br> If the field remains empty and the merchant does have a Bulkpost agreement, the merchant will receive all mobile refunds bulked with a default BulkRef of the MP Enterprise Serialnumber value in the reconciliation file.<br> It must be a merchant decision whether they want all individual postings or an bulk posting per store or the entire group as one posting.</td>
  </tr>
  <tr>
    <td>CaptureType</td>
    <td>String</td>
    <td>required</td>
    <td> Full = Fixed to always capture full (initial) amount when ReservationCapture is called<br>Partial = Amounts equal to or lower than initial amount sent with ReservationStart is accepted</td>
  </tr>
</table>

### Response
HTTP 200 – Ok
```json
{
"CustomerToken":null,
}
```
<table>
  <tr>
    <th>Parameter</th>
    <th>Type</th>
    <th>Description</th>	
  </tr>
  <tr>
    <td>CustomerToken</td>
    <td>String</td>
    <td>Contains loyalty card number, if user is part of a loyalty program with the current Merchant, and has enrolled a loyalty card with MobilePay, the loyalty card <br>Null if not part of loyalty program. </td>
  </tr>
</table>

HTTP 400 – See ReservationStart error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### ReservationStart Error Codes
<table>
  <tr>
    <th>Error code</th>
    <th>Error text</th>
  </tr>
    <td>10</td>
    <td>Missing or invalid parameters.</td>
  </tr>
  <tr>
    <td>30</td>
    <td>The key “MerchantId, LocationId and PoSId” does not exist</td>
  </tr>
  <tr>
    <td>40</td>
    <td>Reservation already exists and has been accepted.</td>
  </tr>
  <tr>
    <td>50</td>
    <td>Reservation already in progress. </td>
  </tr>
</table>

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
<table>
  <tr>
    <th>Parameter</th>
    <th>Type</th>
    <th>Required</th>
    <th>Description</th>	
  </tr>
  <tr>
    <td>MerchantId</td>
    <td>String</td>
    <td>required</td>
    <td>Id of the merchant.</td>
  </tr>
  <tr>
    <td>LocationId</td>
    <td>String</td>
    <td>required</td>
    <td>Location ID related to current merchant ID and PoS ID.</td>
  </tr>
  <tr>
    <td>PoSId</td>
    <td>String</td>
    <td>required</td>
    <td>Current Point of Sale ID (cash register/terminal).</td>
  </tr>
  <tr>
    <td>OrderId</td>
    <td>String</td>
    <td>required</td>
    <td>Order ID related to current reservation status request. </td>
  </tr>
</table>

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
<table>
  <tr>
    <th>Parameter</th>
    <th>Type</th>
    <th>Description</th>	
  </tr>
  <tr>
    <td>PoSId</td>
    <td>String</td>
    <td>Unique ID that identifies the PoS that has initiated current reservation  request.</td>
  </tr>
  <tr>
    <td>PaymentStatus</td>
    <td>Integer</td>
    <td>See status values below <br>- <b>10 ('Idle')</b> No reservation  request in the queue<br>- <b>20 ('Issued')</b> - reservation  request is sent to customer.<br>- <b>30 ('AwaitCheckIn')</b> - Await customer check-in.<br>- <b>40 ('Cancel')</b> - Customer has cancelled/rejected reservation  request.<br>- <b>50 ('Error')</b> - MobilePay is not able to handle the reservation  – the PoS should cancel the MobilePay reservation  request.<br>- <b>80 ('ReservationAccepted')</b> - The reservation request is accepted by the customer – await reservation  confirmation from the reservation  transaction system.<br>- <b>100 ('Done')</b> - "reservation  Confirmed" and TransactionId, PaymentSignature, Amount, CustomerId (optional) will contain a value.</td>
  </tr>
  <tr>
    <td>OrderId</td>
    <td>String</td>
    <td>The OrderId assigned to current reservation .</td>
  </tr>
  <tr>
    <td>TransactionId</td>
    <td>String</td>
    <td>Unique ID that identifies the payment (transaction ID). ID is generated by MobilePay and is shown on the receipt inside the MobilePay app.  </td>
  </tr>
  <tr>
    <td>Amount</td>
    <td>Number</td>
    <td>The amount for the reservation. <br>Note: Decimal point is “.”</td>
  </tr>
  <tr>
    <td>CustomerId</td>
    <td>String</td>
    <td>Unique ID of the customer. The ID is generated by MobilePay. </td>
  </tr>
  <tr>
    <td>CustomerToken</td>
    <td>String</td>
    <td>Contains customer token if customer has checked-In with a merchant token ID related to this merchant’s loyalty program. </td>
  </tr>
  <tr>
    <td>CustomerReceiptToken</td>
    <td>String</td>
    <td>Used for customer receipt token (In DK: Service agreement with Storebox implies that Storebox user Id is provided) <br>Max 32 char.</td>
  </tr>
  <tr>
    <td>PaymentSignature</td>
    <td>null</td>
    <td>null (deprecated).</td>
  </tr>
</table>


HTTP 400 – See GetReservationStatus error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### GetReservationStatus Error Codes
<table>
  <tr>
    <th>Error code</th>
    <th>Error text</th>
  </tr>
    <td>10</td>
    <td>Missing or invalid parameters.</td>
  </tr>
  <tr>
    <td>30</td>
    <td>The key “MerchantId, LocationId and PoSId” does not exist</td>
  </tr>
</table>

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
<table>
  <tr>
    <th>Parameter</th>
    <th>Type</th>
    <th>Required</th>
    <th>Description</th>	
  </tr>
  <tr>
    <td>MerchantId</td>
    <td>String</td>
    <td>required</td>
    <td>Id of the merchant.</td>
  </tr>
  <tr>
    <td>LocationId</td>
    <td>String</td>
    <td>required</td>
    <td>Location ID related to current merchant ID and PoS ID.</td>
  </tr>
  <tr>
    <td>PoSId</td>
    <td>String</td>
    <td>required</td>
    <td>Current Point of Sale ID (cash register/terminal).</td>
  </tr>
  <tr>
    <td>OrderId</td>
    <td>String</td>
    <td>required</td>
    <td>Order ID related to current reservation</td>
  </tr>
</table>

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
<table>
  <tr>
    <th>Error code</th>
    <th>Error text</th>
  </tr>
    <td>10</td>
    <td>Missing or invalid parameters.</td>
  </tr>
  <tr>
    <td>30</td>
    <td>The key “MerchantId, LocationId and PoSId” does not exist</td>
  </tr>
  <tr>
    <td>50</td>
    <td>Action not possible. </td>
  </tr>
</table>

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
<table>
  <tr>
    <th>Parameter</th>
    <th>Type</th>
    <th>Required</th>
    <th>Description</th>	
  </tr>
  <tr>
    <td>MerchantId</td>
    <td>String</td>
    <td>required</td>
    <td>Id of the merchant.</td>
  </tr>
  <tr>
    <td>LocationId</td>
    <td>String</td>
    <td>required</td>
    <td>Location ID related to current merchant ID and PoS ID.</td>
  </tr>
  <tr>
    <td>PoSId</td>
    <td>String</td>
    <td>required</td>
    <td>Current Point of Sale ID (cash register/terminal).</td>
  </tr>
</table>

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
<table>
  <tr>
    <th>Parameter</th>
    <th>Type</th>
    <th>Required</th>
    <th>Description</th>	
  </tr>
  <tr>
    <td>MerchantId</td>
    <td>String</td>
    <td>required</td>
    <td>Id of the merchant.</td>
  </tr>
  <tr>
    <td>LocationId</td>
    <td>String</td>
    <td>required</td>
    <td>Location ID related to current merchant ID and PoS ID.</td>
  </tr>
  <tr>
    <td>PoSId</td>
    <td>String</td>
    <td>required</td>
    <td>Current Point of Sale ID (cash register/terminal).</td>
  </tr>
  <tr>
    <td>OrderId</td>
    <td>String</td>
    <td>required</td>
    <td>The OrderId is a unique id that identifies the payment. The OrderId is issued by the merchant and is attached to the reservation inside MobilePay system. <br>The order ID must be unique for the merchant/location combination. This means that there should be only one completed reservation with any given order ID for the same merchant and location (store) during the lifetime of the merchant/location.<br>CASE SENSITIVE</td>
  </tr>
  <tr>
    <td>Amount</td>
    <td>String</td>
    <td>required</td>
    <td>The amount for the payment. <br>Always with 2 decimals and no thousand separators.<br>For capture of reservations made with CaptureType = Full – amount is to be set to 0.00<br>Note: Decimal point is "."</td>
  </tr>
  <tr>
    <td>BulkRef</td>
    <td>String</td>
    <td>required</td>
    <td>An option for grouping the payments – a text or ID. The field has a maximum length of 18 characters.<br>If the field remains empty and the merchant does not have a Bulkpost agreement, the merchant will receive all mobile payments from any connected shops as individual postings in the reconciliation file.<br>If the field remains empty and the merchant does have a Bulkpost agreement, the merchant will receive all mobile payments bulked with a default bulkref of the MP Enterprise Serialnumber value in the reconciliation file.<br>It must be a merchant decision whether they want all individual postings or a bulk posting per store or the entire group as one posting.<br>The field is mandatory in the request even though it might be an empty string.</td>
  </tr>
</table>

### Response
HTTP 200 – Ok
```json
{
}
```
<table>
  <tr>
    <th>Parameter</th>
    <th>Type</th>
    <th>Description</th>	
  </tr>
  <tr>
    <td>CustomerReceiptToken</td>
    <td>String</td>
    <td>Used for customer receipt token (In DK: Service agreement with Storebox implies that Storebox user Id is provided).<br>May be null.<br>Max 32 characters.</td>
  </tr>
</table>



HTTP 400 – See ReservationCapture error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### ReservationCapture Error Codes
<table>
  <tr>
    <th>Error code</th>
    <th>Error text</th>
  </tr>
    <td>10</td>
    <td>Missing or invalid parameters.</td>
  </tr>
  <tr>
    <td>30</td>
    <td>The key “MerchantId, LocationId and PoSId” does not exist</td>
  </tr>
  <tr>
    <td>50</td>
    <td>Error during card capture process.</td>
  </tr>
</table>

