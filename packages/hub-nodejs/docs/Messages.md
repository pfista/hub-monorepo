# Messages

A Farcaster Message represents an action taken by a user.

Messages are atomic updates that add or remove content from the network. For example, a user can make a new cast by generating a `CastAdd` message and remove it with a `CastRemove` message. See the [protocol spec](https://github.com/farcasterxyz/protocol#3-delta-graph) for more information on how messages work.

Messages are protobufs which are converted by @farcaster/hub-nodejs converts into the Typescript types documented below. Each message is signed by a key pair that is provably controlled by the user. Some messages must be signed by the Ethereum address that controls the user's fid on-chain while other messages must be signed by an EdDSA key pair known as a Signer, which is authorized to act on behalf of the Ethereum address. The protocol specification defines the message types as:

| Message                   | Action                                                              |
| ------------------------- | ------------------------------------------------------------------- |
| SignerAdd                 | Add a new Ed25519 key pair that can sign messages for a user        |
| SignerRemove              | Remove an existing Ed25519 key pair and all messages signed by it   |
| UserDataAdd               | Update some metadata about a user (e.g. Display Name)               |
| CastAdd                   | Add a new Cast                                                      |
| CastRemove                | Remove an existing cast                                             |
| ReactionAdd               | Add a Reaction to an existing item (e.g. like a cast)               |
| ReactionRemove            | Remove an existing Reaction from an existing item                   |
| VerificationEthAddressAdd | Add a signed message verifying that a user owns an Ethereum address |
| VerificationRemove        | Remove a previously added verification message                      |

## Message\<Data>

A generic container which holds the contents of the message(`MessageData`) and metadata to authenticate it.

| Name              | Type                                  | Description                                       |
| :---------------- | :------------------------------------ | :------------------------------------------------ |
| `data`            | [`MessageData`](#messagedata)         | Contents of the message                           |
| `hash`            | `string`                              | Hash digest of data                               |
| `hashScheme`      | [`HashScheme`](#hashscheme)           | Hash scheme that produced the hash digest         |
| `signature`       | `string`                              | Signature of the hash digest                      |
| `signatureScheme` | [`SignatureScheme`](#signaturescheme) | Signature scheme that produced the signature      |
| `signer`          | `string`                              | Public key or address that produced the signature |

## MessageData\<Body,Type>

A generic container which holds common properties and properties specific to the message instance (`MessageType`, `MessageBody`).

| Name        | Type                                    | Description                                    |
| :---------- | :-------------------------------------- | ---------------------------------------------- |
| `body`      | [`MessageBody`](#messagebody)           | Properties specific to the MessageType         |
| `fid`       | `number`                                | Farcaster ID of the user producing the message |
| `network`   | [`FarcasterNetwork`](#farcasternetwork) | Farcaster network the message is intended for  |
| `timestamp` | `number`                                | Farcaster epoch timestamp in seconds           |
| `type`      | [`MessageType`](#messagetype)           | Type of Message contained in the body          |

## MessageBody

A composite type which holds properties unique to the type of the message (`MessageType`).

Some `MessageBody` types map to multiple `MessageTypes` since the only difference is the type of the operation. For instance, both ReactionBody maps to both ReactionAdd and ReactionRemove. The type value must be one of: [`SignerBody`](#signerbody) \| [`UserDataBody`](#userdatabody) \| [`CastAddBody`](#castaddbody) \| [`CastRemoveBody`](#castremovebody) \| [`ReactionBody`](#reactionbody) \| [`VerificationAddEthAddressBody`](#verificationaddethaddressbody) \| [`VerificationRemoveBody`](#verificationremovebody)

### SignerBody

Body of a `MESSAGE_TYPE_SIGNER_ADD` or `MESSAGE_TYPE_SIGNER_REMOVE`

| Name     | Type     | Description                        |
| :------- | :------- | :--------------------------------- |
| `signer` | `string` | Public key of the Ed25519 key pair |

### UserDataBody

Body of a `MESSAGE_TYPE_USER_DATA_ADD`

| Name    | Type                            |
| :------ | :------------------------------ |
| `type`  | [`UserDataType`](#userdatatype) |
| `value` | `string`                        |

### CastAddBody

Body of a `MESSAGE_TYPE_CAST_ADD`

| Name                 | Type                | Description                           |
| :------------------- | :------------------ | ------------------------------------- |
| `embeds?`            | `string[]`          | URLs to be embedded in the cast       |
| `mentions?`          | `number[]`          | Fids mentioned in the cast            |
| `mentionsPositions?` | `number[]`          | Parent cast of the cast               |
| `parent?`            | [`CastId`](#castid) | Text of the cast                      |
| `text`               | `string`            | Positions of the mentions in the text |

### CastRemoveBody

Body of a `MESSAGE_TYPE_CAST_REMOVE`

| Name         | Type     | Description                |
| :----------- | :------- | :------------------------- |
| `targetHash` | `string` | Hash of the cast to remove |

### ReactionBody

Body of `MESSAGE_TYPE_REACTION_ADD` or `MESSAGE_TYPE_REACTION_REMOVE`

| Name     | Type               |
| :------- | :----------------- |
| `target` | [`CastId`]()       |
| `type`   | [`ReactionType`]() |

### VerificationAddEthAddressBody

Body of `MESSAGE_TYPE_VERIFICATION_ADD_ETH_ADDRESS`

| Name           | Type     | Description                                                   |
| :------------- | :------- | ------------------------------------------------------------- |
| `address`      | `string` | Ethereum address being verified                               |
| `blockHash`    | `string` | Hash of the latest Ethereum block when the claim was produced |
| `ethSignature` | `string` | Signature of a valid [VerificationEthAddressClaim]() hash     |

### VerificationRemoveBody

Body of `MESSAGE_TYPE_VERIFICATION_REMOVE`

| Name      | Type     | Description                           |
| :-------- | :------- | ------------------------------------- |
| `address` | `string` | Address of the Verification to remove |

## Enumerations

### FarcasterNetwork

The Farcaster network that will accept the message.

| Name                      | Number | Description            |
| ------------------------- | ------ | ---------------------- |
| FARCASTER_NETWORK_NONE    | 0      |                        |
| FARCASTER_NETWORK_MAINNET | 1      | Public primary network |
| FARCASTER_NETWORK_TESTNET | 2      | Public test network    |
| FARCASTER_NETWORK_DEVNET  | 3      | Private test network   |

### HashScheme

The hashing scheme used to produce a digest of the MessageData.

| Name               | Number | Description                            |
| ------------------ | ------ | -------------------------------------- |
| HASH_SCHEME_NONE   | 0      |                                        |
| HASH_SCHEME_BLAKE3 | 1      | Default scheme for hashing MessageData |

### SignatureScheme

The signature scheme used to sign the Message hash.

| Name                     | Number | Description                          |
| ------------------------ | ------ | ------------------------------------ |
| SIGNATURE_SCHEME_NONE    | 0      |                                      |
| SIGNATURE_SCHEME_ED25519 | 1      | Ed25519 signature (default)          |
| SIGNATURE_SCHEME_EIP712  | 2      | ECDSA signature using EIP-712 scheme |

### MessageType

The type of the Farcaster Message, according to the protocol specification.

| Name                                      | Number | Description                                               |
| ----------------------------------------- | ------ | --------------------------------------------------------- |
| MESSAGE_TYPE_NONE                         | 0      | Invalid default value                                     |
| MESSAGE_TYPE_CAST_ADD                     | 1      | Add a new Cast                                            |
| MESSAGE_TYPE_CAST_REMOVE                  | 2      | Remove an existing Cast                                   |
| MESSAGE_TYPE_REACTION_ADD                 | 3      | Add a Reaction to a Cast                                  |
| MESSAGE_TYPE_REACTION_REMOVE              | 4      | Remove a Reaction from a Cast                             |
| MESSAGE_TYPE_VERIFICATION_ADD_ETH_ADDRESS | 7      | Add a Verification of an Ethereum Address                 |
| MESSAGE_TYPE_VERIFICATION_REMOVE          | 8      | Remove a Verification                                     |
| MESSAGE_TYPE_SIGNER_ADD                   | 9      | Add a new Ed25519 key pair that signs messages for a user |
| MESSAGE_TYPE_SIGNER_REMOVE                | 10     | Remove an Ed25519 key pair that signs messages for a user |
| MESSAGE_TYPE_USER_DATA_ADD                | 11     | Add metadata about a user                                 |

### ReactionType

The type of the Reaction contained in the message.

| Name                 | Number | Description                                  |
| -------------------- | ------ | -------------------------------------------- |
| REACTION_TYPE_NONE   | 0      | Invalid default value                        |
| REACTION_TYPE_LIKE   | 1      | Like the target cast                         |
| REACTION_TYPE_RECAST | 2      | Share target cast to the user&#39;s audience |

### UserDataType

The UserData property being changed by the message.

| Name                   | Number | Description                           |
| ---------------------- | ------ | ------------------------------------- |
| USER_DATA_TYPE_NONE    | 0      | Invalid default value                 |
| USER_DATA_TYPE_PFP     | 1      | Profile Picture for the user          |
| USER_DATA_TYPE_DISPLAY | 2      | Display Name for the user             |
| USER_DATA_TYPE_BIO     | 3      | Bio for the user                      |
| USER_DATA_TYPE_URL     | 5      | URL of the user                       |
| USER_DATA_TYPE_FNAME   | 6      | Preferred Farcaster Name for the user |

## Miscellaneous Types

### CastId

Unique identifier for a Cast that includes its author.

| Name   | Type     | Description                          |
| :----- | :------- | ------------------------------------ |
| `fid`  | `number` | Fid of the user who created the cast |
| `hash` | `string` | Hash of the cast                     |

### VerificationEthAddressClaim

An object that is hashed, signed and included in `VerificationAddEthAddressBody` to prove ownership of an Ethereum address.

| Name        | Type                                    | Description                                                   |
| :---------- | :-------------------------------------- | ------------------------------------------------------------- |
| `address`   | `string`                                | Ethereum address being verified                               |
| `blockHash` | `string`                                | Hash of the latest Ethereum block when the claim was produced |
| `fid`       | `BigNumber`                             | Fid of the user who claims to own the Ethereum address        |
| `network`   | [`FarcasterNetwork`](#farcasternetwork) | Farcaster Network on which the claim will be broadcast        |