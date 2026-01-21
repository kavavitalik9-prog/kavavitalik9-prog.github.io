<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<title>Графік відключення світла — Львівська область</title>

<style>
body{
  margin:0;
  font-family:Arial, sans-serif;
  background:#0d0d0d;
  color:#fff;
}
.container{
  max-width:1000px;
  margin:auto;
  padding:20px;
}
h1{
  text-align:center;
  color:#ffd000;
}
select, button{
  padding:10px;
  margin:10px 0;
  background:#1a1a1a;
  color:#fff;
  border:1px solid #333;
  border-radius:6px;
}
button{
  cursor:pointer;
}
.card{
  background:#151515;
  border-radius:10px;
  padding:15px;
  margin-bottom:15px;
  box-shadow:0 0 10px #000;
}
.card h3{
  margin-top:0;
  color:#ffd000;
}
.slot{
  display:flex;
  justify-content:space-between;
  padding:8px;
  margin-bottom:5px;
  border-radius:6px;
}
.on{ background:#063; color:#6aff9a; }
.off{ background:#400; color:#ff7a7a; }

.status{
  margin-top:10px;
  font-weight:bold;
}
.countdown{
  font-size:14px;
  opacity:0.8;
}

.modal{
  display:none;
  position:fixed;
  inset:0;
  background:rgba(0,0,0,0.95);
  overflow:auto;
  padding:20px;
}
.modal-content{
  max-width:900px;
  margin:auto;
  background:#111;
  padding:20px;
  border-radius:10px;
}
.close{
  font-size:28px;
  cursor:pointer;
  float:right;
}
.footer{
  text-align:center;
  opacity:0.6;
  margin-top:20px;
}
</style>
</head>

<body>
<div class="container">
  <h1>⚡ Львівська область — Середа</h1>

  <label>Оберіть вашу групу:</label><br>
  <select id="groupSelect"></select><br>

  <div id="current"></div>

  <button id="showAll">Показати всі групи</button>

  <div class="footer">⏱ Дані оновлюються автоматично</div>
</div>

<div class="modal" id="modal">
  <div class="modal-content">
    <span class="close" id="closeModal">&times;</span>
    <h2>Всі групи (Середа)</h2>
    <div id="allGroups"></div>
  </div>
</div>

<script>
// ===== ГРАФІК (СЕРЕДА) =====
const schedule = {
 "1.1":[["00:00","18:00","on"],["18:00","20:00","off"],["20:00","23:59","on"]],
 "1.2":[["00:00","01:30","off"],["01:30","23:59","on"]],
 "2.1":[["00:00","20:00","on"],["20:00","23:59","off"]],
 "2.2":[["00:00","23:59","on"]],
 "3.1":[["00:00","20:00","on"],["20:00","23:59","off"]],
 "3.2":[["00:00","23:59","on"]],
 "4.1":[["00:00","20:00","on"],["20:00","22:00","off"],["22:00","23:59","on"]],
 "4.2":[["00:00","18:00","on"],["18:00","20:00","off"],["20:00","23:59","on"]],
 "5.1":[["00:00","18:00","on"],["18:00","20:00","off"],["20:00","23:59","on"]],
 "5.2":[["00:00","23:59","on"]],
 "6.1":[["00:00","01:30","off"],["01:30","23:59","on"]],
 "6.2":[["00:00","23:59","on"]]
};

const groups = Object.keys(schedule);
const groupSelect = document.getElementById("groupSelect");
const currentDiv = document.getElementById("current");
const modal = document.getElementById("modal");
const allGroupsDiv = document.getElementById("allGroups");

// заповнюємо селект
groups.forEach(g=>{
  const opt=document.createElement("option");
  opt.value=g;
  opt.textContent="Група "+g;
  groupSelect.appendChild(opt);
});

// хвилини з часу
function toMin(t){
  const [h,m]=t.split(":").map(Number);
  return h*60+m;
}

function renderGroup(g){
  currentDiv.innerHTML="";
  const now=new Date();
  const nowMin=now.getHours()*60+now.getMinutes();

  const card=document.createElement("div");
  card.className="card";
  card.innerHTML=`<h3>Група ${g}</h3>`;

  let statusText="";

  schedule[g].forEach(s=>{
    const [st,en,state]=s;
    const stM=toMin(st);
    const enM=toMin(en);

    const row=document.createElement("div");
    row.className="slot "+(state==="on"?"on":"off");
    row.innerHTML=`<span>${st}–${en}</span><span>${state==="on"?"Світло є":"Світла нема"}</span>`;
    card.appendChild(row);

    if(nowMin>=stM && nowMin<enM){
      const diff=enM-nowMin;
      const h=Math.floor(diff/60);
      const m=diff%60;
      statusText=`${state==="on"?"🟢 Світло є":"🔴 Світла нема"} — ${state==="on"?"до відключення":"до увімкнення"} ${h} год ${m} хв`;
    }
  });

  const status=document.createElement("div");
  status.className="status";
  status.textContent=statusText || "⏳ Немає даних";
  card.appendChild(status);

  currentDiv.appendChild(card);
}

// подія
groupSelect.onchange=()=>renderGroup(groupSelect.value);
renderGroup(groups[0]);

// ===== ВСІ ГРУПИ =====
document.getElementById("showAll").onclick=()=>{
  modal.style.display="block";
  allGroupsDiv.innerHTML="";
  groups.forEach(g=>{
    const c=document.createElement("div");
    c.className="card";
    c.innerHTML=`<h3>Група ${g}</h3>`;
    schedule[g].forEach(s=>{
      const r=document.createElement("div");
      r.className="slot "+(s[2]==="on"?"on":"off");
      r.innerHTML=`${s[0]}–${s[1]} — ${s[2]==="on"?"Світло є":"Світла нема"}`;
      c.appendChild(r);
    });
    allGroupsDiv.appendChild(c);
  });
};
document.getElementById("closeModal").onclick=()=>modal.style.display="none";

// автооновлення
setInterval(()=>renderGroup(groupSelect.value),60000);
</script>
</body>
</html>
