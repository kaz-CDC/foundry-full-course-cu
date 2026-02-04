# Complete Blockchain Developer Tutorial
## From Zero to Smart Contract Expert with Solidity & Foundry

This tutorial is based on the Cyfrin Updraft Blockchain Developer Career Path. It will take you from complete beginner to building production-ready smart contracts.

---

## Table of Contents

1. [Part 1: Blockchain Fundamentals](#part-1-blockchain-fundamentals)
2. [Part 2: Solidity Basics](#part-2-solidity-basics)
3. [Part 3: Foundry Development Environment](#part-3-foundry-development-environment)
4. [Part 4: Testing & Deployment](#part-4-testing--deployment)
5. [Part 5: Advanced Smart Contracts](#part-5-advanced-smart-contracts)
6. [Part 6: DeFi & Security](#part-6-defi--security)

---

# Part 1: Blockchain Fundamentals

## What is a Blockchain?

A blockchain is a **distributed, immutable ledger** that records transactions across many computers. Key concepts:

- **Decentralization**: No single entity controls the network
- **Immutability**: Once data is recorded, it cannot be changed
- **Transparency**: All transactions are publicly visible
- **Consensus**: Network agrees on the state of the ledger

## Key Components

### 1. Transactions
Every action on a blockchain is a transaction containing:
- **Nonce**: Transaction counter for the sender
- **Gas Price**: Fee per unit of computation
- **Gas Limit**: Maximum computation allowed
- **To**: Recipient address
- **Value**: Amount of ETH to send
- **Data**: Function call data (for smart contracts)
- **v, r, s**: Signature components

### 2. Blocks
Transactions are grouped into blocks containing:
- Block number
- Timestamp
- Previous block hash (creating the "chain")
- List of transactions
- Nonce (for Proof of Work)

### 3. Smart Contracts
**Smart contracts are programs stored on the blockchain** that execute automatically when conditions are met. They have:
- An address (like a wallet)
- Storage (persistent state)
- Code (immutable logic)
- Balance (can hold ETH)

## The Ethereum Virtual Machine (EVM)

The EVM is the runtime environment for smart contracts. It:
- Executes bytecode compiled from Solidity
- Is Turing complete (can compute anything)
- Uses gas to prevent infinite loops
- Is deterministic (same input = same output)

---

# Part 2: Solidity Basics

## Your First Smart Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract SimpleStorage {
    // State variable - stored on blockchain
    uint256 private s_favoriteNumber;
    
    // Store a number
    function store(uint256 _favoriteNumber) public {
        s_favoriteNumber = _favoriteNumber;
    }
    
    // Retrieve the number (view = no gas when called externally)
    function retrieve() public view returns (uint256) {
        return s_favoriteNumber;
    }
}
```

### Key Elements Explained:

1. **License Identifier**: `// SPDX-License-Identifier: MIT`
   - Required by Solidity for open-source compliance

2. **Pragma**: `pragma solidity ^0.8.19;`
   - Specifies compiler version
   - `^` means "this version or higher within major version"

3. **Contract Declaration**: `contract SimpleStorage { }`
   - Similar to a class in other languages

4. **State Variables**: Variables stored permanently on the blockchain
   - Naming convention: prefix with `s_` for storage variables

5. **Functions**: Code that can be called
   - `public`: Anyone can call
   - `view`: Doesn't modify state (free to call externally)
   - `returns`: Specifies return type

## Solidity Data Types

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract DataTypes {
    // Value Types
    bool public isActive = true;
    uint256 public favoriteNumber = 88;     // Unsigned integer (0 to 2^256-1)
    int256 public negativeNumber = -5;      // Signed integer
    address public myAddress = 0x1234567890123456789012345678901234567890;
    bytes32 public myBytes = "hello";       // Fixed-size byte array
    
    // Reference Types
    string public greeting = "Hello, World!";
    
    // Arrays
    uint256[] public dynamicArray;          // Dynamic array
    uint256[3] public fixedArray;           // Fixed-size array
    
    // Mappings (key-value store)
    mapping(address => uint256) public balances;
    
    // Structs (custom types)
    struct Person {
        string name;
        uint256 age;
    }
    
    Person public myPerson = Person("Alice", 25);
    Person[] public people;                 // Array of structs
    
    // Enums (named constants)
    enum Status { Pending, Active, Completed }
    Status public currentStatus = Status.Pending;
}
```

## Functions Deep Dive

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract FunctionTypes {
    uint256 public number;
    
    // Public: callable by anyone
    function setNumber(uint256 _number) public {
        number = _number;
    }
    
    // Private: only callable within this contract
    function _internalHelper() private pure returns (uint256) {
        return 42;
    }
    
    // Internal: callable within this contract and derived contracts
    function _multiply(uint256 a, uint256 b) internal pure returns (uint256) {
        return a * b;
    }
    
    // External: only callable from outside (more gas efficient for large data)
    function externalFunction(uint256[] calldata data) external pure returns (uint256) {
        return data.length;
    }
    
    // View: reads state but doesn't modify
    function getNumber() public view returns (uint256) {
        return number;
    }
    
    // Pure: doesn't read or modify state
    function add(uint256 a, uint256 b) public pure returns (uint256) {
        return a + b;
    }
    
    // Payable: can receive ETH
    function deposit() public payable {
        // msg.value contains the ETH sent
    }
}
```

## Memory, Storage, and Calldata

Solidity has three data locations:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract DataLocations {
    // Storage: permanent, stored on blockchain
    string public storedName;
    
    function setName(string memory _name) public {
        // memory: temporary, exists during function execution
        storedName = _name;
    }
    
    function processData(string calldata _data) external pure returns (uint256) {
        // calldata: read-only, most gas efficient for external function inputs
        return bytes(_data).length;
    }
    
    function getString() public view returns (string memory) {
        // Must specify 'memory' for reference types in returns
        return storedName;
    }
}
```

**Key Rules:**
- **Storage**: State variables (default)
- **Memory**: Function parameters, local variables (temporary)
- **Calldata**: External function parameters (read-only, gas efficient)

## Working with ETH: The Fund Me Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract FundMe {
    // Track who funded how much
    mapping(address => uint256) public addressToAmountFunded;
    address[] public funders;
    address public immutable i_owner;
    
    uint256 public constant MINIMUM_USD = 5e18; // $5 with 18 decimals
    
    // Custom error (more gas efficient than require strings)
    error FundMe__NotOwner();
    
    constructor() {
        i_owner = msg.sender;
    }
    
    // Modifier: reusable condition check
    modifier onlyOwner() {
        if (msg.sender != i_owner) revert FundMe__NotOwner();
        _;  // Continue with the function
    }
    
    // Accept ETH
    function fund() public payable {
        require(msg.value >= 1e15, "Minimum 0.001 ETH required");
        
        addressToAmountFunded[msg.sender] += msg.value;
        funders.push(msg.sender);
    }
    
    // Owner can withdraw all funds
    function withdraw() public onlyOwner {
        // Reset all funder balances
        for (uint256 i = 0; i < funders.length; i++) {
            address funder = funders[i];
            addressToAmountFunded[funder] = 0;
        }
        
        // Reset funders array
        funders = new address[](0);
        
        // Transfer ETH to owner (three ways to send ETH):
        
        // 1. transfer - reverts on failure, 2300 gas limit
        // payable(msg.sender).transfer(address(this).balance);
        
        // 2. send - returns bool, 2300 gas limit
        // bool success = payable(msg.sender).send(address(this).balance);
        // require(success, "Send failed");
        
        // 3. call - recommended, returns bool + data, no gas limit
        (bool success, ) = payable(msg.sender).call{value: address(this).balance}("");
        require(success, "Call failed");
    }
    
    // Receive ETH sent directly to contract
    receive() external payable {
        fund();
    }
    
    // Fallback for calls with data but no matching function
    fallback() external payable {
        fund();
    }
}
```

### Key Concepts Explained:

1. **`msg.sender`**: Address that called the function
2. **`msg.value`**: Amount of ETH sent (in wei)
3. **`immutable`**: Set once in constructor, gas efficient
4. **`constant`**: Set at compile time, most gas efficient
5. **Custom Errors**: More gas efficient than `require` with strings
6. **Modifiers**: Reusable access control patterns

---

# Part 3: Foundry Development Environment

## Installation

```bash
# Install Foundry
curl -L https://foundry.paradigm.xyz | bash
foundryup

# Verify installation
forge --version
cast --version
anvil --version
```

## Create a New Project

```bash
# Create new project
forge init my-project
cd my-project

# Project structure:
# ├── lib/          # Dependencies
# ├── script/       # Deployment scripts
# ├── src/          # Smart contracts
# ├── test/         # Test files
# └── foundry.toml  # Configuration
```

## Writing Smart Contracts

Create `src/Counter.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract Counter {
    uint256 public number;
    
    event NumberChanged(uint256 indexed oldNumber, uint256 indexed newNumber);
    
    function setNumber(uint256 newNumber) public {
        uint256 oldNumber = number;
        number = newNumber;
        emit NumberChanged(oldNumber, newNumber);
    }
    
    function increment() public {
        number++;
    }
    
    function decrement() public {
        require(number > 0, "Cannot go below zero");
        number--;
    }
}
```

## Compiling

```bash
# Compile all contracts
forge build

# Clean and rebuild
forge clean && forge build
```

---

# Part 4: Testing & Deployment

## Writing Tests

Create `test/Counter.t.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Test, console} from "forge-std/Test.sol";
import {Counter} from "../src/Counter.sol";

contract CounterTest is Test {
    Counter public counter;
    address public USER = makeAddr("user");
    
    // Runs before each test
    function setUp() public {
        counter = new Counter();
        counter.setNumber(0);
    }
    
    function test_InitialValueIsZero() public view {
        assertEq(counter.number(), 0);
    }
    
    function test_SetNumber() public {
        counter.setNumber(42);
        assertEq(counter.number(), 42);
    }
    
    function test_Increment() public {
        counter.setNumber(5);
        counter.increment();
        assertEq(counter.number(), 6);
    }
    
    function test_DecrementRevertsAtZero() public {
        counter.setNumber(0);
        vm.expectRevert("Cannot go below zero");
        counter.decrement();
    }
    
    // Fuzz testing - Foundry generates random inputs
    function testFuzz_SetNumber(uint256 x) public {
        counter.setNumber(x);
        assertEq(counter.number(), x);
    }
    
    // Test with different users
    function test_AnyoneCanSetNumber() public {
        vm.prank(USER);  // Next call will be from USER
        counter.setNumber(100);
        assertEq(counter.number(), 100);
    }
    
    // Test events
    function test_EmitsNumberChanged() public {
        vm.expectEmit(true, true, false, false);
        emit Counter.NumberChanged(0, 42);
        counter.setNumber(42);
    }
}
```

## Running Tests

```bash
# Run all tests
forge test

# Run with verbosity (show logs)
forge test -vv

# Run specific test
forge test --match-test test_Increment

# Run with gas report
forge test --gas-report

# Check coverage
forge coverage
```

## Foundry Cheatcodes

Foundry provides powerful testing utilities:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Test, console} from "forge-std/Test.sol";

contract CheatcodesDemo is Test {
    function test_Cheatcodes() public {
        // Create addresses
        address alice = makeAddr("alice");
        address bob = makeAddr("bob");
        
        // Give ETH to an address
        vm.deal(alice, 10 ether);
        assertEq(alice.balance, 10 ether);
        
        // Impersonate an address for the next call
        vm.prank(alice);
        
        // Impersonate for multiple calls
        vm.startPrank(alice);
        // ... multiple calls as alice
        vm.stopPrank();
        
        // Manipulate block time
        vm.warp(block.timestamp + 1 days);
        
        // Manipulate block number
        vm.roll(block.number + 100);
        
        // Expect a revert
        vm.expectRevert();
        // ... call that should revert
        
        // Expect specific revert message
        vm.expectRevert("Error message");
        
        // Expect custom error
        vm.expectRevert(CustomError.selector);
        
        // Log values
        console.log("Value:", 42);
        console.log("Address:", alice);
    }
}
```

## Deployment Scripts

Create `script/DeployCounter.s.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Script, console} from "forge-std/Script.sol";
import {Counter} from "../src/Counter.sol";

contract DeployCounter is Script {
    function run() external returns (Counter) {
        // Get private key from environment
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        
        vm.startBroadcast(deployerPrivateKey);
        
        Counter counter = new Counter();
        counter.setNumber(0);
        
        vm.stopBroadcast();
        
        console.log("Counter deployed at:", address(counter));
        
        return counter;
    }
}
```

## Deploying

```bash
# Deploy to local Anvil network
anvil  # Start local node in one terminal

# In another terminal:
forge script script/DeployCounter.s.sol --rpc-url http://localhost:8545 --broadcast

# Deploy to testnet (Sepolia)
forge script script/DeployCounter.s.sol \
    --rpc-url $SEPOLIA_RPC_URL \
    --broadcast \
    --verify \
    -vvvv
```

## Using a Makefile

Create a `Makefile` for common commands:

```makefile
-include .env

build:
	forge build

test:
	forge test

test-v:
	forge test -vvv

coverage:
	forge coverage

snapshot:
	forge snapshot

deploy-anvil:
	forge script script/DeployCounter.s.sol --rpc-url http://localhost:8545 --broadcast

deploy-sepolia:
	forge script script/DeployCounter.s.sol --rpc-url $(SEPOLIA_RPC_URL) --private-key $(PRIVATE_KEY) --broadcast --verify --etherscan-api-key $(ETHERSCAN_API_KEY) -vvvv
```

---

# Part 5: Advanced Smart Contracts

## ERC20 Token Standard

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyToken is ERC20 {
    constructor(uint256 initialSupply) ERC20("MyToken", "MTK") {
        _mint(msg.sender, initialSupply);
    }
    
    // Optional: Add custom functionality
    function burn(uint256 amount) public {
        _burn(msg.sender, amount);
    }
}
```

## ERC721 NFT Standard

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {ERC721} from "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import {Base64} from "@openzeppelin/contracts/utils/Base64.sol";

contract MoodNFT is ERC721 {
    uint256 private s_tokenCounter;
    
    enum Mood { HAPPY, SAD }
    mapping(uint256 => Mood) private s_tokenIdToMood;
    
    string private constant HAPPY_SVG = "data:image/svg+xml;base64,...";
    string private constant SAD_SVG = "data:image/svg+xml;base64,...";
    
    constructor() ERC721("Mood NFT", "MOOD") {
        s_tokenCounter = 0;
    }
    
    function mintNft() public {
        _safeMint(msg.sender, s_tokenCounter);
        s_tokenIdToMood[s_tokenCounter] = Mood.HAPPY;
        s_tokenCounter++;
    }
    
    function flipMood(uint256 tokenId) public {
        require(ownerOf(tokenId) == msg.sender, "Not owner");
        
        if (s_tokenIdToMood[tokenId] == Mood.HAPPY) {
            s_tokenIdToMood[tokenId] = Mood.SAD;
        } else {
            s_tokenIdToMood[tokenId] = Mood.HAPPY;
        }
    }
    
    function tokenURI(uint256 tokenId) public view override returns (string memory) {
        string memory imageURI = s_tokenIdToMood[tokenId] == Mood.HAPPY 
            ? HAPPY_SVG 
            : SAD_SVG;
            
        return string(
            abi.encodePacked(
                "data:application/json;base64,",
                Base64.encode(
                    bytes(
                        abi.encodePacked(
                            '{"name": "Mood NFT",',
                            '"description": "An NFT that reflects mood",',
                            '"image": "', imageURI, '"}'
                        )
                    )
                )
            )
        );
    }
}
```

## Using Chainlink Oracles

### Price Feeds

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {AggregatorV3Interface} from "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";

contract PriceConsumer {
    AggregatorV3Interface internal priceFeed;
    
    constructor(address _priceFeed) {
        // Sepolia ETH/USD: 0x694AA1769357215DE4FAC081bf1f309aDC325306
        priceFeed = AggregatorV3Interface(_priceFeed);
    }
    
    function getLatestPrice() public view returns (int256) {
        (
            /* uint80 roundID */,
            int256 price,
            /* uint256 startedAt */,
            /* uint256 timeStamp */,
            /* uint80 answeredInRound */
        ) = priceFeed.latestRoundData();
        
        return price; // Price has 8 decimals
    }
    
    function getConversionRate(uint256 ethAmount) public view returns (uint256) {
        int256 price = getLatestPrice();
        // ethAmount is in wei (18 decimals), price has 8 decimals
        // Result: (ethAmount * price) / 1e8 gives USD with 18 decimals
        return (ethAmount * uint256(price)) / 1e8;
    }
}
```

### Chainlink VRF (Verifiable Random Function)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {VRFConsumerBaseV2Plus} from "@chainlink/contracts/src/v0.8/vrf/dev/VRFConsumerBaseV2Plus.sol";
import {VRFV2PlusClient} from "@chainlink/contracts/src/v0.8/vrf/dev/libraries/VRFV2PlusClient.sol";

contract RandomLottery is VRFConsumerBaseV2Plus {
    uint256 private immutable i_subscriptionId;
    bytes32 private immutable i_keyHash;
    uint32 private immutable i_callbackGasLimit;
    uint16 private constant REQUEST_CONFIRMATIONS = 3;
    uint32 private constant NUM_WORDS = 1;
    
    address[] public players;
    address public recentWinner;
    
    event WinnerPicked(address indexed winner);
    
    constructor(
        uint256 subscriptionId,
        address vrfCoordinator,
        bytes32 keyHash,
        uint32 callbackGasLimit
    ) VRFConsumerBaseV2Plus(vrfCoordinator) {
        i_subscriptionId = subscriptionId;
        i_keyHash = keyHash;
        i_callbackGasLimit = callbackGasLimit;
    }
    
    function enterLottery() public payable {
        require(msg.value >= 0.01 ether, "Not enough ETH");
        players.push(msg.sender);
    }
    
    function pickWinner() external {
        // Request random number from Chainlink VRF
        s_vrfCoordinator.requestRandomWords(
            VRFV2PlusClient.RandomWordsRequest({
                keyHash: i_keyHash,
                subId: i_subscriptionId,
                requestConfirmations: REQUEST_CONFIRMATIONS,
                callbackGasLimit: i_callbackGasLimit,
                numWords: NUM_WORDS,
                extraArgs: VRFV2PlusClient._argsToBytes(
                    VRFV2PlusClient.ExtraArgsV1({nativePayment: false})
                )
            })
        );
    }
    
    // Chainlink VRF calls this function with the random number
    function fulfillRandomWords(
        uint256 /* requestId */,
        uint256[] calldata randomWords
    ) internal override {
        uint256 winnerIndex = randomWords[0] % players.length;
        address winner = players[winnerIndex];
        recentWinner = winner;
        
        // Reset players
        players = new address[](0);
        
        // Send prize to winner
        (bool success, ) = winner.call{value: address(this).balance}("");
        require(success, "Transfer failed");
        
        emit WinnerPicked(winner);
    }
}
```

## Upgradeable Contracts (UUPS Pattern)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {UUPSUpgradeable} from "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
import {Initializable} from "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
import {OwnableUpgradeable} from "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

contract BoxV1 is Initializable, UUPSUpgradeable, OwnableUpgradeable {
    uint256 internal number;
    
    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() {
        _disableInitializers();
    }
    
    function initialize() public initializer {
        __Ownable_init(msg.sender);
        __UUPSUpgradeable_init();
    }
    
    function getNumber() external view returns (uint256) {
        return number;
    }
    
    function setNumber(uint256 _number) external {
        number = _number;
    }
    
    function _authorizeUpgrade(address newImplementation) internal override onlyOwner {}
}

// Version 2 with new functionality
contract BoxV2 is Initializable, UUPSUpgradeable, OwnableUpgradeable {
    uint256 internal number;
    
    function getNumber() external view returns (uint256) {
        return number;
    }
    
    function setNumber(uint256 _number) external {
        number = _number;
    }
    
    // New function in V2!
    function increment() external {
        number++;
    }
    
    function _authorizeUpgrade(address newImplementation) internal override onlyOwner {}
}
```

---

# Part 6: DeFi & Security

## Building a Stablecoin

Key concepts for a decentralized stablecoin:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {ERC20Burnable, ERC20} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";

/**
 * @title DecentralizedStableCoin
 * @notice A stablecoin pegged to USD, backed by crypto collateral
 * Collateral: ETH, wBTC (exogenous)
 * Stability: Algorithmic (minting/burning)
 * Peg: USD
 */
contract DecentralizedStableCoin is ERC20Burnable, Ownable {
    error DSC__MustBeMoreThanZero();
    error DSC__BurnAmountExceedsBalance();
    
    constructor() ERC20("DecentralizedStableCoin", "DSC") Ownable(msg.sender) {}
    
    function burn(uint256 _amount) public override onlyOwner {
        uint256 balance = balanceOf(msg.sender);
        if (_amount <= 0) revert DSC__MustBeMoreThanZero();
        if (balance < _amount) revert DSC__BurnAmountExceedsBalance();
        
        super.burn(_amount);
    }
    
    function mint(address _to, uint256 _amount) external onlyOwner returns (bool) {
        if (_to == address(0)) revert DSC__MustBeMoreThanZero();
        if (_amount <= 0) revert DSC__MustBeMoreThanZero();
        
        _mint(_to, _amount);
        return true;
    }
}
```

## Security Best Practices

### 1. Check-Effects-Interactions Pattern

```solidity
function withdraw() external {
    // CHECKS
    uint256 amount = balances[msg.sender];
    require(amount > 0, "No balance");
    
    // EFFECTS (update state before external calls)
    balances[msg.sender] = 0;
    
    // INTERACTIONS (external calls last)
    (bool success, ) = msg.sender.call{value: amount}("");
    require(success, "Transfer failed");
}
```

### 2. Reentrancy Guard

```solidity
import {ReentrancyGuard} from "@openzeppelin/contracts/utils/ReentrancyGuard.sol";

contract SafeContract is ReentrancyGuard {
    function withdraw() external nonReentrant {
        // Protected from reentrancy
    }
}
```

### 3. Access Control

```solidity
import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";
import {AccessControl} from "@openzeppelin/contracts/access/AccessControl.sol";

contract SecureContract is Ownable, AccessControl {
    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN_ROLE");
    
    constructor() Ownable(msg.sender) {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(ADMIN_ROLE, msg.sender);
    }
    
    function adminFunction() external onlyRole(ADMIN_ROLE) {
        // Only admins can call
    }
}
```

### 4. Input Validation

```solidity
function deposit(uint256 amount) external {
    // Validate inputs
    require(amount > 0, "Amount must be positive");
    require(amount <= type(uint256).max - totalDeposits, "Overflow");
    require(token.balanceOf(msg.sender) >= amount, "Insufficient balance");
    
    // Continue with logic
}
```

## Invariant/Fuzz Testing

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Test} from "forge-std/Test.sol";
import {StdInvariant} from "forge-std/StdInvariant.sol";

contract InvariantTest is StdInvariant, Test {
    MyProtocol protocol;
    Handler handler;
    
    function setUp() public {
        protocol = new MyProtocol();
        handler = new Handler(protocol);
        
        // Tell Foundry to only call functions on the handler
        targetContract(address(handler));
    }
    
    // Invariant: total deposits should always equal sum of all balances
    function invariant_totalDepositsEqualsSumOfBalances() public view {
        uint256 sum = 0;
        address[] memory users = handler.getUsers();
        
        for (uint256 i = 0; i < users.length; i++) {
            sum += protocol.balanceOf(users[i]);
        }
        
        assertEq(protocol.totalDeposits(), sum);
    }
}

// Handler constrains how the fuzzer calls the protocol
contract Handler is Test {
    MyProtocol protocol;
    address[] public users;
    
    constructor(MyProtocol _protocol) {
        protocol = _protocol;
    }
    
    function deposit(uint256 amount, uint256 userSeed) public {
        // Bound inputs to reasonable values
        amount = bound(amount, 1, 1000 ether);
        address user = _getOrCreateUser(userSeed);
        
        vm.deal(user, amount);
        vm.prank(user);
        protocol.deposit{value: amount}();
    }
    
    function _getOrCreateUser(uint256 seed) internal returns (address) {
        if (users.length == 0) {
            address newUser = makeAddr(string(abi.encodePacked("user", seed)));
            users.push(newUser);
            return newUser;
        }
        return users[seed % users.length];
    }
    
    function getUsers() external view returns (address[] memory) {
        return users;
    }
}
```

---

## Next Steps

1. **Practice**: Build projects from the course repositories:
   - [foundry-simple-storage-cu](https://github.com/Cyfrin/foundry-simple-storage-cu)
   - [foundry-fund-me-cu](https://github.com/Cyfrin/foundry-fund-me-cu)
   - [foundry-smart-contract-lottery-cu](https://github.com/Cyfrin/foundry-smart-contract-lottery-cu)
   - [foundry-defi-stablecoin-cu](https://github.com/Cyfrin/foundry-defi-stablecoin-cu)

2. **Learn Security**: Take the security course at [Cyfrin Updraft](https://updraft.cyfrin.io)

3. **Participate in Hackathons**: Apply your skills in real competitions

4. **Contribute to Open Source**: Help improve the ecosystem

5. **Join the Community**: 
   - [Cyfrin Discord](https://discord.gg/cyfrin)
   - [Ethereum Stack Exchange](https://ethereum.stackexchange.com/)

---

## Resources

- [Solidity Documentation](https://docs.soliditylang.org/)
- [Foundry Book](https://book.getfoundry.sh/)
- [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts)
- [Chainlink Documentation](https://docs.chain.link/)
- [Ethereum.org Developer Resources](https://ethereum.org/developers)

---

*This tutorial was created based on the Cyfrin Updraft Blockchain Developer Career Path. For the complete video course, visit [updraft.cyfrin.io](https://updraft.cyfrin.io).*
