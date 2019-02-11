---
oip: <to be assigned>
title: <Staking Beneficiary Exists>
author: <Benjamin Bollen (@benjaminbollen)>
discussions-to: <discuss.openst.org>
status: Draft
type: <Standards Track>
category (*only required for Standard Track): <Gateway>
created: <2018-12-05>
requires (*optional): <OIP number(s)>
replaces (*optional): <OIP number(s)>
---

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the OIP.-->
Utility Token should expose an interface `UT.exists(address) returns bool` so
that cogateway can `require(UT.exists(_beneficiary))` on `confirmStakingIntent`.
This blocks a stake-and-mint operation to confirm if `exists(beneficiary)`
returns false, and hence the operation can be reverted if erroneously
initiated for a non-existant beneficiary.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->
By default all addresses exist in the Ethereum specification.  For internal
token economies addresses that are not internal should return `false`
for `UT.exists`. By including this check on `cogateway.confirmStakingIntent`,
a `gateway.stake` for a non-internal beneficiary can either be reverted (or
the beneficiary can be registered as an internal actor on the utility token).
This removes a grid-lock state where the tokens would have been confirmed to be
minted, but cannot be minted because the beneficiary is not registered as an
internal actor.


## Motivation
<!--The motivation is critical for OIPs that want to change the OpenST protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the OIP solves. OIP submissions without sufficient motivation may be rejected outright.-->
This dead-lock state of confirmed-unmintable tokens has been identified before,
and should be resolved.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations.-->

Extend the Utility Token interface with `exists`.

```solidity
interface UtilityToken {
    function increaseSupply(
        address _beneficiary,
        uint256 _amount
    )
        external
        returns (bool success_);
    
    function decreaseSupply(
        uint256 _amount
    )
        external
        returns (bool success_);

    function exists(
        address _actor
    )
        returns (bool exists_);
}
```
