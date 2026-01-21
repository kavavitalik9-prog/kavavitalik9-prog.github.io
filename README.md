<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Графік світла — Львівська область</title>
<style>
body{margin:0;background:#0e0e0e;color:#fff;font-family:system-ui,sans-serif;}
.container{max-width:520px;margin:auto;padding:14px;}
.header{border:2px solid #ffc400;border-radius:14px;padding:12px;margin-bottom:10px;}
.header h1{margin:0;font-size:20px;}
.header .meta{margin-top:6px;font-size:14px;opacity:.9;display:flex;justify-content:space-between;}
select,button{width:100%;padding:12px;margin-top:10px;border-radius:10px;border:none;background:#1f1f1f;color:#fff;font-size:15px;}
button{cursor:pointer;}
.card{background:#1a1a1a;border-radius:14px;padding:14px;margin-top:12px;}
.big{text-align:center;font-size:22px;font-weight:700;}
.schedule-line{margin:6px 0;font-size:15px;}
.timer{margin-top:8px;text-align:center;opacity:.85;}
.footer{margin-top:14px;text-align:center;font-size:13px;opacity:.6;}
</style>
</head>
<body>
<div class="container">

<div class="header">
  <h1>⚡ Львівська область</h1>
  <div class="meta">
    <div>👁 <span id="views"></span></div>
    <div id="lastUpdate"></div>
  </div>
</div>

<select id="day"></select>
<select id="group"></select>
<button onclick="pinGroup()">📌 Закріпити мою групу</button>
<div class="card" id="statusCard"></div>
<div class="footer">Автооновлення • Демо</div>

</div>

<script>
// ========== НАЛАШТУВАННЯ ==========
const updatedAt = new Date("2026-01-21T19:50:00");

// ГРАФІКИ (приклад)
const schedules = {
  wed:{
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
  },
  thu:{
    "1.1":[["00:00","03:00","off"],["03:00","13:30","on"],["13:30","17:00","off"],["17:00","24:00","on"]],
    "1.2":[["00:00","06:30","on"],["06:30","10:00","off"],["10:00","13:30","on"],["13:30","17:00","off"],["17:00","24:00","on"]],
    "2.1":[["00:00","08:00","on"],["08:00","13:30","off"],["13:30","24:00","on"]],
    "2.2":[["00:00","10:00","on"],["10:00","13:30","off"],["13:30","17:00","on"],["17:00","22:00","off"],["22:00","24:00","on"]],
    "3.1":[["00:00","03:00","off"],["03:00","17:00","on"],["17:00","22:00","off"],["22:00","24:00","on"]],
    "3.2":[["00:00","10:00","on"],["10:00","13:30","off"],["13:30","17:00","on"],["17:00","22:00","off"],["22:00","24:00","on"]],
    "4.1":[["00:00","06:30","on"],["06:30","10:00","off"],["10:00","13:30","on"],["13:30","17:00","off"],["17:00","24:00","on"]],
    "4.2":[["00:00","03:00","on"],["03:00","06:30","off"],["06:30","20:30","on"],["20:30","24:00","off"]],
    "5.1":[["00:00","08:00","on"],["08:00","13:30","off"],["13:30","17:00","on"],["17:00","20:30","off"],["20:30","24:00","on"]],
    "5.2":[["00:00","13:30","on"],["13:30","17:00","off"],["17:00","20:30","on"],["20:30","24:00","off"]],
    "6.1":[["00:00","03:00","on"],["03:00","06:30","off"],["06:30","13:30","on"],["13:30","17:00","off"],["17:00","24:00","on"]],
    "6.2":[["00:00","08:00","on"],["08:00","13:30","off"],["13:30","17:00","on"],["17:00","20:30","off"],["20:30","24:00","on"]]
  }
};

// ===== ІНІЦІАЛІЗАЦІЯ =====
const days = ["mon","tue","wed","thu","fri","sat","sun"];
const dayNames = {mon:"Понеділок",tue:"Вівторок",wed:"Середа",thu:"Четвер",fri:"Пʼятниця",sat:"Субота",sun:"Неділя"};

const daySel = document.getElementById("day");
days.forEach(d=>daySel.innerHTML+=`<option value="${d}">${dayNames[d]}</option>`);
daySel.value = days[new Date().getDay()-1] || "mon";

const groupSel = document.getElementById("group");
for(let g=1;g<=6;g++){
  ["1","2"].forEach(s=>groupSel.innerHTML+=`<option value="${g}.${s}">${g}.${s}</option>`);
}
groupSel.value = localStorage.getItem("group") || "1.1";

const viewsEl = document.getElementById("views");
const lastUpdate = document.getElementById("lastUpdate");
const statusCard = document.getElementById("statusCard");

// ===== ФУНКЦІЇ =====
function toMin(t){let[h,m]=t.split(":");return +h*60 + +m;}
function nowMin(){let d=new Date();return d.getHours()*60 + d.getMinutes();}

function render(){
  const day = daySel.value;
  const group = groupSel.value;
  const daySchedule = schedules[day] ? schedules[day][group] : null;

  let html = `<div class="big">Група ${group}</div>`;
  if(!daySchedule){
    html += `<div class="schedule-line">⏳ Графік ще формується</div>`;
    statusCard.innerHTML = html;
    return;
  }

  const n = nowMin();
  let current = null;
  let next = null;

  daySchedule.forEach(s=>{
    const from = toMin(s[0]), to = toMin(s[1]);
    const isOn = s[2]==="on";
    html += `<div class="schedule-line">${isOn?"🟢":"⚫"} ${s[0]}-${s[1]} — ${isOn?"є світло":"нема світла"}</div>`;
    if(n>=from && n<to) current = {from,to,isOn};
    if(n<from && !next) next = {from,to,isOn};
  });

  if(current){
    let left = current.to - n;
    let h=Math.floor(left/60), m=left%60;
    html += `<div class="timer">⏱ До ${current.isOn?"вимкнення":"увімкнення"}: ${h} год ${m} хв</div>`;
  }

  statusCard.innerHTML = html;
}

function pinGroup(){localStorage.setItem("group",groupSel.value);}

function updateMeta(){
  let diff=Math.floor((Date.now()-updatedAt)/60000);
  let t="щойно";
  if(diff>=1) t=diff+" хв тому";
  if(diff>=60){let h=Math.floor(diff/60), m=diff%60; t=h+" год "+m+" хв тому";}
  lastUpdate.textContent="Оновлено: "+t;
  viewsEl.textContent=Math.floor(975+Math.random()*700000);
}

// ===== ОНОВЛЕННЯ =====
render();
updateMeta();
daySel.onchange=render;
groupSel.onchange=render;
setInterval(()=>{render();updateMeta();},60000);
</script>
</body>
</html>
