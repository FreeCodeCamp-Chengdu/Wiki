---
title: "Programming DeFi: Uniswap. Part 1"
date: 2021-06-07T00:00:00.000Z
authorURL: ""
originalURL: https://jeiwan.net/posts/programming-defi-uniswap-1/
translator: ""
reviewer: ""
---

# Programming DeFi: Uniswap. Part 1

<!-- more -->

![Scale](/images/piret-ilver-98MbUldcDJY-unsplash.jpg) Photo by [Piret Ilver][1] on [Unsplash][2]

## Introduction

The best way to learn something is to teach others. Second best way to learn something is to do it yourself. I decided to combine the two ways and teach myself and you how to program DeFi services on Ethereum (and any other blockchains based on EVM – Ethereum Virtual Machine).

Our main focus will be on how those services work, we’ll try to understand the economical mechanics that make them what they are (and they all based on economical mechanics). We’ll find out, decompose, learn, and build their core mechanisms.

However, we’ll only work on smart contracts: building front-end for smart-contracts is also a big and interesting task, but it’s out of the scope of this series.

Let’s begin our journey with Uniswap.

> You can find full source codes in this repo:  
> [https://github.com/Jeiwan/zuniswap/tree/part\_1][3]

## Different versions of Uniswap

As of June 2021, three versions of Uniswap have been launched.

First version was launched in November 2018 and it allowed only swaps between ether and a token. Chained swaps were also possible to allow token-token swaps.

V2 was launched in March 2020 and it was a huge improvement of V1 that allowed direct swaps between any ERC20 tokens, as well as chained swaps between any pairs.

V3 was launched in May 2021 and it significantly improved capital efficiency, which allowed liquidity providers to remove a bigger portion of their liquidity from pools and still keep getting the same rewards (or squeeze the capital in smaller price ranges and get up to 4000x of profits).

In this series, we’ll dig into each of the versions and will try to build simplified copies of each of them from scratch.

**This blog post specifically focuses on Uniswap V1** to respect the chronological order and better understand how previous solutions were improved.

## What is Uniswap?

In simple terms, [Uniswap][4] is a decentralized exchange (DEX) that aims to be an alternative to centralized exchanges. It’s running on the Ethereum blockchain and it’s fully automated: there are no admins, managers, or users with privileged access.

On the lower lever, it’s an algorithm that allows to make pools, or token pairs, and fill them with liquidity to let users exchange tokens using this liquidity. Such algorithm is called _automated market maker_ or _automated liquidity provider_.

Let’s learn more about market makers.

Market makers are entities that provide liquidity (trading assets) to markets. Liquidity is what makes trades possible: if you want to sell something but no one is buying it, there won’t be a trade. Some trading pairs have high liquidity (e.g. BTC-USDT), but some have low or no liquidity at all (like some scammy or shady altcoins).

A DEX must have enough (or a lot of) liquidity to function and serve as an alternative to centralized exchanges. One way to get that liquidity is for the developers of the DEX to put their own money (or money of their investors) in it and become market makers. However, this is not a realistic solution because they would need a lot of money to provide enough liquidity for all pairs, considering that DEXes allow exchanges between any tokens. Moreover, this would make the DEX centralized: as the only market makers, the developers would have a lot of power in their hands.

A better solution is to allow **anyone to be a market maker**, and this is what makes Uniswap an automated market maker: any user can deposit their funds into a trading pair (and benefit from that).

Another important role that Uniswap plays is **price oracle**. Price oracles are services that fetch token prices from centralized exchanges and provide them to smart contracts – such prices are usually hard to manipulate because volumes on centralized exchanges are often very big. However, while not having that big volumes, Uniswap can still serve as a price oracle.

Uniswap acts as a secondary market that attracts arbitrageurs who make profit on differences in prices between Uniswap and centralized exchanges. This makes prices on Uniswap pools as close as possible to those on bigger exchanges. And that wouldn’t have been possible without proper pricing and reserves balancing functions.

## Constant product market maker

You probably have already heard this definition, let’s see what it means.

Automated market maker is a general term that embraces different decentralized market maker algorithms. The most popular ones (and those that gave birth to the term) are related to prediction markets - markets that allow to make profit on predictions. Uniswap and other on-chain exchanges are a continuation of those algorithms.

At the core of Uniswap is the constant product function: $$x \* y = k$$ Where \\(x\\) is ether reserve, \\(y\\) is token reserve (or vice versa), and \\(k\\) is a constant. Uniswap requires that \\(k\\) remains the same no matter how much of reserves of \\(x\\) or \\(y\\) there are. When you trade ether for tokens you deposit your ethers into the contract and get some amount of tokens in return. Uniswap ensures that after each trade \\(k\\) remains the same (this is not really true, we’ll see later why).

This formula is also responsible for pricing calculations, and we’ll soon see how.

## Smart contracts development

To really understand how Uniswap works we’ll build a copy of it. We’ll write smart contracts in [Solidity][5] and will use [HardHat][6] as our development environment. HardHat is a really nice tool that greatly simplifies development, testing, and deployment of smart contracts. Highly recommended!

If you’re new to smart contracts development, I highly recommend you to finish [this course][7] (at least the basic path) – that’ll be a huge help to you!

### Setting up the project

First, create an empty directory (I called mine `zuniswap`), `cd` into it and install HardHat:

```shell
$ mkdir zuniswap && cd $_
$ yarn add -D hardhat
```

We’ll also need a token contract, let’s use ERC20 contracts provided by [OpenZeppelin][8].

```shell
$ yarn add -D @openzeppelin/contracts
```

Initialize a HardHat project and remove everything from `contract`, `script`, and `test` folders.

```bash
$ yarn hardhat
...follow the instructions...
$ rm ...
$ tree -a
.
├── .gitignore
├── contracts
├── hardhat.config.js
├── scripts
└── test
```

Final touch: we’ll use the latest version of Solidity, which is 0.8.4 at the time of writing . Open your `hardhat.config.js` and update Solidity version at the bottom of it.

### Token contract

Uniswap V1 supports only ether-token swaps. To make them possible we need an ERC20 token contract. Let’s write it!

```solidity
// contracts/Token.sol
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract Token is ERC20 {
  constructor(
    string memory name,
    string memory symbol,
    uint256 initialSupply
  ) ERC20(name, symbol) {
    _mint(msg.sender, initialSupply);
  }
}
```

This is all we need: we’re extending the ERC20 contract provided by OpenZeppelin and defining our own constructor that allows us to set token name, symbol, and initial supply. The constructor also mints `initialSupply` of tokens and sends them to token creator’s address.

Now, the most interesting part begins!

### Exchange contract

Uniswap V1 has only two contracts: Factory and Exchange.

Factory is a registry contract that allows to create exchanges and keeps track of all deployed exchanges, allowing to find exchange address by token address and vice versa. Exchange contract actually defines exchanging logic. Each pair (eth-token) is deployed as an exchange contract and allows to exchange ether to/from only one token.

We’ll build Exchange contract and leave Factory to a later blog post.

Let’s create a new blank contract:

```solidity
// contracts/Exchange.sol
pragma solidity ^0.8.0;

contract Exchange {}
```

Since every exchange allows swaps with only one token, we need to connect Exchange with a token address:

```solidity
contract Exchange {
  address public tokenAddress;

  constructor(address _token) {
    require(_token != address(0), "invalid token address");

    tokenAddress = _token;
  }
}
```

Token address is a state variable which makes it accessible from any other contract function. Making it public allows users and developers to read it and to find out what token this exchange is linked to. In the constructor, we’re checking that provided token is valid (not the zero address) and save it to the state variable.

### Providing liquidity

As we have already learned, liquidity makes trades possible. Thus, we need a way to add liquidity to the exchange contract:

```solidity
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

contract Exchange {
    ...

    function addLiquidity(uint256 _tokenAmount) public payable {
        IERC20 token = IERC20(tokenAddress);
        token.transferFrom(msg.sender, address(this), _tokenAmount);
    }
}
```

By default, contracts cannot receive ethers, which can be fixed via the `payable` modifier that enables ethers receiving in a function: any ethers sent along with a function call are added to contract’s balance.

Depositing tokens is a different thing: since token balances are stored on token contracts, we have to use `transferFrom` function (as defined by the ERC20 standard) to transfer tokens from transaction sender’s address to the contract. Also, transaction sender would have to call `approve` function on the token contract to allow our exchange contract to get their tokens.

> This implementation of `addLiquidity` is not complete. I intentionally made it so to focus more on pricing functions. We’ll fill the gap in a later part.

Let’s also add a helper function that returns token balance of an exchange:

```solidity
function getReserve() public view returns (uint256) {
  return IERC20(tokenAddress).balanceOf(address(this));
}
```

And we can now test `addLiquidity` to ensure everything’s correct:

```javascript
describe("addLiquidity", async () => {
  it("adds liquidity", async () => {
    await token.approve(exchange.address, toWei(200));
    await exchange.addLiquidity(toWei(200), { value: toWei(100) });

    expect(await getBalance(exchange.address)).to.equal(toWei(100));
    expect(await exchange.getReserve()).to.equal(toWei(200));
  });
});
```

First, we let the exchange contract spend 200 of our tokens by calling `approve`. Then, we call `addLiquidity` to deposit 200 tokens (the exchange contract calls `transferFrom` to get them) and 100 ethers, which are sent along with the function call. We then ensure that the exchange did in fact receive them.

> I’ve omitted a lot of boilerplate code in tests for brevity. Please check [full source code][9] if something is not clear.

### Pricing function

Now, lets think about how we would calculate exchange prices.

It might be tempting to think that price is simply a relation of reserves, e.g.: $$P\_X=\\frac y x, P\_Y=\\frac x y$$

And this makes sense: exchange contracts don’t interact with centralized exchanges or any other external price oracle, so they cannot know the right price. In fact, exchange contract is a price oracle. Everything they know is ether and token reserves, and this is the only information we have to calculate prices.

Let’s stick to this idea and build a pricing function:

```solidity
function getPrice(uint256 inputReserve, uint256 outputReserve)
  public
  pure
  returns (uint256)
{
  require(inputReserve > 0 && outputReserve > 0, "invalid reserves");

  return inputReserve / outputReserve;
}
```

And let’s test it:

```javascript
describe("getPrice", async () => {
  it("returns correct prices", async () => {
    await token.approve(exchange.address, toWei(2000));
    await exchange.addLiquidity(toWei(2000), { value: toWei(1000) });

    const tokenReserve = await exchange.getReserve();
    const etherReserve = await getBalance(exchange.address);

    // ETH per token
    expect(
      (await exchange.getPrice(etherReserve, tokenReserve)).toString()
    ).to.eq("0.5");

    // token per ETH
    expect(await exchange.getPrice(tokenReserve, etherReserve)).to.eq(2);
  });
});
```

We deposited 2000 tokens and 1000 ethers and we’re expecting the price of token to be 0.5 ethers and the price of ether to be 2 tokens. However, the test fails: it says we’re getting 0 ethers in exchange for our tokens. Why is that?

The reason is that Solidity supports integer division with only floor rounding. The price of 0.5 gets rounded to 0! Let’s fix that by increasing the precision:

```solidity
function getPrice(uint256 inputReserve, uint256 outputReserve)
  public
  pure
  returns (uint256)
{
    ...

  return (inputReserve * 1000) / outputReserve;
}
```

After updating the test, it’ll pass:

```javascript
// ETH per token
expect(await exchange.getPrice(etherReserve, tokenReserve)).to.eq(500);

// token per ETH
expect(await exchange.getPrice(tokenReserve, etherReserve)).to.eq(2000);
```

So, now 1 token equals to 0.5 ethers and 1 ether equals to 2 tokens.

Everything looks correct but what will happen if we swap 2000 tokens for ether? We’ll get 1000 ethers and this is everything we have on the contract! **The exchange would be drained!**

Apparently, something is wrong with the pricing function: it allows to drain an exchange, and this is not something we want to happen.

The reason of that is that the pricing function belongs to a constant **sum** formula, which defines \\(k\\) as a constant sum of \\(x\\) and \\(y\\). The function of this constant sum formula is a straight line:

It crosses \\(x\\) and \\(y\\) axes, which means it allows 0 in any of them! We definitely don’t want that.

### Correct pricing function

Let’s recall that Uniswap is a constant product market maker, which means it’s based on a constant product formula: $$x \* y = k$$

Does this formula produce a better pricing function? Let’s see.

The formula states that \\(k\\) remains constant no matter what reserves (\\(x\\) and \\(y\\)) are. Every trade increases a reserve of either ether or token and decreases a reserve of either token or ether – let’s put that logic in a formula:

$$(x + \\Delta x) (y - \\Delta y) = x y$$

Where \\(\\Delta x\\) is the amount of ethers or tokens we’re trading for \\(\\Delta y\\), amount of tokens or ethers we’re getting in exchange. Having this formula we can now find \\(\\Delta y\\): $$\\Delta y = \\frac{y \\Delta x}{x + \\Delta x}$$

This looks interesting: the function now respects input amount. Let’s try to program it, but note that we’re now dealing with amounts, not prices.

```solidity
function getAmount(
  uint256 inputAmount,
  uint256 inputReserve,
  uint256 outputReserve
) private pure returns (uint256) {
  require(inputReserve > 0 && outputReserve > 0, "invalid reserves");

  return (inputAmount * outputReserve) / (inputReserve + inputAmount);
}
```

This is a low-level function, so let it be private. Let’s make two high-level wrapper functions to simplify calculations:

```solidity
function getTokenAmount(uint256 _ethSold) public view returns (uint256) {
  require(_ethSold > 0, "ethSold is too small");

  uint256 tokenReserve = getReserve();

  return getAmount(_ethSold, address(this).balance, tokenReserve);
}

function getEthAmount(uint256 _tokenSold) public view returns (uint256) {
  require(_tokenSold > 0, "tokenSold is too small");

  uint256 tokenReserve = getReserve();

  return getAmount(_tokenSold, tokenReserve, address(this).balance);
}
```

And test them:

```javascript
describe("getTokenAmount", async () => {
  it("returns correct token amount", async () => {
    ... addLiquidity ...

    let tokensOut = await exchange.getTokenAmount(toWei(1));
    expect(fromWei(tokensOut)).to.equal("1.998001998001998001");
  });
});

describe("getEthAmount", async () => {
  it("returns correct eth amount", async () => {
    ... addLiquidity ...

    let ethOut = await exchange.getEthAmount(toWei(2));
    expect(fromWei(ethOut)).to.equal("0.999000999000999");
  });
});
```

So, now we’re getting 1.998 tokens for 1 ether and 0.999 ether for 2 tokens. Those amounts are very close to the ones produced by the previous pricing function. However, they’re slightly smaller. Why is that?

The constant product formula we based our prices calculations on is, in fact, a hyperbola:

Hyperbola never crosses \\(x\\) or \\(y\\), thus neither of the reserves is ever 0. **This makes reserves infinite!**

And there’s another interesting implication: the price function causes price slippage. The bigger the amount of tokens traded in relative to reserves, the higher the price would be.

This is what we saw in the tests: we got slightly less than we expected. This might be seemed as a drawback of constant product market makers (since every trade has a slippage), however this is the same mechanism that protects pools from being drained. This also aligns with the law of supply and demand: the higher the demand (the bigger output amount you want to get) relative to the supply (the reserves), the higher the price (the less you get).

Let’ improve our tests to see how slippage affects prices:

```javascript
describe("getTokenAmount", async () => {
  it("returns correct token amount", async () => {
    ... addLiquidity ...

    let tokensOut = await exchange.getTokenAmount(toWei(1));
    expect(fromWei(tokensOut)).to.equal("1.998001998001998001");

    tokensOut = await exchange.getTokenAmount(toWei(100));
    expect(fromWei(tokensOut)).to.equal("181.818181818181818181");

    tokensOut = await exchange.getTokenAmount(toWei(1000));
    expect(fromWei(tokensOut)).to.equal("1000.0");
  });
});

describe("getEthAmount", async () => {
  it("returns correct ether amount", async () => {
    ... addLiquidity ...

    let ethOut = await exchange.getEthAmount(toWei(2));
    expect(fromWei(ethOut)).to.equal("0.999000999000999");

    ethOut = await exchange.getEthAmount(toWei(100));
    expect(fromWei(ethOut)).to.equal("47.619047619047619047");

    ethOut = await exchange.getEthAmount(toWei(2000));
    expect(fromWei(ethOut)).to.equal("500.0");
  });
});
```

As you can see, when we’re trying to drain the pool, we’re getting only a half of what we’d expect.

A final thing to note here: our initial, reserves ratio based, pricing function wasn’t wrong. In fact, it’s correct when the amount of tokens we’re trading in is very small compared to reserves. But to make an AMM we need something more sophisticated.

### Swapping functions

Now, we’re ready to implement swapping.

```solidity
function ethToTokenSwap(uint256 _minTokens) public payable {
  uint256 tokenReserve = getReserve();
  uint256 tokensBought = getAmount(
    msg.value,
    address(this).balance - msg.value,
    tokenReserve
  );

  require(tokensBought >= _minTokens, "insufficient output amount");

  IERC20(tokenAddress).transfer(msg.sender, tokensBought);
}
```

Swapping ethers for tokens means sending some amount of ethers (stored in `msg.value` variable) to a payable contract function and getting tokens in return. Note that we need to subtract `msg.value` from contract’s balance because by the time the function is called the ethers sent have already been added to its balance.

Another important variable here is `__minTokens` – this is a minimal amount of tokens the user wants to get in exchange for their ethers. This amount is calculated in UI and always includes slippage tolerance; user agrees to get at least that much but not less. This is a very important mechanism that protects users from front-running bots that try to intercept their transactions and modify pool balances to for their profit.

Finally, the last piece of code for today:

```solidity
function tokenToEthSwap(uint256 _tokensSold, uint256 _minEth) public {
  uint256 tokenReserve = getReserve();
  uint256 ethBought = getAmount(
    _tokensSold,
    tokenReserve,
    address(this).balance
  );

  require(ethBought >= _minEth, "insufficient output amount");

  IERC20(tokenAddress).transferFrom(msg.sender, address(this), _tokensSold);
  payable(msg.sender).transfer(ethBought);
}
```

The function basically transfers `_tokensSold` of tokens from user’s balance and sends them `ethBought` of ethers in exchange.

## Conclusion

That’s it for today! We’re not done yet, but we did a lot. Our Exchange contract can accept liquidity from users, calculate prices in a way that protects from draining, and allows users to swap eth for tokens and back. This is a lot, but some important pieces are still missing:

1.  Adding new liquidity can cause huge price changes.
2.  Liquidity providers are not rewarded; all swaps are free.
3.  There’s no way to remove liquidity.
4.  No way to swap ERC20 tokens (chained swaps).
5.  Factory is still not implemented.

We’ll do these in a future part!

## Useful links

1.  [Introduction to Smart Contracts][10] a lot of fundamental information about smart contracts, blockchains, and EVM, to learn before beginning developing smart contracts.
2.  [Let’s run on-chain decentralized exchanges the way we run prediction markets][11], a post on Reddit from Vitalik Buterin where he proposed to use the mechanics of prediction markets to build decentralized exchanges. This gave an idea to use a constant product formula.
3.  [Uniswap V1 Documentation][12]
4.  [Uniswap V1 Whitepaper][13]
5.  [Constant Function Market Makers: DeFi’s “Zero to One” Innovation][14]
6.  [Automated Market Making: Theory and Practice][15]

[1]: https://unsplash.com/@saltsup?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText
[2]: https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText
[3]: https://github.com/Jeiwan/zuniswap/tree/part_1
[4]: https://uniswap.org/
[5]: https://soliditylang.org
[6]: https://hardhat.org
[7]: https://cryptozombies.io/en/course/
[8]: https://openzeppelin.com/
[9]: https://github.com/Jeiwan/zuniswap/tree/part_1
[10]: https://docs.soliditylang.org/en/latest/introduction-to-smart-contracts.html
[11]: https://www.reddit.com/r/ethereum/comments/55m04x/lets_run_onchain_decentralized_exchanges_the_way/
[12]: https://uniswap.org/docs/v1/
[13]: https://hackmd.io/@HaydenAdams/HJ9jLsfTz
[14]: https://medium.com/bollinger-investment-group/constant-function-market-makers-defis-zero-to-one-innovation-968f77022159
[15]: http://reports-archive.adm.cs.cmu.edu/anon/2012/CMU-CS-12-123.pdf