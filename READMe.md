# Reviewing the <a target="_blank" href="https://github.com/coinvise/contracts/blob/main/contracts/NFTMarketplace.sol">Coinvise Market Place </a>Contract

## About Coinvise
---
Coinvise is an open platform on Ethereum where creators can launch a social & build a tokenized community. It is built for internet creators and the future of digital collectives.
####
The coinvise `NFTMarketplace` smart contract is written such that creatives can list or unlist their NFTs for sale, have people buy, and pay in ethers or the tokens of their choice, while coinvise gets a cut from each sale.

## Imports
---
<a id="initializable"></a>
`import {Initializable} from "@openzeppelin/contracts-upgradeable/proxy/Initializable.sol"`
- This is an abstract contract which is inherited in the contract and  contains a modifier `initializer` which prevents an initializier function from being invoked twice.
----

<a id="ownableupgradeable"></a>`import {OwnableUpgradeable} from "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol"`

This is an abstract contract which is inherited in the contract and contains functions:
- <a id="ownable"></a>`__Ownable_init()` => that initializes the contract by setting the contract owner initially as the person that deploys the contract
- `renounceOwnership()` => that renounces the contract owner, by changing the owner to address(0); and
- `transferOwnership()` => that transfers ownership to the address passed to it.
----


`import {SafeMath} from "@openzeppelin/contracts/math/SafeMath.sol"`
- This is a library that contains arithmetic operations in solidity with added overflow and underflow checks.
-----


`import {IERC721} from "@openzeppelin/contracts/token/ERC721/IERC721.sol"` 

- This is an interface that's required for an ERC721 compliant contract
----

`import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol"`

- This is an interface that's required for an ERC20 compliant contract
----

`import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/SafeERC20.sol"`
- This is a library that adds safety checks to all the ERC20 functions by calling the [`_callOptionalReturn()`](#_callOptionalReturn) function on all the said functions

<a id="_callOptionalReturn"></a>
- `_callOptionalReturn()` a private function that performs a low level call to the target token address and verifies the contract of said token contains code and also asserts for success in the low-level calls made.
<a id="safeTransfer"></a>
- `safeTransfer()` => Takes in the parameters:
    - IERC20 `token` => contract instance of the ERC20 token
    - address `to` => address the token is to be sent to
    - uint256 `value` => amount of tokens to be transferred

    - calls the private function [`_callOptionalReturn()`](#_callOptionalReturn) with the defined parameters

<a id="safeTransferFromERC20"></a>
- `safeTransferFrom()` => Takes in the parameters:
    - IERC20 token => contract instance of the ERC20 token
    - address `from`
    - address `to` => address the token is to be sent to
    - uint256 `value` => amount of tokens to be transferred
    - It calls the private function [`_callOptionalReturn()`](#_callOptionalReturn) with the defined parameters
---

`import {EnumerableSet} from "@openzeppelin/contracts/utils/EnumerableSet.sol"`
- This is a library for managing bytes32 data types. It contains functions to add, remove, etc. bytes32 from a bytes32 array.
---
<a id="ReentrancyGuardUpgradeable"></a>
##
`import "@openzeppelin/contracts-upgradeable/utils/ReentrancyGuardUpgradeable.sol"`
- This is an abstract contract that contains `nonReentrant` a modifier that prevents a contract from calling itself directly or indirectly
---
 
`import {EIP712MetaTransactionUpgradeable} from "./lib/EIP712MetaTransactionUpgradeable/EIP712MetaTransactionUpgradeable.sol"`

This contract inherits the [`Initializable`](#initializable) and `EIP712BaseUggradable` contracts 
 
The `EIP712BaseUpgradeable` contract contains functions:
- <a id="initialize"></a>`_initialize()` it takes in the strings `name` and `version` as paramenters, and does a keccak hash of the abi.encode of the EIP712_DOMAIN_TYPEHASH and the name, version, the network chainId and contract address.
- `getChainID()` which uses assembly to generate the chain ID
- `getDomainSeperator()` returns the hashed value in function _initialize()
- `toTypedMessageHash()` returns the hashed message in EIP712 compatible form, so that it can be used to recover signer from signature signed using EIP712 formatted data.
<a id ="EIP712MetaTransactionUpgradeable"> </a>
`EIP712MetaTransactionUpgradeable` contains functions:
- `_initialize()` takes in the string parameters _name, and _version which makes use of the EIP712BaseUpgradeable's initialize function.
- `executeMetaTransaction()` => a  ` payable`  public function that:
    - takes in the `address` of the user, the function signature in `bytes`, the sigR in  `bytes32`, sigS in `bytes32` and the sigV in `uint8` as parameters
    - compares the function signature from the signature generated from the user to the function signature being called in the contract, passes the check if `true`
    - verifies the signer as the user using the verify function that calls the ecrecover function that returns the address associated with the public key from elliptic curve signature or return zero on error.
    - lastly, after all checks are passed, it calls the function in the function signature and requires a success is gotten from said function call, otherwise, reverts the whole operation.
<br>
<br>

# Contract `NFTMarketPlace`
It inherits contracts [`Initializable`](#initializable), [`OwnableUpgradeable`](#ownableupgradeable), [`EIP712MetaTransactionUpgradeable`](#EIP712MetaTransactionUpgradeable) and [`ReentrancyGuardUpgradeable`](#ReentrancyGuardUpgradeable).

## Events
----
<a id="NftListed"></a>
##
event `NftListed` is emitted when a NFT is listed. It follows the structure below:

    event NftListed(
            uint256 listingId,
            address nftTokenAddress,
            uint256 nftTokenId,
            address listedBy,
            uint256 paymentType,
            uint256 paymentAmount,
            address paymentTokenAddress
    );

<a id="NftDelisted"></a>
##
event `NftDelisted` is emitted when a NFT is delisted. It follows the structure below:

    event NftDelisted(uint256 listingId);  

<a id="NftListingLiked"></a>
##
event `NftListingLiked` is emitted when a listed NFT is liked. It follows the structure below:

    event NftListingLiked(address indexed likedBy, uint256 nftListingId);

<a id="NftListingLikeReverted"></a>
##
event `NftListingLikeReverted` is emitted when a listed liked NFT is unliked. It follows the structure below:

    event NftListingLikeReverted(address indexed likedBy, uint256 nftListingId);

<a id="NftBought"></a>
##
event `NftBought` is emitted when a listed NFT is bought. It follows the structure below:

    event NftBought(address indexed buyer, uint256 nftListingId);

<a id="WithdrawnEthPremiums"></a>
##
event `WithdrawnEthPremiums` is emitted when all the contract ETH is withdrawn by the contract owner. It follows the structure below:

    event WithdrawnEthPremiums(address indexed recipient, uint256 amount);

<a id="WithdrawnErc20Premiums"></a>
##
event `WithdrawnErc20Premiums` is emitted when a all the contract ERC20 balance of a particular token is withdrawn by the contract owner. It follows the structure below:

    event WithdrawnErc20Premiums(
        address indexed recipient,
        IERC20 erc20Token,
        uint256 amount
    );

## Data Structures
----
<a id="NftListing"></a>
##
- `NftListing`: This is a struct that contains details of each NFT listing. It includes:
    - the listing id
    - the nft Token address,
    - the nft token ID
    - the address of the EOA that listed it
    - the payment type (this is defaultly 1 as an ERC20 token, or 0 as Ethers)
    - payment amount expected; and
    - the address of the ERC20 token given in exchange.
    
    ##
       {
        uint256 listingId;
        address nftTokenAddress;
        uint256 nftTokenId;
        address  payable listedBy;
        uint256 paymentType;
        uint256 paymentAmount;
        address paymentTokenAddress;
        }


## Modifiers
<a id="pausible"></a>
##
---
    pausable
- it requires `isPaused` to be `false`, otherwise, it reverts the whole operation.

<a id="onlyOwnerMeta"></a>
##
    onlyOwnerMeta
- requires that `msg.sender` is equal to assigned owner in the [`OwnableUpgradeable`](#ownableupgradeable) contract, otherwise, reverts the whole operation

## Functions
---
    
    initialize()
    
- This public function takes in the :
    - `_premiumPercentage` => premium percentage, 
    - `_premiumPercentageDecimals` => premium percentage decimals,
    - `_nftListingFee` => a listing fee,
    - `whiteListedTokens` => an array of whitelisted tokens. 
    ##
- It uses a modifier `initializer` which was inherited from `Initializable` contract and ensures the function is only called once.
- It calls the function [`__Ownable_init()`](#ownable) from `OwnableUpgradeable` contract to ensure this function is only called once.
- It also calls the function [`_initialize()`](#initialize) from `EIP712MetaTransactionUpgradeable` contract.
- It then asigns the `_premiumPercentage`, `_premiumPercentageDecimals` and `_nftListingFee` to their corresponding state variables.
- It contract is also defined as not paused, by setting the `isPaused` variable to false, and the `isPaymentTokenWhiteListActive` is set to true
- Finally, there's a loop through the array of `whiteListedTokens`, where the mapping called `isPaymentTokenWhiteListed` takes in the addresses to be whitelisted and maps them to a boolean of `true`.

##
    pauseMarketplace()
- This ` external`  function uses the modifier [`onlyOwnerMeta`](#onlyOwnerMeta) and sets the `isPaused` variable to true, thereby decativating the functionalities of the contract.

##
    resumeMarketplace()
- This ` external`  function uses the modifier [`onlyOwnerMeta`](#onlyOwnerMeta) and sets the `isPaused` variable to false, thereby activating the functionalities of contract.

##
    setIsWhiteListActive()
- This ` external`  function takes in a boolean as a parameter, uses the modifier [`onlyOwnerMeta`](#onlyOwnerMeta) and sets the `isPaymentTokenWhiteListActive` variable to the parameter.

##
    whiteListTokens()
- This ` external`  function takes in a an array of addresses to be whitelisted as a parameter
- It uses the modifier [`onlyOwnerMeta`](#onlyOwnerMeta)
- It loops through the parameter array, and sets the mapping `isPaymentTokenWhiteListed` of each address to true, thereby whitelisting each address.

##
    blackListTokens()
- This ` external`  function takes in a an array of addresses to be blacklisted as a parameter. 
- It uses the modifier [`onlyOwnerMeta`](#onlyOwnerMeta)
- It loops through the parameter array, and sets the mapping `isPaymentTokenWhiteListed` of each address to false, thereby blacklisting each address.

##
    setPremiumPercentage()
- This ` external`  function takes in the numbers: premium percentage and premium percentage decimals as parameters
- It uses the modifier [`onlyOwnerMeta`](#onlyOwnerMeta)
- It updates the state variables `premiumPercentage` and  `premiumPercentageDecimals` accordingly.

<a id="_calculateCut"></a>

##
    _calculateCut()
- This  ` internal`   ` view`  function takes in the number `amount` as a parameter and calculates the Coinvise's cut of the listed amount of an NFT.
- It returns the multiplication of the `amount` parameter and the `premiumPercentage`, divided by the multiplication of the exponent of (`premiumPercentageDecimals` + 2).

##
    getCurrentNftListingIds()
- This ` external`   ` view`  function returns an array of the IDs of all the listings currently active on the market.

##
    getCurrentNftListingCount()
- This ` external`   ` view`  function returns the length of the array of the IDs of all the listings currently active on the market.

##
    listNftInEth()
- This ` external`   ` payable`  function takes in the:
  - `_nftTokenAddress` => address of the nft contract,
  - `_nftTokenId` => the nft token id, and 
  - `_priceInWei` => the price of the NFT in wei
- It uses the modifier `nonReentrant` from [`ReentrancyGuardUpgradeable`](#ReentrancyGuardUpgradeable)
- It uses the modifier [`pausible`](#pausible)
- It assigns msg.sender to an local variable `_sender`
- It creates a local contract instance of the nft token address passed as a parameter into the function called `nftContract`
- it requires the `NFTMarketplace` contract is approved by the nft owner, otherwise reverting the whole operation.
- It requires the `_priceInWei` is greater than 0, i.e, the NFT can't be sold for free.
- It requires that msg.value must be equal to the `nftListingFee`. i.e the listing fee must be paid before an NFT is listed.
- a struct instance of the [`NftListing`](#NftListing) is then created in memory.
- the `listingId` is gotten from the `_getNextNftListingId()` function
- the `nftTokenAddress`, `nftTokenId` and `_priceInWei` are assigned from its corresponding values in the parameter
- the `listedBy` is assigned to the local variable `_sender`
- the `paymentType` is given the number 0, signifying the payment type to be in Ethers.
- the `paymentTokenAddress` is assigned to `address(0)`, indicating the absence of a payment token address since the payment is made in ethers.
- the array of listed NFTtoken ids, `_listedNftTokenIds` is updated with the [`NftListing`](#NftListing)'s `listingId`
- the mapping, `nftListingById` of the `listingId` is assigned to the locally defined struct, [`NftListing`](#NftListing)
- an event [`NftListed`](#NftListed) is emitted with its corresponding argument values.

##
    listNftInErc20()
- This ` external`  function takes in the:
  - `_nftTokenAddress` => address of the nft contract,
  - `_nftTokenId` => the nft token id, and 
  - `_erc20TokenAddress` => the ERC20 token address
  - `_priceInErc20Token` => the price of the NFT in ERC20 tokens
- It uses the modifier `nonReentrant` from [`ReentrancyGuardUpgradeable`](#ReentrancyGuardUpgradeable)
- It uses the modifier [`pausible`](#pausible)
- It assigns msg.sender to an local variable `_sender`
- It checks if the `isPaymentTokenWhiteListActive` is true, then requires that the mapping, `isPaymentTokenWhiteListed` of the `_erc20TokenAddress` is true (checks to see if the token address passed has been whitelisted), otherwise reverts the whole operation.
- It creates a local contract instance of the nft token address passed as a parameter into the function called `nftContract`
- it requires the `NFTMarketplace` contract is approved by the nft owner, otherwise reverting the whole operation.
- It requires the `_priceInErc20Token` is greater than 0, i.e, the NFT can't be sold for free.
- a struct instance of the [`NftListing`](#NftListing) is then created in memory.
- the `listingId` is gotten from the `_getNextNftListingId()` function
- the `nftTokenAddress`, `nftTokenId` and `_priceInErc20Token` are assigned from its corresponding values in the parameter
- the `listedBy` is assigned to the local variable `_sender`
- the `paymentType` is given the number 1, signifying the payment type to be in ERC20 tokens.
- the `paymentTokenAddress` is assigned to `_erc20TokenAddress`, indicating the presence of the payment token address.
- the array of listed NFTtoken ids, `_listedNftTokenIds` is updated with the [`NftListing`](#NftListing)'s `listingId`
- the mapping `nftListingById` of the `listingId` is assigned to the locally defined struct, [`NftListing`](#NftListing)
- an event [`NftListed`](#NftListed) is emitted with its corresponding argument values.

<a id="_unlistNft"></a>

##
    _unlistNft()
- This  ` internal`  function takes in a NFTListing Id and deletes it from the contract's records.
- It removes the the NFT-listing id from the array of `_listedNftTokenIds`
- It deletes the mapping, `nftListingById` of the NFTListing Id
- it emits an event [`NftDelisted`](#NftDelisted)

##
    unlistNftByListingCreator()
- This  ` external`  function is to be called by it's listing creator 
- It takes in a NFTListing Id and deletes it from the contract's records before it's bought by someone.
- It requires the `msg.sender` (person calling the function) is the person that listed the NFT, by checking the mapping, `nftListingById` of the NFTlisting id, which returns the [`NftListing`](#NftListing) of the that id.
- The address `listedBy` of the struct is compared to the address of the caller of the function.
- If `false` the whole operation is reverted.
- Else, it calls the  ` internal`  function [`_unlistNft()`](#_unlistNft) and delists the NFT.

##
    buyNftInEth()
- This  ` external`  ` payable` function takes in a NFTListing Id and transfers the NFT attached to the id to the buyer (address calling the function) if all the conditions are met.
- It gets a memory instance of the struct, [`NftListing`](#NftListing), which returns the details of the `_nftListingId`'s listing, with the mapping, `nftListingById` of `_nftListingId`.
- It assigns the address of the person calling the function to a local variable `_sender`.
- It requires that `_sender` is not the address of the person calling the function, I.e the buyer cannot be the lister.
- It requires that the array of `_listedNFTTokenIds` contains the NFTListing Id passed as the parameter into the function, i.e checks to see if the NFT hs already been delisted, otherwise reverts the whole operation.
- It requires that the paymentType of the listing should be equal to 0, (i.e the payment type is of type Ethers) otherwise, reverts the whole operation.
- It validates the amount sent by requiring the `msg.value` (amount sent with the function call) is equal to the addition of listing amount and the amount got from the internal function [`_calculateCut()`](#_calculateCut) (it takes in the listing amount as its argument).
- It creates a local instance of the nftToken's contract address and requires that the address of the Coinvise NFT marketplace has been approved, otherwise, reverts the whole operation.
- It uses the `transfer` method to send the listing amount attached to the  [`NftListing`](#NftListing) to the address that listed the NFT.
- It then uses a ERC721 function `safeTransferFrom()` to send the NFT ownership from the address that listed it, to the buyer (address of the `_sender`)
 - It closes the listing the NFt by calling the  ` internal`  function [`_unlistNft()`](#_unlistNft), which unlists the NFT
- It emits the event [`NftBought`](#NftBought)

##
    buyNftInErc20Tokens()
- This  ` external` function takes in a NFTListing Id and transfers the NFT attached to the id to the buyer (address calling the function) if all the conditions are met.
- It assigns the address of the person calling the function to a local variable `_sender`.
- It gets a memory instance of the struct, [`NftListing`](#NftListing), which returns the details of the `_nftListingId`'s listing, with the mapping, `nftListingById` of `_nftListingId`.
- It requires that `_sender` is not the address of the person calling the function, I.e the buyer cannot be the lister.
- It requires that the array of `_listedNFTTokenIds` contains the NFTListing Id passed as the parameter into the function, i.e checks to see if the NFT hs already been delisted, otherwise reverts the whole operation.
- It requires that the paymentType of the listing should be equal to 1, (i.e the payment type is of type ERC20) otherwise, reverts the whole operation.
- It validates the amount sent by requiring the `msg.value` (amount sent with the function call) is equal to the addition of listing amount and the amount got from the internal function [`_calculateCut()`](#_calculateCut) (it takes in the listing amount as its argument).
- It creates a local instance of the ERC20's contract address and requires that the address of the Coinvise NFT marketplace has the allowance greater than or equal to the amount of tokens to be collected from `_sender`, otherwise, reverts the whole operation
- It creates a local instance of the nftToken's contract address and requires that the address of the Coinvise NFT marketplace has been approved, otherwise, reverts the whole operation.
- It uses the `transfer` method to send the listing amount attached to the  [`NftListing`](#NftListing) to the address that listed the NFT.
- it calls the `SafeERC20` contract function [`safeTransferFrom()`](#safeTransferFromERC20) to transfer the expected amount of tokens (addition of listing amount and the amount got from the internal function [`_calculateCut()`](#_calculateCut)) from the `_sender` to the Coinvise marketplace contract.
- It then calls the `SafeERC20` contract function [`safeTransfer()`](#safeTransfer)  to transfer only the listing amount from the Coinvise marketplace contract to the address that listed the NFT, while the Coinvise marketplace contract keeps the cut from the internal function, [`_calculateCut()`]
- It then uses a ERC721 function `safeTransferFrom()` to send the NFT ownership from the address that listed it, to the buyer (address of the `_sender`)
 - It closes the listing the NFt by calling the  ` internal`  function [`_unlistNft()`](#_unlistNft), which unlists the NFT
- It emits the event [`NftBought`](#NftBought)

##
    likeNftListing()
- This ` external`  function takes in a NFTListing Id and increases the like counter for a listing if's it's actvie
- It uses the modifier [`pausible`](#pausible)
- It assigns the address of the person calling the function to a local variable `_sender`.
- It requires that the array of `_listedNFTTokenIds` contains the NFTListing Id passed as the parameter into the function, i.e checks to see if the NFT hs already been delisted, otherwise reverts the whole operation.
- It requires the mapping `hasUserLikedNftListing` of `_sender`  to the NFTListing id, is `false` (i.e checking to see if the user has not already liked the NFT), otherwise, reverts the whole operation.
- If false, it then sets the mapping `hasUserLikedNftListing` of `_sender`  to the NFTListing id to `true` (I.e the user likes the NFT)
- it emits an event [`NftListingLiked`](#NftListingLiked)

##
    undoLikeNftListing()
- This ` external` function takes in a NFTListing Id and increases the like counter for a listing if's it's actvie
- It uses the modifier [`pausible`](#pausible)
- It assigns the address of the person calling the function to a local variable `_sender`.
- It requires that the array of `_listedNFTTokenIds` contains the NFTListing Id passed as the parameter into the function, i.e checks to see if the NFT hs already been delisted, otherwise reverts the whole operation.
- It requires the mapping `hasUserLikedNftListing` of `_sender`  to the NFTListing id, is `true` (i.e checking to see if the user has liked the NFT), otherwise, reverts the whole operation.
- If true, it then sets the mapping `hasUserLikedNftListing` of `_sender`  to the NFTListing id to `false` (I.e the user dislikes the NFT)
- it emits an event [`NftListingLikeReverted`](#NftListingLikeReverted)

##
    withdrawEthPremiums()
- This  ` external`  function takes in an address `_to` as a paramenter.
- It sends all the ETH balance of the coinvise contract to the address `to`.
- It uses the modifier [`onlyOwnerMeta`](#onlyOwnerMeta)
- It uses the `transfer` method to send all the ETH balance of the coinvise contract to the address `to`.
- it emits an event [`WithdrawnEthPremiums`](#WithdrawnEthPremiums)

##
    withdrawErc20Premiums()
- This  ` external`  function takes in an address `_to`, and an ERC20 token contract address as paramenters.
- It sends all the balance of the ERC20 tokens in the coinvise contract to the address `to`.
- It uses the modifier [`onlyOwnerMeta`](#onlyOwnerMeta)
- It uses the `SafeERC20` contract function [`safeTransfer()`](#safeTransfer)  to send the balance of the ERC20 tokens in the coinvise contract to the address `to`.
- it emits an event [`WithdrawnErc20Premiums`](#WithdrawnErc20Premiums)

##
    fallback()
- This is a fallback method in the contract that allows ethers and data to be sent to the contract directly.

##
    receive()
- This is a fallback method in the contract that allows ethers to be sent to the contract directly.
