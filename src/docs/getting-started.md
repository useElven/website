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

By default useElven uses [@multiversx/sdk-native-auth-client](https://www.npmjs.com/package/@multiversx/sdk-native-auth-client) under the hood.

Besides authentication, useElven will also help with all the interactions, like sending native $EGLD tokens or even ESDT tokens. It will allow you to make most transactions, including interactions with custom smart contracts. There is also a possibility to query smart contracts. With an ABI file, you can also decode returned data using React hooks.

## How to start using it?

<div class="docs-box docs-info-box">
  Just to let you know, integrating it with frameworks needs some additional configuration. You'll find examples in ready-to-use starter templates: <a href="https://github.com/useElven/react-vite" target="_blank">Vite + React</a> and <a href="https://github.com/xdevguild/nextjs-dapp-template" target="_blank">Next.js template</a>. Check the configuration files in both.
</div>

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
    <Component {...pageProps} />
  );
};
```

Login:

```jsx
import { useLogin } from '@useelven/core';

(...)

const { login, isLoggedIn, error } = useLogin();
```

Sign and send transaction:

```jsx
import { useTransaction } from '@useelven/core';
import { TransactionPayload, TokenTransfer } from '@multiversx/sdk-core';

(...)

const { pending, triggerTx, transaction, txResult, error } = useTransaction();

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

The tools should work with most React setups. For example, with Next.js or React + Vite. But each of the setups requires some additional configuration. This is why there are demo templates that you can clone and treat as a base for your app.

<div class="docs-box docs-info-box">
  You should always use transaction hooks when you are sure that you are in a signed-in context
</div>

## Summary

Okay, so you know what useElven is and how to start using it. You are now ready to look at the [SDK reference](/docs/sdk-reference.html).
