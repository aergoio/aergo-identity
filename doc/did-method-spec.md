# Aergo DID Method Specification

## Author

-   Aergo: <https://www.aergo.io/about#contact>

## Preface

The Aergo DID method specification conforms to the requirements specified in 
the [DID specification](https://w3c-ccg.github.io/did-spec/).

## Abstract

The described DID method allows any Aergo smart contract or key pair account to become a valid identity.
An identity needs no registration. In the case that key management or additional
attributes such as "service endpoints" are required, we deployed did registry smart contracts on:

-   Mainnet: `AmgUfj2ghDBRJsmpuSvBZq1Zp67kMc8QX72r1Eh7aS24ueADStPv`
-   Testnet: `AmhPkt8JXujkN6TWEJbynaXqnQUh1AfUckJEyDvTYoA3PyXXf7as`

Since each Aergo transaction must be funded, in order to update attributes, account balance must be greater than zero.

### Identity Ownership 
By default, each identity is controlled by itself. Each identity can only be controlled by a single 
address at any given time. By default, this is the address of the identity itself. The owner can 
replace themselves with any other Aergo address.

## Context Definition
Note, this DID method specification uses following types:
 * `Secp256k1VerificationKey2018`  ( same as used for Ethereum )
 * `Secp256k1SignatureAuthentication2018` ( same as used for Ethereum )
  
And following options for key values:
 * `aergoAddress`
 * `publicKeyHex`
 * `publicKeyBase64`
 * `publicKeyBase58`


## DID Method Name

The namestring that shall identify this DID method is: `aergo`

A DID that uses this method MUST begin with the following prefix: `did:aergo`. Per the DID specification, this string 
MUST be in lowercase. The remainder of the DID, after the prefix, is specified below.

## Method Specific Identifier

The method specific identifier is same as the following:

    aergo-did = "did:aergo:" specific-idstring
    specific-idstring = [ aergo-network-id ":" ] aergo-address
    aergo-network-id = "mainnet" / "testnet" 
    ethr-address = 52*Base58Check 

The Aergo address is case-sensitive.

Note, if no public Aergo network was specified, it is assumed that the DID is anchored
on the Aergo mainnet per default. This means the following DIDs will resolve to the same
DID Document:

    did:aergo:mainnet:AmgFiAupQBr7tx4CLkoV7uZhsMYDNjM5tsQUREKfdtwpGqsm3R9s
    did:aergo:AmgFiAupQBr7tx4CLkoV7uZhsMYDNjM5tsQUREKfdtwpGqsm3R9s

## CRUD Operation Definitions

### Create (Register)

In order to create a DID, an Aergo address, i.e., key pair, needs to be generated. At this point,
no interaction with the target Aergo network is required. 
The holder of the private key is the entity identified by the DID.

The minimal DID document for a an Aergo address, e.g., `AmgFiAupQBr7tx4CLkoV7uZhsMYDNjM5tsQUREKfdtwpGqsm3R9s` with no
transactions to the did registry looks like this:
```
  {
    '@context': 'https://w3id.org/did/v1',
    id: 'did:aergo:AmgFiAupQBr7tx4CLkoV7uZhsMYDNjM5tsQUREKfdtwpGqsm3R9s',
    publicKey: [{
       id: 'did:aergo:AmgFiAupQBr7tx4CLkoV7uZhsMYDNjM5tsQUREKfdtwpGqsm3R9s#owner',
       type: 'Secp256k1VerificationKey2018',
       owner: 'did:aergo:AmgFiAupQBr7tx4CLkoV7uZhsMYDNjM5tsQUREKfdtwpGqsm3R9s',
       aergoAddress: 'AmgFiAupQBr7tx4CLkoV7uZhsMYDNjM5tsQUREKfdtwpGqsm3R9s'}],
    authentication: [{
       type: 'Secp256k1SignatureAuthentication2018',
       publicKey: 'did:aergo:AmgFiAupQBr7tx4CLkoV7uZhsMYDNjM5tsQUREKfdtwpGqsm3R9s#owner'}]
  }
```

### Read (Resolve)

The DID document is built by using read only functions and contract events on the did registry.

Any value from the registry that returns an Aergo address will be added to the `publicKey` array of the DID 
document with type `Secp256k1VerificationKey2018` and an `aergoAddress` attribute containing the address.

#### Owner Address

Each identity always has an owner address. By default it is the same as the identity address, but check the
read only contract function `identityOwner(address identity)` on the deployed version of the did registry contract.

The identity owner will always have a `publicKey` with the id set as the DID with the fragment `#owner` appended.

An entry is also added to the `authentication` array of the DID document with type `Secp256k1SignatureAuthentication2018`.

#### Enumerating Contract Events to build the DID Document

The did registry contract publishes three types of events for each identity.

-   `DIDOwnerChanged`
-   `DIDDelegateChanged`
-   `DIDAttributeChanged`

If a change has ever been made for an identity the block number is stored in the changed mapping.

The latest event can be efficiently looked up by checking for one of the 3 above events at that exact block.

#### Delegate Keys

Delegate keys are Aergo addresses that can either be general signing keys or optionally also perform
authentication.

The only 2 `delegateTypes` that are currently published in the DID document are:

-   `veriKey` which adds a `Secp256k1VerificationKey2018` to the `publicKey` section of the DID document.
-   `sigAuth` which adds a `Secp256k1SignatureAuthentication2018` to the `publicKey` section of document. An entry
  is also added to the `authentication` section of the DID document.

Only events with a `validTo` in seconds greater or equal to the current time should be included in the DID document.

#### Public Keys

The name of the attribute added to did registry should follow this format:

`did/pub/(Secp256k1)/(veriKey|sigAuth)/(hex|base64|base58)`

##### Hex encoded Secp256k1 Verification Key

A `DIDAttributeChanged` event for the identity `AmgFiAupQBr7tx4CLkoV7uZhsMYDNjM5tsQUREKfdtwpGqsm3R9s` with the name
`did/pub/Secp256k1/veriKey/hex` and the value of `416d67466941757051427237747834434c6b6f5637755a68734d59444e6a4d357473515552454b66647477704771736d33523973`
generates a public key entry like the following:
```
  {
    id: "did:aergo:AmgFiAupQBr7tx4CLkoV7uZhsMYDNjM5tsQUREKfdtwpGqsm3R9s#delegate-1",
    type: "Secp256k1VerificationKey2018",
    owner: "did:aergo:AmgFiAupQBr7tx4CLkoV7uZhsMYDNjM5tsQUREKfdtwpGqsm3R9s",
    publicKeyHex: '416d67466941757051427237747834434c6b6f5637755a68734d59444e6a4d357473515552454b66647477704771736d33523973'
  }
```

##### Base64 encoded Secp256k1 Verification Key

A `DIDAttributeChanged` event for the identity `AmgFiAupQBr7tx4CLkoV7uZhsMYDNjM5tsQUREKfdtwpGqsm3R9s` with the name 
`did/pub/Secp256k1/veriKey/base64` and the value of `QW1nRmlBdXBRQnI3dHg0Q0xrb1Y3dVpoc01ZRE5qTTV0c1FVUkVLZmR0d3BHcXNtM1I5cw==`
generates a public key entry like this:
```
  {
    id: "did:aergo:AmgFiAupQBr7tx4CLkoV7uZhsMYDNjM5tsQUREKfdtwpGqsm3R9s#delegate-1",
    type: "Secp256k1VerificationKey2018",
    owner: "did:aergo:AmgFiAupQBr7tx4CLkoV7uZhsMYDNjM5tsQUREKfdtwpGqsm3R9s",
    publicKeyBase64: "QW1nRmlBdXBRQnI3dHg0Q0xrb1Y3dVpoc01ZRE5qTTV0c1FVUkVLZmR0d3BHcXNtM1I5cw=="
  }
```

##### Base58 encoded Secp256k1 Verification Key

A `DIDAttributeChanged` event for the identity `AmgFiAupQBr7tx4CLkoV7uZhsMYDNjM5tsQUREKfdtwpGqsm3R9s` with the name 
`did/pub/Secp256k1/veriKey/base64` and the value of `0xb97c30de767f084ce3080168ee293053ba33b235d7116a3263d29f1450936b71`
generates a public key entry like this:
```
  {
    id: "did:aergo:AmgFiAupQBr7tx4CLkoV7uZhsMYDNjM5tsQUREKfdtwpGqsm3R9s#delegate-1",
    type: "Secp256k1VerificationKey2018",
    owner: "did:aergo:AmgFiAupQBr7tx4CLkoV7uZhsMYDNjM5tsQUREKfdtwpGqsm3R9s",
    publicKeyBase58: "AmgFiAupQBr7tx4CLkoV7uZhsMYDNjM5tsQUREKfdtwpGqsm3R9s"
  }
```

#### Service Endpoints

The name of the attribute should follow this format:

`did/svc/[ServiceName]`

A `DIDAttributeChanged` event for the identity `AmgFiAupQBr7tx4CLkoV7uZhsMYDNjM5tsQUREKfdtwpGqsm3R9s` with the name 
`did/svc/PowService` and value of the URL `https://pow.aergo.io` generates a service endpoint entry like the following:
```
  {
    type: "PowService",
    serviceEndpoint: "https://pow.aergo.io"
  }
```

### Update / Delete (Revoke)

The DID Document may be updated by invoking changeOwner, setAttribute, revokeAttribute, addDelegate, revokeDelegate on did registry smart contract functions.

These functions will trigger the respective Aergo events which are used to build the DID Document.


## Security Considerations

Aergo Identity is based on Aergo blockchain.
Aergo blockchain provides security against following risks:
* Replay Attacks
* DoS Attacks
* Man-in-the-middle Attacks
* Non-repudiation
* Byzantine fault 

## Privacy Considerations
Aergo Identity does not store any personal data.
DID document is stored in Aergo Identity which contains did, public keys, and service endpoints.


## References

 **[1]** Aergo: <https://www.aergo.io>
 
 **[2]** Aergo Project Github: <https://github.com/aergoio>
 
 **[3]** W3C Decentralized Identifiers (DIDs): <https://w3c-ccg.github.io/did-spec/>

