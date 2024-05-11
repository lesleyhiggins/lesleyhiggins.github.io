---
layout: post-2
title: "Marketing Cloud 🤝 web3: How to Token Gate a CloudPage"
date: 2022-06-01
authors: ["Lesley Higgins"]
categories: ["Marketing Cloud", "web3"]
description: Explore how to implement token gating on a CloudPage using MetaMask, a popular crypto wallet, to provide a seamless user authentication experience leveraging blockchain technology.
thumbnail: "/assets/images/gen/blog/Marketing Cloud 🤝 web3- How to Token Gate a CloudPage Thumbnail.png"
image: "/assets/images/gen/blog/Marketing Cloud 🤝 web3- How to Token Gate a CloudPage Thumbnail.png"
---

Web3. What does that really mean? At its most basic level, web3 is "read, write, and own." Part of that ownership is interoperability of identity - being able to take who you are and what you own digitally with you as you traverse around websites. Imagine not having to remember your username and password for every website because you can log in with one identity.

One approach to that is being able to log into websites with a crypto wallet. There are several wallets, but we are going to start with one of the most popular: MetaMask.

MetaMask enables user interactions and experience on web3. It was created to meet the needs of secure and usable Ethereum-based web sites. In particular, it handles account management and connecting the user to the blockchain. It is currently available as a browser extension and as a mobile app on both Android and iOS devices. More simply: it's a crypto wallet. Crypto wallets store assets like coins (ETH, SOL, DOGE, etc.) or NFTs in the form of erc20 and erc721 tokens.

So, why might you want to connect a wallet to a landing page?

NFTs and crypto assets are the latest approach to proving membership in a community. Many web2 companies have started selling NFTs, mostly experimentally. This includes companies like Adidas, The Gap, Gucci, and Liverpool FC.

I personally own a couple Liverpool FC NFTs and my biggest criticism is that they have yet to build a token gated website for claiming any benefits of holding their NFT. Instead they keep dropping coupon codes into the holder-only section of their Discord server. What is wrong with doing that? Well, here is the last 10% off coupon code that they dropped if anyone wants to try using it: `HEROLFC10SATHK`. Anyone can use it and the benefits of holding the NFT are nil.

If you want to truly offer benefits and insider access to the holders of your tokens, you have to create a wallet login that verifies ownership.

### Start by adding MetaMask to your browser:

1. Install MetaMask in the browser of your choice on your development machine. I use Chrome. [Download here](https://metamask.io/download.html).
2. Write down your Secret Recovery Phrase on paper, do not store this on your computer. Write it down again on another piece of paper and store that elsewhere in case of natural disaster.
3. Once MetaMask is installed and running, you should find that new browser tabs have a `window.ethereum` object available in the developer console. This is how your website will interact with MetaMask.
4. To verify if the browser is running MetaMask, copy and paste the code snippet below in the developer console of your web browser:

```javascript
if (typeof window.ethereum !== 'undefined') {
  console.log('MetaMask is installed!');
}
```
Now, moving to Marketing Cloud. In CloudPages, add an HTML block to the page you want to add a Connect to Wallet button on.
Add an HTML block where you want the button.
I added Bootstrap to my CloudPage to help me style the button.
Create the button:

```javascript
<button class="enableEthereumButton">Connect Wallet</button>
<p>Account: <span class="showAccount"></span></p>
```

```javascript
const ethereumButton = document.querySelector('.enableEthereumButton');
const showAccount = document.querySelector('.showAccount');
ethereumButton.addEventListener('click', () => {
  getAccount();
});

async function getAccount() {
  const accounts = await ethereum.request({ method: 'eth_requestAccounts' });
  const account = accounts[0];
  showAccount.innerHTML = account;
}
```
Here is the full code with some minimal bootstrap styling:
 
```javascript

<div class="container">
  <div class="row">
    <div class="col-md-4 offset-md-8">
      <button type="button" class="btn btn-danger enableEthereumButton">Connect Wallet</button>
    </div>
  </div>
  <div class="container">
    <div class="row">
      <p>Account: <span class="showAccount"></span></p>
    </div>
  </div>
</div>
<script>
  const ethereumButton = document.querySelector('.
```