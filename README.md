# Redemption API

### Description

Once a user is in possession of tokens issued by the new Privacy Pass redemption server, any bearer of a token can redeem it through the provided redemption API. This document describes how to use a JSON-RPC endpoint for this purpose.

When a user submits a token for redemption, the server will verify whether the token and its associated data (bindings) are cryptographically valid. If so, the server will mark this token as spent and will return a sucessful response. Otherwise, the server wll return a response indicating the possible cause of the error.

### Redemption Endpoint

~~~sh
 POST  https://privacypass.cloudflare.com/api/redeem
~~~

### Request Object

The endpoint supports one method called `redeem` that receives as input parameter an object with the following structure:
-   "data" : (required) an array of three base64-encoded values.
    1.  A token `t`, which is an octet string.
    2.  A tag generated as `HMAC(sk,R)`, where `sk` is a shared secret previously agreed between client and server, and `R` is the associated information (or bindings).
    3.  A base64-encoded JSON object, indicating an elliptic curve, a hash function, and a hash-to curve function. Example:
    ```json
    {
        "curve": "p256",
        "hash": "sha256",
        "method": "increment"
    }
    ```
-   "bindings" : (required) an array of associated strings, denoted by `R` in the PrivacyPass protocol.
-   "compressed" : (optional) a boolean value indicating whether the elliptic curve points are compressed. Default is `false`.

#### Example

The following snippet shows how it looks a well-formatted object for redemption.

File: `request.json`
```JSON
{
  "jsonrpc": "2.0",
  "method": "redeem",
  "params": {
    "data": [
      "IE/CRo8sBGWRlDDwNJwWAzPryRJ7UlOTrHJdPh3GQxQ=",
      "dYErmr5VLa9A4XPlj1wz2wD31h20VHKIJgz2BKv2cow=",
      "eyJjdXJ2ZSI6InAyNTYiLCJoYXNoIjoic2hhMjU2IiwibWV0aG9kIjoiaW5jcmVtZW50In0="
    ],
    "bindings": [
      "q7r8rfqhodjkc4bcufs1s",
      "zo3zzmvq77m3qwch82dytj"
    ],
    "compressed": false
  },
  "id": 1
}
```

## Submitting a Request

This `curl` transfer shows how to use the redemption API with the previous request object stored in `request.json` file.

```sh
 $ curl -X POST https://privacypass.cloudflare.com/api/redeem \
        -H "Content-Type: application/json"                   \
        --data @request.json
```

A successful redemption is completed with a HTTP 200 status and a JSON response:

```json
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": "success"
}
```

Every subsequent request with the same token will be detected as an intent of double-spending. This situation is alerted with a HTTP-403 status and with the following error response.

```json
{
    "jsonrpc": "2.0",
    "id": 1,
    "error": {
        "message": "Redemption for token that has already been spent",
        "code": -32021
    }
}
```

### Error codes

Several other situations can cause error responses. The following codes are used to indicate the possible source that produced the error.


| Error code | HTTP Status | Description |
|:-----------|:-----------:|:------------|
| -32020 | 403 | When attempting to verify data, often token verification fails the specified HMAC check. |
| -32021 | 403 | The request is attempting to redeem a token that was already  spent. |
| -32022 | 400 | Unsupported elliptic curve settings. Currently, it is supported only the NIST P-256 curve, and the SHA-256 hash function. |
| -32040 | 400 | Requesting the RPC method `issue`. |
| -32041 | 405 | Connecting to the endpoint using a method different than POST. |
| -32600 | 400 | The JSON-RPC request is invalid. |
| -32601 | 400 | Requesting a non-supported RPC method. |
| -32602 | 400 | Indicates that the parameter object has an invalid structure. |
| -32603 | 500 | Internal server error. |
| -32700 | 400 | Failed to parse the JSON request. |

---
Last Update: October 28, 2019.
