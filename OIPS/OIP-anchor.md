---
oip: 10
title: Anchor - Immutably Archived State Roots
author: Martin Schenck (@schemar) <martin@ost.com>
discussions-to: https://discuss.openst.org/t/anchor-oip-0010/54
status: Draft
type: Standards Track
category (*only required for Standard Track): Mosaic
created: 2019-01-17
requires (*optional): none
replaces (*optional): none
---

<!--You can leave these HTML comments in your merged OIP and delete the visible duplicate text guides, they will not appear and may be helpful to refer to if you edit it again. This is the suggested template for new OIPs. Note that an OIP number will be assigned by an editor. When opening a pull request to submit your OIP, please use an abbreviated title in the filename, `OIP-draft_title_abbrev.md`. The title should be 44 characters or less. Thanks to the Ethereum Improvement Proposal (EIP) process for the model we borrow here.-->
# Anchor - Immutably Archived State Roots

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the OIP.-->
By copying state roots in between blockchains, we can assert events of each
blockchain on the respective other chain.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->
Anchoring a state root means storing the state root of a blockchain on another
blockchain. The blockchain where the state root is from is therefore an anchored
blockchain.

Storing the state root on a blockchain archives it immutably. Anyone
can use the anchored state root to prove the state of the anchored blockchain on
the blockchain where the state root is anchored.

As an example, we can use an anchor to prove, on-chain, that a certain event
occurred on another chain. This way, we can for example prove, that a staker has
staked a specific amount on origin and can mint the corresponding tokens on
auxiliary.

## Motivation
<!--The motivation is critical for OIPs that want to change the OpenST protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the OIP solves. OIP submissions without sufficient motivation may be rejected outright.-->
The anchor is an alternative to a [mosaic core](https://github.com/OpenSTFoundation/mosaic-contracts/blob/develop/contracts/core/MosaicCore.sol).
It radically simplifies the process of how state roots are shared across chains.
Instead of mosaic's set of staked PoS validators, the anchor relies solely on an
organization to provide state roots across chains.

Depending on the use-case, an anchor or a mosaic core may be preferred. An
anchor makes it much, much easier to deploy a side chain as compared to a mosaic
core.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations.-->
An `Anchor` provides a method to anchor a state root. It also implements the
`StateRootInterface` to make the state root later available on-chain.

```solidity
/**
 *  @notice Anchor new state root for a block height.
 *
 *  @param _blockHeight Block height at which the state root was valid.
 *  @param _stateRoot State root at given block height.
 *
 *  @return succes_ Whether anchoring was successful.
 */
function anchorStateRoot(
    uint256 _blockHeight,
    bytes32 _stateRoot
)
    external
    onlyOrganization
    returns (bool success_);
```

```solidity
interface StateRootInterface {

    /**
     * @notice Gets the block number of latest committed state root.
     *
     * @return height_ Block height of the latest committed state root.
     */
    function getLatestStateRootBlockHeight()
        external
        view
        returns (uint256 height_);

    /**
     * @notice Get the state root for the given block height.
     *
     * @param _blockHeight The block height for which the state root is fetched.
     *
     * @return bytes32 State root at the given height.
     */
    function getStateRoot(uint256 _blockHeight)
        external
        view
        returns (bytes32 stateRoot_);

}
```

## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->
The method `anchorStateRoot` is kept simple on purpose. The state root and the
height of the block that contains that state root are sufficient to prove
events. An event can be an EVM log, but also the fact that value has been locked
into a gateway or co-gateway.

We decided to implement the `StateRootInterface` with the Anchor as this enables
interchangeability between implementations that provide the state root to other
contracts, for example the gateway or co-gateway.

An alternative to the anchor is a mosaic core. The main difference is that the
mosaic core requires a set of staked validators. The anchor on the other hand
requires only an "organization" to anchor a state root.

## Implementation
<!--The implementations must be completed before any OIP is given status "Final", but it need not be completed before the OIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->

https://github.com/OpenSTFoundation/mosaic-contracts/blob/develop/contracts/gateway/Anchor.sol
