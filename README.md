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

| Blockchain | Contract Type | Address |
|-|-|-|
| **Ethereum** |Bridge| [0x609c690e8F7D68a59885c9132e812eEbDaAf0c9e](https://etherscan.io/address/0x609c690e8F7D68a59885c9132e812eEbDaAf0c9e) |
| [USDC](https://etherscan.io/token/0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48) |Pool| [0xa7062bbA94c91d565Ae33B893Ab5dFAF1Fc57C4d](https://etherscan.io/address/0xa7062bbA94c91d565Ae33B893Ab5dFAF1Fc57C4d) |
| [USDT](https://etherscan.io/token/0xdAC17F958D2ee523a2206206994597C13D831ec7) |Pool| [0x7DBF07Ad92Ed4e26D5511b4F285508eBF174135D](https://etherscan.io/address/0x7DBF07Ad92Ed4e26D5511b4F285508eBF174135D) |
| **BNB Chain** |Bridge| [0x3C4FA639c8D7E65c603145adaD8bD12F2358312f](https://bscscan.com/address/0x3C4FA639c8D7E65c603145adaD8bD12F2358312f) |
| [USDT](https://bscscan.com/token/0x55d398326f99059fF775485246999027B3197955) |Pool| [0xf833afA46fCD100e62365a0fDb0734b7c4537811](https://bscscan.com/address/0xf833afA46fCD100e62365a0fDb0734b7c4537811) |
| **Tron** |Bridge| [TAuErcuAtU6BPt6YwL51JZ4RpDCPQASCU2](https://tronscan.io/#/contract/TAuErcuAtU6BPt6YwL51JZ4RpDCPQASCU2) |
| [USDC](https://tronscan.io/#/token20/TEkxiTehnzSmSe2XqrBj4w32RUN966rdz8) |Pool| [TP5bMjDTFzHq7wwNij1JvbNgt3TmBhswfS](https://tronscan.io/#/contract/TP5bMjDTFzHq7wwNij1JvbNgt3TmBhswfS) |
| [USDT](https://tronscan.io/#/token20/TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t) |Pool| [TAC21biCBL9agjuUyzd4gZr356zRgJq61b](https://tronscan.io/#/contract/TAC21biCBL9agjuUyzd4gZr356zRgJq61b) |
| **Solana** |Bridge| [BrdgN2RPzEMWF96ZbnnJaUtQDQx7VRXYaHHbYCBvceWB](https://explorer.solana.com/address/BrdgN2RPzEMWF96ZbnnJaUtQDQx7VRXYaHHbYCBvceWB) |
| [USDC](https://explorer.solana.com/address/EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v) |Pool Token Account| [G6Qo3WW7RbWpSmACAocTBVgx6JW5kgRpUhABphEoDMfP](https://explorer.solana.com/address/G6Qo3WW7RbWpSmACAocTBVgx6JW5kgRpUhABphEoDMfP) |
| **Polygon** |Bridge| [0x7775d63836987f444E2F14AA0fA2602204D7D3E0](https://polygonscan.com/address/0x7775d63836987f444E2F14AA0fA2602204D7D3E0) |
| [USDC](https://polygonscan.com/token/0x3c499c542cEF5E3811e1192ce70d8cC03d5c3359) |Pool| [0x4C42DfDBb8Ad654b42F66E0bD4dbdC71B52EB0A6](https://polygonscan.com/address/0x4C42DfDBb8Ad654b42F66E0bD4dbdC71B52EB0A6) |
| [USDC.e](https://polygonscan.com/token/0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174) |Pool| [0x58Cc621c62b0aa9bABfae5651202A932279437DA](https://polygonscan.com/address/0x58Cc621c62b0aa9bABfae5651202A932279437DA) |
| [USDT](https://polygonscan.com/token/0xc2132D05D31c914a87C6611C10748AEb04B58e8F) |Pool| [0x0394c4f17738A10096510832beaB89a9DD090791](https://polygonscan.com/address/0x0394c4f17738A10096510832beaB89a9DD090791) |
| **Arbitrum** |Bridge| [0x9Ce3447B58D58e8602B7306316A5fF011B92d189](https://arbiscan.io/address/0x9Ce3447B58D58e8602B7306316A5fF011B92d189) |
| [USDC](https://arbiscan.io/token/0xaf88d065e77c8cC2239327C5EDb3A432268e5831) |Pool| [0x690e66fc0F8be8964d40e55EdE6aEBdfcB8A21Df](https://arbiscan.io/address/0x690e66fc0F8be8964d40e55EdE6aEBdfcB8A21Df) |
| [USDT](https://arbiscan.io/token/0xFd086bC7CD5C481DCC9C85ebE478A1C0b69FCbb9) |Pool| [0x47235cB71107CC66B12aF6f8b8a9260ea38472c7](https://arbiscan.io/address/0x47235cB71107CC66B12aF6f8b8a9260ea38472c7) |
| **Avalanche C-Chain** |Bridge| [0x9068E1C28941D0A680197Cc03be8aFe27ccaeea9](https://snowtrace.io/address/0x9068E1C28941D0A680197Cc03be8aFe27ccaeea9) |
| [USDC](https://snowtrace.io/token/0xB97EF9Ef8734C71904D8002F8b6Bc66Dd9c48a6E) |Pool| [0xe827352A0552fFC835c181ab5Bf1D7794038eC9f](https://snowtrace.io/address/0xe827352A0552fFC835c181ab5Bf1D7794038eC9f) |
| [USDT](https://snowtrace.io/token/0x9702230A8Ea53601f5cD2dc00fDBc13d4dF4A8c7) |Pool| [0x2d2f460d7a1e7a4fcC4Ddab599451480728b5784](https://snowtrace.io/address/0x2d2f460d7a1e7a4fcC4Ddab599451480728b5784) |
| **Base** |Bridge| [0x001E3f136c2f804854581Da55Ad7660a2b35DEf7](https://basescan.org/address/0x001E3f136c2f804854581Da55Ad7660a2b35DEf7) |
| [USDC](https://basescan.org/token/0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913) |Pool| [0xDA6bb1ec3BaBA68B26bEa0508d6f81c9ec5e96d5](https://basescan.org/address/0xDA6bb1ec3BaBA68B26bEa0508d6f81c9ec5e96d5) |
| **OP Mainnet** |Bridge| [0x97E5BF5068eA6a9604Ee25851e6c9780Ff50d5ab](https://optimistic.etherscan.io/address/0x97E5BF5068eA6a9604Ee25851e6c9780Ff50d5ab) |
| [USDC](https://optimistic.etherscan.io/token/0x0b2C639c533813f4Aa9D7837CAf62653d097Ff85) |Pool| [0x3B96F88b2b9EB87964b852874D41B633e0f1f68F](https://optimistic.etherscan.io/address/0x3B96F88b2b9EB87964b852874D41B633e0f1f68F) |
| [USDT](https://optimistic.etherscan.io/token/0x94b008aA00579c1307B0EF2c499aD98a8ce58e58) |Pool| [0xb24A05d54fcAcfe1FC00c59209470d4cafB0deEA](https://optimistic.etherscan.io/address/0xb24A05d54fcAcfe1FC00c59209470d4cafB0deEA) |


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
- Source chain liquidity provider fee, always `0.15%` of the transfer amount
- Dollar value of the receiving token, if more than `1` the amount will decrease
- Destination chain liquidity provider fee, always `0.15%` of the transfer amount

To summarize, the user always pays `0.3%` to the liquidity providers, the rest is the token value fluctuations explained below.

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
