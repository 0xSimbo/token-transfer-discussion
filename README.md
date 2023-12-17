# ERC721 Approval Changes

Created by: simon nft
Created time: December 17, 2023 12:54 PM

This document proposes a change to ERC721 and possibly to ERC20 approvals. There is no formal name yet and this is still exploratory.

## Abstract

In recent years, millions of dollars have been lost due to un-rejected approvals that remain on exploitable contracts. As we’ve seen with Floor Protocol and with NFTTrader in the past few days. Some protocols take extra precautions and include timestamps on bids or sales, but in case of exploit, these mechanics are not enough.

## Proposal 1 - Time Based Approvals

While this proposal is not 100%, it can reduce many attack surfaces. Many of the most recent exploits have come from lingering approvals on old contracts. Rather than having a `setApprovalForAll(address operator, bool status)` method on ERC721’s we could replace this with something like `setApprovalForAllDuration(address operator, bool status, uint duration)`. The approval would only be valid until `block.timestamp + duration` . In the case of NFTTrader and Floor, this might have prevented forgotten approvals from being exploited. Even though tools exist, it’s hard and cumbersome to maintain approvals.

```solidity
struct Approval {
bool isApproved
uint248 approvalEndTimestamp
}

event TimeLockedApproval(.....)
mapping(address => mapping(address => Approval)) private _approvals;

function setApprovalForAll(address operator,bool status,uint248 timeToAdd) external {
	/// Set the approval until block.timestamp + time to add
  /// Emit time locked approval
}

function transferFrom(address from, address to, uint tokenId) public {
//If msg.sender is not from, check the approval and make sure it's still valid
}
```

## Proposal 2 - 2 Step Transfers

In the current implementation of ERC721, once an operator has approval for an NFT, they can transfer any NFT at will. What if we changed this? Rather than the operator having ultimate approval, in this case, the operator would have the ability to PROPOSE a transfer rather than to execute it. Transfer functions would only be allowed by the owner and operators would call a function `requestTransferTo(address from, address to, uint tokenId,bytes _data)`. Indexers could catch these events and design their marketplaces around this 2-step transfer process. This would mean that for any transfer, the owner would have the ultimate decision around where the NFT ends up. The \_data could be encoded calldata for an external call such as interacting with a marketplace contract. This would make it easier to compose complex systems such as marketplaces without running into massive limitations.

```solidity

mapping(address => mapping(address => bool)) private _requesterStatus;

function setRequesterStatusForAll(address operator,bool status) external {
	/// Set the requester status
}

function requestTransferTo(address from, address to, uint tokenId, bytes _data) external {
 //... Emit an event if the requester is valid
 //.    and if owner owns token etc...
}

function actOnTransferRequestIntent(address from, address to, uint tokenId, bytes _data) external {
/// Check all necessary data
// Act upon the data if non-empty
}
```

---

## Backwards Compatability

Marketplaces would still be able to serve ERC721’s as they’ve been doing, and this schema to NFT’s would most likely incur breaking changes. New events would need to be created and marketplaces would need to design around the time based approvals and/or around 2 step transfer schemas.

---

## Closing Thoughts

None of these are perfect, and they’re more back of the napkin ideas, but I wanted to begin opening up the discussion for a more secure token based approval system

I opened up a public github repo for everyone to share their thoughts.
