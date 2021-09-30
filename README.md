# PoolTogether V4 Contest Details

- $100,000 USDC main award pot
- $5,000 USDC gas optimization award pot
- Join [C4 Discord](https://discord.gg/EY5dvm3evD) to register
- Submit findings [using the C4 form](https://code423n4.com/2021-10-pooltogether-v4-contest/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Preview starts September 30, 2021 00:00 UTC (no submissions accepted)
- Preview ends October 6, 2021 23:59 UTC
- Contest Starts October 7, 2021 00:00 UTC
- Contest Ends October 13, 2021 23:59 UTC

# Contest Scope

This contest is open for two weeks to give wardens time to understand the protocol properly. Submissions can only be made in the second week of the contest. Representatives from PoolTogether will be available in the Code Arena Discord to answer any questions during the contest period. The focus for the contest is to try and find any logic errors or ways to drain funds from the protocol in a way that is advantageous for an attacker at the expense of users with funds invested in the protocol. Wardens should assume that governance variables are set sensibly (unless they can find a way to change the value of a governance variable, and not counting social engineering approaches for this).

# Protocol Overview

PoolTogether promotes financial security by making it fun to save.

PoolTogether V4 is unique as it enables a Prize Pool Network.  A Prize Pool Network allows users across chains, with different assets, or different yield sources to combine their interest and share a single pool of prize liquidity.

1. 🏦 Users deposit into the network
2. 📈 Yield accrues on deposits
3. 🏆 The yield is randomly awarded as prizes to the users.

The protocol turns your interest into fun prizes! You never lose your principal, and have a chance to win big.

For a deeper look at the protocol, refer to the [PoolTogether V4 Documentation](https://v4.docs.pooltogether.com/protocol/introduction)

## Smart Contracts

The contracts under audit are those listed below.  Any other contract can be ignored.

There are two repos you should clone:

**v4-core:**
```
git clone git@github.com:pooltogether/v4-core.git
```

**v4-timelocks:**
```
git clone git@github.com:pooltogether/v4-timelocks.git
```

Both repos should be easy to setup:

```
nvm use; yarn; yarn test
```

### v4-core

The [v4-core contracts](https://github.com/pooltogether/v4-core/tree/master/contracts) repo includes the contracts required for the protocol to function.

| Contract | sloc |
| -------- | ------ |
| [ControlledToken.sol](https://github.com/pooltogether/v4-core/tree/master/contracts/ControlledToken.sol) | 50 |
| [DrawBeacon.sol](https://github.com/pooltogether/v4-core/tree/master/contracts/DrawBeacon.sol) | 200 |
| [DrawCalculator.sol](https://github.com/pooltogether/v4-core/tree/master/contracts/DrawCalculator.sol) | 190 |
| [DrawHistory.sol](https://github.com/pooltogether/v4-core/tree/master/contracts/DrawHistory.sol) | 80 |
| [DrawPrize.sol](https://github.com/pooltogether/v4-core/tree/master/contracts/DrawPrize.sol) | 70 |
| [PrizeDistributionHistory.sol](https://github.com/pooltogether/v4-core/tree/master/contracts/PrizeDistributionHistory.sol) | 150 |
| [PrizeFlush.sol](https://github.com/pooltogether/v4-core/tree/master/contracts/PrizeFlush.sol) | 80 |
| [Reserve.sol](https://github.com/pooltogether/v4-core/tree/master/contracts/Reserve.sol) | 100 |
| [Ticket.sol](https://github.com/pooltogether/v4-core/tree/master/contracts/Ticket.sol) | 300 |
| [prize-pool/PrizePool.sol](https://github.com/pooltogether/v4-core/tree/master/contracts/prize-pool/PrizePool.sol) | 250 |
| [prize-pool/YieldSourcePrizePool.sol](https://github.com/pooltogether/v4-core/tree/master/contracts/prize-pool/YieldSourcePrizePool.sol) | 30 |
| [prize-strategy/PrizeSplit.sol](https://github.com/pooltogether/v4-core/tree/master/contracts/prize-strategy/PrizeSplit.sol) | 100 |
| [prize-strategy/PrizeSplitStrategy.sol](https://github.com/pooltogether/v4-core/tree/master/contracts/prize-strategy/PrizeSplitStrategy.sol) | 40 |
| [libraries/DrawRingBufferLib.sol](https://github.com/pooltogether/v4-core/tree/master/contracts/libraries/DrawRingBufferLib.sol) |  30 |
| [libraries/ObservationLib.sol](https://github.com/pooltogether/v4-core/tree/master/contracts/libraries/ObservationLib.sol) | 50 |
| [libraries/RingBuffer.sol](https://github.com/pooltogether/v4-core/tree/master/contracts/libraries/RingBuffer.sol) | 24 |
| [libraries/TwabLib.sol](https://github.com/pooltogether/v4-core/tree/master/contracts/libraries/TwabLib.sol) | 250 |

### v4-timelocks

The [v4-timelocks contracts](https://github.com/pooltogether/v4-timelocks/tree/master/contracts) repo includes the contracts that manage oracle timelocks.

| Contract | sloc |
| -------- | ------ |
| [L1TimelockTrigger.sol](https://github.com/pooltogether/v4-timelocks/tree/master/contracts/L1TimelockTrigger.sol) | 40 |
| [L2TimelockTrigger.sol](https://github.com/pooltogether/v4-timelocks/tree/master/contracts/L2TimelockTrigger.sol) | 40 |
| [DrawCalculatorTimelock.sol](https://github.com/pooltogether/v4-timelocks/tree/master/contracts/DrawCalculatorTimelock.sol) | 80 |

# Unique Logic

- The [Time-Weighted Average Balance](https://v4.docs.pooltogether.com/protocol/concepts/time-weight-average-balance) is a cornerstone of the V4 design.  The concept (and some code!) was borrowed from Uniswap, but still requires heavy scrutiny.
- The Draw Calculator implements the [Tsunami prize algorithm](https://v4.docs.pooltogether.com/protocol/concepts/prize-distribution).
- The codebase uses ring buffers heavily.  You should be familiar with them.

# Areas of Concern

- Does our approach to generating "Picks" (pseudo-random numbers) have any flaws?
- Is our analysis correct for the estimated number of winners for a degree? `i.e. (2^bitRange)^degree - (2^bitRange)^(degree-1) - (2^bitRange)^(degree-2) ...`
- Does the implementation match the theory behind the Tsunami algorithm?  Are there errors or assumptions in the theory?
- This design relies heavily on timestamps.  We highlight a few ways we've mitigated possible front-running at the bottom [of this page](https://v4.docs.pooltogether.com/protocol/reference/launch-architecture).  Have we missed anything?

# Helpful Resources

- [v4-testnet](https://github.com/pooltogether/v4-testnet) repo deploys the contracts in a very similar way to how we will do it in production, if you wish to see.
- [Prize Pool Network](https://v4.docs.pooltogether.com/protocol/concepts/prize-pools-network)
- [Smart Contracts Overview](https://v4.docs.pooltogether.com/protocol/reference/smart-contracts)
- [Launch Architecture](https://v4.docs.pooltogether.com/protocol/reference/launch-architecture)
