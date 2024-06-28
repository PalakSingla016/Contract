# Contract

This Solidity smart contract is designed to manage deposits and withdrawals of Ether, providing basic functionalities for handling funds and owner-specific operations. It includes safety checks and emits events to log significant actions.

## Contract Definition
```solidity
contract Contract {
    address public owner;
    uint public balance;
```
The contract defines two state variables:
- `owner`: stores the address of the contract owner, set during deployment.
- `balance`: keeps track of the contract's current Ether balance.

## Events
```solidity
    event Deposit(address indexed depositor, uint amount, string message);
    event Withdrawal(address indexed withdrawer, uint amount, string message);
    event ForceRevert(string message);
```
Three events are declared to log actions:
- `Deposit`: emitted when a deposit is made.
- `Withdrawal`: emitted when a withdrawal is made.
- `ForceRevert`: emitted when a force revert function is called.

## Constructor
```solidity
    constructor() {
        owner = msg.sender; // Set the contract deployer as the owner
    }
```
The constructor sets the deployer of the contract (`msg.sender`) as the owner, establishing initial ownership.

## Modifiers
```solidity
    modifier onlyOwner() {
        require(msg.sender == owner, "Caller is not the owner");
        _;
    }
```
The `onlyOwner` modifier restricts access to certain functions, ensuring that only the owner can call them. This is done by checking if the caller's address (`msg.sender`) matches the owner's address.

## Functions

### Deposit Function
```solidity
    function depositAmount(uint amount) public payable {
        require(msg.value == amount, "Sent value must match the specified deposit amount");
        require(amount > 0, "Deposit amount must be greater than zero");
        balance += amount;
        emit Deposit(msg.sender, amount, "Deposit successful");
    }
```
This function allows anyone to deposit Ether into the contract. It includes checks to ensure:
- The sent value matches the specified deposit amount.
- The deposit amount is greater than zero.
Upon a successful deposit, the contract's balance is updated, and a `Deposit` event is emitted.

### Withdraw Function
```solidity
    function withdrawAmount(uint amount) public onlyOwner {
        require(amount <= balance, "Insufficient balance");

        // Using assert to check for internal invariants
        uint oldBalance = balance;
        balance -= amount;
        assert(balance == oldBalance - amount);

        // Transfer the specified amount to the owner
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");

        emit Withdrawal(msg.sender, amount, "Withdrawal successful");
    }
```
This function allows the owner to withdraw a specified amount of Ether from the contract. It includes checks to ensure:
- The requested amount does not exceed the contract balance.
- The balance is correctly updated after the withdrawal.
The function then transfers the specified amount to the owner and emits a `Withdrawal` event.

### Force Revert Function
```solidity
    function forceRevert() public {
        emit ForceRevert("This function always reverts");
        revert("This function always reverts");
    }
```
This function always reverts the transaction and emits a `ForceRevert` event. It can be used for testing or demonstrating transaction reversion.

### Fallback Function
```solidity
    receive() external payable {
        balance += msg.value;
    }
```
A fallback function to accept Ether sent directly to the contract. It updates the contract balance with the received value.

## License

This contract is licensed under the MIT License.
