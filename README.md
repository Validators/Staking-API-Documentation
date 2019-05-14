Validators - Polkadot Public node API Documentation
=====================================
At Validators we strive to evolve the ecosystem of "green" blockchains with two areas of focus: 1. staking as a service ([Validators.com](https://validators.com)) and 2. public node infrastructure for application developers ([Validators.io](https://validators.io)).

Okay, are you ready to engage with the Polkadot blockchain!

Getting started
---------------

1. Register as a developer and create your project at https://developers.validators.io.
2. Choose a Polkadot Network (Testnet or Mainnet)
3. Add API authentication to your RPC requests (see section below for examples).
4. Email us at team@validators.com if you have any questions. We are very friendly ;).
* * *

Polkadot JSON-RPC endpoints
-----

| **Network** | **Endpoint** | **Authentication** |
|------------|-------------------------------------------------|------------|
| Testnet (PoC-4 "Alexander")  | https://polkadot.testnet.validators.io | HMAC |
| Mainnet (Expected Q4 2019)  | ~~https://polkadot.mainnet.validators.io~~ | HMAC |

Testnet is currently running Proof-of-Concept version 4 "Alexander".

Behind each endpoint is a cluster of nodes working to serve your request. A load balancer and caching layer decides what node are serving data.


API Authentication (HMAC)
-----
The API uses a transaction-based HMAC Authentication that protects against man-in-the-middle attacts and authenticates the request. 

Please add this to the http request headers:

| **Header** | **Description**                                                                               |
|------------|-----------------------------------------------------------------------------------------------|
| api-key    | get your API-Key at https://developers.validators.io                                                                                  |
| hash       | the request body signed by your API Secret using the HMAC-SHA512 method - see authentication examples below |


### Node.js authentication

Example of how to sign a request with node.js using the `crypto` module:

```js
const crypto = require("crypto");

const message = {
   "jsonrpc": "2.0",
   "id": "test",
   "method": "getLatestBlockExampleMethod",
   "params": {
      "paramSomething": "2333",
      "paramAnother": "20"
   }
};

const sign = crypto
   .createHmac('sha512', apiSecret)
   .update(JSON.stringify(message))
   .digest('hex');
```

### Postman authentication

Here is a guide to create the transaction hash with postman: 

1. Create new request. Select the `Headers` tab and add `ApiKey` and `Hash` headers. Use postman variable syntax for them in `Value` column. These variables will be updated for each request using the pre-request script.

![Postman headers setup](https://static.validators.com/images/Postman-Hmac-headers.png)

2. Paste the following code to the `Pre-request Script` tab for the request. Fill in the apiKey and apiSecret variables you got from https://developers.validators.io. Be careful not to share your secret.

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

Please goto the official Polkadot JSON-RPC documentation at: https://polkadot.js.org/api/METHODS_RPC.html#json-rpc for full details.


### Example request "chain_getBlock"

As a simple illustration of a call to the JSON RPC endpoint this is taken from the Polkadot documentation. Returns the latest block.


Example request `Body` with authorization headers as described above (not shown here):

```json
{
   "jsonrpc": "2.0",
   "id": "1",
   "method": "chain_getBlock"
}
```

<p></p>
<p>
Example Response:
</p>

```json
{
    "jsonrpc": "2.0",
    "id": "1",
    "result": {
        "block": {
            "extrinsics": [
                "0x0100000344aeda5c",
                "0x010d0000"
            ],
            "header": {
                "digest": {
                    "logs": [
                        "0x04617572612101b6c7790f000000005f6b983aa05c8c75d56e5bd0c2eaddcbeec0f50d200f9ee04fc0273d2d03efd2507c6c9678ec793e4568d1ebe1cf3aac1daddc8557db78d7d1a6906a3ccc0406"
                    ]
                },
                "extrinsicsRoot": "0x7d196442043952b736f1e7dbcfd22e9b89e609c6bbbb87b2beec1f2f8755f159",
                "number": "0x14327f",
                "parentHash": "0x73a3549db2c7f2b322704b0fbb1f0b62764438afa1d01599d01c8965159f8713",
                "stateRoot": "0x957db02495df7dbb45d4b33252ccd7001b2a3b0ade912572ddebd384ea8031e5"
            }
        },
        "justification": null
    }
 }
 ```
