---
eip: <to be assigned>
title: <Versatile NFT Standard>
author: Will Wang <will@unizon.pro>, Mike Meng <myan@solv.finance>, Yee Tsai <yee.tsai@gmail.com>, Ryan Chow <ryanchow@solv.finance>, Zhongxin Wu <wuzhongxin@solv.finance>, Jincai Du <dujincai@solv.finance>
discussions-to: <URL>
status: Draft
type: Standards Track
category: ERC
created: <2020-12-01>
requires: <EIP-721, EIP-165>
---

## Simple Summary
This standard describes a set of smart contract interfaces, as an extension of ERC-721, makes an NFT semi-fungible, that is: besides its ERC-721 ID property, it has a quantitative property, called 'units', which can be treated as quantities contained in an ERC-721 token.

## Abstract

VNFT is a kind of ERC-721 compatible NFT, with each token ID representing a single entity. However, it has a 'units' property, representing the amount contained within the token, since the amount contained in one token is fungible by nature, a single token can be split in to several different tokens, with certain properties maintained unchanged and the sum of the amounts of all newly generated tokens equals to that of the original one. Nevertheless, each VNFT has a 'SLOT' attribute, which means they are logically the same kind of business entities, so that tokens can be merged to a target token with the same slot, resulting in the target token to own all units from the merged tokens, which will then be burnt.

## Motivation

Tokenization of assets is one of the most important patterns in blockchain/DeFi apps nowadays, normally there are two ways for tokenizing assets: the fungible way and the non-fungible way. The first one generally uses the ERC-20 standard, in the case that every user's assets deposited into a DeFi contract are fungible to that of others, so the ERC-20 standard provides the most flexible way to manipulate these derived asset tokens. The second one mainly uses the ERC-721 type token standard, for the purpose that everyone's asset needs to be described by one or more custom properties, for example, in a DEX platform that allows individual positions for certain price ranges, the LP token needs to be implemented in an ERC-721 way, since every position should be distinguished from each other due to their different price ranges, even if the values of assets contained in the two positions are equal. 

Both methods have obvious disadvantages, for the ERC-20 way, one needs to create a separate ERC-20 contract for each different value/combination of customizable properties, which is practically unacceptable. For the ERC-721 way, there is no quantitative relationship between the representing token and the represented underlying assets, hence significantly reduces the liquidity and manageability of the token.
	
The most simple and direct way to solve the problem is to add an amount property directly to ERC-721 token, making it best for both property customization and semi-fungibility. Nevertheless, the ERC-721 compatibility will help the new standard easily utilizing existing infrastructures, resulting in fast adoptance.

	

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

**Every contract implementing the VNFT standard MUST implement the `VNFT` interfaces as follows:**

```solidity
pragma solidity 0.7.6;

/**
 * @title VNFT Versatile Non-Fungible Token Standard
 * @dev See https://eips.ethereum.org/EIPS/eip-VNFT
 * Note: the ERC-165 identifier for this interface is ?????
 */
interface VNFT /* is ERC721 */{

    /**
     * @dev This emits when partial units of a token are transferred to another.
     * @param _from The address of the owner of `_tokenId`
     * @param _to The address of the owner of `_targetTokenId`
     * @param _tokenId The token to partially transfer
     * @param _targetTokenId The token to receive the units transferred
     @ @param _transferUnits The amount of units to transfer
     */
    event PartialTransfer(address indexed _from, address indexed _to, uint256 indexed _tokenId, uint256 _targetTokenId, uint256 _transferUnits);

    /**
     * @dev This emits when a token is split into two.
     * @param _owner The address of the owner of both `_tokenId` and `_newTokenId`
     * @param _tokenId The token to be split
     * @param _newTokenId The new token created after split
     @ @param _splitUnits The amount of units to be split from `_tokenId` to `_newTokenId`
     */
    event Split(address indexed _owner, uint256 indexed _tokenId, uint256 _newTokenId, uint256 _splitUnits);
    
    /**
     * @dev This emits when a token is merged into another.
     * @param _owner The address of the owner of both `_tokenId` and `_targetTokenId`
     * @param _tokenId The token to be merged into `_targetTokenId`
     * @param _targetTokenId The token to receive all units of `_tokenId`
     @ @param _mergeUnits The amount of units to be merged from `_tokenId` to `_targetTokenId`
     */
    event Merge(address indexed _owner, uint256 indexed _tokenId, uint256 indexed _targetTokenId, uint256 _mergeUnits);
    
    /**
     * @dev This emits when the approved units of the approved address for a token is set or changed.
     * @param _owner The address of the owner of the token
     * @param _approved The address of the approved operator
     * @param _tokenId The token to approve
     @ @param _approvalUnits The amount of approved units for the operator
     */
    event ApprovalUnits(address indexed _owner, address indexed _approved, uint256 indexed _tokenId, uint256 _approvalUnits);

    /**
     * @dev Find the slot of a token.
     * @param _tokenId The identifier for a token
     * @return The slot of the token
     */
    function slotOf(uint256 _tokenId)  external view returns(uint256);

    /**
     * @dev Count all tokens holding the same slot.
     * @param _slot The slot of which to count tokens
     * @return The number of tokens of the specified slot
     */
    function supplyOfSlot(uint256 _slot) external view returns (uint256);

    /**
     * @dev Find the number of decimals a token uses for units - e.g. 6, means the user representation of the units of a token can be calculated by dividing it by 1,000,000.
     * @return The number of decimals for units of a token
     */
    function decimals() external view return (uint8);
    /**
     * @dev Enumerate all tokens of a slot.
     * @param _slot The slot of which to enumerate tokens
     * @param _index The index in the token list of the slot
     * @return The id for the `_index`th token in the token list of the slot
     */
    function tokenOfSlotByIndex(uint256 _slot, uint256 _index) external view returns (uint256);

    /**
     * @dev Find the amount of units of a token.
     * @param _tokenId The token to query units
     * @return The amount of units of `_tokenId`
     */
    function unitsInToken(uint256 _tokenId) external view returns (uint256);

    /**
     * @dev Set or change the approved units of an operator for a token.
     * @param _to The address of the operator to be approved
     * @param _tokenId The token to approve
     * @param _units The amount of approved units for the operator
     */
    function approve(address _to, uint256 _tokenId, uint256 _units) external;

    /**
     * @dev Find the approved units of an operator for a token.
     * @param _tokenId The token to find the operator for
     * @param _spender The address of an operator
     * @return The approved units of `_spender` for `_tokenId`
     */
    function allowance(uint256 _tokenId, address _spender) external view returns (uint256);

    /**
     * @dev Split a token into several by separating its units and assigning each portion to a new created token.
     * @param _tokenId The token to split
     * @param _units The amounts to split, i.e., the units of the new tokens created after split
     * @return The ids of the new tokens created after split
     */
    function split(uint256 _tokenId, uint256[] calldata _units) external returns (uint256[] memory);

    /**
     * @dev Merge several tokens into one by merging their units into a target token before burning them.
     * @param _tokenIds The tokens to merge
     * @param _targetTokenId The token to receive all units of the merged tokens
     */
    function merge(uint256[] calldata _tokenIds, uint256 _targetTokenId) external;

    /**
     * @dev Transfer partial units of a token to a newly created token. When transferring to a smart contract, the caller SHOULD check if the recipient is capable of receiving VNFTs.
     * @param _from The address of the owner of the token to transfer
     * @param _to The address of the owner the newly created token
     * @param _tokenId The token to partially transfer
     * @param _units The amount of units to transfer
     * @return The token created after transfer containing the transferred units
     */
    function transferFrom(address _from, address _to, uint256 _tokenId, uint256 _units) external returns (uint256);

    /**
     * @dev Transfer partial units of a token to a newly created token. If `_to` is a smart contract, this function MUST call `onVNFTReceived` on `_to` after transferring and then verify the return value.
     * @param _from The address of the owner of the token to transfer
     * @param _to The address of the owner the newly created token
     * @param _tokenId The token to partially transfer
     * @param _units The amount of units to transfer
     * @param _data
     * @return The token created after transfer containing the transferred units
     */
    function safeTransferFrom(address _from, address _to, uint256 _tokenId, uint256 _units, bytes calldata _data) external returns (uint256);

    /**
     * @dev Transfer partial units of a token to an existing token. When transferring to a smart contract, the caller SHOULD check if the recipient is capable of receiving VNFTs.
     * @param _from The address of the owner of the token to transfer
     * @param _to The address of the owner the token to receive units
     * @param _tokenId The token to partially transfer
     * @param _targetTokenId The token to receive units
     * @param _units The amount of units to transfer
     */
    function transferFrom(address _from, address _to, uint256 _tokenId, uint256 _targetTokenId, uint256 _units) external;

    /**
     * @dev Transfer partial units of a token to an existing token. If `_to` is a smart contract, this function MUST call `onVNFTReceived` on `_to` after transferring and then verify the return value.
     * @param _from The address of the owner of the token to transfer
     * @param _to The address of the owner the token to receive units
     * @param _tokenId The token to partially transfer
     * @param _targetTokenId The token to receive units
     * @param _units The amount of units to transfer
     * @param _data
     */
    function safeTransferFrom(address _from, address _to, uint256 _tokenId, uint256 _targetTokenId, uint256 _units, bytes calldata _data) external;
}
```

### VNFT Token Receiver

Smart contracts MUST implement all of the functions in the IVNFTReceiver interface to accept transfers. See “Safe Transfer Rules” for further detail. 

```javascript
 /**
        @notice Handle the receipt of a VNFT token type.
        @dev A VNFT-compliant smart contract MUST call this function on the token recipient contract, at the end of a `safeTransferFrom` after the balance has been updated.        
        This function MUST return `bytes4(keccak256("onVNFTReceived(address,address,uint256,uint256,bytes)"))` (i.e. 0xb382cdcd) if it accepts the transfer.
        This function MUST revert if it rejects the transfer.
        Return of any other value than the prescribed keccak256 generated value MUST result in the transaction being reverted by the caller.
        @param operator  The address which initiated the transfer (i.e. msg.sender)
        @param from      The address which previously owned the token
        @param id        The ID of the token being transferred
        @param units     The units of tokenId being transferred
        @param data      Additional data with no specified format
        @return           `bytes4(keccak256("onVNFTReceived(address,address,uint256,uint256,bytes)"))`
 */
interface IVNFTReceiver {
    function onVNFTReceived(address operator, address from, uint256 tokenId,
        uint256 units, bytes calldata data) external returns (bytes4);
}
```


### Token Manipulation


#### Scenarios

**_Transfer:_**
	
Since a VNFT token is ERC-721 compatible, it has ID level transfer and units level transfer.

The ID level transfer SHOULD obey ERC721 transfer rules with neither extra restrictions nor special treatments.

The units level transfer has two types of interfaces, and both have safe and unsafe versions:

```solidity
	
1. function transferFrom(address from, address to, uint256 tokenId, uint256 targetTokenId, uint256 units) external; 
   function safeTransferFrom(address from, address to, uint256 tokenId, uint256 targetTokenId, uint256 units, 
	    bytes calldata data) external; 

2. function transferFrom(address from, address to, uint256 tokenId, uint256 units) external returns (uint256 newTokenId);
   function safeTransferFrom(address from, address to, uint256 tokenId, uint256 units, bytes calldata data) 
	    external returns (uint256 newTokenId);

```
The main difference between the two kinds of interface is whether the application or the contract is responsible for determining/generating the target token ID in the transfer.

Since partial transfer of a token will possibly result in new token id creation, it's important to give the implementing contract the ability to do that. On the other hand, since part of a token can be transferred to a token with the same slot, we want to keep the flexibility for dapps to determine whether to use this ability, resulting in less contract complexity and less gas consumption.


**_Merge:_**

Several tokes with the same SLOT can be merged together using `merge(uint256[] calldata tokenIds, uint256 targetTokenId);`. `targetTokenId` should already exist, and cannot be one of `tokenIds`. After merging, `targetTokenId` owns all the units from the merged tokens, and the merged tokens will be burned.


**_Split:_**

One token can be split into several tokens, using`split(uint256 tokenId, uint256[] calldata units) returns (uint256[] memory newTokenIds);`. This will result in several newly generated tokens containing units equal to the parameter `units`.



#### Rules

**_approving rules:_**

- For being compatible with ERC721, there are three kinds of approving operations, which SHOULD be used to indicate different levels of approval.
- `setApprovalForAll` SHOULD indicate the top level of approval, the authorized operators are capable of handling all tokens, including their units, owned by the owner.
- The ID level `approve` SHOULD indicate that the `_tokenId` is approved to the operator, but not the units of that token.
- The units level `approve` with the `_units` parameter SHOULD indicate that only the specified amount of units are approved to the operator, but not the whole token.
- Any `approve` MUST revert if `msg.sender` is equal to `_to` or `_operator`.
- The units level `approve` MUST revert if `msg.sender` is not the owner of `_tokenId` nor set approval for all tokens.
- The units level `approve` MUST emit the `ApprovalUnits` event.

**_splitting rules:_**

- MUST revert if `_tokenId` is not a valid token.
- MUST revert if `msg.sender` is neither the owner of `_tokenId` nor set approval for all.
- MUST revert if the sum of all `_units` exceeds the actual amount of units in `_tokenId`.
- MUST return an array containing the ids of the generated tokens after splitting.
- MUST emit the `Split` event.

**_merging rules:_**

- MUST revert if `_targetTokenId` or any of `_tokenIds` is not a valid token.
- MUST revert if the owner of `tokenId` is not the owner of `_targetTokenId`.
- MUST revert if `msg.sender` is neither the owner of all `_tokenIds` and `targetTokenId` nor having been set approval for all.
- MUST revert if any of `_tokenIds` is equal to `_targetTokenId`.
- Each of `_tokenIds` MUST be burnt after being merged.
- MUST emit the `Merge` event.

**_transferFrom rules:_**

- The ERC721 level `transferFrom` without the `_units` and the `_targetTokenId` parameters SHOULD indicate transferring a whole token, including all of its units.
	- MUST revert unless `msg.sender` is the owner of `_tokenId`, or the ERC721 level approved address, or having been set approval for all tokens.
	- MUST revert if `_tokenId` is not a valid token
	- MUST revert if `_from` is not the current owner of `_tokenId`
	- MUST revert if `_to` is the zero address
	- MUST emit the ERC721 level `Transfer` event
	
- The VNFT level `transferFrom` without the `_targetTokenId` parameter SHOULD indicate transferring partial units to a new token of the recipient.
	- MUST revert unless `msg.sender` is the owner of `_tokenId`, or having been set approval for all tokens, or having been VNFT approved a certain number units of `_tokenId`.	
	- MUST revert if `_tokenId` is not a valid token.
	- MUST revert if `_from` is not the current owner of `_tokenId`.
	- MUST revert if `_to` is the zero address.
	- MUST revert if the transfer amount exceeds the actual amount of units in `_tokenId`.
	- MUST revert if the transfer amount exceeds the VNFT approved units limit.
	- MUST return the newly created token of the recipient containing the transferred units.
	- MUST emit the VNFT level `PartialTransfer` event.
	
- The VNFT level `transferFrom` with both the `_units` and the `_targetTokenId` parameters SHOULD indicate transferring partial units to an existing token of the recipient.
	- MUST revert unless `msg.sender` is the owner of `_tokenId`, or having been set approval for all tokens, or the transfer amount is within the VNFT approved units limit.
	- MUST revert if either `_tokenId` or `_targetTokenId` is not a valid token.
	- MUST revert if `_from` is not the current owner of `_tokenId`.
	- MUST revert if `_to` is not the current owner of `_targetTokenId`.
	- MUST revert if `_to` is the zero address.
	- MUST revert if the transfer amount exceeds the actual amount of units in `_tokenId`.
	- MUST revert if the transfer amount exceeds the VNFT approved units limit.
	- MUST emit the VNFT level `PartialTransfer` event.

**_safeTransferFrom rules:_**

- `safeTransferFrom` SHOULD be used to implement the same function as `transferFrom`, with an extra step to check if the recipient is capable of receiving VNFTs by implementing the `onVNFTReceived` interface.
- MUST obey the above rules set for `transferFrom`.
- MUST check if `_to` is a smart contract (code size > 0). If so, `safeTransferFrom` MUST call `onVNFTReceived` on `_to` and MUST revert if the return value does not match `bytes4(keccak256("onVNFTReceived(address,address,uint256,uint256,bytes)"))`


##### A solidity example of the keccak256 generated constants for the various magic values (these MAY be used by implementation):



### Metadata 


#### Metadata Extensions

VNFT metadata extensions are compatible ERC721 metadata extensions.


#### VNFT Metadata URI JSON Schema

This is the "VNFT Metadata JSON Schema" referenced above.

```json
{
    "title": "Asset Metadata",
    "type": "object",
    "properties": {
        "name": {
            "type": "string",
            "description": "Identifies the asset to which this NFT represents"
        },
        "description": {
            "type": "string",
            "description": "Describes the asset to which this NFT represents"
        },
        "image": {
            "type": "string",
            "description": "A URI pointing to a resource with mime type image/* representing the asset to which this NFT represents. Consider making any images at a width between 320 and 1080 pixels and aspect ratio between 1.91:1 and 4:5 inclusive."
        },
         "decimals": {
            "type": "integer",
            "description": "The number of decimal places that the token amount should display - e.g. 18, means to divide the token amount by 1000000000000000000 to get its user representation."
        },
        "properties": {
            "type": "object",
            "description": "Arbitrary properties. Values may be strings, numbers, object or arrays."
        }
    }
}
```




##### JSON Schema
```json
{
    "title": "Token Metadata",
    "type": "object",
    "properties": {
        "name": {
            "type": "string",
            "description": "Identifies the asset to which this token represents",
        },
        "decimals": {
            "type": "integer",
            "description": "The number of decimal places that the token amount should display - e.g. 18, means to divide the token amount by 1000000000000000000 to get its user representation."
        },
        "description": {
            "type": "string",
            "description": "Describes the asset to which this token represents"
        },
        "image": {
            "type": "string",
            "description": "A URI pointing to a resource with mime type image/* representing the asset to which this token represents. Consider making any images at a width between 320 and 1080 pixels and aspect ratio between 1.91:1 and 4:5 inclusive."
        },
        "properties": {
            "type": "object",
            "description": "Arbitrary properties. Values may be strings, numbers, object or arrays.",
        }
    }
}
```


### Approval

VNFT adds a new approval model, that is, one can approve operators to partial transfer units from a token with certain ID, the new interface is:
```function approve(address to, uint256 tokenId, uint256 units); ```



## Rationale 

### Metadata generation

Since VNFT is designed for representing underlying assets, rather than artifacts for gaming or arts, the implementation should give out the metadata directly from contract code, rather than give a URL of a server for returning metadata.


### Design decision: Keep unsafe transfer

There are mainly two reasons we keep the unsafe transfer interfaces:

1. Since VNFT is ERC721 compatible, we must keep compatibility for all wallets and contracts that are still calling unsafe transfer interfaces for ERC721 tokens.
2. We want to keep the ability that dapps can trigger business logic on contracts by simply transferring VNFT tokens to them, that is, a contract can put business logic in `onVNFTReceived` function so that it can be called whenever a token is transferred using `safeTransferFrom`. However, in this situation, an approved contract with customized transfer functions like deposit etc. SHOULD never call `safeTransferFrom` since it will result in confusion that whether `onVNFTReceived` is called by itself or other dapps that safe transfer a token to it.


### Approval

For maximum semantical compatibility with ERC721, we decided to make the relationship between two levels of approval like that:

1. Approval of an id does not result in the ability to partial transfer units from this id by the approved operator;
2. Approval of all units in a token does not result in the ability to transfer the token entity by the approved operator;
3. `setApprovalForAll` will result in the ability to partial transfer units from any token, as well as the ability to transfer any token entities of the owner.


## Backwards Compatibility
	
As mentioned at the very beginning, a VNFT contract is basically an ERC721 contract, hence it is 100% compatible with ERC721.


## References

**Standards**
- [ERC-721 Non-Fungible Token Standard](./eip-721.md)
- [ERC-165 Standard Interface Detection](./eip-165.md)
- [JSON Schema](https://json-schema.org/)
- [RFC 2119 Key words for use in RFCs to Indicate Requirement Levels](https://www.ietf.org/rfc/rfc2119.txt)

**Implementations**
- [VNFT Reference Implementation (Currently same as SOLV Vouchers)](https://github.com/solv-finance/solv-v2-voucher/tree/main/packages/solv-vnft-core/contracts)
- [SOLV Vouchers - VNFT-core](https://github.com/solv-finance/solv-v2-voucher/tree/main/packages/solv-vnft-core/contracts)

**Articles & Discussions**

- [How vnft can improve position management in uniswap-v3](https://solvprotocol.medium.com/how-vnft-can-improve-position-management-in-uniswap-v3-221ab49a8cb2)
- [What are digital assets](https://unizon.pro/en/7632.html)
- [Vnft tokens vs.erc-20 vs. erc-721](https://medium.com/solv-blog/vnft-tokens-vs-erc-20-vs-erc-721-e75843053786)

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
