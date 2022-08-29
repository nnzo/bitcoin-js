bitcoinjs-rpc.js
===============

[![NPM Package](https://img.shields.io/npm/v/bitcoinjs-rpc.svg?style=flat-square)](https://www.npmjs.org/package/bitcoinjs-rpc)
[![Build Status](https://img.shields.io/travis/bitpay/bitcoinjs-rpc.svg?branch=master&style=flat-square)](https://travis-ci.org/bitpay/bitcoinjs-rpc)
[![Coverage Status](https://img.shields.io/coveralls/bitpay/bitcoinjs-rpc.svg?style=flat-square)](https://coveralls.io/r/bitpay/bitcoinjs-rpc?branch=master)

A client library to connect to Bitcoin Core RPC in JavaScript.

## Get Started

bitcoinjs-rpc.js runs on [node](http://nodejs.org/), and can be installed via [npm](https://npmjs.org/):

```bash
npm install bitcoinjs-rpc
```

## Examples

```javascript
var run = function() {
  var bitcore = require('bitcore');
  var RpcClient = require('bitcoinjs-rpc');

  var config = {
    protocol: 'http',
    user: 'user',
    pass: 'pass',
    host: '127.0.0.1',
    port: '18332',
  };

  // config can also be an url, e.g.:
  // var config = 'http://user:pass@127.0.0.1:18332';

  var rpc = new RpcClient(config);

  var txids = [];

  function showNewTransactions() {
    rpc.getRawMemPool(function (err, ret) {
      if (err) {
        console.error(err);
        return setTimeout(showNewTransactions, 10000);
      }

      function batchCall() {
        ret.result.forEach(function (txid) {
          if (txids.indexOf(txid) === -1) {
            rpc.getRawTransaction(txid);
          }
        });
      }

      rpc.batch(batchCall, function(err, rawtxs) {
        if (err) {
          console.error(err);
          return setTimeout(showNewTransactions, 10000);
        }

        rawtxs.map(function (rawtx) {
          var tx = new bitcore.Transaction(rawtx.result);
          console.log('\n\n\n' + tx.id + ':', tx.toObject());
        });

        txids = ret.result;
        setTimeout(showNewTransactions, 2500);
      });
    });
  }

  showNewTransactions();
};
```