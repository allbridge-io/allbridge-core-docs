# Intro
Allbridge Core is a cross-chain swap built specifically for dollar-pegged tokens (stablecoins). Operates without wrapping tokens and works by keeping liquidity pools for the supported tokens on each blockchain. Incoming tokens go into the pool, the value of those tokens is sent via the messaging protocol to another chain, where it is converted back to the tokens to be paid to the user.

![Simple Diagram](/img/simple.png)

In detail, when the user sends tokens via the Allbridge Core the following happens:

- Bridge exchanges incoming tokens into their dollar value in the token pool for this particular token
- This value is send via the underlining messaging protocol to the destination chain
- Bridge contract on the destination chain swaps dollar value to the tokens from the liquidity pool and sends them to the recipient

This process is illustrated for a sample swap of USDT on Ethereum to BUSD on BNB Chain in the diagram below.

![Architecture Diagram](/img/architecture.png)

The user (or the protocol integrating Allbridge Core) only has to trigger a single transaction on the source chain (line #1 in the diagram above).

# Details

## Blockchains and tokens
Allbridge Core currently supports the following chains and tokens:

- Ethereum ([bridge contract](https://etherscan.io/address/0xA314330482f325D38A83B492EF6B006224a3bea9))
    - USDT ([token contract](https://etherscan.io/token/0xdac17f958d2ee523a2206206994597c13d831ec7))
- BNB Chain ([bridge contract](https://bscscan.com/address/0x7E6c2522fEE4E74A0182B9C6159048361BC3260A))
    - BUSD ([token contract](https://bscscan.com/token/0xe9e7cea3dedca5984780bafc599bd69add087d56))
- Tron ([bridge contract](https://tronscan.io/#/contract/TTpGWvC1ixizai91gCnmMFyutkGS5ZcXK1))
    - USDT ([token contract](https://tronscan.io/#/token20/TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t))
- Solana ([bridge contract](https://explorer.solana.com/address/brdgJFpUjC4jd1d7uZ1b9a8uiY65Xz6yyN3VYzrHNz7))
    - USDC ([token mint](https://explorer.solana.com/address/EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v))

## Messaging protocols
Allbridge Core uses messaging protocols to move information between blockchains. Particularly it is used to send information about transfers and their value to the recepient's blockchain. Allbridge Core architecture allows support for multiple messaging protocols. Currently supported protocols are:

- [Wormhole](https://jumpcrypto.com/compositions/wormhole/) is a decentralized protocol operated by [Jump Crypto](https://jumpcrypto.com/) and used by their own [Portal Bridge](https://www.portalbridge.com/). Does not support Tron.
- Allbridge own messaging protocol which is less decentralized (just 2 validators), but works with all blockchains Allbridge Core supports, including Tron.

Wormhole protocol is more decentralized, this is why it is used by default. Allbridge messaging however is quicker (less time required to reach consensus), less expensive (smaller transactions) and supports Tron.

## Fees

Allbridge Core fees can be divided into two distinct categories: fixed transaction fee not depending on the transfer amount and value adjustment fees.

### Transaction fee
To pay for the receiving transactions on the destination chain, the user pays a transaction fee. It is paid in the gas coin of the sending chain and is attached to the transaction the user submits. It does not depend on the transfer amount.

### Value adjustments
These adjustments affect the final amount received by the user. They include:

- Dollar value of the sending token, if more than `1` the amount will increase
- Source chain liquidity provider fee, always `0.05%` of the transfer amount
- Dollar value of the receiving token, if more than `1` the amount will decrease
- Destination chain liquidity provider fee, always `0.05%` of the transfer amount

To summarize, the user always pays `0.1%` to the liquidity providers, the rest is the token value fluctuations explained below.

## Token value

When transferring tokens between blockchains the amount need to be converted into their internal dollar value. This value depends on the pool condition and the amount being evaluated.

For the sending side (converting tokens into their value):

$$x = x_{orig} + x_{tokens}$$

$$y = \frac{\sqrt{4ad^3x + (4adx - 4ax^2 - dx)^2} + 4adx - 4ax^2 - dx}{8ax}$$

$$y_{value} = y_{orig} - y$$

Where:

- $x_{orig}$ is the token balance in the pool before swap (`tokenBalance` view function in the `Pool` contract)
- $x_{tokens}$ is the amount of tokens you plan to swap
- $y_{orig}$ is the `value` balance in the pool (`vUsdBalance` view function in the `Pool` contract)
- $a$ is a swap curve shape constant (`a` view function in the `Pool` contract, does not change)
- $d$ is a measure of the total liquidity in the pool (`d` view function in the `Pool` contract, does not change unless liquidity is added or removed)
- $y_{value}$ is the resulting dollar value of the $x_{tokens}$ tokens

For the receiving side (converting value into tokens the formula is very similar, but reversed):

$$y = y_{orig} + y_{value}$$

$$x = \frac{\sqrt{4ad^3y + (4ady - 4ay^2 - dy)^2} + 4ady - 4ay^2 - dy}{8ay}$$

$$x_{tokens} = x_{orig} - x$$

The meaning of the variables is the same as before, but for the token pool on the receiving side.

# Integrating Allbridge Core

The easiest way to integrate Allbridge Core into your product is to use Core SDK ([npm package](https://www.npmjs.com/package/@allbridge/bridge-core-sdk), [source](https://github.com/allbridge-io/allbridge-core-js-sdk)).