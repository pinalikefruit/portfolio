# [Beedle Report](https://www.codehawks.com/contests/clkbo1fa20009jr08nyyf9wbx)

## Findings
| Severity | Title | 
|:--:|:---|
| H-01|Anyone can call Fees.sellProfits and perform sandwich attacks| 
| M-01|transferOwnership should not be sent to address(0)| 


## [H-01] Anyone can call Fees.sellProfits and perform sandwich attacks


## Summary
Anyone can call the `sellProfits` function and although the receiver is the contract itselfs, an attacker can take advantage and decrease the profit generated.

## Vulnerability Detail

Because the sellProfits function can be called by anyone, a malicious actor can take advantage of this, since the `amountOutMinimum` is set to 0, as this parameter is a measure of protection against market fluctuations and price changes. price during the transaction allowing the possibility of a sandwich attack, manipulating the price before the transaction takes place and the attacker caught from this.

## Impact

Directly in the fees collected that would be the profit of the protocol

## Code Snippet
https://github.com/Cyfrin/2023-07-beedle/blob/main/src/Fees.sol#L26-L44

## Tool used

Manual code review
    
## Recommendation

Prevent anyone from calling this function.

## [M-01] transferOwnership should not be sent to address(0)


## Summary
It is possible that the current owner can transfer ownership to address(0) in the transferOwnership function of the Ownable contract.

## Vulnerability Detail

Knowing Centralization Risk for trusted owners, it should be verified that the transfer of ownership does not go to an address(0), this would harm everything the project as we saw it detailed, without the possibility of recovery.

## Impact

If it happens, several functionalities would be unsuitable.

## Code Snippet
https://github.com/Cyfrin/2023-07-beedle/blob/main/src/utils/Ownable.sol#L19

## Tool used

Manual code review

## Recommendation

Add a statement that checks cannot be transferred to address(0)