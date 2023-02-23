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

__Please be aware that the docs will be improved in the following days!__

The code samples are not ready to copy and paste. Please search for them in the code of the demo apps linked on the homepage.

#### useNetworkSync()

The hook is responsible for synchronizing the network on each refresh. It should be used in the root component. Here is the `_app.tsx` in Next.js app.

```jsx
import { useNetworkSync } from '@useelven/core';

const NextJSDappTemplate = ({ Component, pageProps }: AppProps) => {
  useNetworkSync();
  return (
    <ChakraProvider theme={theme}>
      <Component {...pageProps} />
    </ChakraProvider>
  );
};
```

#### useLogin

It is the main hook for logging in. The hook is one for all auth providers and can take the auth token as an argument. It can be by any string. Based on this, the auth signature will be generated. This is required when you verify the user account on the backend side. The demos don't use it by default, but you can still pass it.

```jsx
import { useLogin } from '@useelven/core';

(...)

const { login, isLoggedIn, error, walletConnectUri, getHWAccounts } = useLogin({
  token: 'some_hash_here',
});
```

#### useTransaction()

The hook provides all that is required for triggering transactions. useTransaction can also take a callback function as an argument.

```jsx
import { useTransaction } from '@useelven/core';
import { TransactionPayload, TokenPayment } from '@multiversx/sdk-core';

(...)

const { pending, triggerTx, transaction, txResult, error } = useTransaction({ cb });

const handleSendTx = () => {
  const demoMessage = 'Transaction demo!';
  triggerTx({
    address: 'erd123.....',
    gasLimit: 50000 + 1500 * demoMessage.length,
    data: new TransactionPayload(demoMessage),
    value: TokenPayment.egldFromBigInteger(1_000_000_000_000_000_000),
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
    value: TokenPayment.egldFromBigInteger(1_000_000_000_000_000_000),
    data
  });
};
```

#### useScQuery()

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

#### useLoggingIn()

The hook will provide information about the authentication flow state. It will tell if the user is already logged in or is logging in.

```jsx
import { useLoggingIn } from '@useelven/core';

(...)

const { isLoggingIn, error, isLoggedIn } = useLoggingIn();
```

#### useAccount()

The hook will provide information about the user's account data state. The data: address, nonce, balance.

```jsx
import { useAccount } from '@useelven/core';

(...)

const { address, nonce, balance } = useAccount();
```

#### useLoginInfo()

The hook will provide information about the user's auth data state. The data: loginMethod, expires, loginToken, signature. Login token and signature won't always be there. It depends if you'll use the token. Check [Elven Tools Dapp backend integration article](https://www.elven.tools/docs/dapp-backend-integration.html) for more info.

```jsx
import { useLoginInfo } from '@useelven/core';

(...)

const { loginMethod, expires, loginToken, signature } = useLoginInfo();
```

#### useApiCall()

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

#### More to come

There will more of them for sure. Some of the hooks will land in separate repositories/packages. Stay tuned!
