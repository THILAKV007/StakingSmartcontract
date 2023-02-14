// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract Staking {
    ERC20 public token;

    mapping (address => uint256) public stakedAmount;
    mapping (address => uint256) public stakedTimestamp;

    event StakeUpdate(address indexed user, uint256 amount);

    // Constructor to set the token
    constructor (ERC20 _token)  {
        token = _token;
    }

    function stake(uint256 _amount) public {
        require(token.balanceOf(msg.sender) >= _amount, "Insufficient balance");

        token.transferFrom(msg.sender, address(this), _amount);

        stakedAmount[msg.sender] += _amount;
        stakedTimestamp[msg.sender] = block.timestamp;

        emit StakeUpdate(msg.sender, _amount);
    }

    // Function to calculate the interest earned
    function calculateInterest() public view returns (uint256) {
        uint256 interestRate;
        uint256 elapsedTime = block.timestamp - stakedTimestamp[msg.sender];

        // Define the interest rate based on the elapsed time
        if (elapsedTime >= 365 days) {
            interestRate = 20;
        } else if (elapsedTime >= 240 days) {
            interestRate = 15;
        } else if (elapsedTime >= 120 days) {
            interestRate = 10;
        } else {
            interestRate = 5;
        }

        // Calculate the interest earned
        uint256 interest = stakedAmount[msg.sender] * interestRate / 100;

        // Return the interest earned
        return interest;
    }

    // Function to claim rewards
    function claimRewards() public {
        uint256 interest = calculateInterest();

        token.transfer(msg.sender, interest);

        stakedAmount[msg.sender] += interest;

        emit StakeUpdate(msg.sender, stakedAmount[msg.sender]);
    }

    // Function to unstake tokens
    function unstake(uint256 _amount) public {
        require(stakedAmount[msg.sender] >= _amount, "Insufficient staked amount");

        stakedAmount[msg.sender] -= _amount;

        token.transfer(msg.sender, _amount);

        emit StakeUpdate(msg.sender, stakedAmount[msg.sender]);
    }
}
