---
oip: <to be assigned>
title: <UX Composer for Branded Token and Gateway>
author: <Benjamin Bollen (@benjaminbollen)>
discussions-to: <discuss.openst.org>
status: Draft
type: <Meta>
created: <2018-11-28>
---

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the OIP.-->
The Branded Token contract for a token economy should be independent
of the layer-2 scaling technology used to run the application;
nor should the scaling contracts be specific to the Branded Token.
Enforcing a clean separation on the interfaces of these contracts,
in turn requires the user to sign more independent transactions.

We propose a composition pattern to combine actions across different
contracts in to fewer combined actions (signatures) for the user through
the use of an optional UX Composer contract. Such a UX Composer contract
should be stateless and transparant such that only the users intended actions
can be executed.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->
A composition contract can be used to optimise the UX flow of
the user across multiple contracts whoes interfaces should not
tightly couple, but where the user intends to perform a single
combined action.

## Motivation
<!--The motivation is critical for OIPs that want to change the OpenST protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the OIP solves. OIP submissions without sufficient motivation may be rejected outright.-->
It is important that the user clearly understands the intended action
she signs a transaction for.  However, for more advanced actions the single
intended action can involve multiple transactions. A composition contract can
facilitate the user experience for common interaction flows across multiple contracts.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations.-->

For a first version, every staker deploys her own Gateway composer contract (UXC),
because the messagebus nonce in gateway locks a single message per staker -
and the UXC contract address is the `staker` for the gateway contract.
A UXC can be (re-)used for multiple gateways in parallel.

Assumptions:
- staking OST originates from a hardware wallet

```solidity
contract GatewayComposer {

    constructor (
        address _owner,
        address _brandedToken)

    function requestStake(
        /// amount in OST
        uint256 _amount,
        /// expected amount in BT
        uint256 _expectedAmount,
        /// intended gateway
        address _gateway,
        /// intended beneficiary address on the metablockchain
        address _beneficiary,
        /// gas price for facilitator fee in (U)BT
        uint256 _gasPrice,
        /// gas limit for message passing
        uint256 _gasLimit,
        /// messagebus nonce for staker
        uint256 _nonce
    )
        returns (uint256 stakedAmount_)
    {
        require(_expectedAmount == BT.convert(_amount));
        OST.transferFrom(msg.sender, this, _expectedAmount);
        OST.approve(BT, _expectedAmount);
        BT.requestStake(_amount, _expectedAmount);

        stakeRequests[gateway] = StakeRequest({
            expectedAmount: _expectedAmount,
            // gateway: _gateway,
            beneficiary: _beneficiary,
            gasPrice: _gasPrice,
            gasLimit: _gasLimit,
            nonce: _nonce,
        });
    }

    function acceptStakeRequest()
}
```



## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->
The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

## Backwards Compatibility
<!--All OIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The OIP must explain how the author proposes to deal with these incompatibilities. OIP submissions without a sufficient backwards compatibility treatise may be rejected outright.-->
All OIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The OIP must explain how the author proposes to deal with these incompatibilities. OIP submissions without a sufficient backwards compatibility treatise may be rejected outright.

## Test Cases
<!--Test cases for an implementation are mandatory for OIPs that are affecting consensus changes. Other OIPs can choose to include links to test cases if applicable.-->
Test cases for an implementation are mandatory for OIPs that are affecting consensus changes. Other OIPs can choose to include links to test cases if applicable.

## Implementation
<!--The implementations must be completed before any OIP is given status "Final", but it need not be completed before the OIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->
The implementations must be completed before any OIP is given status "Final", but it need not be completed before the OIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.
