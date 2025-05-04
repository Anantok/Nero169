#<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Slot Game</title>
  <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
  <style>
    :root {
      --bg-color: #1e1e2f;
      --text-color: #fff;
      --slot-bg: #333;
      --button-bg: #e91e63;
      --button-hover: #c2185b;
    }
    body.light {
      --bg-color: #f0f0f0;
      --text-color: #111;
      --slot-bg: #ddd;
      --button-bg: #03a9f4;
      --button-hover: #0288d1;
    }
    body {
      font-family: 'Press Start 2P', cursive;
      background: var(--bg-color);
      color: var(--text-color);
      text-align: center;
      padding: 20px;
      max-width: 480px;
      margin: auto;
      transition: all 0.3s ease;
    }
    input, button {
      font-family: inherit;
      padding: 10px;
      margin: 5px;
      font-size: 0.8em;
      border-radius: 8px;
      border: none;
    }
    .hidden {
      display: none;
    }
    .slot-container {
      display: flex;
      justify-content: center;
      margin: 20px 0;
    }
    .slot {
      width: 80px;
      height: 80px;
      margin: 0 5px;
      font-size: 2.5em;
      display: flex;
      align-items: center;
      justify-content: center;
      background: var(--slot-bg);
      border-radius: 16px;
      box-shadow: 0 0 10px #00000080;
      transition: transform 0.3s ease;
    }
    button {
      background: var(--button-bg);
      color: white;
      cursor: pointer;
      transition: 0.2s;
    }
    button:hover {
      background: var(--button-hover);
    }
    .credits, .result, .history, .leaderboard, .avatar, .daily-bonus {
      margin-top: 15px;
      font-size: 0.8em;
    }
    .history ul, .leaderboard ol {
      list-style: none;
      padding: 0;
    }
    .history li, .leaderboard li {
      background: var(--slot-bg);
      margin-bottom: 5px;
      padding: 5px;
      border-radius: 8px;
      text-align: left;
    }
    .slot-glow {
      animation: glow 0.4s ease-in-out;
    }
    @keyframes glow {
      0% { box-shadow: 0 0 0px #fff; transform: scale(1.2); }
      100% { box-shadow: 0 0 10px #fff; transform: scale(1); }
    }
    .toggles {
      margin-top: 10px;
      font-size: 0.7em;
    }
    .toggles label {
      display: block;
      margin: 5px 0;
    }
    .avatar img {
      width: 80px;
      border-radius: 50%;
      margin-top: 10px;
    }
  </style>
</head>
<body>
  <div id="login-area">
    <h2>Slot Game</h2>
    <input type="text" id="username" placeholder="Enter your name" />
    <button onclick="login()">Enter</button>
  </div>
  <div id="game-area" class="hidden">
    <h1><span id="user-display"></span></h1>
    <div class="avatar"><img id="user-avatar" src="https://robohash.org/default?set=set5" alt="avatar"></div><div class="credits">Credits: <span id="credits">10</span></div>
<button onclick="resetCredits()">Reset Credits</button>
<button onclick="logout()">Logout</button>

<div class="daily-bonus">
  <button onclick="claimDailyBonus()" id="bonus-button">Claim Daily Bonus</button>
  <div id="bonus-status"></div>
</div>

<div class="toggles">
  <label><input type="checkbox" id="sound-toggle" checked onchange="toggleSound()"> Sound</label>
  <label><input type="checkbox" id="theme-toggle" onchange="toggleTheme()"> Light Mode</label>
</div>

<div class="slot-container">
  <div class="slot" id="slot1">?</div>
  <div class="slot" id="slot2">?</div>
  <div class="slot" id="slot3">?</div>
</div>

<button onclick="spin()">Spin</button>
<div class="result" id="result"></div>

<div class="history">
  <h3>Spin History</h3>
  <ul id="history-list"></ul>
</div>
<div class="leaderboard">
  <h3>Leaderboard</h3>
  <ol id="leaderboard"></ol>
</div>

  </div><audio id="spin-sound" src="https://freesound.org/data/previews/341/341695_3248244-lq.mp3"></audio> <audio id="win-sound" src="https://freesound.org/data/previews/320/320181_5260877-lq.mp3"></audio> <audio id="lose-sound" src="https://freesound.org/data/previews/256/256113_4486188-lq.mp3"></audio>

  <script>
    const symbols = ["🍒", "🍋", "🔔", "⭐", "7"];
    let credits = 10;
    let username = "";
    let history = [];
    let soundEnabled = true;
    let lastBonusDate = null;

    function login() {
      const input = document.getElementById("username").value.trim();
      if (!input) return alert("Enter username");
      username = input;
      document.getElementById("user-display").textContent = username;
      document.getElementById("user-avatar").src = `https://robohash.org/${username}?set=set5`;
      document.getElementById("login-area").classList.add("hidden");
      document.getElementById("game-area").classList.remove("hidden");

      const saved = JSON.parse(localStorage.getItem(username));
      if (saved) {
        credits = saved.credits;
        history = saved.history;
        lastBonusDate = saved.lastBonusDate || null;
        history.forEach(h => addToHistory(h.slot1, h.slot2, h.slot3, h.result));
      }
      updateCreditsDisplay();
      updateLeaderboard();
      updateBonusStatus();
    }

    function logout() {
      document.getElementById("game-area").classList.add("hidden");
      document.getElementById("login-area").classList.remove("hidden");
      document.getElementById("username").value = "";
      document.getElementById("history-list").innerHTML = "";
      document.getElementById("leaderboard").innerHTML = "";
      credits = 10;
      history = [];
      username = "";
      updateCreditsDisplay();
    }

    function toggleSound() {
      soundEnabled = document.getElementById("sound-toggle").checked;
    }

    function toggleTheme() {
      document.body.classList.toggle("light");
    }

    function updateCreditsDisplay() {
      document.getElementById("credits").textContent = credits;
    }

    function animateSlots() {
      document.querySelectorAll(".slot").forEach(slot => {
        slot.classList.add("slot-glow");
        setTimeout(() => slot.classList.remove("slot-glow"), 400);
      });
    }

    function addToHistory(slot1, slot2, slot3, result) {
      const list = document.getElementById("history-list");
      const item = document.createElement("li");
      item.textContent = `${slot1} ${slot2} ${slot3} - ${result}`;
      list.prepend(item);
    }

    function saveUserData() {
      localStorage.setItem(username, JSON.stringify({ credits, history, lastBonusDate }));
    }

    function updateLeaderboard() {
      let board = JSON.parse(localStorage.getItem("leaderboard")) || [];
      board = board.filter(entry => entry.user !== username);
      board.push({ user: username, credits });
      board.sort((a, b) => b.credits - a.credits);
      localStorage.setItem("leaderboard", JSON.stringify(board));

      const list = document.getElementById("leaderboard");
      list.innerHTML = "";
      board.slice(0, 5).forEach(entry => {
        const li = document.createElement("li");
        li.textContent = `${entry.user}: ${entry.credits} cr.`;
        list.appendChild(li);
      });
    }

    function spin() {
      if (credits <= 0) return document.getElementById("result").textContent = "No credits left!";

      if (soundEnabled) {
        const sfx = document.getElementById("spin-sound");
        sfx.currentTime = 0;
        sfx.play();
      }

      animateSlots();
      credits--;
      updateCreditsDisplay();

      const s1 = symbols[Math.floor(Math.random() * symbols.length)];
      const s2 = symbols[Math.floor(Math.random() * symbols.length)];
      const s3 = symbols[Math.floor(Math.random() * symbols.length)];

      document.getElementById("slot1").textContent = s1;
      document.getElementById("slot2").textContent = s2;
      document.getElementById("slot3").textContent = s3;

      let resultText = "Try again!";
      if (s1 === s2 && s2 === s3) {
        resultText = "JACKPOT! +5 Credits";
        credits += 5;
        if (soundEnabled) document.getElementById("win-sound").play();
      } else {
        if (soundEnabled) document.getElementById("lose-sound").play();
      }

      document.getElementById("result").textContent = resultText;
      history.unshift({ slot1: s1, slot2: s2, slot3: s3, result: resultText });
      addToHistory(s1, s2, s3, resultText);
      updateCreditsDisplay();
      saveUserData();
      updateLeaderboard();
    }

    function resetCredits() {
      credits = 10;
      updateCreditsDisplay();
      saveUserData();
      updateLeaderboard();
      document.getElementById("result").textContent = "Credits reset!";
    }

    function claimDailyBonus() {
      const today = new Date().toISOString().slice(0, 10);
      if (lastBonusDate === today) {
        document.getElementById("bonus-status").textContent = "Bonus already claimed today.";
        return;
      }
      lastBonusDate = today;
      credits += 3;
      updateCreditsDisplay();
      saveUserData();
      updateBonusStatus();
      document.getElementById("bonus-status").textContent = "+3 Bonus Credits!";
    }

    function updateBonusStatus() {
      const today = new Date().toISOString().slice(0, 10);
      if (lastBonusDate === today) {
        document.getElementById("bonus-button").disabled = true;
        document.getElementById("bonus-status").textContent = "Bonus already claimed today.";
      } else {
        document.getElementById("bonus-button").disabled = false;
        document.getElementById("bonus-status").textContent = "";
      }
    }
  </script></body>
</html>
