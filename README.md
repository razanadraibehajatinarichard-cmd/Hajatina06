<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>ERP Comptable IA Ultra Pro</title>

<script src="https://cdn.jsdelivr.net/npm/tesseract.js@5/dist/tesseract.min.js"></script>

<style>
body { font-family: Arial; background:#eef2f7; padding:15px; }

h2 { background:#0b1f3a; color:white; padding:12px; border-radius:10px; }

section { background:white; padding:15px; margin-top:15px; border-radius:10px; }

textarea { width:100%; height:120px; }

button { padding:10px; margin:5px; cursor:pointer; }

table { width:100%; border-collapse:collapse; margin-top:10px; }

th,td { border:1px solid #ddd; padding:8px; }

th { background:#222; color:#fff; }

.badge { background:#1a8; color:white; padding:4px 8px; border-radius:5px; }

.flex { display:flex; gap:10px; flex-wrap:wrap; }
</style>
</head>

<body>

<h2>🏢 ERP COMPTABLE IA ULTRA PRO</h2>

<!-- OCR -->
<section>
<h3>📷 Scan (Caméra / Galerie)</h3>
<input type="file" id="img" accept="image/*" capture="environment">
<button onclick="scan()">📸 OCR Scan</button>
<textarea id="ocr"></textarea>
</section>

<!-- INPUT -->
<section>
<h3>✍️ Écritures comptables ERP</h3>
<small>Format : date;débit;crédit;montant;description</small>
<textarea id="input"></textarea>

<div class="flex">
<button onclick="ai()">🧠 IA Auto-Correction</button>
<button onclick="process()">⚙️ Générer ERP</button>
<button onclick="save()">💾 Sauvegarde</button>
<button onclick="exportCSV()">📊 Export CSV</button>
<button onclick="exportJSON()">📦 Export JSON</button>
</div>
</section>

<!-- JOURNAL -->
<section>
<h3>📒 Journal ERP</h3>
<table id="journal"></table>
</section>

<!-- GRAND LIVRE -->
<section>
<h3>📘 Grand Livre</h3>
<div id="gl"></div>
</section>

<!-- BALANCE -->
<section>
<h3>⚖️ Balance Générale</h3>
<div id="balance"></div>
</section>

<!-- BILAN -->
<section>
<h3>📊 Bilan ERP</h3>
<div id="bilan"></div>
</section>

<!-- RESULTAT -->
<section>
<h3>📈 Compte de Résultat</h3>
<div id="res"></div>
</section>

<script>

let dataERP = [];
let GL = {};

// ================= OCR =================
function scan(){
    let f = document.getElementById("img").files[0];
    if(!f) return alert("Choisir image");

    Tesseract.recognize(f,'fra').then(({data:{text}})=>{
        document.getElementById("ocr").value = text;
        document.getElementById("input").value += "\n"+text;
    });
}

// ================= IA =================
function ai(){
    let t = document.getElementById("input").value;

    t = t.replace(/vente/gi,"Ventes")
         .replace(/achat/gi,"Achats")
         .replace(/banque/gi,"Banque")
         .replace(/caisse/gi,"Caisse")
         .replace(/charge/gi,"Charges")
         .replace(/produit/gi,"Produits");

    document.getElementById("input").value = t;

    alert("🧠 IA ERP : normalisation terminée");
}

// ================= PROCESS =================
function process(){

    let lines = document.getElementById("input").value.trim().split("\n");

    dataERP = [];
    GL = {};

    lines.forEach(l=>{
        if(!l.includes(";")) return;

        let [date,debit,credit,montant,desc] = l.split(";");
        montant = parseFloat(montant)||0;

        dataERP.push({date,debit,credit,montant,desc});

        if(!GL[debit]) GL[debit]={d:0,c:0};
        if(!GL[credit]) GL[credit]={d:0,c:0};

        GL[debit].d += montant;
        GL[credit].c += montant;
    });

    renderJournal();
    renderGL();
    renderBalance();
    renderBilan();
    renderResult();
}

// ================= JOURNAL =================
function renderJournal(){
    let html = `<tr>
    <th>Date</th><th>Débit</th><th>Crédit</th><th>Montant</th><th>Description</th>
    </tr>`;

    dataERP.forEach(j=>{
        html += `<tr>
        <td>${j.date}</td>
        <td>${j.debit}</td>
        <td>${j.credit}</td>
        <td>${j.montant}</td>
        <td>${j.desc}</td>
        </tr>`;
    });

    document.getElementById("journal").innerHTML = html;
}

// ================= GRAND LIVRE =================
function renderGL(){
    let html="";

    for(let c in GL){
        html += `<h4>📌 ${c}</h4>
        <table>
        <tr><th>Débit</th><th>Crédit</th></tr>
        <tr><td>${GL[c].d}</td><td>${GL[c].c}</td></tr>
        </table>`;
    }

    document.getElementById("gl").innerHTML = html;
}

// ================= BALANCE =================
function renderBalance(){
    let html = `<table><tr><th>Compte</th><th>Débit</th><th>Crédit</th><th>Solde</th></tr>`;

    for(let c in GL){
        let solde = GL[c].d - GL[c].c;

        html += `<tr>
        <td>${c}</td>
        <td>${GL[c].d}</td>
        <td>${GL[c].c}</td>
        <td>${solde}</td>
        </tr>`;
    }

    html += `</table>`;

    document.getElementById("balance").innerHTML = html;
}

// ================= BILAN =================
function renderBilan(){
    let actif=0, passif=0;

    for(let c in GL){
        let s = GL[c].d - GL[c].c;
        if(s>0) actif += s;
        else passif += Math.abs(s);
    }

    document.getElementById("bilan").innerHTML=`
    <p>Actif : <span class="badge">${actif}</span></p>
    <p>Passif : <span class="badge">${passif}</span></p>
    <p>${actif===passif?"✔ ÉQUILIBRÉ":"❌ NON ÉQUILIBRÉ"}</p>
    `;
}

// ================= RESULTAT =================
function renderResult(){
    let prod=0, chg=0;

    dataERP.forEach(j=>{
        if(j.credit.toLowerCase().includes("vente")) prod+=j.montant;
        if(j.debit.toLowerCase().includes("achat")||j.debit.toLowerCase().includes("charge"))
            chg+=j.montant;
    });

    document.getElementById("res").innerHTML=`
    <p>Produits : ${prod}</p>
    <p>Charges : ${chg}</p>
    <p><b>Résultat : ${prod-chg}</b></p>
    `;
}

// ================= SAVE =================
function save(){
    localStorage.setItem("ERP_COMPTA",JSON.stringify(dataERP));
    alert("💾 Sauvegardé ERP !");
}

// ================= EXPORT CSV =================
function exportCSV(){
    let csv = "date,debit,credit,montant,desc\n";

    dataERP.forEach(j=>{
        csv += `${j.date},${j.debit},${j.credit},${j.montant},${j.desc}\n`;
    });

    let blob = new Blob([csv],{type:"text/csv"});
    let a = document.createElement("a");
    a.href = URL.createObjectURL(blob);
    a.download = "ERP_compta.csv";
    a.click();
}

// ================= EXPORT JSON =================
function exportJSON(){
    let blob = new Blob([JSON.stringify(dataERP,null,2)],{type:"application/json"});
    let a = document.createElement("a");
    a.href = URL.createObjectURL(blob);
    a.download = "ERP_compta.json";
    a.click();
}

</script>

</body>
</html>