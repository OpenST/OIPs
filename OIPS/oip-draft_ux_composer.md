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
contracts into fewer combined actions (requiring less signatures) for the user
through the use of an optional UX Composer contract. Such a UX Composer
contract should be transparant such that only the users intended actions
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
intended action can involve multiple transactions to multiple contracts.
A composition contract can facilitate the user experience for common
interaction flows across multiple contracts.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations.-->

We can think of the UX composer as a macro automating user actions. Users can
remove their macros and deploy improved ones.

We detail a UX composer for a user who wants to stake OST
in a Branded Token (BT) contract and following that stake the Branded Tokens
into a gateway contract to mint the same amount as Utility Branded Tokens (UBT)
on the sidechain to use within the application of the token.

Every staker deploys her own Gateway composer contract (UXC),
because the messagebus nonce in gateway locks a single message per staker -
and the UXC contract address is the `staker` for the gateway contract.
A UXC can be (re-)used for multiple gateways in parallel.

Assumptions:
- staking OST originates from a hardware wallet (currently no support for
    EIP-721)

Flow BrandedToken+Gateway:

1. User approves transfer of OST to composer `OST.approve(uxc, amount)` (SIGN1)
2. User requests stake from composer `uxc.requestStake(...)` (SIGN2)

```solidity
// see more detailed pseudocode below
function uxc:requestStake(<all-user-params>) 
{
    // move OST from user to uxc
    // uxc.requestStake(stakeVT, mintVBT);
    // store struct with <all-user-params>
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
Sketch pieces of code to guide

```solidity
contract GatewayComposer {

    constructor (
        address _owner,
        eip20tokenI _ost,
        ValueBackedI _brandedToken)

    function requestStake(
        /// amount in VT (Value Token, set to OST)
        uint256 _stakeVT,
        /// expected amount in VBT
        uint256 _mintVBT,
        /// gateway to transfer VBT into
        address _gateway,
        /// beneficiary address on the metablockchain
        address _beneficiary,
        /// gas price for facilitator fee in (U)BT
        uint256 _gasPrice,
        /// gas limit for message passing
        uint256 _gasLimit,
        /// messagebus nonce for staker
        /// note: don't attempt to compute the nonce in UXC
        uint256 _nonce
    )
        onlyOwner
        returns (uint256 stakedAmount_)
    {
        require(_mintVBT == BT.convert(_stakeVT));
        require(OST.transferFrom(msg.sender, this, _stakeVT));
        OST.approve(VBT, _stakeVT);
        require(VBT.requestStake(_stakeVT, _mintVBT));

        stakeRequests[gateway] = StakeRequest({
            stakeVT: stakeVT,
            mintVBT: _mintVBT,
            // gateway: _gateway,
            beneficiary: _beneficiary,
            gasPrice: _gasPrice,
            gasLimit: _gasLimit,
            nonce: _nonce,
        });
    }

    function acceptStakeRequest()

    function transferVT(...) onlyOwner
    function approveVT(...) onlyOwner

    function transferVBT(...) onlyOwner
    function approveVBT(...) onlyOwner
}
```