# Queries

## `config`

```json
{
  "config": {}
}
```

Returns info about contract's configuration

## `merkle_root`

```json
{
  "merkle_root": {}
}
```

Returns merkle root of the Airdrop contract.

## `is_claimed`

```json
{
  "is_claimed": {
    "address": "neutron..."
  }
}
```

Returns true if a given address already claimed the airdrop.

## `total_claimed`

```json
{
  "total_claimed": {}
}
```

Returns total amount of already claimed tokens by users.

## `is_paused`

```json
{
  "is_paused": {}
}
```

Returns true of the contract is on pause.