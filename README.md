# BitBUY.ca Merchant API

https://merchant.bitbuy.ca/

[![Build status](https://ci.appveyor.com/api/projects/status/95vn1krk9hajaax8/branch/master?svg=true)](https://ci.appveyor.com/project/AdemarGonzalez/merchantapi/branch/master)

# REST API

All requests to BITBUY's API require authentication. You can create a set of terminal credentials to use with the API in the Merchant control panel under menu option Terminals.

## API Endpoint

  `https://api.bitbuy.ca/v1`

All API requests should use the application/json content type, and must be encrypted with SSL (https).

## Create a Payment session

* **URL**

  /payments

* **Method:**

  `POST`
  
* **Authentication:**

  *Required*

* **Data Params**

  { "amount" : 1.23,  "currency" : "CAD" }

  ### Required fields.

  **amoun**t : amount of currency to collect from the buyer.  
  **currency** : if currency different than Bitcoin then the amount will be converted to BTC using market rates and such amount collected from the buyer.  

  ### Optional fields

  **data** : optional data field value will be sent back on the callback fi there is any.  
  **ttl** : number of minutes before the payment session expires.  
  **priority** : possible values 0, 1 and 2.  
  **urlCallback** : if exits BitBUY will make an HTTP POST to the url whenever the payment session status change.  
  **urlSuccess** : used in the invoice.  
  **urlFailure** : used in the invoice.  

* **Success Response:**

  * **Code:** 200 <br />
    **Content:** 
   ```json
   {
      "Id":"ehePI2Tuuw7jeg9qeVTe3gYNebKzBRcu",
      "Total":1.23000000,
      "Currency":"CAD",
      "BtcRequired":0.00167342,
      "Data":"",
      "CreatedOn":"2016-08-19T20:19:05",
      "ExpiresOn":"2016-08-19T20:26:05",
      "Status":"WAITING",
      "LastUpdate":"2016-08-19T20:19:05",
      "Url":"https://api.bitbuy.ca/v1/invoice?id=ehePI2Tuuw7jeg9qeVTe3gYNebKzBRcu"
   }
```

  **Id** : Payment Id is a cryptographically strong id of size 32.  
  **Total** : The invoice amount.  
  **Currency** : The invoice currency.  
  **BtcRequired** : The amount of Bitcoin required to pay this invoice.  
  **CreatedOn** : Date the payment session was created in UTC.  
  **ExpiresOn** : Date the payment session expired in UTC.  
  **Status** : Payment status (see Payment status below).  
  **LastUpdate** : Date the payment session was last updated in UTC.  
  **Url** : Invoice URL; redirect customer or show inside IFRAME (see Invoice section below).  
  
* **Example creating a payment with CURL.**

```bash
curl -u terminalid:password -H "content-Type: application/json" \
 -X POST -d '{ "amount" : 1.23,  "currency" : "CAD" }' https://api.bitbuy.ca/v1/payments
```
## Get Payment
* **URL**

  /payments/:id

* **Method:**

  `GET`
  
* **Authentication:**

  *Required*

* **Data Params**

  `NONE`

  ### Required fields.

  **id** : Payment id.
  
* **Success Response:**

  * **Code:** 200 <br />
    **Content:** 
   ```json
   {
     "Id":"ehePI2Tuuw7jeg9qeVTe3gYNebKzBRcu",
     "Total":1.23000000,
     "Currency":"CAD",
     "BtcRequired":0.00167342,
     "Data":"",
     "CreatedOn":"2016-08-19T20:19:05",
     "ExpiresOn":"2016-08-19T20:26:05",
     "Status":"EXPIRED",
     "LastUpdate":"2016-08-19T20:26:28",
     "Url":"https://api.bitbuy.ca/v1/invoice?id=ehePI2Tuuw7jeg9qeVTe3gYNebKzBRcu"
   }
```

* **Example getting a payment with CURL.**

```bash
curl -u terminalid:password https://api.bitbuy.ca/v1/payments/ehePI2Tuuw7jeg9qeVTe3gYNebKzBRcu
```
## Get Payment Status
* **URL**

  /payments/:id/status

* **Method:**

  `GET`
  
* **Authentication:**

  *Not required*

* **Data Params**

  `NONE`

  ### Required fields.

  **id** : Payment id.
  
* **Success Response:**

  * **Code:** 200 <br />
    **Content:** 
   ```json
   {
     "status":"EXPIRED"
   }
```

* **Example getting payment status with CURL.**

```bash
curl https://api.bitbuy.ca/v1/payments/ehePI2Tuuw7jeg9qeVTe3gYNebKzBRcu/status
```

## Payment Status

**"WAITING"** - Waiting for 0-confirmation.

**"PAID"** - The total sum of 0-confirmation payment sent to the invoice address is equal or larger than the amount of Bitcoin required.

**"CONFIRMED"** - Payment is confirmed; Depending on the speed setting it will be triggered at 0, 1 or 6 confirmations.

**"COMPLETED"** - Payment has been credited to the merchant. This happens after 6 confirmations.

**"CANCELLED"** - Payment has been cancelled by the terminal operator.

**"EXPIRED"** - 0-confirmation was not received within TTL minutes.

## API Rate Limits

Rate limiting of the API is primarily considered on a per-terminal basis.

Rate limits are divided into 15 minute intervals. When an application exceeds the rate limit for a given API endpoint, the server will now return an HTTP 429 “Too Many Requests” response code.

API request rate is limited at 180 queries per 15 minute window.

## BitBUY Invoice

For every payment session created an invoice will be shown at `"https://api.bitbuy.ca/v1/invoice?id=xxxx"` where `xxxx` is the payment id. In a web application you would tipically redirect your customer to the invoice URL.

You can control the look of the invoice by uploading a logo and providing merchant contact information in the merchant portal.

## Using the .NET client

In this repository you can find the source code for a .NET client.
This is the client reference implementation.

NuGet
=====

To install the library in your .NET project, run the following command in the Package Manager Console

```bash
PM> Install-Package Monitex.Client
```

Sample client code in C#
========================

```csharp
var client =
  new Client ("https://api.bitbuy.ca"
  // Do not use these credentials, create a new pair in the merchant panel
  , "TerminalId"
  , "TerminalPassword");

var request = new CreatePaymentRequest (1.00M, "btc");

request.Data = "1733"; // Some data, could be your order id
request.Ttl = 30; // minutes

var response = client.CreatePayment (request);

System.Diagnostics.Process.Start (response.Url);
```

Sample code in PHP
==================

Check out the repository https://github.com/MonitexInc/php-merchant-api-client
