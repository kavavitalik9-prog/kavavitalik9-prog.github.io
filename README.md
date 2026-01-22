<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Графік світла — Львівська область</title>

<style>
body{
  margin:0;
  background:linear-gradient(135deg,#0f1115,#1b1e27);
  color:#fff;
  font-family:system-ui,Segoe UI,sans-serif;
}
.container{max-width:480px;margin:auto;padding:10px}
.header{
  background:#222;
  border-radius:16px;
  padding:14px;
  box-shadow:0 4px 15px rgba(0,0,0,.6);
  text-align:center;
  position:relative;
}
.header h1{margin:0;color:#ffc400}
.update-time{
  margin-top:6px;
  font-size:14px;
  opacity:.85;
}

.admin-btn{
  position:absolute;
  right:12px;
  top:12px;
  background:#333;
  border-radius:50%;
  width:36px;
  height:36px;
  display:flex;
  align-items:center;
  justify-content:center;
  cursor:pointer;
}

select,button,textarea{
  width:100%;
  margin-top:8px;
  padding:12px;
  border-radius:12px;
  border:none;
  background:#2a2a2a;
  color:#fff;
  font-size:16px;
}

textarea{resize:none;height:150px}

.group-card{
  background:#1c1f26;
  margin-top:10px;
  padding:12px;
  border-radius:16px;
  box-shadow:0 4px 12px rgba(0,0,0,.5);
}

.line{
  display:flex;
  align-items:center;
  margin:4px 0;
  padding:6px 10px;
  border-radius:10px;
}

.on{background:rgba(0,255,0,.15)}
.off{background:rgba(255,0,0,.15)}
.ind{width:28px;font-size:20px}

.center{
  text-align:center;
  padding:20px;
  opacity:.8;
}

.admin-panel{
  display:none;
  background:#111;
  border-radius:16px;
  padding:12px;
  margin-top:12px;
  box-shadow:0 0 15px rgba(255,196,0,.5);
}
</style>
</head>

<body>
<div class="container">

<div class="header">
  <h1>⚡ Львівська область</h1>
  <div class="update-time" id="updateTime">Останнє оновлення: —</div>
  <div class="admin-btn" onclick="openAdmin()">🔒</div>
</div>

<select id="day"></select>
<select id="group"></select>

<div id="content"></div>

<div class="admin-panel" id="adminPanel">
  <h3>🔧 Редагування графіка</h3>
  <p>Вводь <b>тільки періоди, коли світла нема</b><br>Формат: HH:MM-HH:MM</p>
  <textarea id="editor"></textarea>
  <button onclick="saveSchedule()">💾 Зберегти</button>
</div>

</div>

<script>
const ADMIN_PASSWORD="3709";

/* Дні */
const days=[
 ["mon","Понеділок"],
 ["tue","Вівторок"],
 ["wed","Середа"],
 ["thu","Четвер"],
 ["fri","Пʼятниця"],
 ["sat","Субота"],
 ["sun","Неділя"]
];

const daySel=document.getElementById("day");
days.forEach(d=>daySel.innerHTML+=`<option value="${d[0]}">${d[1]}</option>`);
daySel.value=["sun","mon","tue","wed","thu","fri","sat"][new Date().getDay()];

/* Групи */
const groups=[];
for(let g=1;g<=6;g++)["1","2"].forEach(s=>groups.push(`${g}.${s}`));
const groupSel=document.getElementById("group");
groups.forEach(g=>groupSel.innerHTML+=`<option>${g}</option>`);

/* Дані */
let data=JSON.parse(localStorage.getItem("schedules"))||{};
let lastUpdate=localStorage.getItem("lastUpdate");

/* Доп */
function toMin(t){let[a,b]=t.split(":");return a*60+ +b}

function normalize(off){
  let res=[],p=0;
  off.sort((a,b)=>toMin(a[0])-toMin(b[0]));
  off.forEach(o=>{
    let f=toMin(o[0]),t=toMin(o[1]);
    if(p<f) res.push([p,f,"on"]);
    res.push([f,t,"off"]);
    p=t;
  });
  if(p<1440) res.push([p,1440,"on"]);
  return res;
}

function formatTime(d){
  return d.toLocaleTimeString("uk-UA",{hour12:false});
}

/* Рендер */
function render(){
  const c=document.getElementById("content");
  c.innerHTML="";
  const d=daySel.value;
  const g=groupSel.value;

  if(lastUpdate){
    document.getElementById("updateTime").innerText=
      "Останнє оновлення: "+lastUpdate;
  }

  if(!data[d]||!data[d][g]){
    c.innerHTML=`<div class="center">⏳ Графік ще формується</div>`;
    return;
  }

  normalize(data[d][g]).forEach(s=>{
    c.innerHTML+=`
    <div class="group-card">
      <div class="line ${s[2]}">
        <div class="ind">${s[2]=="on"?"🟢":"⚫"}</div>
        ${String(Math.floor(s[0]/60)).padStart(2,"0")}:${String(s[0]%60).padStart(2,"0")}
        –
        ${String(Math.floor(s[1]/60)).padStart(2,"0")}:${String(s[1]%60).padStart(2,"0")}
      </div>
    </div>`;
  });
}

/* Адмін */
function openAdmin(){
  const pass=prompt("Пароль:");
  if(pass!==ADMIN_PASSWORD){
    alert("❌ Невірний пароль");
    return;
  }
  document.getElementById("adminPanel").style.display="block";
  loadEditor();
}

function loadEditor(){
  const d=daySel.value;
  const g=groupSel.value;
  const arr=data[d]?.[g]||[];
  document.getElementById("editor").value=
    arr.map(a=>`${a[0]}-${a[1]}`).join("\n");
}

function saveSchedule(){
  const d=daySel.value;
  const g=groupSel.value;
  const lines=document.getElementById("editor").value
    .trim().split("\n").filter(Boolean);

  if(!data[d]) data[d]={};
  data[d][g]=lines.map(l=>l.split("-"));

  const now=new Date();
  lastUpdate=formatTime(now);

  localStorage.setItem("schedules",JSON.stringify(data));
  localStorage.setItem("lastUpdate",lastUpdate);

  render();
}

/* Події */
daySel.onchange=()=>{render();loadEditor();}
groupSel.onchange=()=>{render();loadEditor();}
render();
</script>
</body>
</html>
