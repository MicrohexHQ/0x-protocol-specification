# ERC20Proxy

## Transferring ERC20 tokens

The `ERC20Proxy` is responsible for transferring [ERC20 tokens](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md). Users must first approve this contract by calling the `approve` method on the token that will be exchanged. It is recommended that users approve a value of 2^256 -1. This minimizes the amount of times `approve` must be called, and also [increases efficiency](https://github.com/ethereum/EIPs/issues/717) for many ERC20 tokens.

This contract may transfer an ERC20 token if its `transferFrom` method is called from an authorized address.

```solidity
/// @dev Transfers assets. Either succeeds or throws.
/// @param assetData Byte array encoded for the respective asset proxy.
/// @param from Address to transfer asset from.
/// @param to Address to transfer asset to.
/// @param amount Amount of asset to transfer.
function transferFrom(
    bytes assetData,
    address from,
    address to,
    uint256 amount
)
    external;
```

## Encoding assetData

This contract expects ERC20 `assetData` to be encoded using [ABIv2](http://solidity.readthedocs.io/en/latest/abi-spec.html) with the following function signature. The id of this contract is `0xf47261b0`, which can be calculated as the [4 byte function selector](https://solidity.readthedocs.io/en/latest/abi-spec.html#function-selector) of the same signature.

```solidity
/// @dev Function signature for encoding ERC20 assetData.
/// @param tokenAddress Address of ERC20Token contract.
function ERC20Token(address tokenAddress)
    external;
```

In Solidity, this data can be encoded with:

```solidity
bytes memory data = abi.encodeWithSelector(
    0xf47261b0,
    erc20TokenAddress
);
```

NOTE: The `ERC20Proxy` does not enforce strict length checks for [`assetData`](#assetdata), which means that extra data may be appended to this field with any arbitrary encoding. Any extra data will be ignored by the `ERC20Proxy` but may be used in external contracts interacting with the [`Exchange`](#exchange) contract. Relayers that do not desire this behavior should validate the length of all [`assetData`](#assetdata) fields contained in [orders](#orders) before acceptance.

The `ERC20Proxy` performs the transfer by calling the token's `transferFrom` method. The transaction will be reverted if the owner has insufficient balance or if the `ERC20Proxy` does not have sufficient allowance to perform the transfer.

## Authorizations

The `ERC20Proxy` has the following interface for managing which addresses are allowed to call this contract's `transferFrom` method. These authorization functions can only be called by the contract's `owner` (currently, the [`AssetProxyOwner`](v3/v3-specification.md#assetproxyowner) contract).

```solidity
contract IAuthorizable {

    /// @dev Gets all authorized addresses.
    /// @return Array of authorized addresses.
    function getAuthorizedAddresses()
        external
        view
        returns (address[]);

    /// @dev Authorizes an address.
    /// @param target Address to authorize.
    function addAuthorizedAddress(address target)
        external;

    /// @dev Removes authorizion of an address.
    /// @param target Address to remove authorization from.
    function removeAuthorizedAddress(address target)
        external;

    /// @dev Removes authorizion of an address.
    /// @param target Address to remove authorization from.
    /// @param index Index of target in authorities array.
    function removeAuthorizedAddressAtIndex(
        address target,
        uint256 index
    )
        external;
}
```

The contracts that are currently authorized to call the `ERC20Proxy` contract's `transferFrom` method are:

- [Exchange 2.0](v2/v2-specification.md#exchange)
- [Exchange 3.0](v3/v3-specification.md#exchange)
- [MultiAssetProxy](asset-proxy/multi-asset-proxy.md)
