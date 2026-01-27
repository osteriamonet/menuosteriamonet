<!DOCTYPE html>
<html lang="it">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Osteria Monët - Editor Completo</title>

<link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;500&display=swap" rel="stylesheet">
<style>
body {
  font-family: 'Montserrat', sans-serif;
  background: #faf8f5;
  margin: 0;
  padding: 20px;
  color: #2f5d50;
}
.container {
  max-width: 950px;
  margin: auto;
  background: white;
  padding: 20px 30px;
  border: 2px solid #c9a24d;
  border-radius: 10px;
  box-shadow: 0 3px 15px rgba(0,0,0,0.08);
}
h1, h2 {
  text-align: center;
  color: #c9a24d;
}
hr {
  border: none;
  border-top: 1px solid #c9a24d50;
  margin: 20px 0;
}
button {
  font-family: inherit;
  font-size: 14px;
  padding: 4px 8px;
  margin: 2px;
  border-radius: 6px;
  border: 1px solid #ccc;
  cursor: pointer;
  background: #2f5d50;
  color: white;
}
button:hover {
  background: #c9a24d;
  color: #2f5d50;
}
input, textarea, select {
  font-family: inherit;
  font-size: 14px;
  padding: 4px 6px;
  margin: 2px;
  border-radius: 6px;
  border: 1px solid #ccc;
}
ul {
  list-style: none;
  padding-left: 10px;
}
li {
  margin: 4px 0;
}
</style>
</head>
<body>

<div class="container">
<h1>Editor Completo - Osteria Monët</h1>
<p>Gestisci macrocategorie, sottosezioni e piatti. Tutto salvato in <b>localStorage.menuData</b>.</p>

<section>
  <h2>Gestione Macrocategorie</h2>
  <input id="newCategory" placeholder="Nuova categoria...">
  <button onclick="aggiungiCategoria()">➕ Aggiungi Categoria</button>
  <ul id="categoryList"></ul>
</section>

<section>
  <h2>Gestione Sottosezioni</h2>
  <select id="selectCategory"></select>
  <input id="newSection" placeholder="Nuova sottosezione...">
  <button onclick="aggiungiSezione()">➕ Aggiungi Sottosezione</button>
  <ul id="sectionList"></ul>
</section>

<section>
  <h2>Gestione Piatti</h2>
  <select id="selectCatForItem"></select>
  <select id="selectSecForItem"></select>
  <input id="itemName" placeholder="Nome piatto">
  <input id="itemPrice" placeholder="Prezzo (€)">
  <textarea id="itemDesc" placeholder="Descrizione"></textarea>
  <button onclick="aggiungiPiatto()">➕ Aggiungi / Aggiorna Piatto</button>
  <ul id="itemList"></ul>
</section>

<hr>

<section>
<h2>Backup / Gestione</h2>
<button onclick="esportaMenu()">⬇️ Esporta JSON</button>
<input type="file" id="importFile" accept=".json" onchange="importaMenu(event)">
<button onclick="resetMenu()">⚠️ Reset Completo</button>
</section>
</div>

<script>
const defaultMenu = {
  mare: { antipasti: [], primi: [], secondi: [], dolci: [] },
  terra: { antipasti: [], primi: [], secondi: [], dolci: [] },
  vini: { bianchi: [], rossi: [], bollicine: [], dolci: [] },
  cocktail: { classici: [], signature: [], distillati: [] }
};

let menuData = JSON.parse(localStorage.getItem("menuData")) || JSON.parse(JSON.stringify(defaultMenu));

function salvaMenu() {
  localStorage.setItem("menuData", JSON.stringify(menuData));
  mostraTutto();
}

// --- CATEGORIE ---
function aggiornaCategorie() {
  const catSelect1 = document.getElementById("selectCategory");
  const catSelect2 = document.getElementById("selectCatForItem");
  const list = document.getElementById("categoryList");
  catSelect1.innerHTML = "";
  catSelect2.innerHTML = "";
  list.innerHTML = "";

  for (let cat in menuData) {
    const li = document.createElement("li");
    li.innerHTML = `${cat} <button onclick="rinominaCategoria('${cat}')">✏️</button> <button onclick="rimuoviCategoria('${cat}')">❌</button>`;
    list.appendChild(li);

    const opt1 = document.createElement("option");
    opt1.value = cat;
    opt1.innerText = cat;
    catSelect1.appendChild(opt1);

    const opt2 = document.createElement("option");
    opt2.value = cat;
    opt2.innerText = cat;
    catSelect2.appendChild(opt2);
  }

  aggiornaSezioni();
  aggiornaPiatti();
}

function aggiungiCategoria() {
  const val = document.getElementById("newCategory").value.trim().toLowerCase();
  if (!val) return alert("Inserisci un nome categoria");
  if (menuData[val]) return alert("Categoria già esistente");

  // crea categoria vuota
  menuData[val] = {};

  // salva etichetta e icona
  const labels = JSON.parse(localStorage.getItem("categoryLabels") || "{}");
  const icons = JSON.parse(localStorage.getItem("categoryIcons") || "{}");
  labels[val] = val.charAt(0).toUpperCase() + val.slice(1);
  icons[val] = "📜";
  localStorage.setItem("categoryLabels", JSON.stringify(labels));
  localStorage.setItem("categoryIcons", JSON.stringify(icons));

  document.getElementById("newCategory").value = "";
  salvaMenu();
}

function rinominaCategoria(cat) {
  const nuovo = prompt("Nuovo nome categoria:", cat);
  if (!nuovo || nuovo.trim() === "") return;
  if (menuData[nuovo]) return alert("Nome già esistente");

  menuData[nuovo] = menuData[cat];
  delete menuData[cat];

  const labels = JSON.parse(localStorage.getItem("categoryLabels") || "{}");
  const icons = JSON.parse(localStorage.getItem("categoryIcons") || "{}");
  if (labels[cat]) { labels[nuovo] = labels[cat]; delete labels[cat]; }
  if (icons[cat]) { icons[nuovo] = icons[cat]; delete icons[cat]; }
  localStorage.setItem("categoryLabels", JSON.stringify(labels));
  localStorage.setItem("categoryIcons", JSON.stringify(icons));

  salvaMenu();
}

function rimuoviCategoria(cat) {
  if (confirm("Eliminare categoria e tutto il suo contenuto?")) {
    delete menuData[cat];
    const labels = JSON.parse(localStorage.getItem("categoryLabels") || "{}");
    const icons = JSON.parse(localStorage.getItem("categoryIcons") || "{}");
    delete labels[cat];
    delete icons[cat];
    localStorage.setItem("categoryLabels", JSON.stringify(labels));
    localStorage.setItem("categoryIcons", JSON.stringify(icons));
    salvaMenu();
  }
}

// --- SOTTOSEZIONI ---
function aggiornaSezioni() {
  const cat = document.getElementById("selectCategory").value;
  const list = document.getElementById("sectionList");
  const secSelect = document.getElementById("selectSecForItem");
  list.innerHTML = "";
  secSelect.innerHTML = "";

  if (!cat || !menuData[cat]) return;

  for (let sec in menuData[cat]) {
    const li = document.createElement("li");
    li.innerHTML = `${sec} <button onclick="rinominaSezione('${cat}','${sec}')">✏️</button> <button onclick="rimuoviSezione('${cat}','${sec}')">❌</button>`;
    list.appendChild(li);

    const opt = document.createElement("option");
    opt.value = sec;
    opt.innerText = sec;
    secSelect.appendChild(opt);
  }

  aggiornaPiatti();
}

function aggiungiSezione() {
  const cat = document.getElementById("selectCategory").value;
  const sec = document.getElementById("newSection").value.trim().toLowerCase();
  if (!cat) return alert("Seleziona una categoria");
  if (!sec) return alert("Inserisci nome sottosezione");
  if (menuData[cat][sec]) return alert("Sottosezione già esistente");

  menuData[cat][sec] = [];
  document.getElementById("newSection").value = "";
  salvaMenu();
}

function rinominaSezione(cat, sec) {
  const nuovo = prompt("Nuovo nome sottosezione:", sec);
  if (!nuovo || nuovo.trim() === "") return;
  if (menuData[cat][nuovo]) return alert("Nome già esistente");
  menuData[cat][nuovo] = menuData[cat][sec];
  delete menuData[cat][sec];
  salvaMenu();
}

function rimuoviSezione(cat, sec) {
  if (confirm("Eliminare sottosezione e tutti i piatti?")) {
    delete menuData[cat][sec];
    salvaMenu();
  }
}

// --- PIATTI ---
function aggiornaPiatti() {
  const cat = document.getElementById("selectCatForItem").value;
  const sec = document.getElementById("selectSecForItem").value;
  const list = document.getElementById("itemList");
  list.innerHTML = "";

  if (!cat || !sec || !menuData[cat] || !menuData[cat][sec]) return;

  menuData[cat][sec].forEach((item, idx) => {
    const li = document.createElement("li");
    li.innerHTML = `<b>${item.nome}</b> - €${item.prezzo || ""} <i>${item.descrizione || ""}</i>
      <button onclick="modificaPiatto('${cat}','${sec}',${idx})">✏️</button>
      <button onclick="rimuoviPiatto('${cat}','${sec}',${idx})">❌</button>`;
    list.appendChild(li);
  });
}

function aggiungiPiatto() {
  const cat = document.getElementById("selectCatForItem").value;
  const sec = document.getElementById("selectSecForItem").value;
  const nome = document.getElementById("itemName").value.trim();
  const prezzo = document.getElementById("itemPrice").value.trim();
  const desc = document.getElementById("itemDesc").value.trim();

  if (!cat || !sec) return alert("Seleziona categoria e sottosezione");
  if (!nome) return alert("Nome piatto obbligatorio");

  menuData[cat][sec].push({ nome, prezzo, descrizione: desc });
  document.getElementById("itemName").value = "";
  document.getElementById("itemPrice").value = "";
  document.getElementById("itemDesc").value = "";
  salvaMenu();
}

function modificaPiatto(cat, sec, idx) {
  const item = menuData[cat][sec][idx];
  const nome = prompt("Nome piatto:", item.nome);
  if (!nome) return;
  const prezzo = prompt("Prezzo (€):", item.prezzo || "");
  const desc = prompt("Descrizione:", item.descrizione || "");
  menuData[cat][sec][idx] = { nome, prezzo, descrizione: desc };
  salvaMenu();
}

function rimuoviPiatto(cat, sec, idx) {
  if (confirm("Eliminare piatto?")) {
    menuData[cat][sec].splice(idx, 1);
    salvaMenu();
  }
}

// --- BACKUP / RESET ---
function esportaMenu() {
  const blob = new Blob([JSON.stringify(menuData, null, 2)], { type: "application/json" });
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = "menuData.json";
  a.click();
  URL.revokeObjectURL(url);
}

function importaMenu(event) {
  const file = event.target.files[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = e => {
    try {
      menuData = JSON.parse(e.target.result);
      salvaMenu();
      alert("Menù importato correttamente!");
    } catch {
      alert("Errore: file non valido.");
    }
  };
  reader.readAsText(file);
}

function resetMenu() {
  if (confirm("Cancellare tutto il menù?")) {
    menuData = JSON.parse(JSON.stringify(defaultMenu));
    localStorage.removeItem("categoryLabels");
    localStorage.removeItem("categoryIcons");
    salvaMenu();
  }
}

// --- AGGIORNAMENTO DINAMICO ---
function mostraTutto() {
  aggiornaCategorie();
}
document.getElementById("selectCategory").addEventListener("change", aggiornaSezioni);
document.getElementById("selectCatForItem").addEventListener("change", aggiornaSezioni);
document.getElementById("selectSecForItem").addEventListener("change", aggiornaPiatti);

mostraTutto();
</script>

</body>
</html>
