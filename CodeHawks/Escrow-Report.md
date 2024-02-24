# [CodeHawks Escrow Report](https://www.codehawks.com/report/cljyfxlc40003jq082s0wemya)

## Findings
| Severity | Title | Report Link |
|:--:|:---| :---|
| M-01|Prevent i_arbiter from being initialized with address (0)| _[Report-M01](https://www.codehawks.com/finding/clm84908m02ihw9ru43z4hqk1)_ |


## [M-01] Prevent i_arbiter from being initialized with address (0)


## Summary
The `i_arbiter` address can be initialized like `address(0)` and is possible for a DOS.

## Vulnerability Detail

The `i_arbiter` is not evaluated in the constructor, but the evaluation is in `initiateDispute()`. So, if for some reason the buyer and the seller do not reach an agreement, it is not possible to create a dispute and the fund may be blocked. The buyer may forget to initialize the arbiter(default) is `address(0)` and execute the transaction and lose the funds.

## Impact

The have impact on the buyer or seller for receive the payment or refund.

## Code Snippet
https://github.com/Cyfrin/2023-07-escrow/blob/main/src/Escrow.sol#L23

https://github.com/Cyfrin/2023-07-escrow/blob/main/src/Escrow.sol#L49

## Tool used

Manual code review

## Recommendation

You can mitigate this vulnerability by moving these sentences to the constructor. How that :

```diff 
if (address(tokenContract) == address(0)) revert Escrow__TokenZeroAddress();
if (buyer == address(0)) revert Escrow__BuyerZeroAddress();
if (seller == address(0)) revert Escrow__SellerZeroAddress();
+ if (i_arbiter == address(0)) revert Escrow__DisputeRequiresArbiter();
```
and update `initiateDispute()`
```diff 
function initiateDispute() external onlyBuyerOrSeller inState(State.Created) {
- if (i_arbiter == address(0)) revert Escrow__DisputeRequiresArbiter();
s_state = State.Disputed;
emit Disputed(msg.sender);
}
```