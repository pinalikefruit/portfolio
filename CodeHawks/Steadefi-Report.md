# [Steadefi](https://www.codehawks.com/contests/clo38mm260001la08daw5cbuf)

| Severity | Title | 
|:--:|:---|
| M-01 | After ProcessCompoundCancellation(), the status of vault is not reset to Open| 
| M-02 | The vault can be reactivated after "Status.Closed"| 
| M-03 | PnL is incorrectly configured for deposit LP| 


## [M-01] After ProcessCompoundCancellation(), the status of vault is not reset to Open

## Summary
processCompoundCancellation is called after liquidity could not be added using a `compound()` action and the vault status must be reset to Open but is set to `Compound_Failed`

## Vulnerability Detail
When `compound()` is called and this function fails, it is not handled properly as in the diagram flow and documentation as shown.

Within the `GMXCompound` flow, the other function that can accept this state is in verification:



```solidity
  function beforeCompoundChecks(
    GMXTypes.Store storage self
  ) external view {
    if (
      self.status != GMXTypes.Status.Open &&
      self.status != GMXTypes.Status.Compound_Failed
    ) 

```
But, it will only change if the result is correct. Otherwise, if this error that was initially generated persists and the compound cannot be performed, the vault will be left with the status `GMXTypes.Status.Compound_Failed` and the keeper will have to call `emergencyPause` and interrupt the operation of the vault to change this state.

## Impact
Incorrect handling of control checks, interrupting the correct flow and leaving the vault disabled.

## Code Snippet
https://github.com/Cyfrin/2023-10-SteadeFi/blob/main/contracts/strategy/gmx/GMXCompound.sol#L127-L135

## Tool used
* Manual Review

## Recommendation
Handle correctly for `processCompoundCancellation()` and update the final status for OPEN.



## [M-02] The vault can be reactivated after "Status.Closed"

## Summary
The `GMXVault.sol` should not be reactivated again, after `emergencyClose()` and repaying all users, but it may be activated again.

On `emergencyClose()` the final state of this vault is changed to `GMXTypes.Status.Closed`; but the keeper can call `emergencyPause()` -> `GMXEmergency.sol.emergencyPause()` again since it has no check for the state of the vault as the other functions have.

## Vulnerability Detail
When `compound()` is called and this function fails, it is not handled properly as in the diagram flow and documentation as shown.

Within the GMXCompound flow, the other function that can accept this state is in verification:



```solidity
  function emergencyPause(
    GMXTypes.Store storage self
  ) external {
    self.refundee = payable(msg.sender);

    GMXTypes.RemoveLiquidityParams memory _rlp;

    // Remove all of the vault's LP tokens
    _rlp.lpAmt = self.lpToken.balanceOf(address(this));
    _rlp.executionFee = msg.value;

    GMXManager.removeLiquidity(
      self,
      _rlp
    );

    self.status = GMXTypes.Status.Paused; // @audit change the status

    emit EmergencyPause();
  }

```
Because the protocol no longer has LP token, there will be no error when withdrawing liquidity.

After calling `EmergencyClose`. It has to be a one-way action.

## Impact
A malicious keeper can deny emergency withdrawal to all users.

## Code Snippet
https://github.com/Cyfrin/2023-10-SteadeFi/blob/main/contracts/strategy/gmx/GMXEmergency.sol#L47-L66

## Tool used
* Manual Review

## Recommendation
Add a function called `beforeEmergencyPauseChecks()` in `GMXChecks.sol` that does the vault status check like the other checks do.

## [M-03] PnL is incorrectly configured for deposit LP

## Summary
If a user wants to deposit LP tokens, the maximum ratio of (PnL / value of tokens in the pool) was established with the withdrawal limit and not for deposits accordingly.

## Vulnerability Detail
The price of a market token depends on the value of the assets in the pool and net pending PnL of traders open positions. This limit is the PnL factor that calculates the price of the market token.

The caps used to calculate the market token price may differ depending on activity, deposits or withdrawals (in our implementation).

```solidity
   if (isDeposit) {
      _pnlFactorType = keccak256(abi.encode("MAX_PNL_FACTOR_FOR_DEPOSITS"));
    } else {
      _pnlFactorType = keccak256(abi.encode("MAX_PNL_FACTOR_FOR_WITHDRAWALS"));
    }

    (int256 _marketTokenPrice,) = getMarketTokenInfo(
      marketToken,
      indexToken,
      longToken,
      shortToken,
      _pnlFactorType,
      maximize
    );

```
But, when this function is called in `deposit()`:

```solidity
if (dp.token == address(self.lpToken)) {
      // If LP token deposited
      _dc.depositValue = self.gmxOracle.getLpTokenValue(
        address(self.lpToken),
        address(self.tokenA),
        address(self.tokenA),
        address(self.tokenB),
        false,
        false
      )
```
The PnL factor limit when calculating the market token price for deposits, is used with the `MAX_PNL_FACTOR_FOR_WITHDRAWALS`, which would be used when calculating the market token price of the market token for withdrawals.

## Impact
Deposits above `MAX_PNL_FACTOR_FOR_DEPOSITS` are not allowed but the function is called with `MAX_PNL_FACTOR_FOR_WITHDRAWALS`.\

## Code Snippet
https://github.com/steadefi/steadefi-codehawks/blob/main/contracts/oracles/GMXOracle.sol#L244-L248

## Tool used
* Manual Review


## Recommendation
Update to correct state when user wants to deposit with LP tokens:

```diff
      _dc.depositValue = self.gmxOracle.getLpTokenValue(
        address(self.lpToken),
        address(self.tokenA),
        address(self.tokenA),
        address(self.tokenB),
-       false,
+       true,
        false
      )


```