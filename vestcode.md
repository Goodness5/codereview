#### Repository at https://github.com/makerdao/dss-vest/blob/master/src/DssVest.sol 
<br>
<br>

####  A free software from DAI foundation. This is a smart contract called DssVest, which implements a vesting schedule for tokens. The contract allows for the creation of a vesting schedule for a given user, with a specified total amount, starting timestamp, duration, cliff duration, and optional manager. The contract also allows for claiming of vested tokens, up to a specified maximum amount, by the user or the manager (if specified).
<br>
<br>

***
```c
pragma solidity 0.6.12;

interface MintLike {
    function mint(address, uint256) external;
}

interface ChainlogLike {
    function getAddress(bytes32) external view returns (address);
}

interface DaiJoinLike {
    function exit(address, uint256) external;
}

interface VatLike {
    function hope(address) external;
    function suck(address, address, uint256) external;
    function live() external view returns (uint256);
}

interface TokenLike {
    function transferFrom(address, address, uint256) external returns (bool);
}    
