### Introduction
This documentation provides framework guidelines and high-level overview to build upon the API subaccount feature, known as Bittrex Enterprise Wallets (BEW).
The subaccount feature enables the partner to create an independent sub-account for their individual users. The partners can provide their users with the follwoing abilities
  1. Create sub-account deposit addresses.
  2. Manage Deposits
  3. Withdrawa funds
  4. Place orders
  
Subaccount feature allows partners to remove two complex components of their platform such as wallets infrastructure and accounts database. Partners will have to maintain a local database to map their customers details to the SubaccountID which will be unique to the users.

### Getting Started
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

### Authentication

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

__Sample JS Code Snippet: __
```
var uri = 'https://api.bittrex.com/v3/balances';
var apiSecret = “c6df75xxxxxxxa3d92daxxxxxxx375a3”
var preSign = [timestamp, uri, method, contentHash, subaccountId].join('');
var signature = CryptoJS.HmacSHA512(preSign, apiSecret).toString(CryptoJS.enc.Hex);
```

__Example Pre-Signed Value:with subaccount)__

```1542323450016https://api.bittrex.com/v3/balancesGETcf83e1357eefb8bdf1542850d66d8007d620e4050b5715dc83f4a921d36ce9ce47d0d13c5d85f2b0ff8318d2877eec2f63b931bd47417a81a538327af927da3ex111x11x-8968-48ac-b956-x1x11x111111```

__Example Post-Signed Value: __

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
