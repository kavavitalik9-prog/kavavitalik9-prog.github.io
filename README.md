<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Графіки відключення світла — Львівська область</title>

<style>
body{
  margin:0;
  font-family:Arial, sans-serif;
  background:#0d0d0d;
  color:#fff;
}
.container{
  max-width:1100px;
  margin:auto;
  padding:15px;
}
h1{
  text-align:center;
  color:#ffd000;
  font-size:22px;
}
.days{
  display:flex;
  gap:8px;
  overflow-x:auto;
  margin-bottom:15px;
}
.days button{
  padding:8px 12px;
  background:#1a1a1a;
  color:#fff;
  border:1px solid #333;
  border-radius:6px;
  cursor:pointer;
  white-space:nowrap;
}
.days button.active{
  background:#ffd000;
  color:#000;
  font-weight:bold;
}
select, button{
  width:100%;
  padding:10px;
  margin:8px 0;
  background:#1a1a1a;
  color:#fff;
  border:1px solid #333;
  border-radius:6px;
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
  padding:7px;
  margin-bottom:5px;
  border-radius:6px;
  font-size:14px;
}
.on{ background:#063; color:#6aff9a; }
.off{ background:#400; color:#ff7a7a; }
.status{
  margin-top:10px;
  font-weight:bold;
  font-size:14px;
}
.center{
  text-align:center;
  opacity:0.8;
}
.modal{
  display:none;
  position:fixed;
  inset:0;
  background:rgba(0,0,0,0.95);
  overflow:auto;
  padding:15px;
}
.modal-content{
  max-width:900px;
  margin:auto;
  background:#111;
  padding:15px;
  border-radius:10px;
}
.close{
  font-size:26px;
  cursor:pointer;
  float:right;
}
.footer{
  text-align:center;
  opacity:0.6;
  margin-top:15px;
  font-size:13px;
}

/* 📱 адаптація */
@media (max-width:600px){
  h1{font-size:18px;}
  .slot{flex-direction:column; gap:4px;}
}
</style>
</head>

<body>
<div class="container">
  <h1>⚡ Львівська область</h1>

  <div class="days" id="days"></div>

  <label>Оберіть вашу групу:</label>
  <select id="groupSelect"></select>

  <div id="content"></div>

  <button id="showAll">Показати всі групи</button>

  <div class="footer">⏱ Дані оновлюються автоматично</div>
</div>

<div class="modal" id="modal">
  <div class="modal-content">
    <span class="close" id="closeModal">&times;</span>
    <h2>Всі групи — Середа</h2>
    <div id="allGroups"></div>
  </div>
</div>

<script>
// ===== ДНІ =====
const daysList = ["Понеділок","Вівторок","Середа","Четвер","Пʼятниця","Субота","Неділя"];
let currentDay = "Середа";

// ===== ГРАФІК ЛИШЕ НА СЕРЕДУ =====
const wednesday = {
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

const groups = Object.keys(wednesday);
const daysDiv = document.getElementById("days");
const groupSelect = document.getElementById("groupSelect");
const content = document.getElementById("content");
const modal = document.getElementById("modal");
const allGroups = document.getElementById("allGroups");

// кнопки днів
daysList.forEach(d=>{
  const b=document.createElement("button");
  b.textContent=d;
  if(d===currentDay) b.classList.add("active");
  b.onclick=()=>{
    currentDay=d;
    document.querySelectorAll(".days button").forEach(x=>x.classList.remove("active"));
    b.classList.add("active");
    render();
  };
  daysDiv.appendChild(b);
});

// select груп
groups.forEach(g=>{
  const o=document.createElement("option");
  o.value=g;
  o.textContent="Група "+g;
  groupSelect.appendChild(o);
});

function toMin(t){
  const [h,m]=t.split(":").map(Number);
  return h*60+m;
}

function render(){
  content.innerHTML="";

  if(currentDay!=="Середа"){
    content.innerHTML=`<div class="card center">⏳ Графік ще формується</div>`;
    return;
  }

  const g=groupSelect.value;
  const now=new Date();
  const nowMin=now.getHours()*60+now.getMinutes();

  const card=document.createElement("div");
  card.className="card";
  card.innerHTML=`<h3>Група ${g}</h3>`;

  let statusText="";

  wednesday[g].forEach(s=>{
    const [st,en,state]=s;
    const stM=toMin(st), enM=toMin(en);

    const row=document.createElement("div");
    row.className="slot "+(state==="on"?"on":"off");
    row.innerHTML=`<span>${st}–${en}</span><span>${state==="on"?"Світло є":"Світла нема"}</span>`;
    card.appendChild(row);

    if(nowMin>=stM && nowMin<enM){
      const diff=enM-nowMin;
      const h=Math.floor(diff/60), m=diff%60;
      statusText = (state==="on"?"🟢 Світло є":"🔴 Світла нема")
        + " — " + (state==="on"?"до відключення":"до увімкнення")
        + ` ${h} год ${m} хв`;
    }
  });

  const status=document.createElement("div");
  status.className="status";
  status.textContent=statusText || "⏳ Немає активного інтервалу";
  card.appendChild(status);

  content.appendChild(card);
}

groupSelect.onchange=render;
render();

// всі групи
document.getElementById("showAll").onclick=()=>{
  modal.style.display="block";
  allGroups.innerHTML="";
  groups.forEach(g=>{
    const c=document.createElement("div");
    c.className="card";
    c.innerHTML=`<h3>Група ${g}</h3>`;
    wednesday[g].forEach(s=>{
      const r=document.createElement("div");
      r.className="slot "+(s[2]==="on"?"on":"off");
      r.textContent=`${s[0]}–${s[1]} — ${s[2]==="on"?"Світло є":"Світла нема"}`;
      c.appendChild(r);
    });
    allGroups.appendChild(c);
  });
};
document.getElementById("closeModal").onclick=()=>modal.style.display="none";

setInterval(render,60000);
</script>
</body>
</html>
