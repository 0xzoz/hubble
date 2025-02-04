# Builders

Factories that construct and sign Farcaster Messages which can be broadcast to Hubs.

## Prerequisites

Before you can build messages, you'll need construct the following objects:

- An `ed25519Signer`, used to sign most messages.
- A `eip712Signer`, used to sign some messages.
- A `dataOptions`, which contains message metadata.

### Ed25519Signer

A Ed25519Signer is an EdDSA key pair which is necessary for signing most messages on behalf of an fid. This example below shows how to construct a new `Ed25519Signer` using the [@noble](https://paulmillr.com/noble/) library :

```typescript
import { Ed25519Signer } from '@farcaster/js';
import * as ed from '@noble/ed25519';

const privateKey = ed.utils.randomPrivateKey(); // Applications must store this key securely.
const privateKeyHex = ed.utils.bytesToHex(privateKey);

const ed25519Signer = Ed25519Signer.fromPrivateKey(privateKey)._unsafeUnwrap();
```

### Eip712Signer

An Eip712Signer is an ECDSA key pair which is necessary signing for some messages like `SignerAdds` and `Verifications`. This example shows how to construct an `Eip712Signer` from a wallet's recovery phrase:

```typescript
import { Eip712Signer } from '@farcaster/js';
import { ethers } from 'ethers';

const mnemonic = 'your mnemonic apple orange banana ...';
const wallet = ethers.Wallet.fromMnemonic(mnemonic);

const eip712Signer = Eip712Signer.fromSigner(wallet, wallet.address)._unsafeUnwrap();
```

### Data Options

A DataOptions object tells the factory some metadata about the message. This example shows how to create the `dataOptions` object to pass to the Factory:

```typescript
import { types } from '@farcaster/js';

const dataOptions = {
  fid: -9999, // Set to the fid of the user creating the message
  network: types.FarcasterNetwork.DEVNET, // Set to the network that the message is broadcast to.
};
```

## Builder Methods

- [makeSignerAdd](#makesigneradd)
- [makeSignerRemove](#makesignerremove)
- [makeCastAdd](#makecastadd)
- [makeCastRemove](#makecastremove)
- [makeReactionAdd](#makereactionadd)
- [makeReactionRemove](#makereactionremove)
- [makeUserDataAdd](#makeuserdataadd)
- [makeVerificationAddEthAddress](#makeverificationaddethaddress)
- [makeVerificationRemove](#makeverificationremove)

---

### makeSignerAdd

Returns a message which authorizes a new Ed25519 Signer to create messages on behalf of a user.

#### Usage

```typescript
import { makeSignerAdd } from '@farcaster/js';

const signerAdd = await makeSignerAdd({ signer: ed25519Signer.signerKeyHex }, dataOptions, eip712Signer);
```

#### Returns

| Type                               | Description                                                    |
| :--------------------------------- | :------------------------------------------------------------- |
| `HubAsyncResult<SignerAddMessage>` | A `HubAsyncResult` that contains the valid `SignerAddMessage`. |

#### Parameters

| Name          | Type                                        | Description                                                                |
| :------------ | :------------------------------------------ | :------------------------------------------------------------------------- |
| `bodyJson`    | [`SignerBody`](modules/types.md#signerbody) | A valid VerificationAddEd25519 body object containing the data to be sent. |
| `dataOptions` | `MessageDataOptions`                        | Optional arguments to construct the message.                               |
| `signer`      | `Eip712Signer`                              | An Eip712Signer generated from the user's custody address.                 |

---

### makeSignerRemove

Returns a message which revokes a previously authorized Ed25519 Signer.

#### Usage

```typescript
import { makeSignerRemove } from '@farcaster/js';

const signerRemove = await makeSignerRemove({ signer: ed25519Signer.signerKeyHex }, dataOptions, eip712Signer);
```

#### Returns

| Type                                  | Description                                                       |
| :------------------------------------ | :---------------------------------------------------------------- |
| `HubAsyncResult<SignerRemoveMessage>` | A `HubAsyncResult` that contains the valid `SignerRemoveMessage`. |

#### Parameters

| Name          | Type                                        | Description                                                      |
| :------------ | :------------------------------------------ | :--------------------------------------------------------------- |
| `bodyJson`    | [`SignerBody`](modules/types.md#signerbody) | A valid SignerRemove body object containing the data to be sent. |
| `dataOptions` | `MessageDataOptions`                        | Optional metadata to construct the message.                      |
| `signer`      | `Eip712Signer`                              | An Eip712Signer generated from the user's custody address.       |

---

### makeCastAdd

Returns a message that adds a new Cast.

#### Usage

```typescript
import { makeCastAdd, types } from '@farcaster/js';

const cast = await makeCastAdd({ text: 'hello world' }, dataOptions, ed25519Signer);
```

#### Returns

| Type                             | Description                                     |
| :------------------------------- | :---------------------------------------------- |
| `HubAsyncResult<CastAddMessage>` | A Result that contains the valid CastAddMessage |

#### Parameters

| Name          | Type                                          | Description                                                 |
| :------------ | :-------------------------------------------- | :---------------------------------------------------------- |
| `bodyJson`    | [`CastAddBody`](modules/types.md#castaddbody) | A valid CastAdd body object containing the data to be sent. |
| `dataOptions` | `MessageDataOptions`                          | Optional metadata to construct the message.                 |
| `signer`      | `Ed25519Signer`                               | A currently valid Signer for the fid.                       |

---

### makeCastRemove

Returns a message that removes an existing Cast.

#### Usage

```typescript
import { makeCastRemove, types } from '@farcaster/js';

const removeBody = {
  targetHash: '0xf88d738eb7145f4cea40fbe8f3bdf...', // Hash of the CastAdd Message
};
const castRemove = await makeCastRemove(removeBody, dataOptions, ed25519Signer);
```

#### Returns

| Type                                | Description                                                     |
| :---------------------------------- | :-------------------------------------------------------------- |
| `HubAsyncResult<CastRemoveMessage>` | A `HubAsyncResult` that contains the valid `CastRemoveMessage`. |

#### Parameters

| Name          | Type                                                | Description                                                    |
| :------------ | :-------------------------------------------------- | :------------------------------------------------------------- |
| `bodyJson`    | [`CastRemoveBody`](modules/types.md#castremovebody) | A valid CastRemove body object containing the data to be sent. |
| `dataOptions` | `MessageDataOptions`                                | Optional metadata to construct the message.                    |
| `signer`      | `Ed25519Signer`                                     | A currently valid Signer for the fid.                          |

---

### makeReactionAdd

Returns a message that adds a Reaction to an existing Cast.

#### Usage

```typescript
import { makeReactionAdd, types } from '@farcaster/js';

const reactionLikeBody = {
  type: types.ReactionType.LIKE,
  target: {
    fid: -9998, // Fid of the Cast's author, which is being reacted to.
    tsHash: '0x455a6caad5dfd4d...', // Hash of the CastAdd message being reacted to
  },
};

const like = await makeReactionAdd(reactionLikeBody, dataOptions, ed25519Signer);
```

#### Returns

| Type | Description |
| :----------------------------------- | | :--------------------------------------------------------------- |
| `HubAsyncResult<ReactionAddMessage>` | A `HubAsyncResult` that contains the valid `ReactionAddMessage`. |

#### Parameters

| Name          | Type                                            | Description                                                     |
| :------------ | :---------------------------------------------- | :-------------------------------------------------------------- |
| `bodyJson`    | [`ReactionBody`](modules/types.md#reactionbody) | A valid ReactionAdd body object containing the data to be sent. |
| `dataOptions` | `MessageDataOptions`                            | Optional metadata to construct the message.                     |
| `signer`      | `Ed25519Signer`                                 | A currently valid Signer for the fid.                           |

---

### makeReactionRemove

Returns a message that removes an existing Reaction to an existing Cast.

#### Usage

```typescript
import { makeReactionRemove, types } from '@farcaster/js';

const reactionLikeBody = {
  type: types.ReactionType.LIKE,
  target: {
    fid: -9998, // Fid of the Cast's author, which is being reacted to.
    tsHash: '0x455a6caad5dfd4d...', // Hash of the CastAdd message being reacted to
  },
};

const unlike = await makeReactionRemove(reactionLikeBody, dataOptions, ed25519Signer);
```

#### Returns

| Type                                    | Description                                                         |
| :-------------------------------------- | ------------------------------------------------------------------- |
| `HubAsyncResult<ReactionRemoveMessage>` | A `HubAsyncResult` that contains the valid `ReactionRemoveMessage`. |

#### Parameters

| Name          | Type                                            | Description                                                        |
| :------------ | :---------------------------------------------- | :----------------------------------------------------------------- |
| `bodyJson`    | [`ReactionBody`](modules/types.md#reactionbody) | A valid ReactionRemove body object containing the data to be sent. |
| `dataOptions` | `MessageDataOptions`                            | Optional metadata to construct the message.                        |
| `signer`      | `Ed25519Signer`                                 | A currently valid Signer for the fid.                              |

---

### makeUserDataAdd

Returns a message that updates metadata about the user.

#### Usage

```typescript
import { makeUserDataAdd, types } from '@farcaster/js';

const userDataPfpBody = {
  type: types.UserDataType.PFP,
  value: 'https://i.imgur.com/yed5Zfk.gif',
};
const userDataPfpAdd = await makeUserDataAdd(userDataPfpBody, dataOptions, ed25519Signer);
```

#### Returns

| Value                                | Description                                                  |
| :----------------------------------- | :----------------------------------------------------------- |
| `HubAsyncResult<UserDataAddMessage>` | A `HubAsyncResult` that contains a signed `UserDataMessage`. |

#### Parameters

| Name          | Type                                            | Description                                                  |
| :------------ | :---------------------------------------------- | :----------------------------------------------------------- |
| `bodyJson`    | [`UserDataBody`](modules/types.md#userdatabody) | A valid UserData body object containing the data to be sent. |
| `dataOptions` | `MessageDataOptions`                            | Optional metadata to construct the message.                  |
| `signer`      | `Ed25519Signer`                                 | A currently valid Signer for the fid.                        |

---

### makeVerificationAddEthAddress

Returns a message that proves that a user owns an Ethereum address.

#### Usage

```typescript
import { makeVerificationAddEthAddress, types } from '@farcaster/js';

const claimBody = {
  fid: -1, // fid of the user
  address: eip712Signer.signerKeyHex, // ethereum address that owns the fid
  network: types.FarcasterNetwork.DEVNET,
  blockHash: '2c87468704d6b0f4c46f480dc54251de...', // block at which this claim is being made
};

const ethSigResult = await eip712Signer.signVerificationEthAddressClaimHex(claimBody);
const ethSig = ethSignResult._unsafeUnwrap();

const verificationBody = {
  address: eip712Signer.signerKeyHex, // address of the user's signer
  signature: ethSig,
  blockHash: '2c87468704d6b0f4c46f480dc54251de...', // block hash at which the claim is being made
};

const verificationMessage = await makeVerificationAddEthAddress(verificationBody, dataOptions, ed25519Signer);
```

#### Returns

| Type                                               | Description                                                                   |
| :------------------------------------------------- | :---------------------------------------------------------------------------- |
| `HubAsyncResult<VerificationAddEthAddressMessage>` | A `HubAsyncResult` that contains a signed `VerificationAddEthAddressMessage`. |

#### Parameters

| Name          | Type                                 | Description                                                                            |
| :------------ | :----------------------------------- | :------------------------------------------------------------------------------------- |
| `bodyJson`    | [`VerificationAddEthAddressBody`](#) | An object which contains an Eip712 Signature from the Ethereum Address being verified. |
| `dataOptions` | `MessageDataOptions`                 | Optional metadata to construct the message.                                            |
| `signer`      | `Ed25519Signer`                      | A currently valid Signer for the fid.                                                  |

---

### makeVerificationRemove

Returns a message that removes a previously added Verification.

#### Usage

```typescript
import { makeVerificationRemove } from '@farcaster/js';

const verificationRemoveBody = {
  address: '0x1234', // Ethereum Address of Verification to remove
};

const verificationRemoveMessage = await makeVerificationRemove(verificationRemoveBody, dataOptions, ed25519Signer);
```

#### Returns

| Type                                        | Description                                                            |
| :------------------------------------------ | :--------------------------------------------------------------------- |
| `HubAsyncResult<VerificationRemoveMessage>` | A `HubAsyncResult` that contains a signed `VerificationRemoveMessage`. |

#### Parameters

| Name          | Type                                                                | Description                                                          |
| :------------ | :------------------------------------------------------------------ | :------------------------------------------------------------------- |
| `bodyJson`    | [`VerificationRemoveBody`](modules/types.md#verificationremovebody) | An object which contains data about the Verification being removed . |
| `dataOptions` | `MessageDataOptions`                                                | Optional metadata to construct the message.                          |
| `signer`      | `Ed25519Signer`                                                     | A currently valid Signer for the fid.                                |
