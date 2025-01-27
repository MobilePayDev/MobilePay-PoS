# PoS Registration Methods: PoS -> MobilePay Backend
The PoS Administration sequence must be completed before a MobilePay payment request can be issued by the PoS device. 
This administration must be done either through Merchant Backend for all PoS devices or via the individual PoS device as needs arise depended on the PoS Vendor / Merchant Backend solution.
The relation between Merchant Identifier, Location Identifier, PoS Identifier and PoSUnit Identifier must be held by Bank MobilePay Backend.


<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->
- [PoS administration](#RegisterPoS)
    - [RegisterPoS](#RegisterPoS)
    - [UpdateRegisteredPoSName](#UpdateRegisteredPoSName)
    - [UnRegisterPoS](#UnRegisterPoS)
- [PoSUnit administration](#AssignPoSUnitIdToPos)
    - [AssignPoSUnitIdToPos](#AssignPoSUnitIdToPos)
    - [UnAssignPoSUnitIdToPoS](#UnAssignPoSUnitIdToPoS)
- [PoS/PoSUnit Support](#ReadPoSAssignPoSUnitId)
    - [ReadPoSAssignPoSUnitId](#ReadPoSAssignPoSUnitId)
    - [ReadPoSUnitAssignedPoSId](#ReadPoSUnitAssignedPoSId)
    - [GetUniquePoSId](#GetUniquePoSId)
    - [GetCurrentPayment](#GetCurrentPayment)
    - [GetPosList](#GetPosList)
- [Location Support](#GetLocationList)
    - [GetLocationList](#GetLocationList)
<!-- /TOC -->

## <a name="RegisterPos"></a>RegisterPoS
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
    <td>The merchant identification number provided by MobilePay.</td>
  </tr>
  <tr>
    <td>LocationId</td>
    <td>String</td>
    <td>required</td>
    <td>Location ID related to current merchant ID provided by MobilePay.</td>
  </tr>
  <tr>
    <td>PoSId</td>
    <td>String</td>
    <td>Optional</td>
    <td>MobilePay system unique Point of Sale ID (cash register / terminal) – provided by either merchant (256 characters) or MobilePay PoS number generator (36 characters). If the request contains an empty PoSId value, the response will contain current auto-generated PoS assigned ID.<br> CASE SENTITIVE</td>
  </tr>
  <tr>
    <td>Name</td>
    <td>String</td>
    <td>required</td>
    <td>PoS name to be shown in MobilePay App when the customer has checked-in. Example: “Cash register 1”</td>
  </tr>
</table>

### Response
HTTP 200 – Ok
```json
{
"PoSId":"a123456-b123-c123-d123-e12345678901"
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
    <td>MobilePay system unique PoS ID  – either the PoSId value contained in the RegisterPoS request or supplied by merchant.</td>
  </tr>
</table>

HTTP 400 – See RegisterPoS error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### RegisterPoS Error Codes
<table>
  <tr>
    <th>Error code</th>
    <th>Error text</th>
  </tr>
  <tr>
    <td>10</td>
    <td>Missing or invalid parameters.</td>
  </tr>
</table>

## <a name="UpdateRegisteredPoSName"></a>UpdateRegisteredPoSName
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
    <td> Merchant ID related to current PoS ID. </td>
  </tr>
  <tr>
    <td>LocationId</td>
    <td>String</td>
    <td>required</td>
    <td> Location ID related to current merchant ID and PoS ID.</td>
  </tr>
  <tr>
    <td>PoSId</td>
    <td>String</td>
    <td>Required</td>
    <td>Current Point of Sale ID (cash register/terminal)</td>
  </tr>
  <tr>
    <td>Name</td>
    <td>String</td>
    <td>required</td>
    <td>Specified PoS name to be shown in MobilePay App when customer has checked-in</td>
  </tr>
</table>

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
<table>
  <tr>
    <th>Error code</th>
    <th>Error text</th>
  </tr>
  <tr>
    <td>10</td>
    <td>Missing or invalid parameters.</td>
  </tr>
</table>

## <a name="UnRegisterPoS"></a>UnRegisterPoS
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
    <td> Merchant ID related to current PoS ID. </td>
  </tr>
  <tr>
    <td>LocationId</td>
    <td>String</td>
    <td>required</td>
    <td> Location ID related to current merchant ID and PoS ID.</td>
  </tr>
  <tr>
    <td>PoSId</td>
    <td>String</td>
    <td>Required</td>
    <td>Current Point of Sale ID (cash register/terminal)</td>
  </tr>
</table>

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
<table>
  <tr>
    <th>Error code</th>
    <th>Error text</th>
  </tr>
  <tr>
    <td>10</td>
    <td>Missing or invalid parameters.</td>
  </tr>
  <tr>
    <td>30</td>
    <td>The Key “MerchantId, LocationId and PoSId” does not exist. </td>
  </tr>
</table>

## <a name="AssignPoSUnitIdToPos"></a>AssignPoSUnitIdToPos
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
    <td> Merchant ID related to current PoS ID. </td>
  </tr>
  <tr>
    <td>LocationId</td>
    <td>String</td>
    <td>required</td>
    <td> Location ID related to current merchant ID and PoS ID.</td>
  </tr>
  <tr>
    <td>PoSId</td>
    <td>String</td>
    <td>Required</td>
    <td>Current Point of Sale ID (cash register/terminal)</td>
  </tr>
  <tr>
    <td>PoSUnitId</td>
    <td>String</td>
    <td>Required</td>
    <td>Current PoSUnitId to be assigne</td>
  </tr>
</table>

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
<table>
  <tr>
    <th>Error code</th>
    <th>Error text</th>
  </tr>
  <tr>
    <td>10</td>
    <td>Missing or invalid parameters.</td>
  </tr>
</table>

## <a name="UnAssignPoSUnitIdToPoS"></a>UnAssignPoSUnitIdToPoS
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
    <td> Merchant ID related to current PoS ID. </td>
  </tr>
  <tr>
    <td>LocationId</td>
    <td>String</td>
    <td>required</td>
    <td> Location ID related to current merchant ID and PoS ID.</td>
  </tr>
  <tr>
    <td>PoSId</td>
    <td>String</td>
    <td>Required</td>
    <td>Current Point of Sale ID (cash register/terminal)</td>
  </tr>
  <tr>
    <td>PoSUnitId</td>
    <td>String</td>
    <td>Required</td>
    <td>Current PoSUnitId to be assigne</td>
  </tr>
</table>

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
<table>
  <tr>
    <th>Error code</th>
    <th>Error text</th>
  </tr>
  <tr>
    <td>10</td>
    <td>Missing or invalid parameters.</td>
  </tr>
  <tr>
    <td>30</td>
    <td>The Key “MerchantId, LocationId and PoSId” does not exist. </td>
  </tr>
</table>

## <a name="ReadPoSAssignPoSUnitId"></a>ReadPoSAssignPoSUnitId
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
    <td> Merchant ID related to current PoS ID. </td>
  </tr>
  <tr>
    <td>LocationId</td>
    <td>String</td>
    <td>required</td>
    <td> Location ID related to current merchant ID and PoS ID.</td>
  </tr>
  <tr>
    <td>PoSId</td>
    <td>String</td>
    <td>Required</td>
    <td>Point of Sale ID (cash register / terminal) that is assigned to current PoSUnitId.</td>
  </tr>
</table>

### Response
HTTP 200 – Ok
```json
{
"PoSUnitId":"123456789012345"
}
```
<table>
  <tr>
    <th>Parameter</th>
    <th>Type</th>
    <th>Description</th>	
  </tr>
  <tr>
    <td>PoSUnitId</td>
    <td>String</td>
    <td>Current assigned PoSUnitId. Empty string if none assigned.</td>
  </tr>
</table>

HTTP 400 – See ReadPoSAssignPoSUnitId error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### ReadPoSAssignPoSUnitId Error Codes
<table>
  <tr>
    <th>Error code</th>
    <th>Error text</th>
  </tr>
  <tr>
    <td>10</td>
    <td>Missing or invalid parameters.</td>
  </tr>
  <tr>
    <td>30</td>
    <td>The Key “MerchantId, LocationId and PoSId” does not exist. </td>
  </tr>
</table>

## <a name="ReadPoSUnitAssignedPoSId"></a>ReadPoSUnitAssignedPoSId
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
    <td> Merchant ID related to current PoS ID. </td>
  </tr>
  <tr>
    <td>LocationId</td>
    <td>String</td>
    <td>required</td>
    <td> Location ID related to current merchant ID and PoS ID.</td>
  </tr>
  <tr>
    <td>PoSUnitId</td>
    <td>String</td>
    <td>Required</td>
    <td>Current PoSUnitId</td>
  </tr>
</table>

### Response
HTTP 200 – Ok
```json
{
"PoSId":"a123456-b123-c123-d123-e12345678901"
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
    <td>Point of Sale ID (cash register / terminal) that is assigned to current PoSUnitId. </td>
  </tr>
</table>

HTTP 400 – See ReadPoSUnitAssignedPoSId error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### ReadPoSUnitAssignedPoSId Error Codes
<table>
  <tr>
    <th>Error code</th>
    <th>Error text</th>
  </tr>
  <tr>
    <td>10</td>
    <td>Missing or invalid parameters.</td>
  </tr>
  <tr>
    <td>30</td>
    <td>The Key “MerchantId, LocationId and PoSId” does not exist. </td>
  </tr>
</table> 

## <a name="GetUniquePoSId"></a>GetUniquePoSId
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
    <td>Merchant ID must be known in the system.</td>
  </tr>
</table>

### Response
HTTP 200 – Ok
```json
{
"PoSId":"a123456-b123-c123-d123-e12345678901"
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
    <td>GUID – global unique ID to POS.</td>
  </tr>
</table>

HTTP 400 – See GetUniquePoSId error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### GetUniquePoSId Error Codes
<table>
  <tr>
    <th>Error code</th>
    <th>Error text</th>
  </tr>
  <tr>
    <td>10</td>
    <td>Missing or invalid parameters.</td>
  </tr>
  <tr>
    <td>20</td>
    <td>The Key: MerchantId does not exist. </td>
  </tr>
</table>

## <a name="GetCurrentPayment"></a>GetCurrentPayment
### Purpose:
Get the current payment transaction for the provided PoSId.

### Request:
```json
{
"MerchantId":"POSDK99999",
"LocationId":"88888",
"PoSId":" a123456-b123-c123-d123-e12345678901"
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
    <td>Merchant ID</td>
  </tr>
  <tr>
    <td>LocationId</td>
    <td>String</td>
    <td>required</td>
    <td>Location ID</td>
  </tr>
  <tr>
    <td>PoSId</td>
    <td>String</td>
    <td>Required</td>
    <td>PoSId</td>
  </tr>
</table>

### Response
HTTP 200 – Ok
```json
{
  "PoSId":"a123456-b123-c123-d123-e12345678901",
  "PosUnitId": "100000000000001",
  "PaymentStatus": 100,
  "OrderId":"123A124321",
  "TransactionId":"BA12366351512",
  "Amount":1023.43,
  "CustomerId":" f123456-a123-b123-c123-d12345678901",
  "CustomerToken": null,
  "CustomerReceiptToken": null,
  "LastestUpdate": "06-02-2017 09:30:39"
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
    <td>Unique ID that identifies the PoS that has initiated current payment request.</td>
  </tr>
  <tr>
    <td>PoSUnitId</td>
    <td>String</td>
    <td>White Box/Terminal Id</td>
  </tr>
  <tr>
    <td>PaymentStatus</td>
    <td>Integer</td>
    <td>See GetPaymentStatus</td>
  </tr>
  <tr>
    <td>OrderId</td>
    <td>String</td>
    <td>The OrderId assigned to current payment.</td>
  </tr>
  <tr>
    <td>TransactionId</td>
    <td>String</td>
    <td>Unique ID that identifies the payment (transaction ID). ID is generated by MobilePay and is shown on the receipt inside the MobilePay app.</td>
  </tr>
  <tr>
    <td>Amount</td>
    <td>Number</td>
    <td>The amount for the payment. Note: Decimal point is “.”</td>
  </tr>
  <tr>
    <td>CustomerId</td>
    <td>String</td>
    <td>Unique ID of the customer. The ID is generated by MobilePay.</td>
  </tr>
  <tr>
    <td>CustomerToken</td>
    <td>String</td>
    <td>Contains customer token if customer has checked-In with a merchant token ID related to this merchant’s loyalty program</td>
  </tr>
  <tr>
    <td>CustomerReceiptToken</td>
    <td>String</td>
    <td>Used for customer receipt token (In DK: Service agreement with Storebox implies that Storebox user Id is provided) Max 32 char.</td>
  </tr>
  <tr>
    <td>LatestUpdate</td>
    <td>DateTime</td>
    <td>Datetime where the Payment were last updated / changed / accessed</td>
  </tr>
</table>

HTTP 400 – See GetCurrentPayment error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### GetCurrentPayment Error Codes
<table>
  <tr>
    <th>Error code</th>
    <th>Error text</th>
  </tr>
  <tr>
    <td>10</td>
    <td>Missing or invalid parameters.</td>
  </tr>
</table>

## <a name="GetPosList"></a>GetPosList
### Purpose:
Get a list of the current PoSId’s registered to the Merchant/Location.

### Request:
```json
{
"MerchantId":"POSDK99999",
"LocationId":"88888"
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
    <td>Merchant ID</td>
  </tr>
  <tr>
    <td>LocationId</td>
    <td>String</td>
    <td>required</td>
    <td>Location ID</td>
  </tr>
</table>

### Response
HTTP 200 – Ok
```json
{
"Poses": [
    {
      "PosId": "CashRegister1",
      "Name": "Cash Register 1",
      "PosUnitId": "100000000000001",
      "Payment": null
    },
    {
      "PosId": "ea19a7f5-9f0e-4059-89a4-7cd7a5933e7d",
      "Name": "Cash Register 2",
      "PosUnitId": null,
      "Payment": null
    },
    {
      "PosId": "ae77de8f-2d71-4c3e-9a91-ed12452b6690",
      "Name": "Cash register 3",
      "PosUnitId": "100000000000002",
      "Payment": {
        "Status": 40,
        "OrderId": "123131",
        "TransactionId": "BA12366351512",
        "Amount": 0.01,
        "CustomerId":" f123456-a123-b123-c123-d12345678901",
        "CustomerToken": null,
        "CustomerReceiptToken": null,
        "LatestUpdate": "13-02-2017 14:07:55"
      }   
  ]
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
    <td>Unique ID that identifies the PoS that has initiated current payment request.</td>
  </tr>
<tr>
    <td>Name</td>
    <td>String</td>
    <td>POS Name</td>
  </tr>
  <tr>
    <td>PoSUnitId</td>
    <td>String</td>
    <td>White Box/Terminal Id</td>
  </tr>
  <tr>
    <td>Payment</td>
    <td>Integer</td>
    <td>See GetPaymentStatus</td>
  </tr>
</table>

HTTP 400 – See GetPosList error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### GetPosList Error Codes
<table>
  <tr>
    <th>Error code</th>
    <th>Error text</th>
  </tr>
  <tr>
    <td>10</td>
    <td>Missing or invalid parameters.</td>
  </tr>
</table>

## <a name="GetLocationList"></a>GetLocationList
### Purpose:
Get a list of locations registered to a MerchantId. 
Can only be called from same merchant using the correct apikey.

### Request:
```json
{
"MerchantId":"POSDK99999"
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
    <td>Merchant ID</td>
  </tr>
</table>

### Response
HTTP 200 – Ok
```json
{
  "Location": [
    {
      "LocationId": "00001",
      "Name": "Location Road 1",
      "NotifyUrl": null,
      "NumberOfPoS": 3
    },
    {
      "LocationId": "00002",
      "Name": "Location Road 2",
      "NotifyUrl": null,
      "NumberOfPoS": 0
    }
  ]
}
```
<table>
  <tr>
    <th>Parameter</th>
    <th>Type</th>
    <th>Description</th>	
  </tr>
  <tr>
    <td>LocationId</td>
    <td>String</td>
    <td>LocationID</td>
  </tr>
<tr>
    <td>Name</td>
    <td>String</td>
    <td>Location Name</td>
  </tr>
  <tr>
    <td>NotifyUrl</td>
    <td>String</td>
    <td>Link to merchant notification server</td>
  </tr>
  <tr>
    <td>NumberOfPos</td>
    <td>Integer</td>
    <td>Number of PointOfSale terminals registered to the Location. Use GetPosList to get the detailed POS information.</td>
  </tr>
</table>

HTTP 400 – See GetLocationList error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### GetLocationList Error Codes
<table>
  <tr>
    <th>Error code</th>
    <th>Error text</th>
  </tr>
  <tr>
    <td>10</td>
    <td>Missing or invalid parameters.</td>
  </tr>
</table>

