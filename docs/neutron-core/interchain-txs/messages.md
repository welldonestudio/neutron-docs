# Messages

## MsgRegisterInterchainAccount

Attempts to register an interchain account by sending an IBC packet over an IBC connection.

```protobuf
message MsgRegisterInterchainAccount {
  option (gogoproto.equal) = false;
  option (gogoproto.goproto_getters) = false;

  string from_address = 1;
  string connection_id = 2 [ (gogoproto.moretags) = "yaml:\"connection_id\"" ];
  string interchain_account_id = 3 [ (gogoproto.moretags) = "yaml:\"interchain_account_id\"" ];
}
```

* `from_address` must be a smart contract address, otherwise the message will fail;
* `connection_id` must be the identifier of a valid IBC connection, otherwise the message will fail;
* `interchain_account_id` is used to generate the [owner](https://github.com/cosmos/ibc-go/blob/v3.1.1/modules/apps/27-interchain-accounts/controller/keeper/account.go#L17) parameter for ICA's `RegisterInterchainAccount()` call, which is later used for port identifier generation (see below).

<details>
  <summary>IBC ports naming / Interchain Account address derivation</summary>

If a contract with the address `neutron14hj2tavq8fpesdwxxcu44rty3hh90vhujrvcmstl4zr3txmfvw9s5c2epq` sends an `MsgRegisterInterchainAccount` with `interchain_account_id` set to `hub/1`, the generated ICA owner will look like `neutron14hj2tavq8fpesdwxxcu44rty3hh90vhujrvcmstl4zr3txmfvw9s5c2epq.hub/1`, and the IBC port generated by the ICA app will be equal to `icacontroller-neutron14hj2tavq8fpesdwxxcu44rty3hh90vhujrvcmstl4zr3txmfvw9s5c2epq.hub/1`.

ICA's remote address generation concatenates connection identifier and port identifier to use them as the derivation key for the new account:

```go
// GenerateAddress returns an sdk.AccAddress derived using the provided module account address and connection and port identifiers.
// The sdk.AccAddress returned is a sub-address of the module account, using the host chain connection ID and controller chain's port ID as the derivation key
func GenerateAddress(moduleAccAddr sdk.AccAddress, connectionID, portID string) sdk.AccAddress {
	return sdk.AccAddress(sdkaddress.Derive(moduleAccAddr, []byte(connectionID+portID)))
}

```
</details>

> **Note:** your contract needs to implement the `sudo()` entrypoint on order to successfully process the IBC events associated with this message. You can find an example in the [neutron-contracts](https://github.com/neutron-org/neutron-contracts/tree/main/contracts) repository. 

### Response

```protobuf
message MsgRegisterInterchainAccountResponse {}
```

### IBC Events

```go
type MessageOnChanOpenAck struct {
	OpenAck OpenAckDetails `json:"open_ack"`
}

type OpenAckDetails struct {
	PortID                string `json:"port_id"`
	ChannelID             string `json:"channel_id"`
	CounterpartyChannelId string `json:"counterparty_channel_id"`
	CounterpartyVersion   string `json:"counterparty_version"`
}
```

The data from an `OnChanOpenAck` event is passed to the contract using a [Sudo() call](https://github.com/CosmWasm/wasmd/blob/288609255ad92dfe5c54eae572fe7d6010e712eb/x/wasm/keeper/keeper.go#L453). You can have a look at an example handler implementation in the [neutron-contracts](https://github.com/neutron-org/neutron-contracts/tree/main/contracts) repository. 

> Note: you can find the interchain account address in the stored in the `CounterpartyVersion` field as part of [metadata](https://github.com/cosmos/ibc-go/blob/main/modules/apps/27-interchain-accounts/host/keeper/handshake.go#L78).

### State modifications

None.

## MsgSubmitTx

Attempts to execute a transaction on a remote chain.

```protobuf
message MsgSubmitTx {
  option (gogoproto.equal) = false;
  option (gogoproto.goproto_getters) = false;

  string from_address = 1;
  string interchain_account_id = 2;
  string connection_id = 3;
  repeated google.protobuf.Any msgs = 4;
  string memo = 5;
  uint64 timeout = 6;
}
```

* `from_address` must be a smart contract address, otherwise the message will fail;
* `interchain_account_id` is identical to `MsgRegisterInterchainAccount.interchain_account_id`;
* `connection_id` must be the identifier of a valid IBC connection, otherwise the message will fail;
* `memo` is the transaction [memo](https://docs.cosmos.network/master/core/transactions.html);
* `timeout` is a timeout in seconds after which the packet times out.

> **Note:** most networks reject memos longer than 256 bytes.

> **Note:** your contract needs to implement the `sudo()` entrypoint on order to successfully process the IBC events associated with this message. You can find an example in the [neutron-contracts](https://github.com/neutron-org/neutron-contracts/tree/main/contracts) repository.

### Response

```protobuf
message MsgSubmitTxResponse {}
```

### IBC Events

```go
type SudoMessageTimeout struct {
	Timeout struct {
		Request channeltypes.Packet `json:"request"`
	} `json:"timeout"`
}

type SudoMessageResponse struct {
	Response struct {
		Request channeltypes.Packet `json:"request"`
		Data    []byte              `json:"data"` // Message data
	} `json:"response"`
}

type SudoMessageError struct {
	Error struct {
		Request channeltypes.Packet `json:"request"`
		Details string              `json:"details"`
	} `json:"error"`
}
```

While trying to execute an interchain transaction, you can receive an IBC `Timeout` or an IBC `Acknowledgement`, and the latter can contain either a valid response or an error. These three types of transaction results are passed to the contract as distinct messages using a [Sudo() call](https://github.com/CosmWasm/wasmd/blob/288609255ad92dfe5c54eae572fe7d6010e712eb/x/wasm/keeper/keeper.go#L453). You can have a look at an example handler implementation in the [neutron-contracts](https://github.com/neutron-org/neutron-contracts/tree/main/contracts) repository.

You can more find info, recommendations and examples about how process acknowledgements [here](TODO_LINK).

### State modifications

None.