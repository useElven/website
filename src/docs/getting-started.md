---
layout: "docs"
title: "Getting started"
publicationDate: "2023-02-12"
tags:
  - intro
excerpt: "A set of tools for React-based apps using MultiversX JS SDK"
ogTitle: "useElven - React hooks for MultiversX blockchain"
ogDescription: "A set of tools for React-based apps using MultiversX JS SDK"
ogUrl: "https://www.useElven.com/docs/getting-started.html"
twitterTitle: "useElven - React hooks for MultiversX blockchain"
twitterDescription: "A set of tools for React-based apps using MultiversX JS SDK"
twitterUrl: "https://www.useElven.com/docs/getting-started.html"
githubUrl: "https://github.com/useelven/website/edit/main/src/docs/getting-started.md"
---

useElven is a set of hooks and tools designed to work with React-base applications.

The tool is a wrapper for [sdk-js](https://docs.multiversx.com/sdk-and-tools/sdk-js/) - a set of Typescript/Javascript libraries.

## What useElven can do?

The fundamental functionality is connecting and logging the user using auth providers. useElven supports all of [4 signing providers](https://docs.multiversx.com/sdk-and-tools/sdk-js/sdk-js-signing-providers).

There is also an option to pass a unique token and get a signature after authentication, which you can use for additional backend verification.

Besides authentication, useElven will also help with all the interactions, like sending native $EGLD tokens or even ESDT tokens. It will allow you to make most transactions, including interactions with custom smart contracts. There is also a possibility to query smart contracts. With an ABI file, you can also decode returned data using React hooks.

## How to start using it?

Add useElven to your project:
```
npm install @useelven/core --save
```

Then import in React app. For example:

```jsx
import { useNetworkSync } from '@useelven/core';
```

Initialize:

```jsx
import { useNetworkSync } from '@useelven/core';

const NextJSDappTemplate = ({ Component, pageProps }: AppProps) => {

  useNetworkSync({ chainType: 'devnet' });

  return (
    <ChakraProvider theme={theme}>
      <Component {...pageProps} />
    </ChakraProvider>
  );
};
```

Login:

```jsx
import { useLogin } from '@useelven/core';

(...)

const { login, isLoggedIn, error } = useLogin({
  token: 'some_hash_here',
});
```

Sign and send transaction:

```jsx
import { useTransaction } from '@useelven/core';
import { TransactionPayload, TokenPayment } from '@multiversx/sdk-core';

(...)

const { pending, triggerTx, transaction, txResult, error } = useTransaction();

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

The tools should work with most React setups. For example, with Next.js or React + Vite. But each of the setups requires some additional configuration. This is why there are demo templates that you can clone and treat as a base for your app.

## Summary

Okay, so you know what useElven is and how to start using it. You are now ready to look at the [SDK reference](/docs/sdk-reference.html).
