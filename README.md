Validators - Staking API Documentation
=====================================

For Wallet Providers: Give your users a great experience with staking their cryptocurrencies with Validators through our Wallet Provider API.

### **Table of contents**:

* [Getting started](#getting-started)
* [API-Authentication](#api-authentication)
* [API-RPC Methods](#api-endpoints)
  - [getStakingStatus](#get-staking-status)
  - [registerStaking](#register-staking)
  - [updateStakingEmail](#update-staking-email)
  - [getPayoutHistory](#getpayouthistory)

Getting started
---------------

1. Register as a Wallet Provider and get a free API-Key at https://providers.validators.com.
2. The wallet should start by calling:
   * [getStakingStatus](#get-staking-status) (to see if user already has staked with validators.com)
   * If **yes**: then show the response data: Estimated Earnings etc.
   * If **no**: then show a button like **"Earn interest"** or **"Start delegating"** that takes the user to a register page.
      - Show a field that takes an email address.
      - Show a checkmark field with a "Terms of Service" link (Use the "general.termsUrl" from response data).  
      - Then call the [registerStaking](#register-staking) method.
3. Please open an issue in this repository if you have any questions.
* * *

API Authentication
-----
The API uses a transaction-based HMAC Authentication that protects against man-in-the-middle attacts and authenticates the wallet provider. 

Please add this to the http request headers:

| **Header** | **Description**                                                                               |
|------------|-----------------------------------------------------------------------------------------------|
| api-key    | get your API-Key at https://providers.validators.com                                                                                  |
| hash       | the request body signed by your API Secret using the HMAC-SHA512 method - see authentication examples below |


### Node.js authentication

Example of how to sign a request with node.js `crypto` module:

```js
const crypto = require("crypto");

const message = {
   "jsonrpc": "2.0",
   "id": "test",
   "method": "getSomeMethod",
   "params": {
      "idSomething": "2333",
      "valueSomething": "20"
   }
};

const sign = crypto
   .createHmac('sha512', apiSecret)
   .update(JSON.stringify(message))
   .digest('hex');
```

### Postman authentication

Here is a small guide how to create the transaction hash with postman: 

1. Create new request. Select the `Headers` tab and add `ApiKey` and `Hash` headers. Use postman variable syntax for them in `Value` column. These variables will be updated for each request using the pre-request script.

![Postman headers setup](https://static.validators.com/images/Postman-Hmac-headers.png)

2. Paste the following code to the `Pre-request Script` tab for the request. Fill in the apiKey and apiSecret variables you got from https://providers.validators.com. Be very careful not to accidentally share your secret.

```js
eval(postman.getGlobalVariable('crypto-js'))

const crypto = require('crypto-js')

const apiKey = ''
const apiSecret = ''

const hash = crypto.HmacSHA512(request.data, apiSecret).toString()

postman.setEnvironmentVariable('apiKey', apiKey)
postman.setEnvironmentVariable('hash', hash)
```

![Postman pre-request script setup](https://static.validators.com/images/Postman-Hmac-configuration.png)

3. Thats it. Now go to the `Body` tab and make your first JSON-RPC Method call from Postman. 

API RPC-Methods
-----

JSON-RPC Url: https://api.validators.com

### Types explained

| Type | Description |
|----------|----------------------|
| microtezzies | Is the lowest denominator of XTZ: 1.0 XTZ = 1,000,000 microtezzies. (6 zeroes). To get XTZ then multiply by 1,000,000 |
| utc datetime | YYYY-MM-DDTHH:MM:SS example 2019-03-05T10:05:48 |


### Method: getStakingStatus

The first endpoint that needs to be called (from a new wallet installation) in order to show if a user already has staked assets with Validators.

Example Request: (With authentication headers as specified above)

```json
{
   "jsonrpc": "2.0",
   "id": "test",
   "method": "getStakingStatus",
   "params": {
      "address": "tz1ZnxnLAm37HuVdSVMpArZLtcNMdYWV2Bfk",
      "currency": "XTZ"
   }
}
```

<details><summary>Show property description</summary>
<p></p>

| Property | Required | Description |
|----------|----------------------|-------------|
| address     | required | the crypto currency address used by the user |
| currency       | required | the crypto currency symbol |
</details>
<p></p>
<p>
Example Response:
</p>

```json
{
   "jsonrpc": "2.0",
   "id": "test",
   "result": {
      "address":{
         "accountUrl": "",
         "stakedAmount": "2000000",
         "totalEarningsReceived":"0",
         "nextPayout":{
            "estimatedUtc": "2019-03-05T10:05:48",
            "estimatedAmount": "150000",
         }
      },
      "general":{
         "estimatedAnnualPercentage":"5%",
         "logoUrl": "",
         "termsUrl": ""
      }
   }
}
```
Returns an empty **"address"** if no staking has been registered.
<details><summary>Show property description</summary>
<p></p>

| Property | Type | Description |
|----------|----------------------|-------------|
| address.**accountUrl** | URI | The url where user can login using their crypto address |
| address.**stakedAmount**   | microtezzies | the amount that are currently staked |
| address.**totalEarningsReceived** | microtezzies | the total amount received to date |
| address.nextPayout.**estimatedUtc** | utc datetime | estimated next payment date |
| address.nextPayout.**estimatedAmount** | microtezzies |estimated next payout amount |
| ~~address.nextPayout.**estimatePercentage**~~ | percentage | Not ready: estimated next percentage in the given cycle |
| general.**estimatedAnnualPercentage** | percentage | estimated yearly percentage |
| general.**logoUrl** | URI | The url to Validators logo (can be used on registration page) |
| general.**termsUrl** | URI | The url to Validators terms of service (link should be visible on registration page) |

</details>


<p></p>


#### Method: registerStaking

The first endpoint that needs to be called in order to register a stake with Validators. This call also returns the address that should be used when delegating funds to Validators. 


> After this call, the actual staking to the blockchain can be performed:

>**Tezos:** Delegation (staking) call can be performed using [Eztz](https://github.com/TezTech/eztz/) javascript library by calling the [setDelegate](https://github.com/TezTech/eztz/blob/master/src/main.js#L684) method.

Request example: (With authentication headers as specified above)

```json
{
   "jsonrpc": "2.0",
   "id": "test",
   "method": "registerStaking",
   "params": {
      "address": "tz1ZnxnLAm37HuVdSVMpArZLtcNMdYWV2Bfk",
      "balance": "10.123456",
      "currency": "XTZ",
      "email":"example@gmail.com",
      "emailPayoutNotification": "true"
   }
}
```

<details><summary>Show params description</summary>
<p></p>

| Property | Required | Description |
|----------|----------------------|-------------|
| address     | required             | the tezos address of the user |
| balance       | required             | the address balance in tezzies (tz) |
| currency       | required             | the crypto currency symbol |
| email       | required             | email of the wallet user |
| emailPayoutNotification | required | true or false - true means they will receive an email when there is a payout |
</details>
<p></p>
<p>For any questions please create an "issue" here or email us at: team@validators.com.