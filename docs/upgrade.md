# Upgrade Notes

## From Bitcore 3.0.0 to 4.0.0

`sumcore-node@2.1.1` to `sumcore-node@3.0.0`

This major upgrade includes changes to indexes, API methods and services. Please review below details before upgrading.

### Indexes

Indexes include *more information* and are now also *faster*. Because of this a **reindex will be necessary** when upgrading as the address and database indexes are now a part of bitcoind with three new `sumcoin.conf` options:
- `-addressindex`
- `-timestampindex`
- `-spentindex`

To start reindexing add `reindex=1` during the **first startup only**.

### Configuration Options

- The `sumcoin.conf` file in will need to be updated to include additional indexes *(see below)*.
- The `datadir` option is now a part of `bitcoind` spawn configuration, and there is a new option to connect to multiple bitcoind processes (Please see [Bitcoin Service Docs](services/bitcoind.md) for more details). The services `db` and `address` are now a part of the `bitcoind` service. Here is how to update `sumcore-node.json` configuration options:

**Before**:
```json
{
  "datadir": "/home/<username>/.sumcoin",
  "network": "livenet",
  "port": 3001,
  "services": [
    "address",
    "litecoind",
    "db",
    "web"
  ]
}
```

**After**:
```json
{
  "network": "livenet",
  "port": 3001,
  "services": [
    "litecoind",
    "web"
  ],
  "servicesConfig": {
    "sumcoin": {
      "spawn": {
        "datadir": "/home/<username>/.sumcoin",
        "exec": "/home/<username>/sumcore-node/bin/litecoind"
      }
    }
  }
}
```

It will also be necessary to update `sumcoin.conf` settings, to include these fields:
```
server=1
whitelist=127.0.0.1
txindex=1
addressindex=1
timestampindex=1
spentindex=1
zmqpubrawtx=tcp://127.0.0.1:<port>
zmqpubhashblock=tcp://127.0.0.1:<port>
rpcallowip=127.0.0.1
rpcuser=<user>
rpcpassword=<password>
```

**Important**: Once changes have been made you'll also need to add the `reindex=1` option **only for the first startup** to regenerate the indexes. Once this is complete you should be able to remove the `sumcore-node.db` directory with the old indexes.

### API and Service Changes
- Many API methods that were a part of the `db` and `address` services are now a part of the `bitcoind` service. Please see [Bitcoin Service Docs](services/bitcoind.md) for more details.
- The `db` and `address` services are deprecated, most of the functionality still exists. Any services that were extending indexes with the `db` service, will need to manage chain state itself, or build the indexes within `bitcoind`.
