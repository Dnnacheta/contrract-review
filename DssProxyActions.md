# DssProxyActions: Contract Review

contact's purpose is to execute transactions and sequences of transactions by proxy. The contract allows code execution using a persistent identity.

1. LINCENCE IDENTIFIER

```solidity
// SPDX-License-Identifier: AGPL-3.0-or-later 

```

The lincense identifier define public access/grant to this source code according to the software package data exchange standard.

2.PRAGMA

```solidity
pragma solidity >=0.5.12;

```

The pragma line tells the compiler to use .5.12 version of solidity or higher when compiling this source code.

3.INTERFACE: interface standard defines standard functions for contracts    //that are like GemLike, having same functions

```solidity
interface GemLike {
    function approve(address, uint) external;
    function transfer(address, uint) external;
    function transferFrom(address, address, uint) external;
    function deposit() external payable;
    function withdraw(uint) external;
}

```

this `GemLike` interface is template for colletral tokens on the makerDAO protocol

```solidity
interface ManagerLike {
    function cdpCan(address, uint, address) external view returns (uint);
    function ilks(uint) external view returns (bytes32);
    function owns(uint) external view returns (address);
    function urns(uint) external view returns (address);
    function vat() external view returns (address);
    function open(bytes32, address) external returns (uint);
    function give(uint, address) external;
    function cdpAllow(uint, address, uint) external;
    function urnAllow(address, uint) external;
    function frob(uint, int, int) external;
    function flux(uint, address, uint) external;
    function move(uint, address, uint) external;
    function exit(address, uint, address, uint) external;
    function quit(uint, address) external;
    function enter(address, uint) external;
    function shift(uint, uint) external;
}

```

The `ManagerLike` interface is for contracts that will manage the vat(vaults on maker protocol)

```solidity
interface VatLike {
    function can(address, address) external view returns (uint);
    function ilks(bytes32) external view returns (uint, uint, uint, uint, uint);
    function dai(address) external view returns (uint);
    function urns(bytes32, address) external view returns (uint, uint);
    function frob(bytes32, address, address, address, int, int) external;
    function hope(address) external;
    function move(address, address, uint) external;
}

```

The interface for vat contracts, which are vaults thta hold tokens in the protocol

```solidity
interface GemJoinLike {
    function dec() external returns (uint);
    function gem() external returns (GemLike);
    function join(address, uint) external payable;
    function exit(address, uint) external;
}

```

The template for contracts used for depositing `Gem` tokens to the vault

```solidity
interface GNTJoinLike {
    function bags(address) external view returns (address);
    function make(address) external returns (address);
}

```

`GNTJoinLike` define structure for GNT tokens

```solidity
interface DaiJoinLike {
    function vat() external returns (VatLike);
    function dai() external returns (GemLike);
    function join(address, uint) external payable;
    function exit(address, uint) external;
}

```

The template for contracts used for creating and decreating `Dai` stable tokens to the vault

```solidity
interface HopeLike {
    function hope(address) external;
    function nope(address) external;
}

```

This interface

```solidity
interface EndLike {
    function fix(bytes32) external view returns (uint);
    function cash(bytes32, uint) external;
    function free(bytes32) external;
    function pack(uint) external;
    function skim(bytes32, address) external;
}

```

???

```solidity
interface JugLike {
    function drip(bytes32) external returns (uint);
}

```

This Template feature a function `drip` for updating debt tokens(`ilk`) on maker protocol

```solidity
interface PotLike {
    function pie(address) external view returns (uint);
    function drip() external returns (uint);
    function join(uint) external;
    function exit(uint) external;
}

```

This Template also features a function `drip` but for updating `dai` tokens on maker protocol

```solidity
interface ProxyRegistryLike {
    function proxies(address) external view returns (address);
    function build(address) external returns (address);
}

```

Contract for

```solidity
interface ProxyLike {
    function owner() external view returns (address);
}

```

This interface contains `owner()` function, which I believe views owner address

4. Contract Common

```solidity
contract Common {
    uint256 constant RAY = 10 ** 27;

    // Internal functions
    function mul(uint x, uint y) internal pure returns (uint z) {
        require(y == 0 || (z = x * y) / y == x, "mul-overflow");
    }

    // Public functions
    function daiJoin_join(address apt, address urn, uint wad) public {
        // Gets DAI from the user's wallet
        DaiJoinLike(apt).dai().transferFrom(msg.sender, address(this), wad);
        // Approves adapter to take the DAI amount
        DaiJoinLike(apt).dai().approve(apt, wad);
        // Joins DAI into the vat
        DaiJoinLike(apt).join(urn, wad);
    }
}

```

This is the parent contract, every other contract in the source code inherits from.
`uint256 constant RAY = 10 ** 27;`

- The only state variable in the contract, `RAY` stores a uint256 value 10 raised to the power 27.

`function mul(uint x, uint y) internal pure returns (uint z)`:

- The internal function `mul` is takes in two uint256 values in its declearation. it's a pure function(meaning, it does not modofiy storage) that returns a uint256.

`require(y == 0 || (z = x * y) / y == x, "mul-overflow");`:

- The `require()` statement expects the 2nd value to equal `0` or the product of the 1st value and the 2nd value is divided by the 1st value if equals to the 2nd. if this statement is false, the `mul-overflow` error is thrown.

`function daiJoin_join(address apt, address urn, uint wad) public`

- The `daiJoin` function takes in two addresses and a uint256 value.

`DaiJoinLike(apt).dai().transferFrom(msg.sender, address(this), wad);`:

- This line transfers `wad` to from caller to the contract

`DaiJoinLike(apt).dai().approve(apt, wad);`

- this line approves `wad` token for the `apt` contract

`DaiJoinLike(apt).join(urn, wad);`

- The line deposit/withdraw unlocked collateral token(wad) into the makerDAO Vaults.

5. contract DssProxyActions is common

- the DssProxyActions contract inherits common with `is` keyword

```solidity
function sub(uint x, uint y) internal pure returns (uint z) {
        require((z = x - y) <= x, "sub-overflow");
    }

```

- The pure function `sub`, takes in two arguments and requires that result of 1st value substracted from 2nd value be less than or equal to the 1st value. if true, returns the result.

```solidity
function toInt(uint x) internal pure returns (int y) {
        y = int(x);
        require(y >= 0, "int-overflow");
    }

```

- This internal pure function `toInt` takes an argument, and coverts it to a signed interger also requires that result be greater than 0. if true, returns the result.

```solidity
function convertTo18(address gemJoin, uint256 amt) internal returns (uint256 wad) {
        wad = mul(amt, 10 ** (18 - GemJoinLike(gemJoin).dec()));
    }

```

- This internal function takes an amount(uint256) and address of `gemLike` contract. call the `mul` function
