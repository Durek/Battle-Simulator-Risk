<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Risk Battle Simulator</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-image: url('risksimulator_bakgrunn.webp');
            text-align: center;
            padding: 20px;
        }
        .container {
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
            display: inline-block;
        }
        input, button, label {
            margin: 10px;
        }
        button {
            padding: 10px 20px;
            background-color: #28a745;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
        }
        button:hover {
            background-color: #218838;
        }
        #stopButton {
            background-color: #dc3545;
        }
        #stopButton:hover {
            background-color: #c82333;
        }
        #result, #history {
            margin-top: 20px;
            font-size: 18px;
            text-align: left;
            max-width: 400px;
            margin-left: auto;
            margin-right: auto;
        }
        .result-entry {
            padding: 10px;
            border-bottom: 1px solid #ddd;
        }
        .bold {
            font-weight: bold;
        }
        .attacker {
            color: red;
        }
        .defender {
            color: blue;
        }
        #history-container {
            display: none;
            max-height: 150px;
            overflow-y: auto;
            background: white;
            padding: 10px;
            border-radius: 10px;
            box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
            margin-top: 20px;
        }
    </style>
</head>
<body>

    <div class="container">
        <h1>Risk Battle Simulator</h1>
        
        <label for="attacker">Attackers:</label>
        <input type="number" id="attacker" min="1" value="5">
        
        <label for="defender">Defenders:</label>
        <input type="number" id="defender" min="1" value="5">
        
        <br>
        <label for="speed">Pause time (ms): <span id="speedDisplay">500</span></label>
        <input type="range" id="speed" min="1" max="10000" step="100" value="500" oninput="updateSpeed()">
        
        <br>
        <button onclick="startBattle()">Start Battle</button>
        <button id="stopButton" onclick="stopBattle()" disabled>Stop Attack</button>

        <div id="result"></div>
        
        <div id="history-container">
            <h3>Previous Results</h3>
            <div id="history"></div>
        </div>
    </div>

    <script>
        let attackerTroops, defenderTroops, battleInterval, pauseTime = 500;
        let battleActive = false;
        let results = [];
        let history = [];

        function rollDice(n) {
            let dice = [];
            for (let i = 0; i < n; i++) {
                dice.push(Math.floor(Math.random() * 6) + 1);
            }
            return dice.sort((a, b) => b - a);
        }

        function riskBattle() {
            if (!battleActive) return;

            let attackerDice, defenderDice;

            if (attackerTroops >= 4 && defenderTroops >= 2) {
                attackerDice = rollDice(3);
                defenderDice = rollDice(2);
            } else if (attackerTroops === 3 && defenderTroops >= 2) {
                attackerDice = rollDice(2);
                defenderDice = rollDice(2);
            } else if (attackerTroops === 2 && defenderTroops >= 2) {
                attackerDice = rollDice(1);
                defenderDice = rollDice(2);
            } else if (attackerTroops >= 4 && defenderTroops === 1) {
                attackerDice = rollDice(3);
                defenderDice = rollDice(1);
            } else if (attackerTroops === 3 && defenderTroops === 1) {
                attackerDice = rollDice(2);
                defenderDice = rollDice(1);
            } else if (attackerTroops === 2 && defenderTroops === 1) {
                attackerDice = rollDice(1);
                defenderDice = rollDice(1);
            } else {
                endBattle();
                return;
            }

            if (attackerDice[0] > defenderDice[0]) {
                defenderTroops--;
            } else {
                attackerTroops--;
            }

            if (attackerDice.length > 1 && defenderDice.length > 1) {
                if (attackerDice[1] > defenderDice[1]) {
                    defenderTroops--;
                } else {
                    attackerTroops--;
                }
            }

            updateResults(attackerDice, defenderDice);

            if (defenderTroops === 0) {
                updateResults(null, null, `<strong style="color: green;">🎉 Attackers win!</strong>`);
                endBattle();
            } else if (attackerTroops === 1) {
                updateResults(null, null, `<strong style="color: red;">🛡️ Defenders win!</strong>`);
                endBattle();
            } else {
                battleInterval = setTimeout(riskBattle, pauseTime);
            }
        }

        function startBattle() {
            attackerTroops = parseInt(document.getElementById("attacker").value);
            defenderTroops = parseInt(document.getElementById("defender").value);

            if (attackerTroops < 1 || defenderTroops < 1) {
                alert("Both sides must have at least one troop!");
                return;
            }

            results = [];
            history = [];
            document.getElementById("history-container").style.display = "none";
            updateResults(null, null, `⚔️ Battle begins!<br>`);
            battleActive = true;
            document.getElementById("stopButton").disabled = false;
            riskBattle();
        }

        function stopBattle() {
            if (battleActive) {
                battleActive = false;
                clearTimeout(battleInterval);
                updateResults(null, null, `<br><strong style="color: red;">🚨 Attack stopped! Defenders win.</strong>`);
                document.getElementById("stopButton").disabled = true;
                document.getElementById("history-container").style.display = "block";
            }
        }

        function endBattle() {
            battleActive = false;
            clearTimeout(battleInterval);
            document.getElementById("stopButton").disabled = true;
            document.getElementById("history-container").style.display = "block";
        }

        function updateResults(attackerDice, defenderDice, message = null) {
            let resultDiv = document.getElementById("result");
            let historyDiv = document.getElementById("history");

            let resultEntry = message 
                ? `<div class="result-entry">${message}</div>` 
                : `<div class="result-entry">
                        🎲 <strong>Attackers:</strong> ${attackerDice} | <strong>Defenders:</strong> ${defenderDice}<br>
                        💥 After round: 
                        <strong class="attacker">${attackerTroops}</strong> vs 
                        <strong class="defender">${defenderTroops}</strong>
                    </div>`;

            history.unshift(resultEntry);
            results.push(resultEntry);

            if (results.length > 3) {
                results.shift();
            }

            resultDiv.innerHTML = results.join("");
            historyDiv.innerHTML = history.join("");
        }

        function updateSpeed() {
            pauseTime = parseInt(document.getElementById("speed").value);
            document.getElementById("speedDisplay").innerText = pauseTime;
        }
    </script>

</body>
</html>
