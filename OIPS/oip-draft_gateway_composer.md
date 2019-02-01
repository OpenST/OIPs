---
oip: <to be assigned>
title: <Gateway Composer for Branded Token and Gateway>
author: <@jasonklein, @abhayks1, Benjamin Bollen (@benjaminbollen)>
discussions-to: <https://discuss.openst.org/t/uxcomposer-brandedtoken-gateway/53>
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
through the use of an optional Gateway Composer contract. Such a Gateway 
Composer contract should be transparant such that only the users intended 
actions can be executed.

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

We can think of the Gateway Composer as a macro automating user actions. Users
can remove their macros and deploy improved ones.

We detail a Gateway Composer for a user who wants to stake OST
in a Branded Token (BT) contract and following that stake the Branded Tokens
into a gateway contract to mint the same amount as Utility Branded Tokens (UBT)
on the sidechain to use within the application of the token.

Every staker deploys her own Gateway Composer contract,
because the messagebus nonce in gateway locks a single message per staker -
and the Gateway Composer (GC) address is the `staker` for the gateway contract.
A Gateway Composer can be (re-)used for multiple gateways in parallel.

#### Assumptions:
- staking OST originates from a hardware wallet (currently no support for
    EIP-721)

#### Flow BrandedToken+Gateway:

1. User approves transfer of OST to Gateway Composer `OST.approve(gc, amount)` 
(USER_SIGNATURE1)

2. User requests stake from Gateway Composer `gc.requestStake(...)` 
(USER_SIGNATURE2)

```Solidity
// See more detailed pseudocode below
function gc:requestStake(<all-user-params>) 
{
    // Move OST from user to gc
    // gc.requestStake(stakeVT, mintBT);
    // store StakeRequest struct with <all-user-params>
}
```
3. Event from `BT` contract `BT:StakeRequested` is evaluated against the
minting policy of the Branded Token.  A registered workers' key for the
BT's organisation can sign the `stakeRequestHash`; the resulting signature
`(r, s, v)` is required to approve the stakeRequest in the BT contract. (ORG_SIGN1)

4. Facilitator can call `gateway.bounty()` to know the active bounty amount,
and must call `OST.approve(gc, bounty)`. 

5. Facilitator can generate a secret and corresponding hashlock for
`gateway:stake`. However the staker is the Gateway Composer `gc`,
so the facilitator must call on `gc.acceptStakeRequest(...)` 
(FACILITATOR_SIGN1)

```Solidity
// See more detailed pseudocode below
function gc::acceptStakeRequest(_stakeRequestHash, _ORG_SIGN1, _hashLock)
{
    // load sr = StakeRequests[_stakeRequestHash]

    // bounty = GatewayI(sr.gateway).bounty();
    // ost.transferFrom(msg.sender, this, bounty);
    // ost.approve(st.gateway, bounty);

    // require(BT.acceptStakeRequest(_stakeRequestHash, _ORG_SIGN1));
    // BT.approve(sr.gateway, sr.mintBT);
    // GatewayI(sr.gateway).stake(
        sr.mintBT,
        sr.beneficiary,
        sr.gasPrice,
        sr.gasLimit,
        sr.nonce,
        _hashLock
    );

    // remove stakeRequest struct
}
```

## Flow OSTPrime
 
 There is only a gateway contract (no BrandedToken contract) mapping (OST on Ethereum mainnet) -> (OSTPrime on sidechain). 
 So stake and mint of OSTPrime would just take two signatures from the user without the obvious need for a Gateway Composer contract;
 
 Stake and Mint flow of OSTPrime will look like below:
 
 - OST.approve(gateway, amount)
 - Gateway.stake(amount, ..., hashlock)
 
 Hashlock would have to be given to the user (assuming the user is not his own facilitator). We will evaluate this further.

#### Additional requirements

Gateway Composer must also support Below functions:

- `transferVT`, `approveVT` by `onlyOwner`
- `transferBT`, `approveBT` by `onlyOwner`
- `revertStakeRequest` for BT
- `revertStake` for Gateway

Gateway Composer can support

- `destroy` to selfdestruct, but be warned that it risks loss of funds if there
are ongoing BT stake requests or gateway stake operations - on revert they
would refund the destroyed contract address. We can check minimally that the
Gateway Composer has no balances and/or there are no ongoing stake requests
(optionally).

## Implementation
<!--The implementations must be completed before any OIP is given status "Final", but it need not be completed before the OIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->
Sketch pieces of code to guide

```Solidity
contract GatewayComposer {

    constructor (
        address _owner,
        eip20tokenI _ost,
        ValueBackedI _brandedToken
    )

    function requestStake(
        /// Amount in VT (Value Token, set to OST)
        uint256 _stakeVT,
        /// Expected amount in BT
        uint256 _mintBT,
        /// Gateway to transfer BT into
        address _gateway,
        /// Beneficiary address on the metablockchain
        address _beneficiary,
        /// Gas price for facilitator fee in (U)BT
        uint256 _gasPrice,
        /// Gas limit for message passing
        uint256 _gasLimit,
        /// Messagebus nonce for staker
        /// Note: don't attempt to compute the nonce in GC
        uint256 _nonce
    )
        onlyOwner
        returns (bytes32 stakeRequestHash_)
    {
        require(_mintBT == BT.convert(_stakeVT));
        require(OST.transferFrom(msg.sender, this, _stakeVT));
        OST.approve(BT, _stakeVT);
        require(BT.requestStake(_stakeVT, _mintBT));

        stakeRequests[_stakeRequestHash] = StakeRequest({
            stakeVT: stakeVT,
            mintBT: _mintBT,
            gateway: _gateway,
            beneficiary: _beneficiary,
            gasPrice: _gasPrice,
            gasLimit: _gasLimit,
            nonce: _nonce,
        });
    }

    function acceptStakeRequest(
        bytes32 _stakeRequestHash,
        bytes32 _r,
        bytes32 _s,
        uint8 _v,
        bytes32 _hashLock
    )
        returns (bytes32 messageHash_)
    {
        // load sr = StakeRequests[_stakeRequestHash]

        // bounty = GatewayI(sr.gateway).bounty();
        // ost.transferFrom(msg.sender, this, bounty);
        // ost.approve(st.gateway, bounty);

        // require(BT.acceptStakeRequest(_stakeRequestHash, _ORG_SIGN1));
        // BT.approve(sr.gateway, sr.mintBT);
        require(GatewayI(sr.gateway).stake(
            sr.mintBT,
            sr.beneficiary,
            sr.gasPrice,
            sr.gasLimit,
            sr.nonce,
            _hashLock
        ));
    }

    function transferVT(...) onlyOwner
    function approveVT(...) onlyOwner

    function transferBT(...) onlyOwner
    function approveBT(...) onlyOwner
}
```