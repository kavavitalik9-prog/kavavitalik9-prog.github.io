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
textarea{resize:none;height:140px}

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

.admin-panel{
  display:none;
  background:#111;
  border-radius:16px;
  padding:12px;
  margin-top:12px;
  box-shadow:0 0 15px rgba(255,196,0,.5);
}

.center{
  text-align:center;
  padding:20px;
  opacity:.8;
}
</style>
</head>

<body>
<div class="container">

<div class="header">
  <h1>⚡ Львівська область</h1>
  <div class="admin-btn" onclick="openAdmin()">🔒</div>
</div>

<select id="day"></select>
<select id="group"></select>

<div id="content"></div>

<div class="admin-panel" id="adminPanel">
  <h3>🔧 Редагування графіка (четвер)</h3>
  <p>Формат:<br><b>HH:MM-HH:MM off</b></p>
  <textarea id="editor"></textarea>
  <button onclick="saveSchedule()">💾 Зберегти</button>
</div>

</div>

<script>
/* ===== НАЛАШТУВАННЯ ===== */
const ADMIN_PASSWORD="3709";
const EDITABLE_DAY="thu";

/* ===== ДНІ ===== */
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

const today=["sun","mon","tue","wed","thu","fri","sat"][new Date().getDay()];
daySel.value=today;

/* ===== ГРУПИ ===== */
const groups=[];
for(let g=1;g<=6;g++)["1","2"].forEach(s=>groups.push(`${g}.${s}`));
const groupSel=document.getElementById("group");
groups.forEach(g=>groupSel.innerHTML+=`<option>${g}</option>`);

/* ===== ЧЕТВЕР (OFF-ІНТЕРВАЛИ) ===== */
const defaultThu={
"1.1":[["01:00","03:00"],["13:30","17:00"]],
"1.2":[["06:30","10:00"],["13:30","17:00"]],
"2.1":[["01:30","03:00"],["10:00","13:30"]],
"2.2":[["10:00","13:30"],["17:00","22:00"]],
"3.1":[["01:00","03:00"],["17:00","22:00"]],
"3.2":[["10:00","13:30"],["17:00","22:00"]],
"4.1":[["06:30","10:00"],["13:30","17:00"]],
"4.2":[["03:00","04:00"],["06:00","06:30"],["20:30","24:00"]],
"5.1":[["10:00","13:30"],["17:00","20:30"]],
"5.2":[["01:30","03:00"],["13:30","17:00"],["20:30","24:00"]],
"6.1":[["03:00","04:00"],["06:00","06:30"],["13:30","17:00"]],
"6.2":[["10:00","13:30"],["17:00","20:30"]]
};

let thu=JSON.parse(localStorage.getItem("thu"))||defaultThu;

/* ===== ДОП ФУНКЦІЇ ===== */
function toMin(t){let[a,b]=t.split(":");return a*60+ +b}

function normalize(off){
  let res=[],p=0;
  off.forEach(o=>{
    let f=toMin(o[0]),t=toMin(o[1]);
    if(p<f) res.push([p,f,"on"]);
    res.push([f,t,"off"]);
    p=t;
  });
  if(p<1440) res.push([p,1440,"on"]);
  return res;
}

/* ===== РЕНДЕР ===== */
function render(){
  const c=document.getElementById("content");
  c.innerHTML="";

  if(daySel.value!==EDITABLE_DAY){
    c.innerHTML=`<div class="center">⏳ Графік ще формується</div>`;
    document.getElementById("adminPanel").style.display="none";
    return;
  }

  const g=groupSel.value;
  normalize(thu[g]).forEach(s=>{
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

/* ===== АДМІН ===== */
function openAdmin(){
  if(daySel.value!==EDITABLE_DAY){
    alert("Редагування доступне лише для четверга");
    return;
  }
  const pass=prompt("Пароль:");
  if(pass===ADMIN_PASSWORD){
    document.getElementById("adminPanel").style.display="block";
    document.getElementById("editor").value=
      thu[groupSel.value].map(s=>`${s[0]}-${s[1]} off`).join("\n");
  }else alert("❌ Невірний пароль");
}

function saveSchedule(){
  const lines=document.getElementById("editor").value.trim().split("\n");
  thu[groupSel.value]=lines.map(l=>{
    const [t]=l.split(" ");
    return t.split("-");
  });
  localStorage.setItem("thu",JSON.stringify(thu));
  render();
}

daySel.onchange=render;
groupSel.onchange=render;
render();
</script>
</body>
</html>
