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

<div class="docs-box docs-info-box">
Please be aware that versions below 1.0.0 will still have breaking changes. Till then, please 'freeze' the version of useElven in your app, and decide when to upgrade. Please remember that the scope of functionality is limited and does not correspond to that of sdk-dapp. But over time, functionalities are added.
</div>

## What useElven can do?

The fundamental functionality is connecting and logging the user using auth providers. useElven supports all of [4 signing providers](https://docs.multiversx.com/sdk-and-tools/sdk-js/sdk-js-signing-providers).

By default useElven uses [@multiversx/sdk-native-auth-client](https://www.npmjs.com/package/@multiversx/sdk-native-auth-client) under the hood.

Besides authentication, useElven will also help with all the interactions, like sending native $EGLD tokens or even ESDT tokens. It will allow you to make most transactions, including interactions with custom smart contracts. There is also a possibility to query smart contracts. With an ABI file, you can also decode returned data using React hooks.

useElven is designed to work with the MultiversX Web Wallet Hub and xPortal Discover (Hub) functionalities, though these integrations are still experimental. It integrates official webview provider. No additional action is required on your part to use these features.

## How to start using it?

<div class="docs-box docs-info-box">
  Just to let you know, integrating it with frameworks needs some additional configuration. You'll find examples in ready-to-use starter templates: <a href="https://github.com/useElven/react-vite" target="_blank">Vite + React</a> and <a href="https://github.com/xdevguild/nextjs-dapp-template" target="_blank">Next.js template</a>. Check the configuration files in both.
</div>

Add useElven to your project:
```
npm install @useelven/core @multiversx/sdk-core --save
```

Then import in React app. For example:

```jsx
import { useNetworkSync } from '@useelven/core';
```

Initialize (example: Next.js App Router):

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
    value: BigInt('1000000000000000000'),
  });
};
```

The tools should work with most React setups. For example, with Next.js or React + Vite. But each of the setups requires some additional configuration. This is why there are demo templates that you can clone and treat as a base for your app.

<div class="docs-box docs-info-box">
  You should always use transaction hooks when you are sure that you are in a signed-in context
</div>

## Summary

Okay, so you know what useElven is and how to start using it. You are now ready to look at the [SDK reference](/docs/sdk-reference.html).
