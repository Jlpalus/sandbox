<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Phase 10 Score Tracker</title>
  <style>
    body { font-family: Arial, sans-serif; padding: 10px; margin: 0; }
    h2 { text-align: center; }
    .setup {
      display: flex;
      justify-content: center;
      align-items: center;
      flex-wrap: wrap;
      margin-bottom: 10px;
      gap: 6px;
    }
    .setup label { margin-right: 5px; }
    .names input {
      width: 90px;
      margin: 2px;
    }
    .container {
      display: flex;
      flex-wrap: wrap;
      gap: 20px;
      justify-content: center;
      overflow-x: auto;
    }
    table {
      border-collapse: collapse;
      margin-bottom: 20px;
    }
    th, td {
      border: 1px solid #999;
      padding: 4px 6px;
      text-align: center;
      min-width: 60px;
    }
    th { background: #f0f0f0; }
    input[type="number"] {
      width: 50px;
      text-align: center;
      -moz-appearance: textfield;
    }
    input[type="number"]::-webkit-inner-spin-button,
    input[type="number"]::-webkit-outer-spin-button {
      -webkit-appearance: none;
      margin: 0;
    }
    .placement { font-weight: bold; background: #e0e0e0; }
    @media (max-width: 600px) {
      .names input { width: 80px; }
      th, td { font-size: 12px; padding: 2px 4px; }
      input[type="number"] { width: 45px; }
    }
  </style>
</head>
<body>

<h2>Phase 10 Score Tracker</h2>

<div class="setup">
  <label>Number of Players:</label>
  <select id="playerCount" onchange="buildTables()">
 <option>2</option>
  <option>3</option>
  <option selected>4</option>
  <option>5</option>
  <option>6</option>
  </select>
  <span id="nameInputs" class="names"></span>
</div>

<div class="container">
  <table id="scores"></table>
  <table id="totals"></table>
</div>

<script>
const phases = ["1","2","3","4","5","bonus","6","7","8","9","10","bonus"];

function buildTables() {
  const scores = document.getElementById("scores");
  const totals = document.getElementById("totals");
  const nameInputs = document.getElementById("nameInputs");
  const numPlayers = parseInt(document.getElementById("playerCount").value);

  scores.innerHTML = '';
  totals.innerHTML = '';
  nameInputs.innerHTML = '';

  // Build player name fields
  for (let i = 0; i < numPlayers; i++) {
    nameInputs.innerHTML += `<input type="text" value="Player ${i+1}" oninput="updateNames(${i})">`;
  }

  // Build header rows
  let scoreHeader = `<thead><tr><th>Phase</th>`;
  let totalHeader = `<thead><tr><th>Phase</th>`;
  for (let i = 0; i < numPlayers; i++) {
    scoreHeader += `<th class="p${i}">Player ${i+1}</th>`;
    totalHeader += `<th class="p${i}">Player ${i+1}</th>`;
  }
  scoreHeader += `</tr></thead>`;
  totalHeader += `</tr></thead>`;

  scores.innerHTML += scoreHeader + `<tbody id="score-body"></tbody><tfoot><tr class="placement"><td>Placement</td>` +
    Array(numPlayers).fill().map((_, i) => `<td id="rank-${i}"></td>`).join('') + `</tr></tfoot>`;

  totals.innerHTML += totalHeader + `<tbody id="total-body"></tbody><tfoot><tr><th>Total</th>` +
    Array(numPlayers).fill().map((_, i) => `<th id="final-${i}"></th>`).join('') + `</tr></tfoot>`;

  const scoreBody = document.getElementById("score-body");
  const totalBody = document.getElementById("total-body");

  phases.forEach((label, r) => {
    let scoreRow = `<tr><td>${label}</td>`;
    let totalRow = `<tr><td>${label}</td>`;

    for (let p = 0; p < numPlayers; p++) {
      scoreRow += (label === "bonus")
        ? `<td><em>AUTO</em></td>`
        : `<td><input type="number" data-p="${p}" data-r="${r}" oninput="recalc()"></td>`;
      totalRow += `<td id="t-${p}-${r}"></td>`;
    }

    scoreBody.innerHTML += scoreRow + "</tr>";
    totalBody.innerHTML += totalRow + "</tr>";
  });

  recalc();
}

function recalc() {
  const numPlayers = parseInt(document.getElementById("playerCount").value);
  const totals = [];

  for (let p = 0; p < numPlayers; p++) {
    let top = 0, bottom = 0;

    for (let r = 0; r < phases.length; r++) {
      const label = phases[r];
      let val = 0;

      if (label === "bonus") {
        if (r === 5 && top > 220) val = 40;
        if (r === 11 && bottom > 220) val = 40;
      } else {
        const input = document.querySelector(`input[data-p="${p}"][data-r="${r}"]`);
        val = input ? parseInt(input.value) || 0 : 0;
      }

      if (r < 6) {
        top += val;
        document.getElementById(`t-${p}-${r}`).textContent = top;
      } else {
        bottom += val;
        document.getElementById(`t-${p}-${r}`).textContent = bottom;
      }
    }

    const total = top + bottom;
    document.getElementById(`final-${p}`).textContent = total;
    totals[p] = total;
  }

  updateRanks(totals);
}

function updateRanks(totals) {
  const scored = totals.some(t => t > 0);
  const numPlayers = totals.length;

  for (let i = 0; i < numPlayers; i++) {
    document.getElementById(`rank-${i}`).textContent = "";
  }
  if (!scored) return;

  const sorted = totals.map((t, i) => ({i, t}))
                       .sort((a, b) => b.t - a.t)
                       .map((obj, idx) => ({...obj, rank: idx + 1}));

  const ranks = Array(numPlayers);
  sorted.forEach(obj => ranks[obj.i] = `${ordinal(obj.rank)}`);

  for (let p = 0; p < numPlayers; p++) {
    document.getElementById(`rank-${p}`).textContent = ranks[p];
  }
}

function ordinal(n) {
  const s=["th","st","nd","rd"],v=n%100;
  return n+(s[(v-20)%10]||s[v]||s[0]);
}

function updateNames(i) {
  const name = document.querySelectorAll(".names input")[i].value || `Player ${i + 1}`;
  document.querySelectorAll(`.p${i}`).forEach(el => el.textContent = name);
}

buildTables();
</script>

</body>
</html>
