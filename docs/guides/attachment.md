---
Triam Attachment Convention
---

# Attachments

Sometimes there is a need to send more information about a transaction than fits in the provided memo field, for example: KYC info, an invoice, a short note. Such data shouldn't be placed in the [ledger](./concepts/ledger.md) because of it's size or private nature. Instead, you should create what we call an `Attachment`. A Triam attachment is simply a JSON document. The sha256 hash of this attachment is included as a memo hash in the transaction. The actual attachment document can be sent to the receiver through some other channel, most likely through the receiver's [Auth server](./compliance-protocol.md).

## Attachment structure

Attachments have a flexible structure. They can include the following fields but these are optional and there can be extra information attached.

```json
{
  "nonce": "<nonce>",
  "transaction": {
    "sender_info": {
      "first_name": "<first_name>",
      "middle_name": "<middle_name>",
      "last_name": "<last_name>",
      "address": "<address>",
      "city": "<city>",
      "province": "<province>",
      "country": "<country in ISO 3166-1 alpha-2 format>",
      "date_of_birth": "<date of birth in YYYY-MM-DD format>",
      "company_name": "<company_name>"
    },
    "route": "<route>",
    "note": "<note>"
  },
  "operations": [
    {
      "sender_info": <sender_info>,
      "route": "<route>",
      "note": "<note>"
    },
    // ...
  ]
}
```

Name | Data Type | Description
-----|-----------|------------
`nonce` | string | [Nonce](https://en.wikipedia.org/wiki/Cryptographic_nonce) is a unique value. Every transaction you send should have a different value. A nonce is needed to distinguish attachments of two transactions sent with otherwise identical details. For example if you send $10 to Bob two days in a row.
`transaction.sender_info` | JSON | JSON containing KYC info of the sender. This JSON object can be extended with more fields if needed.
`transaction.route` | string | The route information returned by the receiving federation server (`memo` value). Tells the receiver how to get the transaction to the ultimate recipient.
`transaction.note` | string | A note attached to transaction.
`operations[i]` | | `i`th operation data. Can be omitted if transaction has only one operation.
`operations[i].sender_info` | JSON | `sender_info` for `i`th operation in the transaction. If empty, will inherit value from `transaction`.
`operations[i].route` | string | `route` for `i`th operation in the transaction. If empty, will inherit value from `transaction`.
`operations[i].note` | string | `note` for `i`th operation in the transaction. If empty, will inherit value from `transaction`.

## Calculating Attachment hash

To calculate the Attachment hash you need to stringify the JSON object and calculate `sha-256` hash. In Node.js:

```js
const crypto = require('crypto');
const hash = crypto.createHash('sha256');

hash.update(JSON.stringify(attachment));
var memoHashHex = hash.digest('hex');
```

To add the hash to your transaction use the [`TransactionBuilder.addMemo`](http://stellar.github.io/js-stellar-base/TransactionBuilder.html#addMemo) method.

## Sending Attachments

To send an Attachment and its hash (in a transaction) to Auth server of a receiving organization read the [Compliance protocol](./compliance-protocol.md) doc for more information.

## Example

```js
var crypto = require('crypto');

var nonce = crypto.randomBytes(16);
var attachment = {
  "nonce": nonce.toString('hex'),
  "transaction": {
    "sender_info": {
      "name": "Sherlock Holmes",
      "address": "221B Baker Street",
      "city": "London NW1 6XE",
      "country": "UK",
      "date_of_birth": "1854-01-06"
    }
  },
  "operations": [
    // Operation #1: Payment for Dr. Watson
    {
      "route": "watson",
      "note": "Payment for helping to solve murder case"
    },
    // Operation #2: Payment for Mrs. Hudson
    {
      "route": "hudson",
      "note": "Rent"
    }
  ]
};

var hash = crypto.createHash('sha256');
hash.update(JSON.stringify(attachment));
var memoHashHex = hash.digest('hex');
console.log(memoHashHex);
```
