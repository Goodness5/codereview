#### Repository at https://github.com/makerdao/dss-vest/blob/master/src/DssVest.sol 
<br>

####  A free software from DAI foundation. This is a smart contract called DssVest, which implements a vesting schedule for tokens. The contract allows for the creation of a vesting schedule for a given user, with a specified total amount, starting timestamp, duration, cliff duration, and optional manager. The contract also allows for claiming of vested tokens, up to a specified maximum amount, by the user or the manager (if specified).
<br>
<br>

***
```solidity
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
```

#### The code block above begins the code and it shows the various interfaces being decleared and used in the vesting code

* MintLike interface allows the contract to mint new tokens.
* ChainlogLike interface provides an address registry for the system.
* DaiJoinLike interface allows the contract to exit DAI from the system.
* VatLike interface allows the contract to manage the state of the system's accounting engine.
it contains 3 functions the <br> **hope(address) external**: This function takes in one parameter address which is the address of another smart contract or an externally owned account. It is used to give permission to the provided address to interact with the smart contract that implements this interface. The address that is passed as a parameter to this function is allowed to call certain functions on the contract.<br>
**suck(address, address, uint256) external:** This function takes in three parameters: src, dst, and wad. src is the address of the token holder who wants to transfer their tokens, dst is the address of the recipient who will receive the tokens, and wad is the amount of tokens to be transferred. This function is used to transfer tokens from one address to another. The contract that implements this interface must have the required tokens before calling this function.<br>
**live() external view returns (uint256):** This function does not take any parameters and returns a uint256 value. It is used to retrieve the current timestamp or block number of the blockchain where the contract is deployed. The view keyword indicates that this function does not modify the state of the contract.<br><br>
* TokenLike interface allows the contract to transfer tokens and check the allowance of an account by implementing the transferFrom function.

#### The codebase is an Abstract contract i.e none of the functions were implemented or used which means the contract can also function as a reusable code source almost like a library.
The contract was already decleared as an abstract contract in the contract initialization 
```solidity

abstract contract DssVest {
    // --- Data ---
    mapping (address => uint256) public wards;

    struct Award {
        address usr;   // Vesting recipient
        uint48  bgn;   // Start of vesting period  [timestamp]
        uint48  clf;   // The cliff date           [timestamp]
        uint48  fin;   // End of vesting period    [timestamp]
        address mgr;   // A manager address that can yank
        uint8   res;   // Restricted
        uint128 tot;   // Total reward amount
        uint128 rxd;   // Amount of vest claimed
    }
    mapping (uint256 => Award) public awards;

    uint256 public cap; // Maximum per-second issuance token rate

    uint256 public ids; // Total vestings

    uint256 internal locked;

    uint256 public constant  TWENTY_YEARS = 20 * 365 days;

    // --- Events ---
    event Rely(address indexed usr);
    event Deny(address indexed usr);

    event File(bytes32 indexed what, uint256 data);

    event Init(uint256 indexed id, address indexed usr);
    event Vest(uint256 indexed id, uint256 amt);
    event Restrict(uint256 indexed id);
    event Unrestrict(uint256 indexed id);
    event Yank(uint256 indexed id, uint256 end);
    event Move(uint256 indexed id, address indexed dst);
```
The code snippet shows the contract, state variables and events initialization
<br>
* The first state variable which is a mapping of public visibility helps keep track of the address that has been granted permission by the contract owner to create new vesting awards. it returns the award Id of the created award which is in turn used to access the award struct VIA the **awards** mapping which takes in a uint award id as a parameter to return the **Awards** struct.

* **The Award Struct:** this contains 8 variables<br>
    * the address usr: this is the address of the recipient of the vesting reward. the usr address will receive the vested tokens at the end of the stipulated time.
    * bgn: This variable is of type uint48 and represents the start of the vesting period. The value of this variable is a timestamp in Unix time (number of seconds since January 1, 1970) that indicates when the vesting period begins.
    * clf: This variable is also of type uint48 and represents the cliff date. The cliff date is the point in time after which the recipient becomes eligible to receive a portion of the vested tokens. The value of this variable is also a timestamp in Unix time.

    * fin: This variable is of type uint48 and represents the end of the vesting period. The value of this variable is a timestamp in Unix time that indicates when the vesting period ends.

    * mgr: This variable is of type address and represents the address of a manager who has the ability to "yank" (i.e., withdraw) the vested tokens before they are fully distributed to the recipient. This variable is optional and may not be present in every Award struct.

    * res: This variable is of type uint8 and represents whether the vested tokens are restricted in any way. The value of this variable is a flag that indicates whether the tokens are subject to any restrictions or limitations.

    * tot: This variable is of type uint128 and represents the total amount of tokens that are subject to vesting. This is the total amount of tokens that will be distributed to the recipient over the course of the vesting period.

    * rxd: This variable is also of type uint128 and represents the amount of tokens that have already been claimed or received by the recipient. This value is updated as the recipient receives more tokens over time.
        
we also have variables like the <br> **cap** which represents the maximum amount of tokens that can be issued per seconds<br>
**Ids** which represents the total number of vestings that has been created and it's incremented by 1 after each vesting creation.<br>
**locked** which is used to prevent replay attacks 
**TWENTY_YEARS** which is a constant and is used to store the number of days in 20 years.

## The events
**event Rely(address indexed usr)** - This event is emitted when an address is granted permission to create new vesting awards by being added to the wards mapping.

**event Deny(address indexed usr)** - This event is emitted when an address has its permission to create new vesting awards revoked by being removed from the wards mapping.

**event File(bytes32 indexed what, uint256 data)** - This event is emitted when a contract parameter is updated using the file function. The what parameter represents the name of the parameter that was updated, and the data parameter represents the new value of the parameter.

**event Init(uint256 indexed id, address indexed usr)** - This event is emitted when a new vesting award is created using the init function. The id parameter represents the unique identifier of the vesting award, and the usr parameter represents the address of the recipient of the vesting award.

**event Vest(uint256 indexed id, uint256 amt)** - This event is emitted when a portion of a vesting award is claimed by the recipient using the vest function. The id parameter represents the unique identifier of the vesting award, and the amt parameter represents the amount of tokens that were claimed.

**event Restrict(uint256 indexed id)** - This event is emitted when a vesting award is restricted from being claimed by the recipient using the restrict function. The id parameter represents the unique identifier of the vesting award.

**event Unrestrict(uint256 indexed id)** - This event is emitted when a vesting award is unrestricted and can be claimed by the recipient using the unrestrict function. The id parameter represents the unique identifier of the vesting award.

**event Yank(uint256 indexed id, uint256 end)** - This event is emitted when a vesting award is yanked by the contract owner using the yank function. The id parameter represents the unique identifier of the vesting award, and the end parameter represents the timestamp at which the vesting period is ended and the remaining tokens are returned to the contract owner.

**event Move(uint256 indexed id, address indexed dst)** - This event is emitted when a vesting award is transferred to a new recipient using the move function. The id parameter represents the unique identifier of the vesting award, and the dst parameter represents the address of the new recipient.

## constructor
   `the constructor for the contract  initializes the contract by adding the msg.sender to the wards mapping with an id of 1 and emmiting the Rely event`.

## modifiers
    the first modifier is the locked modifier which  requires that the locked status of each Awards is 0 and returns the system locked error if the locked isn't 0, if the locked status is 0 the modifier sets the locked value to 1 and then runs the function setting <br> the locked value to 1 in the modifier prevents attackers from carrying out a replay attack. after the function has been executed the locked status is set to 0 again, this ensures that while the function is being executed and denies any further calls o the function.

    the second modifier is the authentication modifier, this requires that  the msg.sender is the ward with id 1 which is also the person that deploys the contract as seen in the constructor

## Functions
 **A. Getter functions:** <br>there are 8 getter functions to return the information stored in the struct they are view functions i.e they don't make any changes to the state. they take in the award Id as input to access the struct and return each information in the struct we have:..
 * get USR  function
 * get BGN function
 * get CLF function
 * get FIN function
 * get MGR function
 * get RES function
 * get TOT function
 * get RXD function

`i have expalined each of these variables above`





