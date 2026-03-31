# Simple-Crowdfunding-ICO
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Crowdfunding {
    address public owner;
    uint256 public goal;
    uint256 public deadline;
    uint256 public totalRaised;
    mapping(address => uint256) public contributions;

    event Funded(address contributor, uint256 amount);
    event GoalReached(uint256 total);

    constructor(uint256 _goal, uint256 _durationInDays) {
        owner = msg.sender;
        goal = _goal;
        deadline = block.timestamp + (_durationInDays * 1 days);
    }

    function contribute() public payable {
        require(block.timestamp < deadline, "Campaign ended");
        contributions[msg.sender] += msg.value;
        totalRaised += msg.value;
        emit Funded(msg.sender, msg.value);
    }

    function withdraw() public {
        require(msg.sender == owner, "Only owner");
        require(totalRaised >= goal, "Goal not reached");
        payable(owner).transfer(address(this).balance);
    }

    function refund() public {
        require(block.timestamp > deadline, "Campaign still active");
        require(totalRaised < goal, "Goal was reached");
        uint256 amount = contributions[msg.sender];
        contributions[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }
}

5. Lottery Contractsolidity

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Lottery {
    address[] public players;
    address public winner;
    uint256 public ticketPrice = 0.01 ether;

    function enter() public payable {
        require(msg.value == ticketPrice, "Incorrect ticket price");
        players.push(msg.sender);
    }

    function pickWinner() public {
        require(players.length > 0, "No players");
        uint256 index = uint256(keccak256(abi.encodePacked(block.timestamp, block.prevrandao))) % players.length;
        winner = players[index];
        payable(winner).transfer(address(this).balance);
    }

    function getPlayers() public view returns (address[] memory) {
        return players;
    }
}

6. Timelock Walletsolidity

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Timelock {
    address public owner;
    uint256 public unlockTime;

    constructor(uint256 _unlockTime) {
        owner = msg.sender;
        unlockTime = block.timestamp + _unlockTime;
    }

    function deposit() public payable {}

    function withdraw() public {
        require(msg.sender == owner, "Not owner");
        require(block.timestamp >= unlockTime, "Still locked");
        payable(owner).transfer(address(this).balance);
    }
}

7. Multi-Signature Wallet (2-of-3 Simple)solidity

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract MultiSigWallet {
    address[3] public owners;
    uint256 public required = 2;
    uint256 public transactionCount;

    struct Transaction {
        address to;
        uint256 value;
        bool executed;
    }

    mapping(uint256 => Transaction) public transactions;
    mapping(uint256 => mapping(address => bool)) public confirmations;

    constructor(address[3] memory _owners) {
        owners = _owners;
    }

    function submitTransaction(address _to, uint256 _value) public {
        transactionCount++;
        transactions[transactionCount] = Transaction(_to, _value, false);
    }

    function confirmTransaction(uint256 _txId) public {
        confirmations[_txId][msg.sender] = true;
    }

    function executeTransaction(uint256 _txId) public {
        Transaction storage tx = transactions[_txId];
        require(!tx.executed, "Already executed");
        uint256 count = 0;
        for (uint i = 0; i < 3; i++) {
            if (confirmations[_txId][owners[i]]) count++;
        }
        require(count >= required, "Not enough confirmations");
        tx.executed = true;
        payable(tx.to).transfer(tx.value);
    }

    receive() external payable {}
}

8. Vesting Contract (Token Release Over Time)solidity

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Vesting {
    IERC20 public token;
    address public beneficiary;
    uint256 public start;
    uint256 public duration;
    uint256 public released;

    constructor(address _token, address _beneficiary, uint256 _duration) {
        token = IERC20(_token);
        beneficiary = _beneficiary;
        start = block.timestamp;
        duration = _duration;
    }

    function release() public {
        uint256 vested = vestedAmount();
        uint256 unreleased = vested - released;
        require(unreleased > 0, "No tokens to release");
        released += unreleased;
        token.transfer(beneficiary, unreleased);
    }

    function vestedAmount() public view returns (uint256) {
        if (block.timestamp < start) return 0;
        if (block.timestamp >= start + duration) return token.balanceOf(address(this));
        return (token.balanceOf(address(this)) * (block.timestamp - start)) / duration;
    }
}

interface IERC20 {
    function transfer(address to, uint256 amount) external returns (bool);
    function balanceOf(address account) external view returns (uint256);
}

9. Simple DAO Proposal (Voting)solidity

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimpleDAO {
    struct Proposal {
        string description;
        uint256 voteCount;
        bool executed;
    }

    Proposal[] public proposals;
    mapping(address => bool) public members;

    constructor() {
        members[msg.sender] = true; // Founder
    }

    function addMember(address _member) public {
        require(members[msg.sender], "Not member");
        members[_member] = true;
    }

    function createProposal(string memory _description) public {
        proposals.push(Proposal(_description, 0, false));
    }

    function vote(uint256 _proposalId) public {
        require(members[msg.sender], "Not a member");
        proposals[_proposalId].voteCount++;
    }
}

10. Yield Optimizer (Basic)solidity

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract YieldOptimizer {
    IERC20 public token;
    uint256 public totalStaked;
    mapping(address => uint256) public userStaked;

    constructor(address _token) {
        token = IERC20(_token);
    }

    function deposit(uint256 amount) public {
        token.transferFrom(msg.sender, address(this), amount);
        userStaked[msg.sender] += amount;
        totalStaked += amount;
    }

    function withdraw(uint256 amount) public {
        require(userStaked[msg.sender] >= amount, "Insufficient");
        userStaked[msg.sender] -= amount;
        totalStaked -= amount;
        token.transfer(msg.sender, amount);
    }

    // Simulate yield - in real use call external strategy
    function getAPY() public pure returns (uint256) {
        return 12; // 12% APY example
    }
}

interface IERC20 {
    function transferFrom(address from, address to, uint256 amount) external returns (bool);
    function transfer(address to, uint256 amount) external returns (bool);
}

Tips when deploying these on Base Mainnet:Use OpenZeppelin imports (Remix can auto-import them).
For contracts using OpenZeppelin, make sure you are on Remix with Solidity Compiler set to ^0.8.20.
Test on Base Sepolia first before mainnet.

Bạn muốn mình giải thích chi tiết contract nào hoặc chỉnh sửa thêm tính năng không?
Ví dụ: thêm pause, tax, reflection, upgradeable, v.v. Just tell me! 

Giải thích chi tiết Staking Contract

Giới thiệu DeFi trên Base

