# 👋 Smart Contract Audits and Findings by Piña 🍍

🔐 Blockchain Security Researcher | 💔 Love breaking things 

🕷️ What I'm up to:
- 🏆 Participating in exciting contests at:
  
  - 🐺 [Code4rena](https://code4rena.com)
  - 🕵️ [Sherlock](https://audits.sherlock.xyz/contests)
  - 🦅 [CodeHawks](https://www.codehawks.com)

🚀 What I'm building:

- 🛠 Created a comprehensive repository on fuzz testing techniques and best practices. [Fuzzing Repository](https://github.com/pinalikefruit/fuzzing.git)
- 🤖 Developed an automation project to streamline my setup process for participating in security review contests. [Setup Repository](https://github.com/pinalikefruit/setup-security-review)

# Audit Competitions Summary

| Overall | High risk |  Medium risk |
|:--:|:--:|:--:|

| 20 High/Medium | 2 High | 18 Medium |

| Date | Project | Severity  | Finding | 
| :---: | :---: | --- | :---: |
|07-23 | [Escrow](https://www.codehawks.com/contests/cljyfxlc40003jq082s0wemya) | [Prevent i_arbiter from being initialized with address (0)](https://codehawks.cyfrin.io/c/2023-07-escrow/s/228) | Medium | 
|07-23 | [BeedleFi](https://www.codehawks.com/contests/clkbo1fa20009jr08nyyf9wbx) | [Anyone can call Fees.sellProfits and perform sandwich attacks](https://codehawks.cyfrin.io/c/2023-07-beedle/s/1556) | High | 
|07-23 | [BeedleFi](https://www.codehawks.com/contests/clkbo1fa20009jr08nyyf9wbx) | [transferOwnership should not be sent to address(0)](https://codehawks.cyfrin.io/c/2023-07-beedle/s/1448) | Medium | 
|07-23 | [Stablecoin](https://www.codehawks.com/contests/cljx3b9390009liqwuedkn0m0) | [liquidate can be front-run](https://codehawks.cyfrin.io/c/2023-07-foundry-defi-stablecoin/s/785) | Medium | 
|07-23 | [Stablecoin](https://www.codehawks.com/contests/cljx3b9390009liqwuedkn0m0) | [TIMEOUT value is hardcoded](https://codehawks.cyfrin.io/c/2023-07-foundry-defi-stablecoin/s/763) | Medium | 
|07-23 | [Stablecoin](https://www.codehawks.com/contests/cljx3b9390009liqwuedkn0m0) | [updatedAt can be zero](https://codehawks.cyfrin.io/c/2023-07-foundry-defi-stablecoin/s/767) | Medium | 
|07-23 | [Stablecoin](https://www.codehawks.com/contests/cljx3b9390009liqwuedkn0m0) | [TIMEOUT value is too large](https://codehawks.cyfrin.io/c/2023-07-foundry-defi-stablecoin/s/766) | Medium | 
|07-23 | [Stablecoin](https://www.codehawks.com/contests/cljx3b9390009liqwuedkn0m0) | [answeredInRound is Deprecated](https://codehawks.cyfrin.io/c/2023-07-foundry-defi-stablecoin/s/768) | Medium | 
|09-23 | [Allo V2](https://audits.sherlock.xyz/contests/109) | [The `Anchor.sol` sets up the registry with an incorrect address](https://github.com/sherlock-audit/2023-09-Gitcoin-judging/issues/335) | High | 
|09-23 | [Venus Primer](https://code4rena.com/contests/2023-09-venus-prime#top) | [DoS and gas griefing of calls to Prime.updateScores()](https://github.com/code-423n4/2023-09-venus-findings/issues/556) | Medium | 
|10-23 | [Steadefi](https://www.codehawks.com/contests/clo38mm260001la08daw5cbuf) | [The vault can be reactivated after "Status.Closed"](https://codehawks.cyfrin.io/c/2023-10-SteadeFi/s/216) | Medium | 
|10-23 | [Steadefi](https://www.codehawks.com/contests/clo38mm260001la08daw5cbuf) | [After `ProcessCompoundCancellation()`, the status of vault is not reset to Open](https://codehawks.cyfrin.io/c/2023-10-SteadeFi/s/246) | Medium | 
|10-23 | [Steadefi](https://www.codehawks.com/contests/clo38mm260001la08daw5cbuf) | [PnL is incorrectly configured for deposit LP](https://codehawks.cyfrin.io/c/2023-10-SteadeFi/s/245) | Medium | 
|01-24 | [Salty](https://code4rena.com/contests/2023-09-venus-prime#top) | [No proposal time limit traps sponsors of unpopular proposals ](https://github.com/code-423n4/2024-01-salty-findings/issues/362) | Medium | 
|01-24 | [Salty](https://code4rena.com/contests/2023-09-venus-prime#top) | [DOS of proposals by abusing ballot names without important parameters](https://github.com/code-423n4/2024-01-salty-findings/issues/621) | Medium | 
|01-24 | [Salty](https://code4rena.com/contests/2023-09-venus-prime#top) | [Impossible to change managed wallets with proposeWallets after first rejection](https://github.com/code-423n4/2024-01-salty-findings/issues/838) | Medium | 
|07-24 | [MagicSea](https://audits.sherlock.xyz/contests/437) | [Protocol Incompatibility with Rebasing Tokens](https://github.com/sherlock-audit/2024-06-magicsea-judging/issues/578) | Medium | 
|07-24 | [MagicSea](https://audits.sherlock.xyz/contests/437) | [Unclaimed Extra Rewards Stuck in Contract After setExtraRewarder Update](https://github.com/sherlock-audit/2024-06-magicsea-judging/issues/559) | Medium | 
|07-24 | [MagicSea](https://audits.sherlock.xyz/contests/437) | [Miscalculation of avgDuration in MlumStaking::addToPosition Causes Extended Lock Periods](https://github.com/sherlock-audit/2024-06-magicsea-judging/issues/345) | Medium | 
|07-24 | [MagicSea](https://audits.sherlock.xyz/contests/437) | [Fee on Transfer Tokens Miscalculation in MasterChef Contract](https://github.com/sherlock-audit/2024-06-magicsea-judging/issues/337) | Medium | 

