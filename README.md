# DebitMyData's mining pools

The pools that mine DMD (the DebitMyData coin), for DebitMyData's sites:

- **`pools-v2.json`**: who found each block. The mempool explorer
  ([mempool.debitmydata.io](https://mempool.debitmydata.io)), DMD Stats
  ([stats.debitmydata.io](https://stats.debitmydata.io)), the analytics site
  ([analytics.debitmydata.io](https://analytics.debitmydata.io)) and DMD Mining
  ([mining.debitmydata.io](https://mining.debitmydata.io)) read it. Its format is mempool's
  (below), with each pool's `country`.
- **`pools-info.json`**: what DMD Mining shows about each pool besides: its API, how to connect,
  its fees. Only DMD Mining reads it, so a mistake there never changes who found a block.

The pools' logos are in [mining-pool-logos](https://github.com/DebitMyData-PoW/mining-pool-logos),
named as the pool's slug: its name's lowercase letters and digits (`Coin-Miners`: `coinminers.svg`).

Every change and every pull request is checked automatically (the **JSON check** on GitHub):
both files must be valid JSON, each pool needs its own `id` and name with its `link`, `tags` and
`addresses`, and each entry in `pools-info.json` must be for a pool in `pools-v2.json`. A green
tick on a pull request means it's safe to merge.

## DMD Mining: `pools-info.json`

One entry per pool, by its `id` in `pools-v2.json`. Every field but `id` is optional: a pool
without an entry still shows, with its blocks from the chain.

| Field | |
| :-- | :-- |
| `id` | The pool's `id` in `pools-v2.json` |
| `title` | The name to show, when it isn't the one in `pools-v2.json` (`Official`: `DebitMyData pool`) |
| `website` | The page to send miners to (else the `link` in `pools-v2.json`) |
| `software` | `MiningCore`, `YIIMP`, `NOMP` or `Custom` |
| `api` | The pool's API: for MiningCore, its address up to `/api` (`https://cyberpool.pro/api`), for YIIMP its `/api` |
| `api_coin` | YIIMP: the coin's symbol in its API (`DMD`) |
| `hosts` | The stratum servers' addresses, without ports: the ports come from the API (MiningCore) |
| `username`, `account` | What miners put as the username; `true` when the pool needs an account first |
| `modes` | One per way to mine there (shared, solo), below |
| `notes` | A sentence for its page |

Each mode: `label` (`PPLNS`, `Solo`…), `api_id` (MiningCore: the pool's id in its API, such as
`dmd1`), `scheme` (`PPLNS`, `PPS`, `PROP`, `SOLO`…), `fee` (percent), `min_payout` (DMD),
`stratum` (`host:port` addresses, when the API doesn't give the ports), `password` and `note`.
A MiningCore pool's API gives the scheme, fee, minimum payout and ports itself: what you write
here only fills in what it doesn't.

DMD Mining reads both files and the logos from GitHub every hour: a pool added here shows there
within the hour.

---

# Bitcoin Mining Pools

Mining pools definition used on https://mempool.space/mining/pools

# Contributing

Contributions welcome. All changes must be applied in `pools-v2.json` file.

## Adding a new mining pool

Regardless of the choosen method, we recommend adding a appropriate slug to each
new mining pool you add to `pools-v2.json`. The slug will be used as a unique tag for
the mining pool, for example in the public facing urls like https://mempool.space/mining/pool/foundryusa (here `foundryusa` is the slug).

You can specify mining pool slugs in the `slugs` object in `pools-v2.json`. If you
don't specify one, we will automatically generate one [as such](https://github.com/mempool/mempool/blob/02820b0e6836c4202c2e346195e8aace357e3483/backend/src/api/pools-parser.ts#L106-L110).

```javascript
if (slug === undefined) {
  // Only keep alphanumerical
  slug = poolNames[i].replace(/[^a-z0-9]/gi, '').toLowerCase();
  logger.warn(`No slug found for '${poolNames[i]}', generating it => '${slug}'`);
}
```

### Add a new mining pool by `coinbase_tags`

You can add a new mining pool by specifying the coinbase tag they're using in
the coinbase transaction.

To add a new pool, you must add a new JSON object in the `coinbase_tags` object.
Note that you can add multiple tags for the same mining pool, but you *must* use
the exact same values for `name` and `link` in each new entry.
For example:

```json
"Foundry USA Pool" : {
  "name" : "Foundry USA",
  "link" : "https://foundrydigital.com/"
},
"Foundry USA Pool another tag" : {
  "name" : "Foundry USA",
  "link" : "https://foundrydigital.com/"
},
```

Each coinbase tag will be use as a regex to match blocks with their mining pool.
This is how we use it in mempool application. You can see the code [here](https://github.com/mempool/mempool/blob/02820b0e6836c4202c2e346195e8aace357e3483/backend/src/api/blocks.ts#L238-L246).
```javascript
const regexes: string[] = JSON.parse(pools[i].regexes);
for (let y = 0; y < regexes.length; ++y) {
  const regex = new RegExp(regexes[y], 'i');
  const match = asciiScriptSig.match(regex);
  if (match !== null) {
    return pools[i];
  }
}
```

### Add a new mining pool by `payout_addresses`

You can add a new mining pool by specifying the receiving address they're using in
the coinbase transaction to receive the miner reward.

To add a new pool, you must add a new JSON object in the `payout_addresses` object.
Note that you can add multiple addresses for the same mining pool, but you *must* use
the exact same values for `name` and `link` in each new entry.
For example:

```json
"1FFxkVijzvUPUeHgkFjBk2Qw8j3wQY2cDw" : {
    "name" : "Foundry USA",
    "link" : "https://foundrydigital.com/"
},
"12KKDt4Mj7N5UAkQMN7LtPZMayenXHa8KL" : {
    "name" : "Foundry USA",
    "link" : "https://foundrydigital.com/"
},
```

Each address will be use to match blocks with their mining pool by matching the
coinbase transaction output address.
This is how we use it in mempool application. You can see the code [here](https://github.com/mempool/mempool/blob/02820b0e6836c4202c2e346195e8aace357e3483/backend/src/api/blocks.ts#L230-L236).
```javascript
const address = txMinerInfo.vout[0].scriptpubkey_address;
for (let i = 0; i < pools.length; ++i) {
  if (address !== undefined) {
    const addresses: string[] = JSON.parse(pools[i].addresses);
    if (addresses.indexOf(address) !== -1) {
      return pools[i];
    }
  }
```

## Change an existing mining pool metadata

You can also change an existing mining pool's name, link and slug. In order to
do so properly, you must update all existing entry in the `pools-v2.json` file.

For example, if you'd like to rename `Foundry USA` to `Foundry Pool`, you must replace
all occurences of the old string with the new one in `pools-v2.json` file, with no
exception, otherwise you'll end with two mining pools. The samme idea applies if
you want to change the link or the slug.

For example, to rename `Foundry USA` to `Foundry Pool` you'd need to update the
following (using today's `pools-v2.json` as reference):

```json
// Original
"Foundry USA Pool" : {
    "name" : "Foundry USA",
    "link" : "https://foundrydigital.com/"
},
  "/2cDw/" : {
    "name" : "Foundry USA",
    "link" : "https://foundrydigital.com/"
},
// Renamed
"Foundry USA Pool" : {
    "name" : "Foundry Pool",
    "link" : "https://foundrydigital.com/"
},
  "/2cDw/" : {
    "name" : "Foundry Pool",
    "link" : "https://foundrydigital.com/"
},
```
```json
// Original
"1FFxkVijzvUPUeHgkFjBk2Qw8j3wQY2cDw" : {
    "name" : "Foundry USA",
    "link" : "https://foundrydigital.com/"
},
"12KKDt4Mj7N5UAkQMN7LtPZMayenXHa8KL" : {
    "name" : "Foundry USA",
    "link" : "https://foundrydigital.com/"
},
// Renamed
"1FFxkVijzvUPUeHgkFjBk2Qw8j3wQY2cDw" : {
    "name" : "Foundry Pool",
    "link" : "https://foundrydigital.com/"
},
"12KKDt4Mj7N5UAkQMN7LtPZMayenXHa8KL" : {
    "name" : "Foundry Pool",
    "link" : "https://foundrydigital.com/"
},
```

```json
// Original
"Foundry USA": "foundryusa",
// Renamed - Be aware, this will also change the mining pool page link from
mempool.space/mining/pool/foundryusa to mempool.space/mining/pool/foundrypool
"Foundry Pool": "foundrypool",
```

## Block re-indexing

When a mining pool's coinbase tag or addresses is updated in `pools.jon`,
mempool can automatically re-index the appropriate blocks in order to re-assign
them to the correct mining pool.
"Appropriate" blocks here concern all blocks which are not yet assigned to a
mining pool (`unknown` pool), from block 130635 (first known mining pool block)
as well as all blocks from the update mining pool.
You can find the re-indexing logic [here](https://github.com/mempool/mempool/blob/02820b0e6836c4202c2e346195e8aace357e3483/backend/src/api/pools-parser.ts#L224-L249)

You can enable/disable this behavior using by setting the following backend
configuration variable:
```
{
  "MEMPOOL": {
    "AUTOMATIC_BLOCK_REINDEXING": false
  }
}
```

If you set it to false, no re-indexing will happen automatically, but this also
means that you will need to delete blocks manually from your database. Upon
restarting your backend, missing indexed blocks are always be re-indexed using
the latest mining pool data.

## Mining pool definition

When the mempool backend starts, we automatically fetch the latest `pools-v2.json`
version from github. By default the url points to https://github.com/mempool/mining-pools/blob/master/pools-v2.json but you can configure it to points to another repo by setting
the following backend variables:

```
{
  "MEMPOOL": {
    'POOLS_JSON_URL': 'https://raw.githubusercontent.com/mempool/mining-pools/master/pools-v2.json',
    'POOLS_JSON_TREE_URL': 'https://api.github.com/repos/mempool/mining-pools/git/trees/master'
  }
}
```