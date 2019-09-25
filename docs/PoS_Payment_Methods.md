# PoS Payment Methods: PoS -> MobilePay Backend

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->
- [PaymentStart](#PaymentStart)
- [GetPaymentStatus](#GetPaymentStatus)
- [PaymentCancel](#PaymentCancel)
- [PaymentRefund](#PaymentRefund)
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
|OrderId           |String      | required | The OrderId is a unique id that identifies the payment. The OrderId is issued by the merchant and is attached to the payment inside MobilePay system. <br>The order ID must be unique for the merchant/location combination. This means that there should be only one completed payment with any given order ID for the same merchant and location (store) during the lifetime of the merchant/location. <br>CASE SENTITIVE |
|Amount            |String      | required | The amount for the payment. Always with 2 decimals and no thousand separators. <br>Note: Decimal point is "." |
|BulkRef           |String      | required | An option for grouping the payments – a text or ID. The field has a maximum length of 18 characters. <br>If the field remains empty and the merchant does not have a Bulkpost agreement, the merchant will receive all mobile payments from any connected shops as individual postings in the reconciliation file. <br>If the field remains empty and the merchant does have a Bulkpost agreement, the merchant will receive all mobile payments bulked with a default bulkref of the MP Enterprise Serialnumber value in the reconciliation file. <br>It must be a merchant decision whether they want all individual postings or a bulk posting per store or the entire group as one posting.<br>The field is mandatory in the request even though it might be an empty string.|
|Action            |String      | required | Action values:<br>"Start": Initiate a payment.<br>"Update": Update a current payment after recalculation. |
|CustomerTokenCalc |String      | required | The field indicate if the loyalty payment flow must be initiated if applicable or not. <br>“0”: Loyalty flow will be initiated if customer is a member of the merchant’s loyalty program. <br>“1”: Ignore loyalty flow |
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
|ReCalc       |Integer      |0 – normal usage.<br>1 – recalculation must be executed and payment updated.|
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

## GetPaymentStatus
### Purpose:
Get a payment status for current PoS ID.
Used for polling a payment status. Polling has to be done every 1 second until the PaymentStatus is 100 ('Done') or if it rejects the payment request (PaymentStatus 40 ('Cancel') or 50 ('Error')).

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
|OrderId         |String      | required | Order ID related to current payment status request. |

### Response
HTTP 200 – Ok
```json
{
"PoSId":"a123456-b123-c123-d123-e12345678901",
"PaymentStatus":20,
"OrderId":"123A124321",
"TransactionId":"BA12366351512",
"Amount":1023.43,
"CustomerId":"abc123",
"CustomerToken":"",
"CustomerReceiptToken":"",
"PaymentSignature":"null"
}
```
|Parameter    |Type        |Description                                                      |
|---------------------|------------|-----------------------------------------------------------------|
|PoSId                |String      |Unique ID that identifies the PoS that has initiated current payment request.|
|PaymentStatus        |Integer     |See status values below <br>- **10 ('Idle')** No payment request in the queue<br>- **20 ('Issued')** - Payment request is sent to customer.<br>- **30 ('AwaitCheckIn')** - Await customer check-in.<br>- **40 ('Cancel')** - Customer has cancelled/rejected payment request.<br>- **50 ('Error')** - MobilePay is not able to handle the payment – the PoS should cancel the MobilePay payment request.<br>- **60 (‘AwaitTokenRecalc’)** – Await for PoS system to update payment after recalculation.<br>- **80 ('PaymentAccepted')** - The payment request is accepted by the customer – await payment confirmation from the payment transaction system.<br>- **100 ('Done')** - "Payment Confirmed" and TransactionId, PaymentSignature, Amount, CustomerId (optional) will contain a value. |
|OrderId              |String      |The OrderId assigned to current payment. |
|TransactionId        |String      |Unique ID that identifies the payment (transaction ID). ID is generated by Danske Bank and is shown on the receipt inside the MobilePay app. |
|Amount               |Number      |The amount for the payment. <br>Note: Decimal point is “.” |
|CustomerId           |String      |Unique ID of the customer. The ID is generated by MobilePay. |
|CustomerToken        |String      |Contains customer token if customer has checked-In with a merchant token ID related to this merchant’s loyalty program. |
|CustomerReceiptToken |String      |Used for customer receipt token (In DK: Service agreement with Storebox implies that Storebox user Id is provided) <br>Max 32 char. |
|PaymentSignature     |null        |null (deprecated). |

HTTP 400 – See GetPaymentStatus error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### GetPaymentStatus Error Codes
|Error code   |Error text       |
|-------------|-----------------|
|10           |Missing or invalid parameters. |
|30           |The key "MerchantId, LocationId and PoSId" does not exist. |

## PaymentCancel
### Purpose:
Cancel payment request for current PoS ID.
Cancel is principal possible as long as earlier request for payment hasn't been finalized (status 100).
A PaymentCancel will delete current payment entity active or not unless earlier finished payment ended in status Done (status code 100) which will remains until a new payment starts.

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
|PoSId        |String      | required | Current Point of Sale ID (cash register/terminal). |

### Response
HTTP 200 – Ok
```json
{
}
```

HTTP 400 – See PaymentCancel error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### PaymentCancel Error Codes
|Error code   |Error text       |
|-------------|-----------------|
|10           |Missing or invalid parameters. |
|30           |The key "MerchantId, LocationId and PoSId" does not exist. |
|50           |Action not possible. |

## PaymentRefund
### Purpose:
Refund part of or the entire amount of the payment.
A payment refund can be made days/weeks after the original payment has been made.
"PaymentRefund" is a stand-alone method and must be called directly. The response code from this call will indicate success or failure. 
The "OrderId" value must be entered from either PoS or merchant backend.

### Request:
```json
{
"MerchantId":"POSDK99999",
"LocationId":"88888",
"PoSId":"a123456-b123-c123-d123-e12345678901",
 "OrderId":"123A124321",
"Amount":"100.00",
"BulkRef":""
}
```
|Parameter    |Type        |Required  |Description                                                      |
|-------------|------------|----------|-----------------------------------------------------------------|
|MerchantId   |String      | required | Id of the merchant. |
|LocationId   |String      | required | Location ID related to current merchant ID and PoS ID.|
|PoSId        |String      | required | Current Point of Sale ID (cash register / terminal). |
|OrderId      |String      | required | Order ID that identifies the payment to refund. |
|Amount       |String      | required | Amount to refund. <br>If  0.00 or blank, the whole transaction will be refunded.<br>Note: Decimal point is “.”<br>Examples: "Amount": "0.00"  -  "Amount": "100.00" |
|BulkRef      |String      | required | Note: this parameter is required but currently only a placeholder for future use. For now, passed values will not be used, but we recommend preparing your solution for this feature. We expect to implement this in an upcoming update of the API.<br> An option for grouping the refunds – a text or ID. The field has a maximum length of 18 characters.<br> If the field remains empty and the merchant does not have a Bulkpost agreement, the merchant will receive all mobile refunds from any connected shops as individual postings in the reconciliation file.<br> If the field remains empty and the merchant does have a Bulkpost agreement, the merchant will receive all mobile refunds bulked with a default BulkRef of the MP Enterprise Serialnumber value in the reconciliation file.<br> It must be a merchant decision whether they want all individual postings or an bulk posting per store or the entire group as one posting.|

### Response
HTTP 200 – Ok
```json
{
"Remainder":10.75,
"TransactionId":"1122334455A"
}
```

HTTP 400 – See PaymentRefund error codes
```json
{
"StatusCode":10,
"StatusText":"Missing or invalid parameters"
}
```
### PaymentRefund Error Codes
|Error code   |Error text       |
|-------------|-----------------|
|1            |Order cannot be found (this order has not been paid by the customer, or it has been more than 3 months since the order has been paid) |
|2            |Amount is larger than remaining refundable amount on transaction |
|3            |Transaction is already refunded |
|10           |Missing or invalid parameters |
|30           |The key "MerchantId, LocationId and PoSId" does not exist |
|99           |Technical error (refund cannot be performed) |


[![](assets/images/6.5.1 PoS Payment E2E-sequence.jpg)](assets/images/6.5.1 PoS Payment E2E-sequence.jpg)
