# Venus Report

# [Venus](https://code4rena.com/reports/2023-09-venus)

| Severity | Title | Report Link |
|:--:|:---| :---|
| M-01 | `updateScores` can fall into an infinite loop| _[Report-M01](https://github.com/code-423n4/2023-09-venus-findings/issues/556)_ |


## [M-01] `updateScores` can fall into an infinite loop

## Vulnerability Detail
First, anyone can call `updateScores`:

```solidity
function updateScores(address[] memory users) external {
        if (pendingScoreUpdates == 0) revert NoScoreUpdatesRequired();
        if (nextScoreUpdateRoundId == 0) revert NoScoreUpdatesRequired();

        for (uint256 i = 0; i < users.length; ) {
            address user = users[i];

            if (!tokens[user].exists) revert UserHasNoPrimeToken();
            if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;

            address[] storage _allMarkets = allMarkets;
            for (uint256 j = 0; j < _allMarkets.length; ) {
                address market = _allMarkets[j];
                _executeBoost(user, market);
                _updateScore(user, market);

                unchecked {
                    j++;
                }
            }

            pendingScoreUpdates--;
            isScoreUpdated[nextScoreUpdateRoundId][user] = true;

            unchecked {
                i++;
            }

            emit UserScoreUpdated(user);
        }
    }
    ```

    The focus is on the first loop, a check statement "if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;" and if this is true it is because the user has their score updated.

    This situation occurred because the i is not incremented because the sentence is at the end of the function and is ignored if the user was updated.

```solidity
unchecked {
                i++;
            }
```

Therefore, it is never incremented and the loop will always verify the same user who is the one that is updated, remaining in that loop forever.

This situation is verified in this example contract:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.18;

contract TestLoop {

    uint public number;

    function loopAlwaysTrue() public  {

        for(uint i = 1; i <= 10; ) {

            if(i % 2  == 0) continue;
            number++;

            unchecked {
                i++;
            }
        }
    }
}
```
In this case, assuming that if(i % 2 == 0) continue is equal to if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;. This means that 5 non-upgraded users and 5 upgraded users will be passed to represent our scenario in this example, but when the validation is successful "(2 % 2 == 0)", it will never be incremented again and the transaction will never complete.

As you can see, the loop index never increases. Then, all the transaction gas will be consumed.

## Impact
When someone calls `updateScores()`, they have to pass a list of addresses they want to update or their own wallet. But, if someone/Venus Protocol calls back using at least one wallet that was passed through the function before, the function falls into an infinite loop and consumes all the gas of the user/protocol.

Various scenarios can occur, such as:

Venus protocol calls updateScores() for that round. But then, a user wants to updateScores because he thinks his score is not the current one, but when he calls the function, it consumes all the gas. Even the protocol will have new users and wants to updateScores, but if there is only one user in that list it will have isScoreUpdated[nextScoreUpdateRoundId][user] = true, then the function will consume all the gas.

This scenario can happen in the opposite way, a user updates his score seconds before the protocol, but the protocol then calls this function and this user is in the list that is passed as an argument in the function, the function will consume all the gas.

A malicious attacker can call before the user/protocol the same transaction and consequently the user/protocol loses all the gas money.

These scenarios can lead to loss of gas (money) for users/protocol due to an implementation error in the first for loop of the function.

## Code Snippet
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L208
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L224-L226

## Tool used
* Manual Review
* Remix


## Recommendation
Make sure to increment the loop index whenever a user passes, like:

```diff 
- for (uint256 i = 0; i < users.length; ) {
+ for (uint256 i = 0; i < users.length; i++ ) {
```