---
layout: manual
title: Integration via SDK iOS
description: Gateway Braspag Technical Integration
search: true
translated: true
categories: manualp
sort_order: 4
tags:
  - 4. EMV 3DS (3DS 2.0)
---

# What is 3DS 2.0?

For more details about 3DS 2.0, please visit: [https://braspag.github.io//en/manualp/emv3ds](https://braspag.github.io//en/manualp/emv3ds).

# Step 1 - Access Token Request

The solution comprises the API access token request step and authentication request from the APP.

|Environment|Endpoint|Authorization|
|---|---|---|
|**SANDBOX**|https://authsandbox.braspag.com.br/oauth2/token|**Basic _(Authorization)_**<br><br>The Authorization value must be obtained by concatenating the value of the "ClientID", colon (":"), and "ClientSecret"<br><br>E.g.: b4c14ad4-5184-4ca0-8d1a-d3a7276cead9:qYmZNOSo/5Tcjq7Nl2wTfw8wuC6Z8gqFAzc/utxYjfs=<br><br>and then encode the result in base 64. <br>This will generate an alphanumeric access code that will be used in the access token. For testing purposes, use the following data: <br>ClientID: **dba3a8db-fa54-40e0-8bab-7bfb9b6f2e2e **<br>ClientSecret:** D/ilRsfoqHlSUChwAMnlyKdDNd7FMsM7cU/vo02REag =**|
|---|---|
|**PRODUCTION**|https://auth.braspag.com.br/oauth2/token|Request the "ClientID" and "ClientSecret" data to the Support team after completing sandbox development.|

### Request

<aside class="request"><span class="method post">POST</span> <span class="endpoint">/v2/auth/token</span></aside>

```json
{
     "EstablishmentCode":"1006993069",
     "MerchantName": "Example Store Ltda",
     "MCC": "5912"
}
```

|**Parameter**|**Description**|**Type/Size**|
|---|---|---|
|EstablishmentCode|Establishment Code in Cielo E-Commerce 3.0 or Getnet, or PV at Rede|Numeric [10 positions]|
|MerchantName|Name of registered establishment in the acquirer|Alphanumeric [up to 25 positions]|
|MCC|Merchant Category Code|Numeric [4 positions]|

### Response

```json
{
      "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJjbGllbnRfbmFtZSI6IlFBXzNEU19BdXRoZW50aWNhdG9yIiwiY2xpZW50X2lkIjoiZGJhM2E4ZGItZmE1NC00MGUwLThiYWItN2JmYjliNmYyZTJlIiwic2NvcGVzIjoie1wiU2NvcGVcIjpcIjNEU0F1dGhlbnRpY2F0b3JcIixcIkNsYWltc1wiOlt7XCJOYW1lXCI6XCJNZXJjaGFudE5hbWVcIixcIlZhbHVlc1wiOFwiVmFsdWVzXCI6W1wiNTU1NVwiXX0se1wiTmFtZVwiOlwiUmVmZXJlbmNlSWRcIixcIlZhbHVlc1wiOltcImY3MjE1YmQ3LWM0OTQtNGQ5Yi1NzEyfQ.daMqXko3dZOV0TzNFQ2vSsVSKqOsrwuswg7RB82ecAASSSSSSSSSSSSFFFFFFFFFFFFFGGGGGGGGGGGGGGGGGGGGGGGG",
      "token_type": "bearer",
      "expires_in": "2018-07-23T11:29:32"
}
```

|**Parameter**|**Description**|**Type/Size**|
|---|---|---|
|access\_token|Token required to perform authentication|Alphanumeric [variable size]|
|token\_type|Fixed "bearer"|Alphanumeric|
|expires\_in|Time in minutes to expire token|Numeric|

# Step 2 - Using the SDK

To use the SDK it is necessary to import the Braspag3dsSdk module:

```swift
import Braspag3dsSdk
```

Then you must pass to the client side (APP) the *access_token* generated in the previous step:

```swift
let braspag3ds = Braspag3ds(accessToken: "[Access_Token gerado no passo 1]", environment: Environment.production)
```

Then it is necessary to use the `authenticate` method, informing the buyer data and the *callback* that will receive the response:

```swift
braspag3ds.authenticate(orderData: OrderData(...),
                        cardData: CardData(...),
                        authOptions: OptionsData(...),
                        billToData: BillToData(...),
                        shipToData: ShipToData(...),
                        cart: [CartItemData(...),
                        deviceData: DeviceData(...),
                        userData: UserData(...),
                        airlineData: AirlineData(...),
                        mdd: MddData(...),
                        recurringData: RecurringData(...),
                        deviceIpAddress: "0.0.0.0"){ (status, authenticationResponse) in

                        switch(status){
                        case .success:
                          ...
                        case .unenrolled:
                          ...
                        case failure:
                          ...
                        case error:
                          ...
                        case unsupportedBrand:
                          ...
                        }
  }
```

## Minimum SDK Version

SDK uses minimum version 9.0

## *Authenticate* method input parameters

|**Field**|**Type**|**Description**|**Required**|
|---|---|--|--|
|orderData|OrderData|Payment Request Data|Yes|
|cardData|CardData|Card Data|Yes|
|authOptions|OptionsData?|Additional Settings to the 3DS Process|No|
|billToData|BillToData?|Cardholder Billing Data|No|
|shipToData|ShipToData?|Delivery Data|No|
|cart|[CartItemData]?|Array with Cart Items|No|
|deviceData|[DeviceData]?|Additional Settings to the 3DS Process|No|
|userData|UserData?|User data in your store|No|
|airlineData|AirlineData?|Travel Ticket Data|No|
|mdd|MddData?|Extra data sent by the shopkeeper|No|
|recurringData|RecurringData?|Recurrence data|No|
|deviceIpAddress|String?|Device IP Address|No|

## Description of Callback Status

|**Status**|**Description**|
|---|---|
|success|Returns when the card is eligible and the authentication process has been successfully completed. In this case, the CAVV, XID, and ECI variables will be returned. This data must be sent in the request at the time of authorization. In this scenario, if the transaction is authorized, the liability shift is transferred to the issuer.|
|unenrolled|Returns when the card is not eligible, i.e. the holder and/or issuer does not participate in the authentication program. In this case, only the ECI variable will be returned. If there is a decision to proceed with the authorization anyway, the ECI must be sent at the time of the request. In this scenario, if the transaction is authorized, the liability shift remains with the establishment.|
|failure|Returns when the card is eligible but has not had the authentication process failed for some reason. In this case, only the ECI variable will be returned. If there is a decision to proceed with the authorization anyway, the ECI must be sent at the time of the request. In this scenario, if the transaction is authorized, the liability shift remains with the establishment.|
|error|Returns when the authentication process received a systemic error. In this scenario, if the transaction is authorized, the liability shift remains with the establishment.|
|unsupportedBrand|Returns when card banner is not supported by 3DS 2.0|

## Description of *AuthenticationResponse* fields

|**Exit**|**Description**|**Type/Size**|
|---|---|---|
|cavv|Data representing authentication signature|Text|
|xid|ID representing the authentication request|Text|
|eci|E-commerce indicator code, which represents the result of authentication|Numeric [up to 2 positions]|
|version|3DS Version Applied|Numeric [1 position] 1-3DS 1. 02-3DS 2.0|
|referenceID|ID representing the authentication request|GUID [36 positions]|
|returnCode|Authentication Request Return Code|Alphanumeric [up to 5 positions]|
|returnMessage|Authentication Request Return Message|Alphanumeric [variable]|

# Detailing Requisition Objects

To make it easier to use only what the merchant needs to send, the request is separated into several objects with well-defined data context as the authenticate input parameters table shows. Below we will detail each of the objects used.

<aside class="warning">The greater the number of parameterized fields, the greater the chance of having transparent authentication, as the issuer will have greater subsidy for risk analysis</aside>

## OptionsData

|**Property**|**Description**|**Type/Size**|**Required**|
|---|---|---|---|
|notifyonly|Boolean indicating whether the card transaction will be submitted in "notification only" mode. In this mode, the authentication process will not be triggered, however, the data will be flagged. **VALID ONLY FOR MASTERCARD CARDS**|Boolean: <br>true - notification only mode; <br>false - mode with authentication|No|
|suppresschallenge|Boolean that indicate if ignore or not the challenge, if requested. If a transaction authorized after ignoring the challenge, liability remains with the establishment.|Boolean: <br>true - ignore challenges if any; <br>false - present challenge if any|No|

## OrderData

|**Property**|**Description**|**Type/Size**|**Required**|
|---|---|---|---|
|orderNumber|Order code at establishment|Alphanumeric [up to 50 positions]|Yes|
|currencyCode|Currency Code|Fixed "BRL"|Yes|
|totalAmount|Total transaction amount, sent in cents|Numeric [up to 15 positions]|Yes
|paymentMethod|Type of card to be authenticated. In the case of a multiple card, you must specify either Credit or Debit|*PaymentMethod* <br><br>credit – Credit Card<br>debit – Debit Card|Yes|
|installments|Number of Transaction Installments|Numeric [up to 2 positions]|Yes
|recurrence|Indicates if it is an order that generates future recurrences<br>|<br>Booleanotruefalse|No|
| productCode | Product Code | *ProductCode*<br><br>goodsPurchase: Purchase of goods<br>checkAcceptance: Check acceptance<br>financeAccount: Account financing<br>quasiMoneyTransaction: Quasi-money transaction<br>recharge: recharge<br>| Yes |
|countLast24Hours|Quantity of orders placed by this buyer in the last 24h|Numeric [up to 3 positions]|No|
|countLast6Months|Number of orders placed by this shopper in the last 6 months|Numeric [up to 4 positions]|No|
|countLast1Year|Number of orders placed by this shopper in the last year|Numeric [up to 3 positions]|No|
|cardAttemptsLast24Hours|Number of same card transactions in last 24h|Numeric [up to 3 positions]|No|
|marketingOptIn|Indicates if the shopper has accepted to receive marketing offers|Boolean<br>true – yes<br>false – no|No|
|marketingSource|Identifies the origin of the marketing campaign|Alphanumeric [up to 40 positions]|No|
|transactionMode|Identifies the channel that originated the transaction|M: MOTO<br>R: Varejo<br>S: E-Commerce<br>P: Mobile<br>T: Tablet|No|
|merchantUrl|Establishment Website Address|Alphanumeric [100] Example: http://www.example.com|Yes|

## CardData

|**Property**|**Description**|**Type/Size**|**Required**|
|---|---|---|---|
|number|Card Number|Numeric [up to 19 positions]|Yes|
|expirationMonth|Month of Card Expiration|Numeric [2 positions]|Yes|
|cardexpirationYear|Year of card expiration|Numeric [4 positions]|Yes|
|cardAlias|Card Alias|Alphanumeric [up to 128 positions]|No|
|defaultCard| Indicates if it is a standard customer card in the store|Boolean<br>true - yes<br>false - no|No|

## BillToData

|**Property**|**Description**|**Type/Size**|**Required**|
|---|---|---|---|
|customerId|Identifies the shopper CPF/CNPJ|Numeric [11 to 14 positions]<br>99999999999999|No|
|contactName|Billing Address Contact Name|Alphanumeric [up to 120]|Yes|
|phoneNumber|Billing Address Contact Phone|Numeric [up to 15 positions], in the format: 5511999999999|Yes|
|email|Billing Address Contact Email|Alphanumeric [up to 255], in the format [name@example.com] (mailto: name@example.com)|Yes|
|street1|Address and Billing Address Number|Alphanumeric [up to 60]|Yes|
|street2|Billing Address Supplement and Neighborhood|Alphanumeric [up to 60]|Yes|
|city|Billing Address City|Alphanumeric [up to 50]|Yes|
|state|Billing Address State Acronym|Text [2 positions]|Yes|
|zipCode|Billing Address Zip Code|Alphanumeric [up to 8 positions], in the format: 99999999|Yes|
|country|Billing Address Country|Text [2 positions] Ex. BR|Yes|

## ShipToData

|**Property**|**Description**|**Type/Size**|**Required**|
|---|---|---|---|
|sameAsBillTo|Indicates whether to use the same address provided for billing address|Boolean<br>true<br>false|No|
|addressee|Contact Name of Shipping Address|Alphanumeric [up to 60]|No|
|phoneNumber|Billing Address Contact Phone|Numeric [up to 15 positions], in the format: 5511999999999|Yes|
|email|Billing Address Contact Email|Alphanumeric [up to 255], in the format [name@example.com] (mailto: name@example.com)|Yes|
|street1|Address and Delivery Address Number|Alphanumeric [up to 60]|No|
|street2|Delivery Address Complement and Neighborhood|Alphanumeric [up to 60]|No|
|city|City of delivery address|Alphanumeric [up to 50]|No|
|state|Shipping Address State Acronym|Text [2 positions]|No|
|zipCode|Shipping Address ZIP Code|Alphanumeric [up to 8 positions], in the format: 99999999|No|
|country|Billing Address Country|Text [2 positions] Ex. BR|No|
|shippingMethod|Shipping Method Type|*ShippingMethodEnum*<br><br>LOWCOST: Economical Shipping<br>SAMEDAY: Same shipping<br>ONEDAY: Next day shipping<br>TWODAY: Two days shipping <br>THREEDAY: Three days shipping<br>PICKUP: Delivery at the store<br>OTHER: There is no shippment|No|
|firstUsageDate|Indicates the date when the shipping address was first used|Text<br>YYYY-MM-DD - Date Created|No|

## CartItemData

|**Property**|**Description**|**Type/Size**|**Required**|
|---|---|---|---|
|description|Item Description|Alphanumeric [up to 255 positions]|No|
|name|Item Name|Alphanumeric [up to 255 positions]|Yes|
|sku|Item SKU|Alphanumeric [up to 255 positions]|No|
|quantity|Item Quantity in Cart|Numeric [up to 10 positions]|No|
|unitprice|Cart item unit value in cents|Numeric [up to 10 positions]|No|

## DeviceData

|**Property**|**Description**|**Type/Size**|**Required**|
|---|---|---|---|
|fingerprint|Id returned by Device Finger Print|Alphanumeric [without limitation]|No|
|provider|Device Finger Print Provider Name|Alphanumeric [up to 32 positions] cardinal<br>inauth<br>threatmetrix|No|

## UserData

|**Properties**|**Description**|**Type/Size**|**Required**|
|---|---|---|---|
|guest|Indicates if the shopper is a shopper without login (guest)|Boolean<br>true – yes<br>false – no|No|
|createdDate|Indicates the date when the buyer account was created|Text<br>YYYY-MM-DD - Date Created|No|
|changedDate|Indicates the date when the last buyer account change|Text<br>YYYY-MM-DD - date of last change|No|
|passwordChangedDate|Indicates the date when the shopper account password was changed|Text<br>YYYY-MM-DD - Last Password Change Date|No|
|authenticationMethod|Authentication Method of the shopper in the store|*AuthenticationMethod* <br> NOAUTHENTICATION - Does not happen authentication<br>OWNSTORELOGIN - Login at the own store<br>FEDERATEDLOGIN - Login at federated store <br>FIDOAUTHENTICATOR - Login with FIDO authenticator|No|
|authenticationProtocol|This represents the store login protocol|Alphanumeric [up to 2048 positions]|No|
|authenticationTimestamp|The date and time the buyer logged in the store|Text [19 positions] _YYYY-MM-ddTHH: mm: SS_|No|
|newCustomer|Identifies if is a new buyer in the store|Boolean<br>true – yes<br>false – no|No|

## AirlineData

|**Properties**|**Description**|**Type/Size**|**Required**|
|---|---|---|---|
|numberOfPassengers|Number of Passengers|Numeric [3 positions]|No|
|billToPassportCountry|Passport country code (ISO Standard Country Codes)|Text [2 positions]|No|
|billtoPassportNumber|Passport Number|Alphanumeric [40 positions]|No|
|travelLeg|Excerpt from the trip|TravelLeg|No|
|passenger|Passenger Data|Passenger|No|

## TravelLeg

|**Properties**|**Description**|**Type/Size**|**Required**|
|---|---|---|---|
|carrier|IATA code for the stretch|Alphanumeric [2 positions]|No|
|departureDate|Date of Departure|Text<br>YYYY-MM-DD|No|
|origin|IATA code of origin airport|Alphanumeric [5 positions]|No|
|destination|IATA code of destination airport|Alphanumeric [5 positions]|No|

## Passenger

|**Properties**|**Description**|**Type/Size**|**Required**|
|---|---|---|---|
|name|Passenger Name|Alphanumeric [up to 60 positions]|No|
|ticketPrice|The value of the ticket in cents|Numeric [up to 15 positions],<br>example: $ 125.54 = 12554|No|

## MDD

|**Properties**|**Description**|**Type/Size**|**Required**|
|---|---|---|---|
|mdd1|Extra data defined by the merchant|Alphanumeric [up to 255 positions]|No|
|mdd2|Extra data defined by the merchant|Alphanumeric [up to 255 positions]|No|
|mdd3|Extra data defined by the merchant|Alphanumeric [up to 255 positions]|No|
|mdd4|Extra data defined by the merchant|Alphanumeric [up to 255 positions]|No|
|mdd5|Extra data defined by the merchant|Alphanumeric [up to 255 positions]|No|

## RecurringData

|**Properties**|**Description**|**Type/Size**|**Required**|
|---|---|---|---|
|endDate|Identifies recurrence end date|Text (YYYY-MM-DD)|No|
|frequency|Indicates frequency of recurrence|*RecurringFrequencyEnum*|No| <br><br>MonthlyBimonthly<br>Quarterly<br>Triannual<br>SemiAnnual<br>Yearly|No|
|originalPurchaseDate|Identifies the date of the 1st transaction that originated the recurrence|Text (YYYY-MM-DD)|No|

## Other parameters

|**Properties**|**Description**|**Type/Size**|**Required**|
|---|---|---|---|
|ipAddress|Shopper's Machine IP Address|Alphanumeric [up to 45]|No|

# Test Cards

Use the **test** cards below to simulate various scenarios in the **SANDBOX** environment

## Test cards with challenge

|**CARD**|**BRAND**|**RESULT**|**DESCRIPTION**|  
|---|---|---|---|     
|4000000000001091<br>5200000000001096<br>6505050000001091|VISA<br>MASTER<br>ELO|SUCCESS|Authentication with challenge and cardholder successfully authenticated|  
|4000000000001109<br>5200000000001104<br>6505050000001109|VISA<br>MASTER<br>ELO|FAILURE|Authentication with challenge and cardholder authentication terminated with failure|  
|4000000000001117<br>5200000000001112<br>6505050000001117|VISA<br>MASTER<br>ELO|UNENROLLED|Authentication with challenge currently not available|  
|4000000000001125<br>5200000000001120<br>6505050000001125|VISA<br>MASTER<br>ELO|UNENROLLED|System error during authentication|  

## Test cards without challenge

|**CARD**|**BRAND**|**RESULT**|**DESCRIPTION**|  
|---|---|---|---|     
|4000000000001000<br>5200000000001005<br>6505050000001000|VISA<br>MASTER<br>ELO|SUCCESS|Authentication without challenge and cardholder successfully authenticated|  
|4000000000001018<br>5200000000001013<br>6505050000001018|VISA<br>MASTER<br>ELO|FAILURE|Authentication without challenge and cardholder authentication terminated with failure| 

## Authorization with Authentication

After authentication is completed, it undergoes the authorization process by submitting the authentication data in the external authentication  (node **ExternalAuthentication**).
See more details at: [https://braspag.github.io//en/manualp/authorization-with-authentication](https://braspag.github.io//en/manualp/authorization-with-authentication)

# Last updates

To see the manual latest updates, [click here](https://github.com/Braspag/braspag.github.io/commits/docs/_i18n/en/_posts/emv3ds/2019-09-13-integration-sdk-ios.md)
