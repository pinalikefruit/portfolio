# Salty Report

###  [Salty](https://code4rena.com/reports/2024-01-salty)

| Severity | Title | Report Link |
|:--:|:---| :---|
| M-01 | Proposal Process Stalled: `confirmationWallet` Denial Prevents New Proposal Submissions | _[Report-M01](https://github.com/code-423n4/2024-01-salty-findings/issues/766)_ |
| M-02 | DoS Vulnerability in `Proposals::proposeSendSALT` and ` Proposals::proposeSetContractAddress` by Users with `requiredXSalt` Balance | _[Report-M02](https://github.com/code-423n4/2024-01-salty-findings/issues/855)_ |
| M-03 | Proposal Can Be Stuck Indefinitely Due to Lack of Quorum | _[Report-M03](https://github.com/code-423n4/2024-01-salty-findings/issues/902)_ |


## [M-01]  Proposal Process Stalled: `confirmationWallet` Denial Prevents New Proposal Submissions

## Lines of code 

https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L67-L69

## Vulnerability Detail

### Impact

The `ManagedWallet.sol` is designed with a unique functionality where one wallet (mainWallet) proposes a new pair of addresses, and the `confirmationWallet` confirms this change.

The `confirmationWallet` has two options: to make this change or to reject it. However, upon choosing the latter, the contract becomes inoperative. This is because `proposedMainWallet` is not reset, and the contract includes a validation check for new proposals:

```solidity
require(proposedMainWallet == address(0), "Cannot overwrite non-zero proposed mainWallet.");
```
_./src/ManagedWallet.sol/_

This validation check prevents overwriting a new proposal while another is pending, but it does not reset `proposedMainWallet` to address(0) unless confirmationWallet agrees.

As a result, the contract cannot facilitate any changes. This stalemate harms any changes that `ExchangeConfig::managedTeamWallet` wants to make, impacting their ability to utilize this contract effectively and the handling the address of the `SALT` tokens they receive.

```solidity
    function step11() public onlySameContract {
        uint256 releaseableAmount = VestingWallet(payable(exchangeConfig.teamVestingWallet())).releasable(address(salt));

        // teamVestingWallet actually sends the vested SALT to this contract - which will then need to be sent to the active teamWallet
        VestingWallet(payable(exchangeConfig.teamVestingWallet())).release(address(salt));
        
@>       salt.safeTransfer(exchangeConfig.managedTeamWallet().mainWallet(), releaseableAmount);
    }
```

## Proof of Concept

The following is an adaptation of a test from the protocol's test suite (./src/root_tests/ManagedWallet.t.sol), demonstrating the issue: `testProposeWalletsRevertsIfProposedMainWalletIsNonZero` :

```solidity
function testProposeWalletsRevertsIfProposedMainWalletIsNonZero() public {
        address nonZeroProposedMainWallet = address(0x5555);
        address nonZeroProposedConfirmationWallet = address(0x6666);
        
        address confirmationWallet = address(0x2222);
        ManagedWallet managedWallet = new ManagedWallet(alice, confirmationWallet);

         // mainWallet (ALICE) proposes wallets
        vm.prank(alice);
        managedWallet.proposeWallets(nonZeroProposedMainWallet, nonZeroProposedConfirmationWallet);
        
        // confirmationWallet disagrees
        vm.prank(confirmationWallet);
        uint256 sentValue = 0.0001 ether;
        vm.deal(confirmationWallet,sentValue);
        
        // confirmationWallet declines the proposed wallets
        (bool success,) = address(managedWallet).call{value: sentValue}("");
        require(success);
        
        // mainWallet (ALICE) attempts new proposal
        vm.prank(alice);
        address newProposedMainWallet = address(0x7777);
        address newProposedConfirmationWallet = address(0x8888);
        vm.expectRevert("Cannot overwrite non-zero proposed mainWallet.");
        managedWallet.proposeWallets(newProposedMainWallet, newProposedConfirmationWallet);
    }
```
Command to run the test:
```
COVERAGE="yes" NETWORK="sep" forge test --mt testProposeWalletsRevertsIfProposedMainWalletIsNonZero -vv --rpc-url https://eth-sepolia.g.alchemy.com/v2/RPC_KEY
```

## Tool used

Manual code review


## Recommended Mitigation Steps
To resolve this, ensure that when confirmationWallet declines new proposeWallets, the proposedMainWallet is reset:

```diff
   receive() external payable {
        require(msg.sender == confirmationWallet, "Invalid sender");

        // Confirm if .05 or more ether is sent and otherwise reject.
        // Done this way in case custodial wallets are used as the confirmationWallet - which sometimes won't allow for smart contract calls.
        if (msg.value >= 0.05 ether) {
            activeTimelock = block.timestamp + TIMELOCK_DURATION;
        } // establish the timelock
        else {
            activeTimelock = type(uint256).max;
+           proposedMainWallet = address(0);
+           proposedConfirmationWallet = address(0);
        } // effectively never
    }

```
This modification will prevent the contract from becoming inoperative and ensure that it can continue to facilitate necessary changes effectively.


## [M-02]  DoS Vulnerability in `Proposals::proposeSendSALT` and ` Proposals::proposeSetContractAddress` by Users with `requiredXSalt` Balance

## Lines of code 

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L196
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L240
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L131

## Vulnerability Detail

### Impact
A malicious user calling `Proposals::proposeSetContractAddress` with specific names like priceFeed1, priceFeed2, priceFeed3 or accessManager, or calling `Proposals::proposeSendSALT`, can create a Denial of Service (DoS) for these types of proposals, effectively preventing the setting of contracts or sending of `SALT` token.

For `proposeSetContractAddress`, this is because the contract set is determined by the specific name of the contract in the ballot, which can be the same as the hash generated. Thus, a malicious user could hijack these names after 45 days of deployment.

In the case of `proposeSendSALT`, there's no need to send a specific name, as the function sets the name and only one user can make this proposal at the same time. A user with the minimum `requiredXSalt` balance can make this proposal and deny another proposal after `DAOConfig::ballotMinimumDuration`. They can continually front-run the proposal and send it again if other user try.

## Proof of Concept
Here, it's evident that the `ballotName` must specifically match for any changes to be made.
```solidity
function _executeSetContract(Ballot memory ballot) internal {
        bytes32 nameHash = keccak256(bytes(ballot.ballotName));
     
@>        if (nameHash == keccak256(bytes("setContract:priceFeed1_confirm"))) {
            priceAggregator.setPriceFeed(1, IPriceFeed(ballot.address1));
@>        } else if (nameHash == keccak256(bytes("setContract:priceFeed2_confirm"))) {
            priceAggregator.setPriceFeed(2, IPriceFeed(ballot.address1));
@>        } else if (nameHash == keccak256(bytes("setContract:priceFeed3_confirm"))) {
            priceAggregator.setPriceFeed(3, IPriceFeed(ballot.address1));
@>        } else if (nameHash == keccak256(bytes("setContract:accessManager_confirm"))) {
            exchangeConfig.setAccessManager(IAccessManager(ballot.address1));
        }
       
        emit SetContract(ballot.ballotName, ballot.address1);
    }
```
_./src/dao/DAO.sol_

In the case of proposing to send Salt, the ballotName is set to `sendSALT`. Due to protocol restrictions, only one proposal with a unique name can be submitted.

```solidity
function proposeSendSALT(address wallet, uint256 amount, string calldata description)
        external
        nonReentrant
        returns (uint256 ballotID)
    {
        require(wallet != address(0), "Cannot send SALT to address(0)");

        // Limit to 5% of current balance
        uint256 balance = exchangeConfig.salt().balanceOf(address(exchangeConfig.dao()));
        uint256 maxSendable = balance * 5 / 100;
        require(amount <= maxSendable, "Cannot send more than 5% of the DAO SALT balance");

        // This ballotName is not unique for the receiving wallet and enforces the restriction of one sendSALT ballot at a time.
        // If more receivers are necessary at once, a splitter can be used.
@>        string memory ballotName = "sendSALT";
        return _possiblyCreateProposal(ballotName, BallotType.SEND_SALT, wallet, amount, "", description);
    }
```
_./src/dao/Proposals.sol_


## Tool used

Manual code review

## Recommended Mitigation Steps

Implement a mechanism similar to the protocol used for whitelisting tokens. If a contract needs to be set or SALT sent, the trusted result should be reflected in the election and passed through the proposal process. This approach ensures that anyone with enough tokens to make the proposal can submit a contract_name, and if the correct address wins, it will be sent to the oficial proposal.

## [M-03]  Proposal Can Be Stuck Indefinitely Due to Lack of Quorum

## Lines of code 

https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L396
https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L281

## Vulnerability Detail

### Impact
Proposals within the system may become indefinitely stagnant if they fail to achieve the required quorum. This issue can lead to several consequences, including:

Making a new proposal for the same user will be disabled until the proposal achieves quorum (indefinite time)
Crucial proposals (such as `setContracts` or `sendSalt`) becoming unchangeable.
If the DAO implements changes in the protocol, these active proposals might be exploited by malicious users, especially in sensitive functions like `proposeCallContract`, giving them undue influence as the DAO makes the call.
The difficulty in achieving a quorum is further compounded as the required quorum can vary from 1x to 3x depending on the proposal type, making it harder for certain proposals to reach completion.

## Proof of Concept

The issue becomes apparent when examining the finalizeBallot function. The primary requirement for finalizing a ballot is:

`require(proposals.canFinalizeBallot(ballotID), "The ballot is not yet able to be finalized");`

Within this requirement, several checks are performed:

```solidity
function canFinalizeBallot(uint256 ballotID) external view returns (bool) {
        Ballot memory ballot = ballots[ballotID];
        if (!ballot.ballotIsLive) {
            return false;
        }

        // Check that the minimum duration has passed
        if (block.timestamp < ballot.ballotMinimumEndTime) {
            return false;
        }

        // Check that the required quorum has been reached
@>       if (totalVotesCastForBallot(ballotID) < requiredQuorumForBallotType(ballot.ballotType)) {
            return false;
        } 

        return true;
    }
```
_./src/dao/Proposals.sol_

A proposal can be stuck indefinitely if it doesn’t meet the requiredQuorumForBallotType. As upKeep is called, the total supply increases, making it increasingly difficult to achieve the quorum over time.

This test case illustrates the issue:

```solidity
 function testCanFinalizeBallot() public {
        string memory ballotName = "parameter:2";

        uint256 initialStake = 10000000 ether;

        vm.startPrank(alice);
        staking.stakeSALT(1110111 ether);
        proposals.proposeParameterBallot(2, "description");
        staking.unstake(1110111 ether, 2);
        uint256 ballotID = proposals.openBallotsByName(ballotName);

        // Early ballot, no quorum
        bool canFinalizeBallotStillEarly = proposals.canFinalizeBallot(ballotID);

        // Ballot reached end time, no quorum
        vm.warp(block.timestamp + daoConfig.ballotMinimumDuration() + 1); // ballot end time reached

        vm.expectRevert("SALT staked cannot be zero to determine quorum");
        proposals.canFinalizeBallot(ballotID);
        vm.stopPrank();

        vm.prank(DEPLOYER);
        staking.stakeSALT(initialStake);

        bool canFinalizeBallotPastEndtime = proposals.canFinalizeBallot(ballotID);

        // Almost reach quorum
        vm.prank(alice);
        staking.stakeSALT(1110111 ether);

        // Default user has no access to the exchange, but can still vote
        vm.prank(DEPLOYER);
        salt.transfer(address(this), 1000 ether);

        salt.approve(address(staking), type(uint256).max);
        staking.stakeSALT(1000 ether);
        proposals.castVote(ballotID, Vote.INCREASE);

        vm.startPrank(alice);
        proposals.castVote(ballotID, Vote.INCREASE);

        bool canFinalizeBallotAlmostAtQuorum = proposals.canFinalizeBallot(ballotID);

        // Reach quorum
        staking.stakeSALT(1 ether);

        // Recast vote to include new stake
        proposals.castVote(ballotID, Vote.DECREASE);

        bool canFinalizeBallotAtQuorum = proposals.canFinalizeBallot(ballotID);

        // Assert
        assertEq(canFinalizeBallotStillEarly, false, "Should not be able to finalize live ballot");
        assertEq(canFinalizeBallotPastEndtime, false, "Should not be able to finalize non-quorum ballot");
        assertEq(
            canFinalizeBallotAlmostAtQuorum,
            false,
            "Should not be able to finalize ballot if quorum is just beyond the minimum "
        );
        assertEq(
            canFinalizeBallotAtQuorum,
            true,
            "Should be able to finalize ballot if quorum is reached and past the minimum end time"
        );
    }
```
_./src/dao/tests/Proposals.t.sol_

## Tool used

Manual code review

## Recommended Mitigation Steps

Add expiration time, if the ballot dot pass more thant `ballotMinimumEndTime` but the `requiredQuorumForBallotType` is not enough may close this proposal wiouth change and give the oportunitie to the user can open a new proposal.