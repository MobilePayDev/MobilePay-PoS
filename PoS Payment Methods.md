# PoS Payment Methods: PoS -> MobilePay Backend

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
- [Location Support](#ReadPoSAssignPoSUnitId)
    - [GetLocationList](#GetLocationList)
<!-- /TOC -->


## PaymentStart
### Purpose:
This method is called when the Point of Sale (cash register / terminal) wishes to start a payment (The customer does not have to be checked-in in advance). 
PaymentStart is only possible if no active MobilePay payment entity exists for current PoS – unless it is an update (Action=Update) for a payment entity in state "AwaitTokenRecalc".
A PaymentStart will delete earlier finished payment entities – i.e., payment entities in status 'Done', 'Cancel' or 'Error'.

### Request:
```json
{
"MerchantId":"POSDK99999",
"LocationId":"88888",
"PoSId":"a123456-b123-c123-d123-e12345678901",
"OrderId":"123A124321",
"Amount":"1023.43",
"BulkRef":"MP Bulk Reference",
"Action":"Start",
"CustomerTokenCalc":0,
"HMAC":"NTsitSPXe9f5o8Nclza8ZQIx9mdom6GdcyoCQvGs9o0="
}
```

|Parameter    |Type        |Required  |Description                                                      |
|------------------|------------|----------|-----------------------------------------------------------------|
|MerchantId        |String      | required | Merchant ID related to current PoS ID. |
|LocationId        |String      | required | Location ID related to current merchant ID and PoS ID. |
|PoSId             |String      | required | Current Point of Sale ID (cash register/terminal). |
|OrderId           |String      | required | The OrderId is a unique id that identifies the payment. The OrderId is issued by the merchant and is attached to the payment inside MobilePay system. The order ID must be unique for the merchant/location combination. This means that there should be only one completed payment with any given order ID for the same merchant and location (store) during the lifetime of the merchant/location. CASE SENTITIVE |
|Amount            |String      | required | The amount for the payment. Always with 2 decimals and no thousand separators.Note: Decimal point is "." |
|BulkRef           |String      | required | An option for grouping the payments – a text or ID. The field has a maximum length of 18 characters. If the field remains empty and the merchant does not have a Bulkpost agreement, the merchant will receive all mobile payments from any connected shops as individual postings in the reconciliation file. If the field remains empty and the merchant does have a Bulkpost agreement, the merchant will receive all mobile payments bulked with a default bulkref of the MP Enterprise Serialnumber value in the reconciliation file. It must be a merchant decision whether they want all individual postings or a bulk posting per store or the entire group as one posting.
The field is mandatory in the request even though it might be an empty string. |
|Action            |String      | required | Action values:
"Start": Initiate a payment.
"Update": Update a current payment after recalculation. |
|CustomerTokenCalc |String      | required | The field indicate if the loyalty payment flow must be initiated if applicable or not. “0”: Loyalty flow will be initiated if customer is a member of the merchant’s loyalty program. “1”: Ignore loyalty flow |
|HMAC              |String      | required | The HMAC is calculated based on the other parameters. The key for the HMAC is a MerchantKey issued by MobilePay. <br><br>Base64 (with padding) encoded HMAC SHA256:<br>HMAC = Base64(hmacSha256(<br>ISO88591Bytes(“{MerchantId+LocationId}#{PoSId}#{OrderId}#{Amount}#{BulkRef}#”), ISO88591Bytes(MerchantKey))) <br>Note that fields must be trimmed, see example [here](https://developer.mobilepay.dk/node/2546) |

### Response
HTTP 200 – Ok
```json
{
"ReCalc":0,
"CustomerToken":null,
"CustomerReceiptToken":null
}
```
|Parameter    |Type        |Description                                                      |
|-------------|------------|-----------------------------------------------------------------|
|ReCalc       |Integer      |0 – normal usage.
1 – recalculation must be executed and payment updated.|
|CustomerToken       |String      |null if no recalculation is needed yet. |
|CustomerReceiptToken       |String      |Used for customer receipt token (In DK: Service agreement with Storebox implies that Storebox user Id is provided). May be null. Max 32 characters. |

HTTP 400 – See PaymentStart error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### PaymentStart Error Codes
|Error code   |Error text       |
|-------------|-----------------|
|10           |Missing or invalid parameters. | 
|30           |The key “MerchantId, LocationId and PoSId” does not exist | 
|40           |Payment already exists and has been paid | 
|50           |Payment already in progress | 