# [CodeHawks Stablecoin Report](https://www.codehawks.com/contests/cljx3b9390009liqwuedkn0m0)

## Findings
| Severity | Title | 
|:--:|:---|
| M-01| liquidate can be front-run| 
| M-02| answeredInRound is Deprecated and TIMEOUT value is too large|
| M-03| updatedAt can be zero |
| M-04| TIMEOUT value is hardcoded|


# [M-01] liquidate can be front-run


## Summary
The liquidate function in the DSCEngine contract presents a front-run risk. This vulnerability may allow attackers to exploit pending transactions for illicit profit.

## Vulnerability Detail

The liquidate function allows users to cover debts and redeem collateral with a 10% bonus. However, it lacks preventative measures against front running, allowing an attacker to view the pending transaction, duplicate the transaction, and increase the gas fee to be confirmed first. This could cause the original user to lose the settlement opportunity and any gas spent.

## Impact

Medium

## Code Snippet
https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/main/src/DSCEngine.sol#L229

## Tool used

Manual code review

## Recommendation

I can recomender use confidential transact like Flashbots.

# [M-02] answeredInRound is Deprecated and TIMEOUT value is too large


## Summary
The variable answeredInRound in Chainlink is used to indicate in which round the current price (answer) was recorded. According to Chainlink's documentation, this variable is marked "deprecated," which typically means that its use is no longer recommended and could be removed or its behavior changed in the future.

If we require the most up-to-date prices possible. The difference in minutes can be critical, so three hours is definitely too long a time frame. And it is superior to the Heartbeat proposed by the two selected pairs.

## Vulnerability Detail

The fact that it is marked as "deprecated" implies that its value should not be trusted for critical DSCEngine logic. In general, it's a good practice to avoid using deprecated features in your code to ensure future compatibility and avoid potential bugs or issues.

In the OracleLib.sol library inside the staleCheckLatestRoundData() function we can see the condition
```
  if (secondsSince > TIMEOUT) revert OracleLib__StalePrice();
```
The selected pairs (BTC/USD & ETH/USD) have a Heartbeat Threshold value of 3600 seconds and a proposed TIMEOUT value of 10800 seconds.

That is, it is possible that our function returns a stale price.

## Impact

High

## Code Snippet
https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/main/src/libraries/OracleLib.sol#L26

https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/main/src/libraries/OracleLib.sol#L19

https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/main/src/libraries/OracleLib.sol#L30
## Tool used

Manual code review

## Recommendation

To make sure, you can add a check between `answeredInRound` and `roundId`, since `answeredInRound` cannot be fully trusted you can add updatedAt in the logic to ensure that the Oracle data is recent.

Update the TIMEOUT value according to the Heartbeat Threshold value of the pairs, even lower, including the tolerance to guarantee having recent data.

# [M-03] updatedAt can be zero


## Summary
The updatedAt variable is the timestamp at which the price was last updated. If updatedAt is equal to zero, it usually means that the data from the last round has not yet been updated or that the price oracle has not provided it yet. no data.

## Vulnerability Detail

Because it is allowed to use any Chainlink data feed, if the selected pair has just been implemented and has not provided any data yet. Until the first price is published, updatedAt will be zero.

Or it could also have resulted from a bug or problem with the price oracle that is preventing it from updating.

Providing a stale price that impacts the DSCEngine

## Impact

High

## Code Snippet
https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/main/src/libraries/OracleLib.sol#L26

## Tool used

Manual code review

## Recommendation

Add a statement that guarantees that `updateAt` is greater than zero.

# [M-04] TIMEOUT value is hardcoded


## Summary
Hardcoding important values like TIMEOUT that will depend on whether the price is not stale in your code is not a good practice, as it makes it difficult to maintain and adapt.

## Vulnerability Detail

The TIMEOUT value was declared as a constant, if ChainLink updates this value or for some security reason you want to change it, this will not be possible, jeopardizing price verification and not returning stale price.

## Impact

High

## Code Snippet
https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/main/src/libraries/OracleLib.sol#L19

## Tool used

Manual code review

## Recommendation

It is recommended to have a timeout state variable that can be set through a function that can only be called by the owner of the contract or whoever has the relevant special permissions.

Considering possible timeout if it were to happen for each proposed pair.

This would give you the flexibility to adjust the timeout based on the changing needs of the DSCEngine.