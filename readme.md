

![alt text](header.png)

[![codecov](https://codecov.io/gh/andrecronje/solidly/branch/master/graph/badge.svg?token=8AZBHZII04)](https://codecov.io/gh/andrecronje/solidly)


Solidly allows low cost, near 0 slippage trades on uncorrelated or tightly correlated assets. The protocol incentivizes fees instead of liquidity. Liquidity providers (LPs) are given incentives in the form of `token`, the amount received is calculated as follows;

* 100% of weekly distribution weighted on votes from ve-token holders

The above is distributed to the `gauge` (see below), however LPs will earn between 40% and 100% based on their own ve-token balance.

LPs with 0 ve* balance, will earn a maximum of 40%.

## AMM

What differentiates Solidly's AMM;

Solidly AMMs are compatible with all the standard features as popularized by Uniswap V2, these include;

* Lazy LP management
* Fungible LP positions
* Chained swaps to route between pairs
* priceCumulativeLast that can be used as external TWAP
* Flashloan proof TWAP
* Direct LP rewards via `skim`
* xy>=k

Solidly adds on the following features;

* 0 upkeep 30 minute TWAPs. This means no additional upkeep is required, you can quote directly from the pair
* Fee split. Fees do not auto accrue, this allows external protocols to be able to profit from the fee claim
* New curve: x3y+y3x, which allows efficient stable swaps
* Curve quoting: `y = (sqrt((27 a^3 b x^2 + 27 a b^3 x^2)^2 + 108 x^12) + 27 a^3 b x^2 + 27 a b^3 x^2)^(1/3)/(3 2^(1/3) x) - (2^(1/3) x^3)/(sqrt((27 a^3 b x^2 + 27 a b^3 x^2)^2 + 108 x^12) + 27 a^3 b x^2 + 27 a b^3 x^2)^(1/3)`
* Routing through both stable and volatile pairs
* Flashloan proof reserve quoting

## token

**TBD**

## ve-token

Vested Escrow (ve), this is the core voting mechanism of the system, used by `BaseV1Factory` for gauge rewards and gauge voting.

This is based off of ve(3,3) as proposed [here](https://andrecronje.medium.com/ve-3-3-44466eaa088b)

* `deposit_for` deposits on behalf of
* `emit Transfer` to allow compatibility with third party explorers
* balance is moved to `tokenId` instead of `address`
* Locks are unique as NFTs, and not on a per `address` basis

```
function balanceOfNFT(uint) external returns (uint)
```

## BaseV1Pair

Base V1 pair is the base pair, referred to as a `pool`, it holds two (2) closely correlated assets (example MIM-UST) if a stable pool or two (2) uncorrelated assets (example FTM-SPELL) if not a stable pool, it uses the standard UniswapV2Pair interface for UI & analytics compatibility.

```
function mint(address to) external returns (uint liquidity)
function burn(address to) external returns (uint amount0, uint amount1)
function swap(uint amount0Out, uint amount1Out, address to, bytes calldata data) external
```

Functions should not be referenced directly, should be interacted with via the BaseV1Router

Fees are not accrued in the base pair themselves, but are transfered to `BaseV1Fees` which has a 1:1 relationship with `BaseV1Pair`

### BaseV1Factory

Base V1 factory allows for the creation of `pools` via ```function createPair(address tokenA, address tokenB, bool stable) external returns (address pair)```

Base V1 factory uses an immutable pattern to create pairs, further reducing the gas costs involved in swaps

Anyone can create a pool permissionlessly.

### BaseV1Router

Base V1 router is a wrapper contract and the default entry point into Stable V1 pools.

```

function addLiquidity(
    address tokenA,
    address tokenB,
    bool stable,
    uint amountADesired,
    uint amountBDesired,
    uint amountAMin,
    uint amountBMin,
    address to,
    uint deadline
) external ensure(deadline) returns (uint amountA, uint amountB, uint liquidity)

function removeLiquidity(
    address tokenA,
    address tokenB,
    bool stable,
    uint liquidity,
    uint amountAMin,
    uint amountBMin,
    address to,
    uint deadline
) public ensure(deadline) returns (uint amountA, uint amountB)

function swapExactTokensForTokens(
    uint amountIn,
    uint amountOutMin,
    route[] calldata routes,
    address to,
    uint deadline
) external ensure(deadline) returns (uint[] memory amounts)

```

## Gauge

Gauges distribute arbitrary `token(s)` rewards to BaseV1Pair LPs based on voting weights as defined by `ve` voters.

Arbitrary rewards can be added permissionlessly via ```function notifyRewardAmount(address token, uint amount) external```

Gauges are completely overhauled to separate reward calculations from deposit and withdraw. This further protect LP while allowing for infinite token calculations.

Previous iterations would track rewardPerToken as a shift everytime either totalSupply, rewardRate, or time changed. Instead we track each individually as a checkpoint and then iterate and calculation.

## Bribe

Gauge bribes are natively supported by the protocol, Bribes inherit from Gauges and are automatically adjusted on votes.

Users that voted can claim their bribes via calling ```function getReward(address token) public```

Fees accrued by `Gauges` are distributed to `Bribes`

### BaseV1Voter

Gauge factory permissionlessly creates gauges for `pools` created by `BaseV1Factory`. Further it handles voting for 100% of the incentives to `pools`.

```
function vote(address[] calldata _poolVote, uint[] calldata _weights) external
function distribute(address token) external
```

### veNFT distribution recipients

| Name | Address | Qty |
| :--- | :--- | :--- |



### Mainnet RED CHAIN AVALANCHE

| Name | Address |
| :--- | :--- |
| WAVAX| [0xB31f66AA3C1e785363F0875A1B74E27b85FD66c7](https://snowtrace.io/address/0xB31f66AA3C1e785363F0875A1B74E27b85FD66c7#code) |
| Printy V2 | [0x6902a8ecF99a732e5a73491Afc14e5E135eE4234](https://snowtrace.io/address/0x6902a8ecF99a732e5a73491Afc14e5E135eE4234#code) |
| BaseV1Factory | [0xc62Ca231Cd2b0c530C622269dA02374134511a36](https://snowtrace.io/address/0xc62Ca231Cd2b0c530C622269dA02374134511a36#code) |
| BaseV1BribeFactory | [0x5C035F691D6C9A23647b84aC3511c47A402670A3](https://snowtrace.io/address/0x5C035F691D6C9A23647b84aC3511c47A402670A3#code) |
| BaseV2GaugesFactory | [0xd51013D5B7f04bB1fDf3b0439eD41893E2aF4026](https://snowtrace.io/address/0xd51013D5B7f04bB1fDf3b0439eD41893E2aF4026#code) |
| BaseV1Router01 | [0xDc72882909252E133a4A46eFB135b3B145366eba](https://snowtrace.io/address/0xDc72882909252E133a4A46eFB135b3B145366eba#code) |
| BaseV1Voter | [0x580770a33B87B209C5a9E1b4F0129f49a8dEab48](https://snowtrace.io/address/0x580770a33B87B209C5a9E1b4F0129f49a8dEab48#code) |
| vePrintyNFT | [0x91E95D75354eC965B67165Cc8DF8249b7B082D3C](https://avascan.info/blockchain/c/address/0x91E95D75354eC965B67165Cc8DF8249b7B082D3C/contract) |
| veNFT-dist | [0x80Ce36E262876aB217c676cfdbB1D953BbB92511](https://snowtrace.io/address/0x80Ce36E262876aB217c676cfdbB1D953BbB92511#code) |
| BaseV1Minter | [0x6146df0D7865e051d97270749a023b4DA03244dD](https://snowtrace.io/address/0x6146df0D7865e051d97270749a023b4DA03244dD#code) |
| PrintyLibrary | [0x1DC44EC22EF22C02cf1Cef1D72Fd915572c153bA](https://snowtrace.io/address/0x1DC44EC22EF22C02cf1Cef1D72Fd915572c153bA#code) |


## Security (!!!solidly)
  
- [Immunefi Bug Bounty Program](https://immunefi.com/bounty/solidly/)  

* [MythX: voter.sol](https://github.com/andrecronje/solidly/blob/master/audits/17faf962f99a7e7e3f26f8bc.pdf)
* [MythX: ve.sol](https://github.com/andrecronje/solidly/blob/master/audits/4094394a6bc512d57672533c.pdf)
* [MythX: gauges.sol](https://github.com/andrecronje/solidly/blob/master/audits/4212b799deea3d9dd8f8620e.pdf)
* [MythX: core.sol](https://github.com/andrecronje/solidly/blob/master/audits/79effbd69276f2d16698b72d.pdf)
* [MythX: minter.sol](https://github.com/andrecronje/solidly/blob/master/audits/dea98051d23c85bcaa80dc5a.pdf)
* [PeckShield](https://github.com/andrecronje/solidly/blob/master/audits/e456a816-3802-4384-894c-825a4177245a.pdf)
