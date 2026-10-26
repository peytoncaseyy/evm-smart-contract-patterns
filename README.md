EVM Smart Contract Security & Architecture Patterns

A reference repository showcasing production-grade, gas-optimized Solidity design patterns, secure escrow mechanisms, and reentrancy protection.

 Core Architecture: All-or-Nothing Escrow & State Machine

This pattern demonstrates a secure crowdfunding escrow lifecycle designed to handle conditional fund disbursement and automatic refunds.

Key Security Features:
Checks-Effects-Interactions Pattern:** Strictly enforces state updates before triggering external token or ETH transfers to eliminate reentrancy attack vectors.
Reentrancy Guard: Leverages OpenZeppelin's modifier safeguards on state-altering payout and refund functions.
Custom Errors (`error InsufficientBalance()`)**: Replaces costly string error messages with gas-efficient custom errors to minimize deployment and execution overhead.
Role-Based Access Control:** Secure modifier implementation ensuring only authorized campaign creators or platform administrators can trigger status transitions.

Sample Implementation (Solidity 0.8.x)
solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SecureCrowdfundingEscrow is ReentrancyGuard {
    enum CampaignState { Active, Successful, Failed, Claimed }
    
    struct Campaign {
        address creator;
        uint256 goal;
        uint256 totalRaised;
        CampaignState state;
    }

    mapping(uint256 => Campaign) public campaigns;
    mapping(uint256 => mapping(address => uint256)) public contributions;

    error CampaignNotActive();
    error TransferFailed();
    error Unauthorized();

    event FundsContributed(uint256 indexed campaignId, address indexed donor, uint256 amount);
    event RefundIssued(uint256 indexed campaignId, address indexed donor, uint256 amount);

    // Secure state transition & contribution logic
    function contribute(uint256 campaignId) external payable nonReentrant {
        Campaign storage campaign = campaigns[campaignId];
        if (campaign.state != CampaignState.Active) revert CampaignNotActive();
        
        contributions[campaignId][msg.sender] += msg.value;
        campaign.totalRaised += msg.value;
        
        emit FundsContributed(campaignId, msg.sender, msg.value);
    }
}
