# IBallot

Source file [../../contracts/IBallot.sol](../../contracts/IBallot.sol).

<br />

<hr />

```javascript
// BK Ok - Will be replaced
pragma solidity ^0.4.15;

// BK Ok
import "./EthearnalRepToken.sol";
import "./VotingProxy.sol";

// BK Ok
contract IBallot {
    // BK Ok
    using SafeMath for uint256;
    // BK Ok
    EthearnalRepToken public tokenContract;

    // Date when vote has started
    // BK Ok
    uint256 public ballotStarted;

    // Registry of votes
    // BK Ok
    mapping(address => bool) public votesByAddress;

    // Sum of weights of YES votes
    // BK Ok
    uint256 public yesVoteSum = 0;

    // Sum of weights of NO votes
    // BK Ok
    uint256 public noVoteSum = 0;

    // Length of `voters`
    // BK Ok
    uint256 public votersLength = 0;

    // BK Ok
    uint256 public initialQuorumPercent = 51;

    VotingProxy public proxyVotingContract;

    // Tells if voting process is active
    // BK OK
    bool public isVotingActive = false;

    // BK Ok - Event
    event FinishBallot(uint256 _time);
    
    modifier onlyWhenBallotStarted {
        require(ballotStarted != 0);
        _;
    }

    function vote(bytes _vote) public onlyWhenBallotStarted {
        require(_vote.length > 0);
        if (isDataYes(_vote)) {
            processVote(true);
        } else if (isDataNo(_vote)) {
            processVote(false);
        }
    }

    function isDataYes(bytes data) public constant returns (bool) {
        // compare data with "YES" string
        return (
            data.length == 3 &&
            (data[0] == 0x59 || data[0] == 0x79) &&
            (data[1] == 0x45 || data[1] == 0x65) &&
            (data[2] == 0x53 || data[2] == 0x73)
        );
    }

    // TESTED
    function isDataNo(bytes data) public constant returns (bool) {
        // compare data with "NO" string
        return (
            data.length == 2 &&
            (data[0] == 0x4e || data[0] == 0x6e) &&
            (data[1] == 0x4f || data[1] == 0x6f)
        );
    }
    
    function processVote(bool isYes) internal {
        // BK Ok
        require(isVotingActive);
        // BK Ok
        require(!votesByAddress[msg.sender]);
        // BK Ok
        votersLength = votersLength.add(1);
        // BK Ok
        uint256 voteWeight = tokenContract.balanceOf(msg.sender);
        if (isYes) {
            // BK Ok
            yesVoteSum = yesVoteSum.add(voteWeight);
        } else {
            // BK Ok
            noVoteSum = noVoteSum.add(voteWeight);
        }
        require(getTime().sub(tokenContract.lastMovement(msg.sender)) > 7 days);
        uint256 quorumPercent = getQuorumPercent();
        if (quorumPercent == 0) {
            isVotingActive = false;
        } else {
            decide();
        }
        votesByAddress[msg.sender] = true;
    }

    function decide() internal {
        // BK Ok
        uint256 quorumPercent = getQuorumPercent();
        // BK Ok
        uint256 quorum = quorumPercent.mul(tokenContract.totalSupply()).div(100);
        // BK Ok
        uint256 soFarVoted = yesVoteSum.add(noVoteSum);
        // BK Ok
        if (soFarVoted >= quorum) {
            // BK Ok
            uint256 percentYes = (100 * yesVoteSum).div(soFarVoted);
            // BK Ok
            if (percentYes >= initialQuorumPercent) {
                // does not matter if it would be greater than weiRaised
                proxyVotingContract.proxyIncreaseWithdrawalChunk();
                FinishBallot(now);
                // BK Ok
                isVotingActive = false;
            } else {
                // do nothing, just deactivate voting
                // BK Ok
                isVotingActive = false;
                FinishBallot(now);
            }
        }
        
    }

    function getQuorumPercent() public constant returns (uint256);

    function getTime() internal returns (uint256) {
        // Just returns `now` value
        // This function is redefined in EthearnalRepTokenCrowdsaleMock contract
        // to allow testing contract behaviour at different time moments
        return now;
    }
    
}
```
