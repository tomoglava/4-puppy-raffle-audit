### [S-#] Looping through players array in `PuppyRaffle::enterRaffle` is potential DoS attack, incrementicg gas cost for future entrants

IMPACT: medium
LIKELIHOOD: medium

**Description:** The `PuppyRaffle::enterRaffle` function loops through `players` array to check for duplicates. However, the longer the `PupplyRaffle::players` array is the more check a new player will have to make. This means the gas costs for players who enter right when the raffle start will be automaticaly lower than those who enter later. Every additional address in the `players` array is an additional check the loop will have to make.

```javascript
        // @audit DoS attack possible if players array is too large
        for (uint256 i = 0; i < players.length - 1; i++) {
            for (uint256 j = i + 1; j < players.length; j++) {
                require(players[i] != players[j], "PuppyRaffle: Duplicate player");
            }
        }
```

**Impact:** The gas cost for raffle entrants will greatly increase as more players enter the raffle. Discouraging later users from entering and causing a rudh at the start of a raffle to be one of the first entrant in the queue.

An attacker might make the `PuppyRaffle::entrants ` array so big, that no one else neters, guaranteeing themselces the win.

**Proof of Concept:**

If we have 2 sets of 100 players enter, the gas cost increase dramatically:

<details>
<summary>PoC</summary>
Place the following test into `PuppyRaffleTest.t.sol`.

```javascript
    function test_denialOfService() public {
        vm.txGasPrice(1);

        uint256 playersNum = 100;
        address[] memory players = new address[](playersNum);
        for (uint256 i = 0; i < playersNum; i++) {
            players[i] = address(i);
        }

        uint256 gasStart = gasleft();
        puppyRaffle.enterRaffle{value: entranceFee * playersNum}(players);
        uint256 gasEnd = gasleft();

        uint256 gasUsedFirst = (gasStart - gasEnd) * tx.gasprice;
        console.log("Gas used for first 100 players: %d", gasUsedFirst);

        address[] memory playersTwo = new address[](playersNum);
        for (uint256 i = 0; i < playersNum; i++) {
            playersTwo[i] = address(i + playersNum);
        }

        uint256 gasStart2 = gasleft();
        puppyRaffle.enterRaffle{value: entranceFee * playersNum}(playersTwo);
        uint256 gasEnd2 = gasleft();

        uint256 gasUsedFirst2 = (gasStart2 - gasEnd2) * tx.gasprice;
        console.log("Gas used for second 100 players: %d", gasUsedFirst2);

        assert(gasUsedFirst2 > gasUsedFirst);
    }
```

</details>

**Recommended Mitigation:** There are few recomendations:
1. Consider allowing duplicates. Users can make new wallet anywhays, so a duplicate check doesn't prevent the same person from entering multiple times.
2. Consider using a mapping to check for duplicates. This would allow constant time lookup of weather a user has already entered.

```diff
    function enterRaffle(address[] memory newPlayers) public payable {
        // q were custom reverts a thing in solidity 0.7.6?
        // q what if it's 0?
        require(msg.value == entranceFee * newPlayers.length, "PuppyRaffle: Must send enough to enter raffle");
        for (uint256 i = 0; i < newPlayers.length; i++) {
            players.push(newPlayers[i]);
+           playerToRaffleId[newPlayers[i]] = raffleId;
        }

+       for(uint256 i = 0; i < newPlayers.length; i++) {
+           require(playerToRaffleId[newPlayers[i]] != raffleId, "PuppyRaffle: Duplicate player");
+       }

        // Check for duplicates
-        for (uint256 i = 0; i < players.length - 1; i++) {
-            for (uint256 j = i + 1; j < players.length; j++) {
-                require(players[i] != players[j], "PuppyRaffle: Duplicate player");
-            }
-        }
        emit RaffleEnter(newPlayers);



    function selectWinner() external {
+       raffleId = raffleId + 1;
        require(block.timestamp >= raffleStartTime + raffleDuration, "PuppyRaffle: Raffle not over");
        ...
```