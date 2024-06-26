---
layout: "docs"
title: "SDK Reference"
publicationDate: "2022-09-20"
tags:
  - intro
excerpt: "A set of tools for React-based apps using MultiversX JS SDK"
ogTitle: "useElven - React hooks for MultiversX blockchain - reference"
ogDescription: "A set of tools for React-based apps using MultiversX JS SDK"
ogUrl: "https://www.useElven.com/docs/sdk-reference.html"
twitterTitle: "useElven - React hooks for MultiversX blockchain - reference"
twitterDescription: "A set of tools for React-based apps using MultiversX JS SDK"
twitterUrl: "https://www.useElven.com/docs/sdk-reference.html"
githubUrl: "https://github.com/useElven/website/edit/main/src/docs/sdk-reference.md"
---

<div class="docs-box docs-info-box">
  The code samples are not ready to copy and paste. Please search for them in the code of the demo apps: <a href="https://github.com/useElven/react-vite" target="_blank">Vite + React</a> and <a href="https://github.com/xdevguild/nextjs-dapp-template" target="_blank">Next.js template</a>. Also, please check the configuration files in both.
</div>

#### useNetworkSync

The hook is responsible for synchronizing the network on each refresh. It should be used in the root component like the root layout. For example:

```jsx
import { useNetworkSync } from '@useelven/core';
import { ElvenInit } from './components';

const RootLayout = () => {
  return (
    <html lang="en">
      <body className={inter.className}>
        <ElvenInit />
        (...)
      </body>
    </html>
  );
};
```

where ElvenInit:

```jsx
'use client';

import { useNetworkSync } from '@useelven/core';

export const ElvenInit = () => {
  useNetworkSync({
    chainType: 'devnet',
    // If you want to use xPortal signing, 
    // you would need to configure your Wallet Connect project id here: https://cloud.walletconnect.com
    walletConnectV2ProjectId: '<your_wallet_connect_project_id_here>'
  });
  return null;
};
```

Configuration options in `useNetworkSync`:

```typescript
chainType?: string;
shortId?: string;
name?: string;
egldLabel?: string;
egldDenomination?: string;
decimals?: string;
gasPerDataByte?: string;
walletConnectDeepLink?: string;
xAliasAddress?: string;
walletAddress?: string;
apiAddress?: string;
explorerAddress?: string;
IPFSGateway?: string;
apiTimeout?: string;
txWatcherTimeout?: string;
txWatcherPatience?: string;
walletConnectV2RelayAddresses?: string[];
walletConnectV2ProjectId?: string;
```

You can overwrite each of the settings. But when you don't overwrite all of them and overwrite the `chainType`, all other values will default by the `chainType`. You can find all defaults [here](https://github.com/useElven/core/blob/main/src/config/network.ts).

So, for example, when you set the `chainType` to `testnet`, and `apiTimeout` to `10000` and nothing more. Then you will get all default settings for the testnet, like the API endpoint, `testnet-api.multiversx.com`, but the `apiTimeout` will be kept at `10000`.

#### useLogin

It is the main hook for logging in. By default useElven uses [@multiversx/sdk-native-auth-client](https://www.npmjs.com/package/@multiversx/sdk-native-auth-client) under the hood.

```jsx
import { useLogin, LoginMethodsEnum } from '@useelven/core';

(...)

const {
  login,
  isLoggedIn,
  error,
  walletConnectUri,
  getHWAccounts
} = useLogin(params);

(...)

login(LoginMethodsEnum.extension);
```

where:
```ts
enum LoginMethodsEnum {
  ledger = "ledger",
  walletconnect = "walletconnect",
  wallet = "wallet",
  extension = "extension",
  xalias = "xalias",
  extra = "extra",
  none = ""
}
```

Params:

- `callbackUrl` - Optional redirect URL, by default, will use `window.location.href` in case of Web wallet or 2FA redirects (Pass the pathname without the origin)

#### useLogout

The hook is used to disconnect/logout of the wallet.

```ts
const {
  logout,
  pending,
  loggedIn,
  error,
} = useLogout(params);

(...)

logout();
```

Params:

- `callbackUrl` - Optional redirect URL, by default, will use `window.location.href` in case of Web wallet or 2FA redirects (Pass the pathname without the origin)

#### useTransaction

The hook provides all that is required for triggering transactions. useTransaction can also take a callback function as an argument.

<div class="docs-box docs-info-box">
  You should always use transaction hooks when you are sure that you are in a signed-in context (Check Authenticated component in <a href="https://github.com/xdevguild/nextjs-dapp-template">Next.js Dapp Template</a>).
</div>

```jsx
import { useTransaction } from '@useelven/core';
import { TransactionPayload, TokenTransfer } from '@multiversx/sdk-core';

(...)

const {
  pending,
  triggerTx,
  transaction, // transaction data before signing
  txResult, // transaction result on chain
  error
} = useTransaction({ id, cb, callbackUrl });

const handleSendTx = () => {
  const demoMessage = 'Transaction demo!';
  triggerTx({
    address: 'erd123.....',
    // additional 50000 will be added internally when working with guarded accounts
    gasLimit: 50000 + 1500 * demoMessage.length,
    data: new TransactionPayload(demoMessage),
    value: TokenTransfer.egldFromBigInteger(1_000_000_000_000_000_000),
  });
};
```

Example with Smart Contract data payload. For example, when you want to call an endpoint on a custom smart contract.

```jsx
import { useTransaction } from '@useelven/core';
import { U32Value, ContractFunction, ContractCallPayloadBuilder } from '@multiversx/sdk-core';

(...)

const { triggerTx } = useTransaction();

const handleSendTx = () => {
  const data = new ContractCallPayloadBuilder()
    .setFunction(new ContractFunction('mint'))
    .setArgs([new U32Value(1)])
    .build();

  triggerTx({
    address: 'erd123.....',
    gasLimit: 14000000,
    value: TokenTransfer.egldFromBigInteger(1_000_000_000_000_000_000),
    data
  });
};
```

Params:

- `id` - custom ID for a transaction. It is helpful when there is a need to have multiple calls on the same view, especially for web wallet redirections, but generally it is optional
- `cb` - optional callback function
- `callbackUrl` - Optional redirect URL, by default, will use `window.location.href` in case of Web wallet or 2FA redirects (Pass the pathname without the origin)

**Important**
From version 0.11.0, you can also pass whole Transaction object to the `triggerTx`. It is helpful when you need some custom Transaction logic and builders.

It could look like:
```jsx
 triggerTx({
  tx: transactionObject
});
```
Where `transactionObject` is your Transaction object, the same as type as the one prepared in `triggerTx`.

#### useTokenTransfer

The hook is responsible for transferring ESDT tokens (Fungible/Non-fungible/Semi-fungible/Meta). You can also send and call the smart contract endpoint.

Example:

```typescript
(...)
const {
  pending, 
  transfer,
  transaction, // transaction data before signing
  txResult, // transaction result on chain
  error
} = useTokenTransfer({ id, cb, callbackUrl }); // useTokenTransfer params are optional, read more about them below

(...)

const tokensArr: MultiTransferToken[] = [
  {
    type: ESDTType.FungibleESDT,
    amount: '150',
    tokenId: 'FUNG-303171',
  },
  {
    type: ESDTType.SemiFungibleESDT,
    amount: '120', // (here the amount is 120, no need for denomination etc.)
    tokenId: 'BURN-19ee93-01', // token id, not collection id
  },
  {
    type: ESDTType.MetaESDT,
    amount: '10.55', // (here the amount is 10.55)
    tokenId: 'META-18r993-01',
  },
  {
    type: ESDTType.NonFungibleESDT,
    tokenId: 'ELVEN-14e593-01',
  },
];

transfer({
  tokens: tokensArr,
  receiver: 'erd1qqqqqqqqqq...', // smart contract address
  gasLimit: 60000000,
  endpointName: 'burn',
  endpointArgs: [],
});
(...)
```



Where `tokens` is an array of objects with type `MultiTransferToken` (can be imported from the lib).

You can also send ESDTs to standard wallet addresses. In that case omit `endpointName`, `endpointArgs` and `gasLimit`.

MultiTransferToken:
```typescript
{
  type: ESDTType;
  tokenId: string;
  amount: string;
}
```

Params:

- `id` - custom ID for a transaction. It is helpful when there is a need to have multiple calls on the same view, especially for web wallet redirections, but generally it is optional
- `cb` - optional callback function
- `callbackUrl` - Optional redirect URL, by default, will use `window.location.href` in case of Web wallet or 2FA redirects (Pass the pathname without the origin)

ESDTType enum:

```typescript
enum ESDTType {
    FungibleESDT = "FungibleESDT",
    MetaESDT = "MetaESDT",
    NonFungibleESDT = "NonFungibleESDT",
    SemiFungibleESDT = "SemiFungibleESDT"
}
```

#### useScQuery

The hook uses useSWR under the hood and can be triggered on a component mount or remotely on some action. It has two different states for the pending action. For initial load and on revalidate. It also takes one of three return data types: 'number', 'string', 'boolean'. For now, it assumes that you know what data type will be returned by a smart contract. Later it will get more advanced functionality.

```jsx
import { SCQueryType, useScQuery } from '@useelven/core';

(...)

const {
  data: queryResult,
  fetch, // you can always trigger the query manually if 'autoInit' is set to false
  isLoading, // pending state for initial load
  isValidating, // pending state for each revalidation of the data, for example using the mutate
  error,
} = useScQuery<number>({
  type: SCQueryType.NUMBER, // can be NUMBER, STRING, BOOLEAN or COMPLEX
  payload: {
    scAddress: 'erd123....',
    funcName: 'getSomethingBySomething',
    args: [], // arguments for the query in hex format, you can use sdk-core for that, for example: args: [ new Address('erd1....').hex() ] etc. It will be also simplified in the future.
  },
  autoInit: false, // you can enable or disable the trigger of the query on the component mount
  abiJSON: yourImportedAbiJSONObject // required for SCQueryType.COMPLEX type
});
```

**Example** with `SCQueryType.COMPLEX`. This type uses `/vm-values/query`, ABI and ResultParser. The ABI JSON contents are required here. You can copy abi.json and import it in the same place you use useScQuery. Put the abi JSON file wherever you like in the codebase. I chose the `config` directory. See the example below:

```jsx
import { TypedOutcomeBundle } from '@multiversx/sdk-core';
import abiJSON from '../config/abi.json'; // you need to be able to load JSON files in React app

const { data } = useScQuery<TypedOutcomeBundle>({
  type: SCQueryType.COMPLEX,
  payload: {
    scAddress: 'erd1qqq...',
    funcName: 'yourScFunction',
    args: [], // args in hex format, use sdk-core for conversion, see above
  },
  autoInit: true,
  abiJSON,
});
```

The `data` here will be a `TypedOutcomeBundle`. Which is:

```typescript
interface TypedOutcomeBundle {
  returnCode: ReturnCode;
  returnMessage: string;
  values: TypedValue[];
  firstValue?: TypedValue;
  secondValue?: TypedValue;
  thirdValue?: TypedValue;
  lastValue?: TypedValue;
}
```

You can then process the data. For example `data.firstValue.valueOf()` or `data.firstValue.toString()` if applicable. The returned type can be further processed using [sdk-core](https://github.com/multiversx/mx-sdk-js-core).

#### useScDeploy

The hook allows you to deploy a smart contract directly from the provided path to file (also URL) or as an ArrayBuffer.

```jsx
import { useScDeploy } from '@useelven/core';

(...)

const { deploy, pending, transaction, error, txResult, scAddress } = useScDeploy({ id, cb, callbackUrl });

const handleDeploy = () => {
  deploy({ source: '/mysmartcontract.wasm' });
  // or:
  // deploy({ source: 'https://www.somedomain.com/mysmartcontract.wasm' });
  // or:
  // const response = await fetch(source);
  // const bytes = await response.arrayBuffer();
  // deploy({ source: bytes });
};

(...)

// scAddress is computed from the sender's address and nonce,
// it will be available immediately
// so in your app, the best is to wait for txResult and only then show the scAddress
// txResult is transaction data on chain after all processes are finished
console.log(scAddress)
```

deploy arguments:
```ts
export interface ScDeployArgs {
  source: Buffer | string;
  gasLimit?: number;
  isUpgradeable?: boolean;
  isReadable?: boolean;
  isPayable?: boolean;
  isPayableBySmartContract?: boolean;
  initArguments?: T[];
}
```

- `source` The source of your smart contract
- `gasLimit` Gas limit for the transaction
- `isUpgradeable` Whether the contract is upgradeable
- `isReadable` Whether other contracts can read this contract's data (without calling one of its pure functions)
- `isPayable` Whether the contract is payable
- `isPayableBySmartContract` Whether the contract is payable by other smart contracts
- `initArguments` is a set of TypedValue arguments. For example, if your smart contract needs two arguments in the init function, and they are a BigUint and ManagedBuffer, you could do the following:

```ts
import { useScDeploy } from '@useelven/core';
import { BigUintValue, BytesValue } from '@multiversx/sdk-core';

(...)

deploy({ source: '/mysmartcontract.wasm', initArguments: [new BigUintValue(100), BytesValue.fromUTF8('Some string')] });

(...)
```

Params:

- `id` - custom ID for a transaction. It is helpful when there is a need to have multiple calls on the same view, especially for web wallet redirections, but generally it is optional
- `cb` - optional callback function
- `callbackUrl` - Optional redirect URL, by default, will use `window.location.href` in case of Web wallet or 2FA redirects (Pass the pathname without the origin)

Check sdk-core lib for more data types helpers.

#### useSignMessage

The hook allows you to sign any custom message using your wallet address.

```jsx
import { useSignMessage } from '@useelven/core';

(...)

const { signMessage, pending, signature } = useSignMessage();

const handleSignMessage = () => {
  signMessage({ message: 'Elven Family is awesome!' });
};

(...)

console.log(signature)
```

signMessage arguments:

```ts
type SignMessageArgs = {
  message: string;
  options?: {
      callbackUrl?: string;
  };
};
```

#### useLoggingIn

The hook will provide information about the authentication flow state. It will tell if the user is already logged in or is logging in.

```jsx
import { useLoggingIn } from '@useelven/core';

(...)

const { pending, error, loggedIn } = useLoggingIn();
```

#### useAccount

The hook will provide information about the user's account data state. The data: address, nonce, balance.

```jsx
import { useAccount } from '@useelven/core';

(...)

const { address, nonce, balance } = useAccount();
```

#### useConfig

The hook will provide information about curent global configuration.

```jsx
import { useConfig } from '@useelven/core';

(...)

const {
  chainType,
  shortId,
  name,
  egldLabel,
  egldDenomination,
  decimals,
  gasPerDataByte,
  walletConnectDeepLink,
  xAliasAddress,
  walletAddress,
  apiAddress,
  explorerAddress,
  apiTimeout,
  txWatcherTimeout,
  txWatcherPatience,
  IPFSGateway,
  walletConnectV2RelayAddresses,
  walletConnectV2ProjectId,
} = useConfig();
```

#### useNetwork

The hook will provide the dapp provider and network provider instances. It isn't used much, but in some cases, it can be helpful.

```jsx
import { useNetwork } from '@useelven/core';

(...)

const { dappProvider, apiNetworkProvider } = useAccount();
```

#### useLoginInfo

The hook will provide information about the user's auth data state. The data: loginMethod, expires, loginToken, signature, accessToken.

```jsx
import { useLoginInfo } from '@useelven/core';

(...)

const {
  loginMethod,
  expires,
  loginToken,
  signature,
  accessToken
} = useLoginInfo();
```

#### useApiCall

The hook provides a convenient way of doing custom API calls unrelated to transactions or smart contract queries. By default, it will use MultiversX API endpoint. But it can be any type of API, not only MultiversX API. In that case, you would need to pass the `{ baseEndpoint: "https://some-api.com" }`

```jsx
const { data, isLoading, isValidating, fetch, error } = useApiCall<Token[]>({
  url: `/accounts/<some_erd_address_here>/tokens`, // can be any API path without the host, because the host is already handled internally
  autoInit: true, // similar to useScQuery
  type: 'get', // can be get, post, delete, put
  payload: {},
  options: {}
  baseEndpoint: undefined, // any custom API endpoint, by default MultiversX API
});
```

You can pass the response type. Returned object is the same as in `useScQuery`
The hook uses `swr` and native `fetch` under the hood.

#### formatAmount

The utility which helps with amounts formatting.

```jsx
import { formatAmount } from '@useelven/core';

(...)

formatAmount('1000000000000', 18, 18) // 0.000001
```

Arguments:
```ts
type AmountArgs = {
  amount: string;
  decimals: number;
  rounding?: number;
};
```

Returns string.

#### parseAmount

The utility which helps with amounts parsing.

```jsx
import { parseAmount } from '@useelven/core';

(...)

parseAmount('0.000001', 18) // 1000000000000n
```

Arguments:
```ts
type AmountArgs = {
  amount: string;
  decimals: number;
};
```

Returns bigInt.

#### Real life examples:

To get more familiarity with the use Elven library, you can check two open-source projects that use a lot of it:

- [Buildo.dev](https://github.com/xdevguild/buildo.dev)
- [Next.js Dapp Template](https://github.com/xdevguild/nextjs-dapp-template)
- [Piggy Bank Dapp](https://github.com/xdevguild/piggy-bank-dapp)

#### More to come

There will more of them for sure. Some of the hooks will land in separate repositories/packages. Stay tuned!
