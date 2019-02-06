---
oip: <to be assigned>
title: <Gateway Stake and Mint>
author: <Deepesh, Sarvesh, Benjamin>
discussions-to: <discuss.openst.org>
status: Draft
type: <Standards Track>
category (*only required for Standard Track): <Mosaic>
created: <2018-11-23>
---

## Summary
A Gateway allows EIP-20 tokens to be staked on Ethereum mainnet (origin chain)
in the gateway contract, for the equivalent (1:1) amount of tokens minted
on the auxiliary chain (by the cogateway contract).

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

## Motivation
<!--The motivation is critical for OIPs that want to change the Ethereum protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the OIP solves. OIP submissions without sufficient motivation may be rejected outright.-->
The motivation is critical for OIPs that want to change the Ethereum protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the OIP solves. OIP submissions without sufficient motivation may be rejected outright.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations.-->


```solidity
interface Stake {
    function stake(
        uint256 _amount,
        address _beneficiary,
        address _staker,
        uint256 _gasPrice,
        uint256 _gasLimit,
        uint256 _nonce,
        bytes32 _hashLock,
        bytes _signature
    )
        returns (bytes32 messageHash_);
}
```

```solidity
EIP20Gateway is Gateway
Gateway is GatewayBase
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
