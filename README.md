# Casper storage [![CI](https://github.com/CasperDash/casper-storage/actions/workflows/ci-master.yml/badge.svg?branch=master)](https://github.com/CasperDash/casper-storage/actions/workflows/ci-master.yml) [![codecov](https://codecov.io/gh/CasperDash/casper-storage/branch/master/graph/badge.svg?token=9B1WI1WXGE)](https://codecov.io/gh/CasperDash/casper-storage)
> Following crypto standard libraries, BIPs, SLIPs, etc this library provides a generic solution which lets developers have a standard way to manage wallets

## Documents
- [Initial design](https://github.com/CasperDash/casper-storage/blob/master/document/01-casper-storage-design.md)
- [Technical document](https://casperdash.github.io/casper-storage/)

## Setup

### Browser
`<script src="https://cdn.jsdelivr.net/npm/casper-storage"></script>`

### Node
`npm install casper-storage` or `yarn add casper-storage`

### React-native
> Due to missing features of JavascriptCore, we need to polyfill and overrides some features (e.g BigInt, encoding, etc)

[Click here](https://github.com/CasperDash/casper-storage/blob/master/supports/react-native/README.md) for more detailed information

## Usage

### Key generator
1. In order to use work with keys, import the *KeyFactory* from *casper-storage*
```
import { KeyFactory } from "casper-storage";
const keyManager = KeyFactory.getInstance();
```
2. To generate a new random key (default is mnemonic provider)
```
keyManager.generate();
// output will be something like *enlist announce climb census combine city endorse anxiety wedding combine sleep casino curtain only hedgehog venture inject funny banner cattle early erosion feed observe*
```
3. To convert the key to a seed, so we can use in as a master seed for HD wallet
```
keyManager.toSeedArray("your keyphrase here");
// output will be *an instance of Uint8Array*
```
4. To validate if a phrase is a valid key, following BIP39 standard
```
keyManager.validate("your keyphrase here");
// output will be *either true or false*
```

### Casper HD wallet (with keyphrase)
```
import { KeyFactory, EncryptionType, CasperHDWallet } from "casper-storage"
const keyManager = KeyFactory.getInstance();
```
1. Create a new keyphrase (master key), default is 24-words-length phrase
```
const masterKey = keyManager.generate()
```
2. Convert the master key to the master seed
```
const masterSeed = keyManager.toSeed(masterKey)
const masterSeedArray = keyManager.toSeedArray(masterKey)
```
3. Create a new instance HDWallet from the master seed (either hex value or array buffer), with desired encryption mode
```
const hdWallet = new CasperHDWallet(masterSeed, EncryptionType.Ed25519);
```
4. Get account wallet
```
const acc0 = await hdWallet.getAccount(0)
const acc1 = await hdWallet.getAccount(1)
```
5. Play with wallets
```
// Get the private key
acc0.getPrivateKey()

// Get the public key
await acc0.getPublicKey()

// Get the public address
await acc0.getPublicAddress()

// Get the public address's hash
await acc0.getPublicHash()
```

### Casper legacy wallet (with single private key)
```
import { KeyFactory, EncryptionType, CasperLegacyWallet } from "casper-storage"
```
1. Prepare the private key (input from user, or read from a file)
2. Create a new instance of CasperLegacyWallet with that private key (either hex string or Uint8Array data)
```
const wallet = new CasperLegacyWallet("a-private-key-hex-string", EncryptionType.Ed25519)
const wallet = new CasperLegacyWallet(privateUint8ArrayData, EncryptionType.Secp256k1)
```
3. This wallet will also share the same methods from a wallet of HDWallet
```
await wallet.getPublicAddress()
```

### User
```
import { User } from "casper-storage"
```
1. Prepare a new user instance
```
const user = new User("user-provided-password")
```
2. Set user's HD wallet with encryption type
```
user.setHDWallet("user-generated-master-key", EncryptionType.Ed25519);
```
3. Add user's default first account
```
user.addWalletAccount(0);
const acc0 = await user.getWalletAccount(0);
user.setWalletInfo(acc0.getKey().getPath(), new WalletDescriptor("Account 1"));
```
4. Scan all available users's account (index from 1+, maximum upto 20 following BIP's standard) and add them into the user instance
5. Optional, add user's legacy wallets
```
const wallet = new LegacyWallet("user-wallet-private-key", EncryptionType.Ed25519);
user.addLegacyWallet(wallet, new WalletDescriptor("Legacy wallet 1"));
```
6. Retrive all HD wallet's derived wallets to show on UI
```
user.getHDWallet().derivedWallets
```
7. Retrieve all legacy wallets to show on UI
```
user.getLegacyWallets()
```
8. Each derived wallet or legacy wallet provides a WalletDescriptor with its detailed information
```
const name = user.getHDWallet().derivedWallets[0].descriptor.name
const name = user.getLegacyWallets()[0].descriptor.name
```
9. Serialize/Deserialize user's information
```
const userInfo = user.serialize();
const user2 = user.deserialize(userInfo);
```

### Storage
```
import { StorageManager } from "casper-storage;
const storage = StorageManager.getInstance();

// Set item into storage
await storage.set("key", "value");

// Get items from storage
const value = await storage.get("key");

// Other utils
const exists = await storage.has("key");
await storage.remove("key");
await storage.clear();
```

## Progress
- [x] Key generator (mnemonic)
- [x] Cryptography
  - [x] Asymmetric key implementation
    - [x] Ed25519
    - [x] Secp256k1
  - [x] Utilities
    - [x] Common hash functions (HMAC, SHA256, SHA512, etc)
    - [x] AES encryption
- [x] Wallet
  - [x] HD wallet
  - [x] Legacy wallet
- [x] User management
  - [x] Manage HD wallets, legacy wallets
  - [x] Serialize/Deserialize (encrypted) user's information
- [x] Storage manament
  - [x] Cross-platform storage which supports web, react-native (iOS, Android)
