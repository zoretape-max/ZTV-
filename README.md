
<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="theme-color" content="#07111f">
<title>AVIA-MINES AI</title>
<style>
*{box-sizing:border-box}
body{
  margin:0;padding:20px 14px;
  background:#07111f;color:#f5f7ff;
  font-family:Arial,sans-serif;text-align:center
}
main{max-width:430px;margin:auto}
h1{color:#43e6b1;font-size:27px;margin:10px 0}
.subtitle{color:#aab9d0;font-size:14px;line-height:1.5}
.panel{
  background:#101e31;border:1px solid #243b53;
  border-radius:18px;padding:16px;margin:16px 0
}
label{display:block;margin-bottom:8px;color:#dbe7f6}
select,button{
  font-size:16px;border:0;border-radius:10px;padding:13px
}
select{background:#20344c;color:white;width:100%}
.grid{
  display:grid;grid-template-columns:repeat(5,minmax(0,1fr));
  gap:8px;margin:18px 0
}
.cell{
  aspect-ratio:1;background:#1b3450;color:white;
  border:2px solid #2b4968;border-radius:10px;
  font-size:25px;font-weight:bold;padding:0;
  cursor:pointer;position:relative
}
.cell.recommended{
  background:#174c40;border-color:#43e6b1;
  box-shadow:0 0 10px #43e6b155
}
.cell.mine{background:#57272e;border-color:#ff737e}
.cell.gem{background:#16473e;border-color:#43e6b1}
button.primary{
  width:100%;background:#43e6b1;color:#071b18;
  font-weight:bold;margin:5px 0
}
button.secondary{
  width:100%;background:#263950;color:white;margin:5px 0
}
#result{font-weight:bold;line-height:1.6;margin-top:12px}
.notice{
  font-size:12px;line-height:1.6;color:#aab9d0;
  text-align:left
}
.legend{font-size:13px;color:#c9d6e8;line-height:1.9}
</style>
</head>
<body>
<main>
  <h1>💎 AVIA-MINES AI</h1>
  <p class="subtitle">Grille Mines · 5 × 5 · 25 cases</p>

  <section class="panel">
    <label for="mineCount">Nombre de mines choisi dans le jeu</label>
    <select id="mineCount">
      <option value="1">1 mine</option>
      <option value="3">3 mines</option>
      <option value="5" selected>5 mines</option>
      <option value="7">7 mines</option>
      <option value="10">10 mines</option>
      <option value="15">15 mines</option>
      <option value="20">20 mines</option>
      <option value="24">24 mines</option>
    </select>

    <div class="grid" id="grid"></div>

    <button class="primary" id="analyze">🔎 ANALYSER</button>
    <button class="secondary" id="clear">↻ EFFACER</button>

    <div id="result" aria-live="polite">
      Appuie sur ANALYSER pour afficher les cases suggérées.
    </div>
  </section>

  <section class="panel">
    <div class="legend">
      💠 Case recommandée par la simulation<br>
      💎 Case marquée manuellement comme diamant<br>
      💣 Case marquée manuellement comme mine<br>
      ⬛ Case non renseignée
    </div>
  </section>

  <section class="panel notice">
    <strong>⚠️ Limite importante</strong><br>
    Les recommandations sont générées aléatoirement par cette
    simulation. Elles ne détectent pas les mines cachées et ne
    prédisent pas le résultat réel du casino. Une case suggérée
    peut contenir une mine. Ne mise pas d'argent sur la base de
    ces suggestions.
  </section>
</main>

<script>
const grid = document.getElementById("grid");
const result = document.getElementById("result");
const mineCount = document.getElementById("mineCount");
const analyzeButton = document.getElementById("analyze");

let cells = Array(25).fill("unknown");
let recommendations = [];

function renderGrid() {
  grid.innerHTML = "";

  cells.forEach((state, index) => {
    const button = document.createElement("button");
    button.className = "cell";
    button.type = "button";
    button.setAttribute("aria-label", "Case " + (index + 1));

    if (state === "gem") {
      button.textContent = "💎";
      button.classList.add("gem");
    } else if (state === "mine") {
      button.textContent = "💣";
      button.classList.add("mine");
    } else if (state === "unknown") {
      button.textContent = "";
      button.setAttribute("aria-label", "Case " + (index + 1) + ", non renseignée");
    }

    if (recommendations.includes(index) && state === "unknown") {
      button.classList.add("recommended");
      button.textContent = "✦";
      button.setAttribute("aria-label", "Case " + (index + 1) + ", suggestion expérimentale");
    }

    button.addEventListener("click", () => {
      const cycle = {
        unknown: "gem",
        gem: "mine",
        mine: "unknown"
      };
      cells[index] = cycle[cells[index]];
      recommendations = [];
      renderGrid();
      result.textContent =
        "Case " + (index + 1) + " mise à jour. Tu peux relancer l’analyse.";
    });

    grid.appendChild(button);
  });
}

function analyze() {
  const mineTotal = Number(mineCount.value);
  const unknownCells = cells
    .map((state, index) => state === "unknown" ? index : -1)
    .filter(index => index !== -1);

  if (unknownCells.length === 0) {
    result.textContent = "Aucune case non renseignée à suggérer. Appuie sur EFFACER pour recommencer.";
    recommendations = [];
    renderGrid();
    return;
  }

  // Tirage aléatoire de suggestions : aucune détection de mines réelles.
  const shuffled = [...unknownCells].sort(() => Math.random() - 0.5);
  const suggestedCount = Math.min(
    3,
    unknownCells.length,
    Math.max(1, Math.round(unknownCells.length * (25 - mineTotal) / 25))
  );

  recommendations = shuffled.slice(0, suggestedCount);
  renderGrid();

  const numbers = recommendations.map(index => index + 1).sort((a,b) => a-b);

  result.textContent =
    "Suggestions expérimentales : cases " + numbers.join(", ") +
    ". Ce tirage est aléatoire et ne révèle pas les vraies mines.";
}

function clearGrid() {
  cells = Array(25).fill("unknown");
  recommendations = [];
  renderGrid();
  result.textContent = "Grille effacée. Appuie sur ANALYSER pour recommencer.";
}

analyzeButton.addEventListener("click", analyze);
document.getElementById("clear").addEventListener("click", clearGrid);

renderGrid();
</script>
</body>
</html>

