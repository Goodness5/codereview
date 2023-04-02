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

#### The first contract is an Abstract contract i.e none of the functions were implemented or used which means the contract can also function as a reusable code source almost like a library.
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
    the first modifier is the locked modifier which  requires that the locked status of each Awards is 0 and returns the system locked error if the locked isn't 0, if the locked status is 0 the modifier sets the locked value to 1 and then runs the function setting the locked value to 1 in the modifier prevents attackers from carrying out a replay attack. after the function has been executed the locked status is set to 0 again, this ensures that while the function is being executed and denies any further calls o the function.

    the second modifier is the authentication modifier, this requires that  the msg.sender is the ward with id 1 which is also the person that deploys the contract as seen in the constructor. this could also be regarded as a special priviledge and administration account.

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

**B. Abstract Contract functions:** <br>
* The rely function takes an address _usr as an argument and sets the value of wards[_usr] to 1. This grants the address _usr permission to participate in the vesting as a recipient. The auth modifier indicates that only a specific authorized user which was initialized in the constructor can call this function. A rely event is emmited with the address of the person added.
* The deny function takes an address _usr as an argument and sets the value of wards[_usr] to 0. This fuction revokes the permission of the address _usr from participating. Similar to the rely function, the auth modifier indicates that only an authorized user or contract can call this function while an event is being emmited with the adress of the revoked or denied person.
* **The file function:** The function takes two arguments: what, which is a bytes32 variable that indicates what parameter is being set, and data, which is a uint256 variable that contains the value to which the parameter is being set. the file function is basically used to set the cap value  which is why we have an IF statement that checks if the parameter being set is the cap else it reverts while if true it sets the cap variable to the uint data parameter passed in.
```solidity 
      if(what == "cap")
              cap = data; 
        else revert("DssVest/file-unrecognized-param");
```
* The next set of functions are pure functions which dosn't read or modifies the state, they are used to perform mathematical operations while checking for overflows and underflows to prevent a mistake when handling mathematical operations that dosn't return a whole number. <br>
The reqire statement after these functions ensures that the calculations does not overflow which can become a very serious bug.
    ```solidity

    function min(uint256 x, uint256 y) internal pure returns (uint256 z) {
        z = x > y ? y : x;
    }
    function add(uint256 x, uint256 y) internal pure returns (uint256 z) {
        require((z = x + y) >= x, "DssVest/add-overflow");
    }
    function sub(uint256 x, uint256 y) internal pure returns (uint256 z) {
        require((z = x - y) <= x, "DssVest/sub-underflow");
    }
    function mul(uint256 x, uint256 y) internal pure returns (uint256 z) {
        require(y == 0 || (z = x * y) / y == x, "DssVest/mul-overflow");
    }
    function toUint48(uint256 x) internal pure returns (uint48 z) {
        require((z = uint48(x)) == x, "DssVest/uint48-overflow");
    }
    function toUint128(uint256 x) internal pure returns (uint128 z) {
        require((z = uint128(x)) == x, "DssVest/uint128-overflow");
    }
    ```
* **The create function:** This function initializes a new vesting award by taking in the parameters and setting them to the struct by making use of the id to map an award. the function can only be called or executed by the ward with id 1 i.e the address that deploys the contract as is set in the constructor This function creates a new vesting award for a given user. It takes in several parameters:<br>
    _usr: the address of the user who will receive the award<br>
    _tot: the total amount of tokens to be awarded<br>
    _bgn: the beginning of the vesting period, measured in Unix time (seconds since the Unix epoch)<br>
    _tau: the duration of the vesting period, also measured in seconds <br>
    _eta: the cliff period, which is the amount of time that must pass before any tokens can be awarded <br>
    _mgr: the address of the manager who will oversee the award <br>
    <br>
    There are also several require statements to  check each of the users input to ensure that they are valid. This is a sanity check as the saying goes "you can't trust your users." There is also a require statement that ensures that the vesting duration is not greater than 20 years. this can be altered based on various design decisions.
    After the sanity checks then we increment the Id from the last used Id by 1 using the command id = ++ids;, ids is the state variable that keeps track of the ids while id is a local variable decleared in the function and is being returned. After the id is being set the struct is initialized with those parameters accepted from the user and the struct is accessed using the mapping of uint to struct which takes in the id as the key to the struct.
    After setting the values in the struct, an event is emmitted with the _usr address and the Id of the Award. we can also notice the conversion of the uint256 values to uint48 this happens since the Award struct is set to accept thhose values as uint48 and the user's end dosn't recorgnize this and may pass in a value greater than uint48 so the contract handles this  

*  **The Vest:** there are 3 vest functions
    ```solidity
    function vest(uint256 _id) external {
        _vest(_id, type(uint256).max);
    }


    function vest(uint256 _id, uint256 _maxAmt) external {
        _vest(_id, _maxAmt);
    }

    function _vest(uint256 _id, uint256 _maxAmt) internal lock {
        Award memory _award = awards[_id];
        require(_award.usr != address(0), "DssVest/invalid-award");
        require(_award.res == 0 || _award.usr == msg.sender, "DssVest/only-user-can-claim");
        uint256 amt = unpaid(block.timestamp, _award.bgn, _award.clf, _award.fin, _award.tot, _award.rxd);
        amt = min(amt, _maxAmt);
        awards[_id].rxd = toUint128(add(_award.rxd, amt));
        pay(_award.usr, amt);
        emit Vest(_id, amt);
    }
    ```
    - The first and second external vest functions implements the function overload properties i.e same function name with different input parameters
    the first vest function provides the max-amount for the _vest function as the highest number available in the uint on the EVM this can be used when we want to allow unlimmited number while the second vest allows us to set a limit
    - the _vest function contains the main logic that vests the token it is an internal function that takes in 2 parameters the Id and max amount. the id is used to retrieve the Award details from the Award struct using the mapping of id to struct. The function implements the lock modifier which has been explained above. <br>
    The function firsts checks if the _usr address in the struct is not a zero address if it return a zero address this means that there is no Award with that id number i.e the data trying to be fetched does not exist, the second check requires that the msg.sender i.e the person calling the function is the _usr address i.e the vesting recipient the amount is then calculated using the unpaid function which will be discussed later. the amount is then passed to the min function alongside the maximum amount the user can get the min function returns the smaller value among these 2 parameters. this process helps to ensure that a user does not claim above the highest claimable at a time. then the rxd value is updated by adding the amount recieved by the user to the rxd values the rxd variable keeps track of the amount the user has received. then the amount get paid to the caller's address using the pay function which we will discuss as we progress.

* **function accrued:** <br>
**A.** The first function returns the amount of tokens accrued over a period of time it is external view function that only reads from the state and does not modify the state. it makes use of the second accrued function which is internal and pure to make the main calculations.<br>
**B.** the second accrued function is an internal and pure function i.e it neither reads nor modifies the state. The function first checks if the current time is before the vesting period starts. If so, it returns 0. If the current time is after or equal to the vesting period end time, it returns the total amount of tokens to be vested.
If the current time is between the vesting period start and end times, the function calculates the amount of tokens that have vested so far. It does this by taking the total amount of tokens to be vested and multiplying it by the time elapsed between the vesting period start and the current time, and then dividing that by the total duration of the vesting period. The resulting value represents the proportion of the total tokens that have vested so far.

* **Function unpaid** <br>
There are 2 different  unpaid functions<br>
    - The external view unpaid function: this function returns the amount of claimable rewards of a particular award by taking in the id as parameter and using it to send the award struct information to the second internal unpaid function. The function also checks if the address of _usr is a zero address this will ensure that the award has been created.
    - the internal pure function which takes in the parameters initialzed from the external unpaid function and returnsthe amount that has been accumulated after calculating based on the time.

* **Restrict and Unrestrict function**
    * The Restrict function takes in an award Id and also implemets the lock modifier to prevent reentrancy attacks this function also reuires admin access i.e requires either the owner of the award or the contract owner to call and sets the res variable in the struct to 1 which is also the restriction variable i explained above.
    * The unrestrict does the same thing with the restrict function the only difference is that instead of setting the res variable to 1 sets it to 0 instead.

* **Yank function:** 
    we have 3 yank functions
    ```solidity
    function yank(uint256 _id) external {
        _yank(_id, block.timestamp);
    }
    ``` 
    this yank function is an external function that calls the internal _yank function passing in the id it took as parameter and the block.timestamp which returns the current epoch time. the yank function is used to remove a vesting immediately from the contract.

    ```solidity
     function yank(uint256 _id, uint256 _end) external {
        _yank(_id, _end);
    }
    ```
    this yank function also removes the vesting from the contract but not immediately but at the time set by the caller of the function.

    **the _yank function** is an internal function that contains the logic that removes a vesting from the contract it takes in the vest id and the time it should be removed as parameters and ensures that the caller is either an admin or the vest owner, it also checks if the address _usr is not address zero i.e the vest actually exists. it checks if the _end time is less than the block.timestamp if true it resets the end time to the current block.timestamp this will make up for every time change during the transaction as the evm is relatively slow. if the new end time is less than the original end time, the vesting contract is modified to end at the new end time. If the new end time is less than the original cliff time, the vesting contract is modified to end and cliff at the new end time and the total amount of tokens vesting is set to zero. If the new end time is less than the original vesting end time but greater than or equal to the original cliff time, the vesting contract is modified to end at the new end time and the total amount of tokens vesting is set to the sum of the previously vested tokens and the new amount of tokens that would vest from the modified vesting contract.

    it then emmits the yank event with the award id and the new endtime.

* **The Move function:** <br>
This function allows the user who owns the vesting contract with the given _id to move ownership of the contract to a new address _dst.
The function first checks that the caller of the function is the current owner of the vesting contract. It then checks that the new owner address _dst is not the zero address.
If both checks pass, the function updates the usr field of the Award struct associated with the given _id to the new owner address _dst. Finally, it emits a Move event with the _id and _dst as arguments to signal the ownership transfer.

* **Valid function:** <br>
The function checks if the amount of rewards received so far (awards[_id].rxd) is less than the total amount of rewards that the contract is supposed to distribute (awards[_id].tot). If the amount of rewards received is less than the total amount, the function returns true, indicating that the contract is still valid and has rewards to distribute. Otherwise, the function returns false, indicating that the contract has distributed all of its rewards and is no longer valid.

* **the pay function:** <br>
This fuction contains no logic and the developer is expected to write their payment logic. the function takes 2 parameters the _guy(recipient) and the _amt(amount to be sent).
****


# DSSVESTMINTABLE Contract 
 This contract is the contract that adds minting functionality when a user comes to claim their vested tokens, it inherits the DSSVest contract.

 ## variables
 There is only one state variable (gem) which is of datatype ``Mintlike interface``
 the gem variable is set to immutable and is deployed with the contract in the constructor. the constructor takes in an address parameter which is the address of the token being vested, it checks if it isn't address zero. and then passes the address into the Mintlike interface then assigns it to the gem variable since the gem variable is of type `Mintlike`

 There is only one function added to the contract the **pay function** which i expalined above but the logic is being implemented and the gem address is calling the mint function in the gem address and minting to the _guy address the _amt(amount).

 # DssVestSuckable Contract 
 The DssVestSuckable contract is a contract that inherits from the DssVest contract and adds functionality to handle the payment of ERC-20 Dai tokens by sucking them from the MakerDAO Vat using the suck function and then exiting them to the specified recipient using the exit function of the DaiJoin contract.

 ## variables
* RAY is a constant with the value of 10^27.
* chainlog is an instance of the ChainlogLike contract interface, which represents the MCD chainlog contract.
* vat is an instance of the VatLike contract interface, which represents the MCD vat contract.
* daiJoin is an instance of the DaiJoinLike contract interface, which represents the MCD Dai join contract.

    **Notes:** MCD stands for "Multi-Collateral Dai". It is a decentralized stablecoin system built on the Ethereum blockchain. The system is designed to allow users to deposit various types of collateral, such as Ether and other ERC-20 tokens, in exchange for generating the stablecoin Dai. MCD also allows for the creation of new collateral types through a community-driven governance process. The system is governed by MKR token holders who participate in making decisions on system upgrades, collateral risk parameters, and other key governance aspects.

The constructor of the contract takes in an address _chainlog which is used to set the chainlog variable. The chainlog is a contract that acts as a registry of all the system's contracts and addresses 
The require function is used to check against weather the chainlog address is not address zero then saves the _chainlog value to the chainlog variable after being typecasted with the chainloglike interface.
```solidity
   constructor(address _chainlog) public DssVest() {
        require(_chainlog != address(0), "DssVestSuckable/Invalid-chainlog-address");
        ChainlogLike chainlog_ = chainlog = ChainlogLike(_chainlog);
        VatLike vat_ = vat = VatLike(chainlog_.getAddress("MCD_VAT"));
        DaiJoinLike daiJoin_ = daiJoin = DaiJoinLike(chainlog_.getAddress("MCD_JOIN_DAI"));

        vat_.hope(address(daiJoin_));
    }
```
The ChainlogLike contract is used to obtain the addresses of other contracts that are part of the MCD system. The getAddress() function is used to obtain the address of the MCD_VAT contract, which is then used to initialize the VatLike contract instance.

The DaiJoinLike contract is used to interact with the MCD_JOIN_DAI contract, which is responsible for converting Dai tokens to the internal gem token used by the MCD system.
The hope() function is called on the VatLike contract to give permission to the DaiJoinLike contract to interact with the VatLike contract.

## **fuctions**

**pay function** <br>
This pay function transfers Dai to the recipient _guy by first "sucking" the Dai from the MakerDAO Vat contract using vat.suck, and then transferring it to the _guy address using daiJoin.exit.

The vat.suck function takes three arguments: the first is the address of the Vow contract, which is where excess Dai is sent in the MakerDAO system. The second is the address of the contract that will receive the Dai being sucked, which in this case is the DssVestSuckable contract itself. The third argument is the amount of Dai to suck from the Vat, which is calculated by multiplying _amt by the constant RAY, which has a value of 10^27. This converts the _amt from units of Dai (where 1 Dai = 10^18) to units of "rad", where 1 rad = 10^45.

Finally, the daiJoin.exit function is called to transfer the Dai to the _guy address. This function takes two arguments: the first is the address to which the Dai will be transferred (in this case, _guy), and the second is the amount of Dai to transfer.