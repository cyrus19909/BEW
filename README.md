## Table of Content
- Content
  - [Introduction](#Introduction)
  - [Getting Started](#GettingStarted)
  - [Authentication](#Authentication)
  - [Wallets](#Wallets)
  	- [Overview](#Overview)
  	- [Subaccounts](#Subaccounts)
  	- [Balances](#Balances)
  	- [Addresses](#Addresses)
  	- [Deposits](#Deposits)
  	- [Withdrawals](#Withdrawals)
  - [Internal Transfers](#internaltransfers)


## Introduction
This documentation provides framework guidelines and high-level overview to build upon the API subaccount feature, known as Bittrex Enterprise Wallets (BEW).
The subaccount feature enables the partner to create an independent subaccount for their individual users. The partners can provide their users with the follwoing abilities
  1. Create subaccount deposit addresses.
  2. Manage Deposits
  3. Withdraw funds
  4. Place orders
  
Subaccount feature allows partners to remove two complex components of their platform such as wallets infrastructure and accounts database. Partners will have to maintain a local database to map their customers details to the SubaccountID which will be unique to the users.

## Getting Started
The partners need to commit to the following prerequisites to have access to subaccount feature 

* Contact your Bittrex representative, and they will assist you with enabling Bittrex Enterprise Wallets.
* Enable 2FA on your account. API Keys cannot be generated unless 2FA is enabled.
* API key must be created manually through the UI right now.
* Please reach out to your Bittrex representative or  corpcare@bittrex.com to get your API key whitelisted and enable subaccount feature.
* All REST requests must be sent to https://api.bittrex.com/v3 using the application/json content type. Non-HTTPS requests will be redirected to HTTPS, possibly causing functional or performance issues with your application.

### Subaccount Limitations
Some subaccount limitations include 
* API Keys cannot be created for a subaccount for the users.
* 2FA must be enabled on the account 
* Whitelist must be enforced and implemented by the partner. 

## Authentication

#### Overview
In order to properly sign an authenticated request for the Bittrex v3 API, the following headers must be included:
```
Api-Key
Api-Timestamp
Api-Content-Hash
Api-Signature
Api-Subaccount-Id
```
The following sections are instructions for properly populating these headers.
##### Api-Key
Populate this header with your API key.

```8d4bxxxxxxx048d7be1cd112eed86a2```

##### Api-Timestamp
Populate this header with the current time as a UNIX timestamp, in epoch-millisecond format.

Sample JS Code Snippet:
```
var timestamp = new Date().getTime();
Example Value: 1542323450016
```
##### Api-Content-Hash
Populate this header with a SHA512 hash of the request contents, Hex-encoded. If there are no request contents, populate this header with a SHA512 hash of an empty string. For authentication example we use empty string.

Sample JS Code Snippet:
```
var content = ““ 
var contentHash = CryptoJS.SHA512(content).toString(CryptoJS.enc.Hex);

cf83e1357eefb8bdf1542850d66d8007d620e4050b5715dc83f4a921d36ce9ce47d0d13c5d85f2b0ff8318d2877eec2f63b931bd47417a81a538327af927da3e
```

##### Api-Subaccount-Id (Only for subaccount feature)
If you wish to make a request on behalf of a subaccount, you will need to:
	* Authenticate using all 4 of the headers above referring to your master account.
	* Populate the Api-Subaccount-Id header with the Guid of the subaccount you wish to impersonate for this request. The specified subaccount must be a subaccount of the master account used to authenticate the request. (please refer to SUBACCOUNTS section on how to generate or retrieve subaccount ID)
	* Include the Api-Subaccount-Id header at the end of the pre-signed signature, as indicated in the next section.
	
```x111x11x-8968-48ac-b956-x1x11x111111```

##### Api-Signature
Create a pre-signature string formed from the following items and concatenating them together:
  1.Contents of your Api-Timestamp header
  2.The full URI you are using to make the request (including query string)
  3.The HTTP method of the request, in all caps (GET, POST, DELETE, etc.)
  4.Contents of your Api-Content-Hash header
  5.Content of your Api-Subaccount-Id header (or an empty string if not present)

Once you have created this pre-sign string, sign it via HmacSHA512, using your API secret as the signing secret. Hex-encode the result of this operation and populate the Api-Signature header with it.

__Sample JS Code Snippet:__
```
var uri = 'https://api.bittrex.com/v3/balances';
var apiSecret = “c6df75xxxxxxxa3d92daxxxxxxx375a3”
var preSign = [timestamp, uri, method, contentHash, subaccountId].join('');
var signature = CryptoJS.HmacSHA512(preSign, apiSecret).toString(CryptoJS.enc.Hex);
```

__Example Pre-Signed Value:with subaccount)__

```1542323450016https://api.bittrex.com/v3/balancesGETcf83e1357eefb8bdf1542850d66d8007d620e4050b5715dc83f4a921d36ce9ce47d0d13c5d85f2b0ff8318d2877eec2f63b931bd47417a81a538327af927da3ex111x11x-8968-48ac-b956-x1x11x111111```

__Example Post-Signed Value:__

```47623f0efbe10bfbb32f18e5d8885b2a91be3c3cea82adf0dd2d20892b20bcb6a10a91fec3afcedcc009f2b2a86c5366974cfadcf671fe0490582568f51f```

__Header example:__
```
var header = {
 		"Api-Key": "<apiKey>",
 		"Api-Timestamp": "<timestamp>",
 		"Api-Content-Hash": "<contentHash>",
 		"Api-Signature": "<signature>"
 	}
```

__Below is the POST/ addresses example of the REST call implemented in JS.__
```
var CryptoJS = require("crypto-js")
const request = require('request')
apiKey = "8dxxxxxxe5f048d7be1cdxxxxxx86a27"
apiSecret = "e6xxxxxx2d314548a7c2a4xxxxxxd1ee"
var content = {
    "currencySymbol": "BTC"
}
var timestamp = new Date().getTime();
var contentHash = CryptoJS.SHA512(content).toString(CryptoJS.enc.Hex);
console.log(contentHash)
uri = 'https://api.bittrex.com/v3/addresses'
var method = ‘POST’
var subaccountId = “”
var preSign = [timestamp, uri, contentHash, method, subaccountId].join('');
var signature = CryptoJS.HmacSHA512(preSign, apiSecret).toString(CryptoJS.enc.Hex);
console.log(signature)
request.post('https://api.bittrex.com/v3/balances', {
    json: content
}, (error, res, body) => {
    if (error) {
        console.error(error)
        return
    }
    console.log(`statusCode: ${res.statusCode}`)
    console.log(body)
}) 
```

## WALLETS

### Overview

Wallets page entitles one to perform the following operations.
	1. Display individual holdings of all the different coins in possession of the person.
	2. Display estimated total holding by summing up the individual balances.
	3. Allows one to provision a Wallet address for a coin and make deposit to it.
	4. Allows one to make a withdrawal to an account outside Bittrex.
	5. Allows one to see the withdrawal history and status of the withdrawal.
	6. Allows one to see the deposit history and status of the deposit.
	
### Subaccounts
This section is an overview of subaccounts features.

#### GET /subaccounts
List subaccounts. (NOTE: This API is limited to partners and not available for traders.) Pagination and the sort order of the results are in inverse order of the CreatedAt field.

##### Headers (Explained under Authentication)
##### Request URI https://api.bittrex.com/v3/subaccounts
##### Response Example (200OK)
```
{
   "id": "5c838f11-f13c-47f7-8961-a5bfc0e982c2",
   "createdAt": "2019-06-18T17:56:00.087Z"
} 
```
#### POST /subaccounts
Create a new subaccount. (NOTE: This API is limited to partners and not available for traders.) 

##### Headers (Explained under Authentication)
##### Request URI https://api.bittrex.com/v3/subaccounts
##### Request Body
```
{}
```
##### Response Example (200OK)
```
{
    "id": "5c838f11-f13c-47f7-8961-a5bfc0e982c2",
    "createdAt": "2019-06-18T17:56:00.087Z"
}
```
#### GET / subaccounts /{subaccountId}
Retrieve details for a specified subaccount. (NOTE: This API is limited to partners and not available for traders.)  

##### Headers (Explained under Authentication)
##### Request URI https://api.bittrex.com/v3/subaccounts

##### Response Example (200OK)
```
{
    "id": "5c838f11-f13c-47f7-8961-a5bfc0e982c2",
    "createdAt": "2019-06-18T17:56:00.087Z"
}
```

### Balances

#### GET /balances
List account balances across available currencies. Returns a Balance entry for each currency for which there is either a balance or an address. 

##### Headers (Explained under Authentication)
##### Request URI  https://api.bittrex.com/v3/balances 
##### Response Example (200OK)
```
[ 
    { 
        "currencySymbol": "BTC", 
        "total": "0", 
        "available": "0" 
    }, 
    { 
        "currencySymbol": "PESOS", 
        "total": "1000000.00000000", 
        "available": "1000000.00000000" 
    }, 
    { 
        "currencySymbol": "QRL", 
        "total": "0", 
        "available": "0" 
    } 
] 
```

#### GET /balances/{currencySymbol}
Retrieve account balance for a specific currency. Request will always succeed when the currency exists, regardless of whether there is a balance or address. 

##### Headers (Explained under Authentication)
##### Request URI  https://api.bittrex.com/v3/balances/btc
##### Response Example (200OK)
```
{ 
    "currencySymbol": "BTC", 
    "total": "0.65363625", 
    "available": "0.65363625" 
}   
```

### Addresses

When a user wants to make a deposit, the address for the specific wallet must be generated.The following endpoints can be used to create or query the existing wallet addresses for individual user.

#### GET /addresses

List deposit addresses that have been requested or provisioned. 

##### Headers (Explained under Authentication)
##### Request URI  https://api.bittrex.com/v3/addresses 
##### Response Example (200OK)
```
[ 
    { 
        "status": "PROVISIONED", 
        "currencySymbol": "BTM", 
        "cryptoAddress": "tm1q386xw6g7ke64m3ptm9jw0zda6xmalx0k8h9k4g" 
    }, 
    { 
        "status": "PROVISIONED", 
        "currencySymbol": "XLM", 
        "cryptoAddress": "GB6YPGW5JFMMP2QB2USQ33EUWTXVL4ZT5ITUNCY3YKVWOJPP57CANOF3", 
        "cryptoAddressTag": "ad7154d0733649009c3" 
    } 
] 
```

#### POST /addresses 
Request provisioning of a deposit address for a currency for which no address has been requested or provisioned. 

##### Headers (Explained under Authentication)
##### Request URI  https://api.bittrex.com/v3/addresses
##### Request Body
```
{ 
    "currencySymbol": "BTC"
}
```
##### Response Example (200OK)
```
{ 
    "status": "PROVISIONED", 
    "currencySymbol": "BTC", 
    "cryptoAddress": "2N8e4ZjZofi9W33P4ZrJjmqRF7zZTXHB16t " 
} 
```

#### GET /addresses/{currencySymbol} 
Retrieve the status of the deposit address for a particular currency for which one has been requested or provisioned.

##### Headers (Explained under Authentication)
##### Request URI  https://api.bittrex.com/v3/addressses/btc
##### Response Example (200OK)
```
{ 
    "status": "PROVISIONED", 
    "currencySymbol": "BTC", 
    "cryptoAddress": "2N8e4ZjZofi9W33P4ZrJjmqRF7zZTXHB16t" 
}
```

### Deposits

When a user wants to make a deposit or query existing deposits, the following endpoints can be used to create or query open/closed  deposits.  

#### GET /deposits/open
List open deposits. Results are sorted in inverse order of UpdatedAt and are limited to the first 1000.

##### Headers (Explained under Authentication)
##### Request URI  https://api.bittrex.com/v3/deposits/open
##### Response Example (200OK)
```
[
    { 
	"id": "string (uuid)", 
	"currencySymbol": "string", 
	"quantity": "number (double)", 
	"cryptoAddress": "string", 
	"cryptoAddressTag": "string", 
	"txId": "string", 
	"confirmations": "integer (int32)", 
	"updatedAt": "string (date-time)", 
	"completedAt": "string (date-time)",
	"status": "string"
    }
] 
```

#### GET /deposits/closed
List closed deposits. StartDate and EndDate filters apply to the CompletedAt field. Pagination and the sort order of the results are in inverse order of the CompletedAt field.

##### Headers (Explained under Authentication)
##### Request URI  https://api.bittrex.com/v3/deposits/open
##### Response Example (200OK)
```
[
    { 
	"id": "string (uuid)", 
	"currencySymbol": "string", 
	"quantity": "number (double)", 
	"cryptoAddress": "string", 
	"cryptoAddressTag": "string", 
	"txId": "string", 
	"confirmations": "integer (int32)", 
	"updatedAt": "string (date-time)", 
	"completedAt": "string (date-time)", 
	"status": "string" 
    }
] 
```

#### GET /deposits/ByTxId/{txId} 
Retrieves all deposits for this account with the given TxId. 

##### Headers (Explained under Authentication)
##### Request URI  https://api.bittrex.com/v3/deposits/ByTxId/{txId}
##### Response Example (200OK)
```
[
    { 
	"id": "string (uuid)", 
	"currencySymbol": "string", 
	"quantity": "number (double)", 
	"cryptoAddress": "string", 
	"cryptoAddressTag": "string", 
	"txId": "string", 
	"confirmations": "integer (int32)", 
	"updatedAt": "string (date-time)", 
	"completedAt": "string (date-time)", 
	"status": "string" 
    }
] 
```

#### GET /deposits/{depositId}
Retrieve information for a specific deposit.

##### Headers (Explained under Authentication)
##### Request URI  https://api.bittrex.com/v3/deposits/{depositId}
##### Response Example (200OK)
```
[
    { 
	"id": "string (uuid)", 
	"currencySymbol": "string", 
	"quantity": "number (double)", 
	"cryptoAddress": "string", 
	"cryptoAddressTag": "string", 
	"txId": "string", 
	"confirmations": "integer (int32)", 
	"updatedAt": "string (date-time)", 
	"completedAt": "string (date-time)", 
	"status": "string" 
    }
] 
```

### Withdrawals

When the user wants to make or query existing withdrawals, the following endpoints can be used. The following open/closed endpoint can be used to determine the state of withdrawals (AUTHORIZED,PENDING,COMPLETED) . One can also cancel the withdrawal if the status has not reached PENDING.

#### GET /withdrawals/open
List open withdrawals. Results are sorted in inverse order of the CreatedAt field and are limited to the first 1000.

##### Headers (Explained under Authentication)
##### Request URI  https://api.bittrex.com/v3/withdrawals/open
##### Response Example (200OK)
```
 [ 
    { 
        "id": "37f81c21-fec1-49a5-b4a8-10215ce7476b", 
        "currencySymbol": "BTC", 
        "quantity": "0.00095000", 
        "cryptoAddress": "mv4rnyY3Su5gjcDNzbMLKBQkBicCtHUtFB", 
        "cryptoAddressTag": "", 
        "txCost": "0.00005000", 
        "status": "PENDING", 
        "createdAt": "2019-05-10T16:52:37.467Z" 
    } 
] 
```

#### GET /withdrawals/closed 
List closed withdrawals. StartDate and EndDate filters apply to the CompletedAt field. Pagination and the sort order of the results are in inverse order of the CompletedAt field.

##### Headers (Explained under Authentication)
##### Request URI  https://api.bittrex.com/v3/withdrawals/closed
##### Response Example (200OK)
```
[ 
    { 
        "id": "1e411139-df4a-4406-8353-25abbf793f75", 
        "currencySymbol": "BTC", 
        "quantity": "0.00095000", 
        "cryptoAddress": "bc1qljg8qdesd49jyz7s0gamc7pqz3r2wmxa9vrmug", 
        "cryptoAddressTag": "", 
        "txCost": "0.00005000", 
        "status": "CANCELLED", 
        "createdAt": "2019-05-30T22:28:28.817Z", 
        "completedAt": "2019-06-11T19:16:21.67Z" 
    } 
] 
```

#### GET /withdrawals/ByTxId/{txId} 
Retrieves all withdrawals for this account with the given TxId.

##### Headers (Explained under Authentication)
##### Request URI  https://api.bittrex.com/v3/withdrawals/ByTxId/{txId}
##### Response Example (200OK)
```
[ 
    { 
        "id": "1e411139-df4a-4406-8353-25abbf793f75", 
        "currencySymbol": "BTC", 
        "quantity": "0.00095000", 
        "cryptoAddress": "bc1qljg8qdesd49jyz7s0gamc7pqz3r2wmxa9vrmug", 
        "cryptoAddressTag": "", 
        "txCost": "0.00005000", 
        "status": "CANCELLED", 
        "createdAt": "2019-05-30T22:28:28.817Z", 
        "completedAt": "2019-06-11T19:16:21.67Z" 
    } 
]  
```

#### GET /withdrawals/{withdrawalId}
Retrieve information on a specified withdrawal.

##### Headers (Explained under Authentication)
##### Request URI  https://api.bittrex.com/v3/withdrawals/ByTxId/{txId}
##### Response Example (200OK)
```
[ 
    { 
	"id": "1e411139-df4a-4406-8353-25abbf793f75", 
	"currencySymbol": "BTC", 
	"quantity": "0.00095000", 
	"cryptoAddress": "bc1qljg8qdesd49jyz7s0gamc7pqz3r2wmxa9vrmug", 
	"cryptoAddressTag": "", 
	"txCost": "0.00005000", 
	"status": "CANCELLED", 
	"createdAt": "2019-05-30T22:28:28.817Z", 
	"completedAt": "2019-06-11T19:16:21.67Z" 
    } 
] 
```

#### DELETE /withdrawals/{withdrawalId}
Cancel a withdrawal. (Withdrawals can only be cancelled if status is REQUESTED, AUTHORIZED, or ERROR_INVALID_ADDRESS.)

##### Headers (Explained under Authentication)
##### Request URI  https://api.bittrex.com/v3/withdrawals/{withdrawalId}
##### Response Example (200OK)
```
N/A
```

#### POST /withdrawals
Create new withdrawal. Please note: You will get an error if you try to send coins directly between Bittrex Accounts.  

##### Headers (Explained under Authentication)
##### Request URI  https://api.bittrex.com/v3/withdrawals
##### Request Body
```
{
    "currencySymbol": "BTC", 
    "quantity": "1", 
    "address": "1J8rcVPwRjJdi1jhk95AsjMfsLBbP7x7bS" 
}
```
##### Response Example (200OK)
```
{ 
    "id": "b239807d-b921-40e7-bc71-553fdff2d1fb",
    "currencySymbol": "BTC",
    "quantity": "0.00095000",
    "cryptoAddress": "2MsM3kfaANtqz4ASuSFCmk5skspYkpj3a1y",
    "cryptoAddressTag": "",
    "txCost": "0.00005000",
    "status": "AUTHORIZED",
    "createdAt": "2019-06-18T16:55:35.95Z"
}
```

## Internal Transfers

General notes on internal transfers.
1. The minimum that can be transferred is 1Sats (BTC 0.00000001).
2. Transfers work on available balance and NOT total balance. Same applies to deposits and withdrawals.

#### POST /v3/transfers

#### Transfer from subaccount to master.
Append subaccountID to the header and preSignature
##### Request URI  https://api.bittrex.com/v3/transfers
##### Request Body
```
{ 
    "toMasterAccount": true, 
    "currencySymbol": "BTC", 
    "amount": "0.0001"
}
```
##### Response Example (200OK)
```
{ 
    "id": "dc0ab541-4026-4451-b994-d20d44f58ed4", 
    "executedAt": "2019-06-17T21:13:30.57Z" 
}
```

#### Transfer from master to subaccount.
##### Request URI  https://api.bittrex.com/v3/transfers
##### Request Body
```
{ 
    "toSubaccountId": "592c8a5b-703f-4b3c-899f-5d5ea1ad8fea", 
    "currencySymbol": "BTC", 
    "amount": "0.0001" 
}
```
##### Response Example (200OK)
```
{ 
    "id": "01e203ec-bb4a-4b52-9f49-914a88d52b2b", 
    "executedAt": "2019-06-17T21:25:58.05Z"
}
```

#### Transfer from subaccount to subaccount.
Append subaccountID to the header and preSignature
##### Request URI  https://api.bittrex.com/v3/transfers
##### Request Body
```
{ 
    "toSubaccountId": "b61bdb98-ed6a-4bbf-8cec-43ff31518b01", 
    "currencySymbol": "BTC", 
    "amount": "0.0001"
} 
```
##### Response Example (200OK)
```
{ 
    "id": "01e203ec-bb4a-4b52-9f49-914a88d52b2b", 
    "executedAt": "2019-06-17T21:25:58.05Z"
}
```

#### GET /v3/transfers/{transferID}

#### Get transfer by ID (from master to subaccount)
##### Request URI  https://api.bittrex.com/v3/transfers/{transferID}
##### Response Example (200OK)
```
{ 
    "toSubaccountId": "592c8a5b-703f-4b3c-899f-5d5ea1ad8fea", 
    "id": "dc0ab541-4026-4451-b994-d20d44f58ed4", 
    "currencySymbol": "BTC", 
    "amount": "0.00010000", 
    "executedAt": "2019-06-17T21:13:30.57Z" 
} 
```

#### Get transfer by ID (from subaccount to subaccount)
Append subaccountID to the header and preSignature
##### Request URI  https://api.bittrex.com/v3/transfers/{transferID}
##### Response Example (200OK)
```
{ 
    "toSubaccountId": "b61bdb98-ed6a-4bbf-8cec-43ff31518b01", 
    "id": "01e203ec-bb4a-4b52-9f49-914a88d52b2b", 
    "currencySymbol": "BTC", 
    "amount": "0.00010000", 
    "executedAt": "2019-06-17T21:25:58.05Z" 
}
```

#### Get transfer by ID (from subaccount to master)
Header: SubaccountId required and the preSignature needs the subaccountID.
##### Request URI  https://api.bittrex.com/v3/transfers/{transferID}
##### Response Example (200OK)
```
{
    "toMasterAccount": true,
    "id": "dc0ab541-4026-4451-b994-d20d44f58ed4", 
    "currencySymbol": "BTC", 
    "amount": "0.00010000", 
    "executedAt": "2019-06-17T21:13:30.57Z"
}
```

### GET /v3/transfers/sent

#### Get list of sent transfers from master to subaccounts
##### Request URI  https://api.bittrex.com/v3/transfers/sent
##### Response Example (200OK)
```
[ 
    { 
        "toSubaccountId": "592c8a5b-703f-4b3c-899f-5d5ea1ad8fea", 
        "id": "4ca1e662-ca6c-4071-b5b2-2786080e0926", 
        "currencySymbol": "BTC", 
        "amount": "0.00010000", 
        "executedAt": "2019-06-17T21:07:45.3Z" 
    }, 
    { 
        "toSubaccountId": "592c8a5b-703f-4b3c-899f-5d5ea1ad8fea", 
        "id": "ca3bdd7b-f799-4992-aae3-605309bffc38", 
        "currencySymbol": "BTC", 
        "amount": "0.00010000", 
        "executedAt": "2019-06-17T21:07:43.4Z" 
    } 
] 
```

#### Get list of sent transfers from subaccount to subaccounts and master
Append subaccountID to the header and preSignature
##### Request URI  https://api.bittrex.com/v3/transfers/sent
##### Response Example (200OK)
```
[ 
    { 
        "toSubaccountId": "b61bdb98-ed6a-4bbf-8cec-43ff31518b01", 
        "id": "01e203ec-bb4a-4b52-9f49-914a88d52b2b", 
        "currencySymbol": "BTC", 
        "amount": "0.00010000", 
        "executedAt": "2019-06-17T21:25:58.05Z" 
    }, 
    { 
        "toMasterAccount": true, 
        "id": "dc0ab541-4026-4451-b994-d20d44f58ed4", 
        "currencySymbol": "BTC", 
        "amount": "0.00010000", 
        "executedAt": "2019-06-17T21:13:30.57Z" 
    } 
] 
```

### GET /v3/transfers/received

#### Get list of received transfers by master
##### Request URI  https://api.bittrex.com/v3/transfers/received
##### Response Example (200OK)
```
[ 
    { 
        "fromSubaccountId": "592c8a5b-703f-4b3c-899f-5d5ea1ad8fea", 
        "id": "dc0ab541-4026-4451-b994-d20d44f58ed4", 
        "currencySymbol": "BTC", 
        "amount": "0.00010000", 
        "executedAt": "2019-06-17T21:13:30.57Z" 
    } 
] 
```

#### Get list of received transfers by subaccount
Append subaccountID to the header and preSignature
##### Request URI  https://api.bittrex.com/v3/transfers/received
##### Response Example (200OK)
```
[ 
    { 
        "fromMasterAccount": true, 
        "id": "4ca1e662-ca6c-4071-b5b2-2786080e0926", 
        "currencySymbol": "BTC", 
        "amount": "0.00010000", 
        "executedAt": "2019-06-17T21:07:45.3Z" 
    }, 
    { 
        "fromMasterAccount": true, 
        "id": "ca3bdd7b-f799-4992-aae3-605309bffc38", 
        "currencySymbol": "BTC", 
        "amount": "0.00010000", 
        "executedAt": "2019-06-17T21:07:43.4Z" 
    }
] 
```
