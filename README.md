Version : 0.1 (alpha) - This Document is subject to future changes and will most likely include even API-breaking changes.

# Introduction

The USN is a decentralized network based on blockchain technology. Every Device has exactly one resolveable set of rules and conditions, which are defined as part of a smart contract in the ethereum blockchain. 

![Alt text](https://download.slock.it/private/AccessControl.svg)

This Contract is used to enforce them by access control acting as Gateway and enables control over a device. This software is connected with its contract on the chain and verify any requests by executing the incoming messages and state-changes in the contract. 

The Access Control manages and holds the state and verifies the rules by calling functions in the smart contract. Even though the smart contract is also able to hold a state, if users send transaction to it. This state can only be enforeced by the access control and will therefore be synced.


## Identification and ENS

Each Object on this a network needs a unique Identifier. This ID needs to be resolveable to a smart contract. 

In order to make them human readable, all identifiers are based the [ENS](https://ens.domains/).

Because one smartcontract is able to hold rules for multiple devices, the identifiers uses a similiar approach like email-addresses: 


`<deviceName>[#<deviceCounter>]@<contractName>`

* **contractName** - the domainname or contractName will be resolved by using the ENS.   
```js
ResolverContract.at(
  ens.resolver(namehash('myname.eth'))
).addr(namehash('myname.eth')) 
```    
As a result the address will point to the contract holding the conditions and rules.

* **deviceName** - the deviceId will be generated out of the first 24 bytes of the sha256-hash of the deviceName. If none is given, it will use 0x00.
```js
deviceId = sha256( deviceName ).substr( 0, 50 ) + 
           deviceCounter.toString( 16 ).padStart(16,'0')
```


* **deviceCounter** - this optional number will be added to the deviceId and uses the last 8 bytes of it.

Out of the deviceName and deviceCounter we build the deviceId, which will be used inside the contract to identify a device.

## USNResolver

The ENS uses contracts as Resolvers, which are able to map nodeids to addresses. The USN offers such a resolver, which is used to resolve not only domain-names, but also additional library-contracts, using the special lib-function.

For every domain-name ( hased to the nodeid), 2 actions are needed:
- the onwer needs to set himself as such in the ENS-System, which should be done using the `USNRegistrar`

  ```js
  usnRegistrar.registerObject(_subnode,  newContract);
  ```
  In order to make sure the good domain-names are not simply taken by the first calling this, only domain-names, whcih fullfill this condition, can be registered freely:
  ```js
  require (uint(_subnode) < difficulty );
  ```
  This means any free domain-name generate a sha3-hash lower than a given difficulty. For any other domain, special contracts will manage the auction.

- the owner points this nodeid to a contract address ( using the USN-Resolver)

  ```js
  usnResolver.setAddress(_subnode,  newContract);
  ```

In order to avoid fraud, the USNResolver uses a whitelist for verified contract-codes. So even though  anybody can deploy new contracts, they must use a verified contract-code.

## Multichain-support

Even though the ENS of the public chain is used to register nodeids, the resolver is able to define a IPFS-Hash for a describtion of a different chain this contract was deployed. Only Requirements for the chain:

1. the chain must be publicy available. ( in order to interact with it)
2. the chain must be based on the EVM.
3. at least one remote-client given access to the chain must be specified.
4. the chain must be approved and put on a whitelist.

If these criteria are fullfilled, the USN is able to use a private or consortium-chain and connect to the contract ther

## DeviceContract

Before a device itself can be registered, a contract holding the rules needs to be deployed.
The minmal required Interface this contract needs to implement is described in the EIP-165:

```js
  /// checks, if the contract supports a certain feature by implementing the interface
  /// @param _interfaceID the hash or identifier of the interface
  function supportsInterface(bytes4 interfaceID) public constant returns (bool);
```

This gives max flexibility for even new features on the future.
The USN defines a base-interface : BasicRentRegistry, which can be used as minimal function needed for renting.

## Interfaces

There are 2 kind of features, which can be used for a contract:

1. Internal features : These implement interfaces offering or overriding functions directly in the same contract.
2. External or remote features : Library-Contracts, which implement this function and are called by the contract. Those are supported, if the RemoteFeature-Interface is implemented.

| Interface  | Hash  | Description  |
|---|---|---|
| [AccessSupport](contracts/features/AccessSupport.sol)  | 0xd1559fbc | basic function to support AccessControl to a device |
| [BlacklistSupport](contracts/features/BlacklistSupport.sol) | 0xa3fb4905 | manages a blacklist for a device |
| [DependendAccessSupport](contracts/features/DependendAccessSupport.sol)  | 0x83b8c330 | Enables access to a device depending on access tp a a second or group (packaging access) |
| [DepositSupport](contracts/features/DepositSupport.sol) | 0xd8533f34 | Allows owner to require a deposit, which can be withdrawn later with a optional dispute management |
| [DiscountSupport](contracts/features/DiscountSupport.sol)  | 0xbe4adbf9 | Enables Rules in order to set discounts on the price dependend on time or users |
| [GroupedSupport](contracts/features/GroupedSupport.sol) | 0xd63efd21 | Allows One set of rules for a given number of devices |
| [IdentitySupport](contracts/features/IdentitySupport.sol)  | 0xf9378cb2 | Allows or identifyable users to rent the device |
| [MetaSupport](contracts/features/MetaSupport.sol)  | 0xfb4ca7b3 | enables additional metadata based on IPFS/Swarm or other resources |
| [MultisigSupport](contracts/features/MultisigSupport.sol)  | 0x7a26de3a | resolves Multigs to give access to each key-holder |
| [OffChainSupport](contracts/features/OffChainSupport.sol)  | 0x3d8a1e6b | offers function to verify access and rent for a state managed outside of the contract |
| [OwnerSupport](contracts/features/OwnerSupport.sol)  | 0xb4762fab | assigns a owner per device |
| [RemoteFeatureSupport](contracts/features/RemoteFeatureSupport.sol)  | 0xaf568c9b | supports adding or removing features, which are implemented in its own contract, to a existing contract |
| [RentForSupport](contracts/features/RentForSupport.sol)  | 0xbf89d2ac | allows users to seperate the payer from the user and also book in the future |
| [RentingSupport](contracts/features/RentingSupport.sol)  | 0xb2a80dea | allows basic renting of a device (rent/return) |
| [StateChannelSupport](contracts/features/StateChannelSupport.sol)  | 0x5fa3718b | support for StateChannels |
| [TimeRangeSupport](contracts/features/TimeRangeSupport.sol)  | 0x51533fb7 | defines a minimal and maximal renting time |
| [TokenSupport](contracts/features/TokenSupport.sol)  | 0xe1cae6fe | allows the owner to accept multiple tokens and manages the pricecalculation through exchanges |
| [VerifiedDeviceSupport](contracts/features/VerifiedDeviceSupport.sol)  | 0xa3951baf | Registry for devices and their public address in order to verify the author or messages from them |
| [WeekCalendarSupport](contracts/features/WeekCalendarSupport.sol)  | 0xa7f26b04 | Allows the owner set time within a Week where the device should accessable |
| [WhitelistSupport](contracts/features/WhitelistSupport.sol)  | 0xd63efd09 | Manage direct access to a device based on a whitelist |


## Discovery

The USNResolver will trigger events for all registered or changed contracts. They will follow the ENS-Resolver-Standard:

```js
  event AddrChanged(bytes32 indexed node, address a);
```

Bases on these events a list of all available contracts can be created.

In addition all contracts supporting the `BasicRentRegistry`-interface will trigger events using a fixed constant as second topics of the event. This way all renting related events (state changes) may be read by using the Bloom-Filter.

```js
// returns all renting events form all contracts.
web3.eth.getPastLogs({ fromBlock, toBlock, topics: [null, '0xa2f7689fc12ea917d9029117d32b9fdef2a53462c853462ca86b71b97dd84af6'] })
```

An addition to the device-states each device may provide metadata, which can be fetched and used to discover. These metadata will also hold GPS-Location and additional descriptions.


## RentingSupport

For Renting there is a basic interface, which can be checked using `supportsInterface`. The minimal needed interface to implemet would look like this:

```js
/// @title defines a contract able to rent and return
contract RentingSupport {
 
    /// interface-id for supportInterface-check
    bytes4 public constant ID = 0xb2a80dea;

    /// created after the rent-function was executed
    event LogRented(bytes32 indexed fixFilter, bytes32 indexed id, address controller, uint64 rentedFrom, uint64 rentedUntil, bool noReturn, uint128 amount, address token);
    /// whenever the device was returned.
    event LogReturned(bytes32 indexed fixFilter, bytes32 indexed id, address controller, uint64 rentedFrom, uint64 rentedUntil, uint128 paidBack);

    /// rents a device, which means it will change the state by setting the sender as controller.
    /// @param id the deviceid
    /// @param secondsToRent the time of rental in seconds
    /// @param token the address of the token to pay (See token-addreesses for details)
    function rent(bytes32 id, uint32 secondsToRent, address token) public payable;

    /// returns the Object or Device and also the funds are returned in case he returns it earlier than rentedUntil.
    /// @param id the deviceid
    function returnObject(bytes32 id) public;

    /// the price for rentinng the device
    /// @param id the deviceid
    /// @param user the user because prices may depend on the user (whitelisted or discount)
    /// @param secondsToRent the time of rental in seconds
    /// @param token the address of the token to pay (See token-addreesses for details)
    function price(bytes32 id, address user, uint32 secondsToRent, address token) public constant returns (uint128);

    /// a list of supported tokens for the given device
    /// @param id the deviceid
    function supportedTokens(bytes32 id) public constant returns (address[] memory addresses);

    /// the receiver of the token, which may be different than the owner or even a Bitcoin-address. For fiat this may be hash of payment-data.
    /// @param id the deviceid
    /// @param token the paid token
    function tokenReceiver(bytes32 id, address token) public constant returns (bytes32);

    /// returns the current renting state
    /// @param user the user on which this may depend.
    /// @param id the deviceid
    function getRentingState(bytes32 id, address user) public constant returns (bool rentable, bool free, bool open, address controller, uint64 rentedUntil, uint64 rentedFrom);

}
```

### Renting a device

In order to rent this renting function is used:
```js
  function rent(bytes32 id, uint32 secondsToRent, address token) public payable;
```
The payment needed can be calculated before by using the `price()` function. If the payment is done in ether the transaction-value needs to have the correct amount. In case of any other ERC20-Token, the amount needs to be apporved before calling this function.

In case of a required deposit, the amount must include the deposit as well.

### Tokens

The USN will support any valid ERC20-Token, but the Owner of a device may set the tokens he accepts.

The function `supportedTokens` will always return a list of addresses, which are accepted by the owner. If the user wants to pay with any other token, exchanges need to convert first.

```js
  /// a list of supported tokens for the given device
  /// @param id the deviceid
  function supportedTokens(bytes32 id) public constant returns (address[] memory addresses);
```

Each address points to a ERC20-Token-contract. For addresses with a value lower than `0xFFFF` a special handling is reserved:

| Address  | Symbol  | Description  |
|---|---|---|
| `0x...0000` | *ETH* | represents ether all payments can be done directly in ether (which is the default) |
| `0x...0001` | *ETC* | Ether Classic |
| `0x...0002` | *BTC* | Bitcoin | 
| `0x...0003` | *BCC* | Bitcoin Cash |
| `0x...0004` | *EUR* | Fiat EURO (through a special FIat-Service) |
| `0x...0005` | *USD* | Fiat US Dollar (through a special Fiat-Service) |
| <`0x...FFFF` | ... | any value up to `0xFFFF`will have special coins and handler defined. |
| \>`0x...FFFF` | ... | these addresses will be handled as ERC20-Token. |

In case the contract is deployed on a non public chain, the token address will point to a ERC20-contract within this private chain. But this would mean, the user needs to pay with a token or even ether in the target chain. In the future exchange-services may handle transaction crossing chains or even use polkadot.

# Messages

Accessing a physical device is always done by sending a message to its AccessControl. The AccessControl will then verify the message and execute it. 

There are different TransportLayer to deliver this message to a AccessControl. 
- Whisper 

  as targetPeer of the whisper-message the USN uses the public key as published in the VerifiedDevice-Interface:

  ```js
  web3.ssh.post({ 
    targetPeer : await verifiedDevice.devicePubKey(deviceId),
    payload : toHex(message),
    sig: usersKey,
    topic: '0xffaadd11'
  })
  ``` 

- usn-hub

   The usn-hub keeps connections open for devices using websockets. This enables connections even through a firewall.
   Every Message send will then be forwarded through this hub.

- Other protocols may be supported in the future

In general there are 3 different kind of messages:

## State-Message

This message changes the state of a device. For Renting it usually means, a different user is now able to control the device. The message will have the following structure:

| Field | Type | Description |
|-------|------|-------------|
| **url** | `string` | the device identifier |
| **rentedFrom** | `integer` | start time of rental. starting from this time the `controller` should be set.  The value is a unix-timestamp in seconds since 1970. |
| **rentedUntil** | `integer` | end time of rental. the renting should only be set until this time.  The value is a unix-timestamp in seconds since 1970. |
| **controller** | `address` | public ethereum address of the user controlling the device. |
| **verification** | `object` | Verification data. The data holds a field `paymentType`, which defines the verifier used to validate this data. The `data`-field will then hold a JSON-Structure, which depend on the type. The veriffier should be able to read this structure and validate by asking the contract. This Verification must hold a proof of the required transaction (BTC/ETH-Transaction, Fiat-Payment, Signed State-Channel-Message,.. )|
| **msgType** | `string` | specific action. Currently only `rent`and `return`are valid values. |

The Verification of a state-message depending on the supported Verfifiers. This enables it to be open for any kind of blockchain or even fiat-payments as long as the device supports a verifier and the message is able to provide the required verification-data.  

For a Event on the Eth-Blockchain, the verification-data will simply hold the Event+Transaction-data. The Verifier can then simply verifiy the existence of this transaction by calling `web3.eth.getTransaction(txHash)` on its own client or a trusted remote client.



## ActionMessage

Action messages are send to a device requireing a physical change of state. They need to be signed by the user requesting the action.

| Field | Type | Description |
|-------|------|-------------|
| **url** | `string` | the device identifier |
| **msgId** | `number` | a identifier of the message in order to ensure action-messages are only executed once.|
| **msgType** | `string` | constant: `action` |
| **timestamp** | `integer` | Unix format timestamp (in seconds) of the message. This is needed in order to make sure only current messages are accepted. It must fulfill this condition: ` Math.abs ( timestamp - Date.now()/1000 ) < 10 `   |
| **action** | `string` | Abstract physical action. I.E. open, lock, detonate.   |
| **metadata** | `object` | Object containing data that the physical device expects to trigger change of state.|
| **signature** | `object` | Signature of the message. This object contains at least the require fields `messageHash`, `r`, `s`, `v` |

In order to verify a Action Message the hash is build as:

```js
messageData = message.url + message.timestamp + message.action + JSON.stringify(message.metadata)
````

The signature itself is valid if 

```js
message.signature.messageHash === sha3(messageData) 
&& web3.eth.accounts.recover(message.signature) === currentController
```


## Read-Message

A Read-Message will simply report the current State (both physical and received state) back to the user.
Each Message should be signed with the private key of the device, where the public key i published in the VerifiedDevice-Contract.

| Field | Type | Description |
|-------|------|-------------|
| **url** | `string` | the device identifier |
| **msgId** | `number` | a identifier of the message in order to ensure action-messages are only executed once.|
| **msgType** | `string` | constant: `read` |
| **timestamp** | `integer` | Unix format timestamp (in seconds) of the message. This is needed in order to make sure only current messages are accepted. It must fulfill this condition: ` Math.abs ( timestamp - Date.now()/1000 ) < 10 `   |
| **signature** | `object` | Signature of the message. This object contains at least the require fields `messageHash`, `r`, `s`, `v` |

The signature is optional, but signing a read message enables the user to receive more detailed information, if the user is the current controller. For example if the user is controlling a drone, only the controlling user, will receive the gps-location or other relevant data. But anyone would receive the renting-state.
