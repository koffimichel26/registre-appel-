# registre-appel-
Logiciel permettant de suivre l'assiduité des élèves 

<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Registre d'Appel Lycée Moderne de Treichville'</title>

<style>
body { font-family:Segoe UI; margin:0; background:#f2f5fb;}
header { background:#2f4f88; color:white; padding:20px; text-align:center;}
.container { padding:20px; max-width:1300px; margin:auto;}
.card { background:white; padding:20px; margin-bottom:20px; border-radius:12px; box-shadow:0 4px 10px rgba(0,0,0,0.05);}
button { background:#2f4f88; color:white; padding:8px 15px; border:none; border-radius:6px; cursor:pointer;}
button:hover{ background:#1f3661;}
input, select { padding:6px; border-radius:6px; border:1px solid #ccc;}
table { width:100%; border-collapse:collapse;}
th, td { border:1px solid #ddd; padding:8px; text-align:center;}
th { background:#2f4f88; color:white;}
.hidden { display:none;}
textarea { width:100%; height:60px;}
.smallBtn { padding:5px 10px; font-size:12px;}
.dashboard { font-weight:bold; color:#2f4f88;}
.settings { margin-top:10px; }
</style>
</head>

<body>

<header>
<h2>📘 Registre d'Appel Lycée Moderne de Treichville</h2>
<p>Système d’Alerte Parentale</p>
</header>

<div class="container">

<!-- LOGIN -->
<div class="card" id="loginCard">
<h3>🔐 Accès Administration</h3>
<input type="password" id="password" placeholder="Mot de passe">
<button onclick="login()">Connexion</button>
<p style="font-size:12px;color:gray;">Mot de passe par défaut : admin123</p>
</div>

<!-- APPLICATION -->
<div id="appContent" class="hidden">

<div class="card">
Classe :
<select id="classe">
<option>6e A</option>
<option>6e B</option>
<option>5e A</option>
<option>4e A</option>
<option>3e A</option>
</select>

Semaine :
<input type="week" id="semaine">

Nombre élèves :
<input type="number" id="nbEleves" value="1" min="1">

<button onclick="ajouterLignes()">Ajouter</button>
<button onclick="calculer()">Calculer</button>
<button onclick="exportUSB()">Exporter USB</button>
</div>

<div class="card dashboard">
Total élèves : <span id="total">0</span> |
Moyenne classe : <span id="moyenne">0%</span> |
Élèves critiques (<60%) : <span id="critique">0</span>
</div>

<div class="card">
<table id="tableAppel">
<thead>
<tr>
<th>N°</th>
<th>Nom</th>
<th>Téléphone</th>
<th>L</th><th>M</th><th>Me</th><th>J</th><th>V</th>
<th>Taux</th>
<th>Message</th>
<th>Actions</th>
</tr>
</thead>
<tbody></tbody>
</table>
</div>

<div class="card settings">
<h3>⚙ Modifier le mot de passe</h3>
<input type="password" id="newPassword" placeholder="Nouveau mot de passe">
<button onclick="changerMotDePasse()">Modifier</button>
</div>

</div>
</div>

<script>

let defaultPassword = "admin123";

function getPassword(){
    return localStorage.getItem("appPassword") || defaultPassword;
}

function login(){
    if(document.getElementById("password").value === getPassword()){
        document.getElementById("loginCard").classList.add("hidden");
        document.getElementById("appContent").classList.remove("hidden");
        charger();
    } else {
        alert("Mot de passe incorrect");
    }
}

function changerMotDePasse(){
    let newPass = document.getElementById("newPassword").value;
    if(newPass.length < 4){
        alert("Mot de passe trop court");
        return;
    }
    localStorage.setItem("appPassword", newPass);
    alert("Mot de passe modifié avec succès !");
}

let compteur = 0;

function ajouterLignes(){
    let n = parseInt(document.getElementById("nbEleves").value);
    let tbody = document.querySelector("#tableAppel tbody");

    for(let i=0;i<n;i++){
        compteur++;
        tbody.innerHTML += `
        <tr>
        <td>${compteur}</td>
        <td><input type="text"></td>
        <td><input type="text" placeholder="2250700000000"></td>
        <td><input type="checkbox"></td>
        <td><input type="checkbox"></td>
        <td><input type="checkbox"></td>
        <td><input type="checkbox"></td>
        <td><input type="checkbox"></td>
        <td class="result"></td>
        <td><textarea></textarea></td>
        <td>
        <button class="smallBtn" onclick="envoyerWhatsApp(this)">WhatsApp</button>
        <button class="smallBtn" onclick="supprimer(this)">🗑</button>
        </td>
        </tr>`;
    }

    majDashboard();
    sauvegarder();
}

function supprimer(btn){
    btn.parentElement.parentElement.remove();
    renumeroter();
    majDashboard();
    sauvegarder();
}

function renumeroter(){
    let rows=document.querySelectorAll("#tableAppel tbody tr");
    compteur=0;
    rows.forEach(r=>{
        compteur++;
        r.cells[0].innerText=compteur;
    });
}

function calculer(){
    let rows=document.querySelectorAll("#tableAppel tbody tr");
    let total=0, critique=0;

    rows.forEach(r=>{
        let checks=r.querySelectorAll("input[type='checkbox']");
        let present=0;
        checks.forEach(c=>{ if(c.checked) present++; });

        let taux=(present/5)*100;
        r.querySelector(".result").innerText=taux+"%";
        total+=taux;

        let nom=r.cells[1].querySelector("input").value;
        let message;

        if(taux<60){
            message=`Bonjour, votre enfant ${nom} a un taux de présence de ${taux}%. Merci de prendre des dispositions immédiates.`;
            critique++;
        } else {
            message=`Bonjour, votre enfant ${nom} a un bon taux de présence de ${taux}%. Continuez ainsi.`;
        }

        r.querySelector("textarea").value=message;
    });

    let moyenne=rows.length? total/rows.length:0;
    document.getElementById("total").innerText=rows.length;
    document.getElementById("moyenne").innerText=moyenne.toFixed(1)+"%";
    document.getElementById("critique").innerText=critique;

    sauvegarder();
}

function envoyerWhatsApp(btn){
    let row=btn.parentElement.parentElement;
    let numero=row.cells[2].querySelector("input").value.replace("+","");
    let message=row.querySelector("textarea").value;

    if(numero===""){ alert("Numéro manquant"); return;}

    let url="https://wa.me/"+numero+"?text="+encodeURIComponent(message);
    window.open(url,"_blank");
}

function sauvegarder(){
    let rows=document.querySelectorAll("#tableAppel tbody tr");
    let data=[];

    rows.forEach(r=>{
        let obj={
            nom:r.cells[1].querySelector("input").value,
            tel:r.cells[2].querySelector("input").value,
            jours:[],
            message:r.querySelector("textarea").value
        };

        r.querySelectorAll("input[type='checkbox']").forEach(c=>{
            obj.jours.push(c.checked);
        });

        data.push(obj);
    });

    localStorage.setItem("registreLMTR", JSON.stringify(data));
}

function charger(){
    let saved=localStorage.getItem("registreLMTR");
    if(saved){
        let data=JSON.parse(saved);
        data.forEach(eleve=>{
            ajouterLignes();
            let last=document.querySelector("#tableAppel tbody tr:last-child");

            last.cells[1].querySelector("input").value=eleve.nom;
            last.cells[2].querySelector("input").value=eleve.tel;

            let checks=last.querySelectorAll("input[type='checkbox']");
            eleve.jours.forEach((val,i)=>{ checks[i].checked=val; });

            last.querySelector("textarea").value=eleve.message;
        });
        calculer();
    }
}

function exportUSB(){
    let data=localStorage.getItem("registreLMTR");
    let blob=new Blob([data],{type:"application/json"});
    let link=document.createElement("a");
    link.href=URL.createObjectURL(blob);
    link.download="registre_direction.json";
    link.click();
}

</script>

</body>
</html>
