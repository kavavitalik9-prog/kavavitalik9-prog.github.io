<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Графік світла — Львівська область</title>
<style>
body {margin:0;background:#0e0e0e;color:#fff;font-family:system-ui,sans-serif;}
.container{max-width:480px;margin:auto;padding:12px;}
.header{border:2px solid #ffc400;border-radius:16px;padding:12px;text-align:center;margin-bottom:10px;background:#1a1a1a;}
.header h1{margin:0;font-size:22px;}
.meta{margin-top:6px;font-size:14px;display:flex;justify-content:space-between;}
select,button{width:100%;padding:12px;margin-top:8px;border-radius:10px;border:none;background:#1f1f1f;color:#fff;font-size:16px;}
button{cursor:pointer;}
.statusCard{margin-top:12px;}
.group-card{background:#1a1a1a;border-radius:14px;padding:12px;margin:6px 0;transition:all 0.3s ease;position:relative;}
.group-name{font-weight:700;font-size:18px;margin-bottom:8px;}
.status-line{display:flex;align-items:center;margin:4px 0;}
.status-line span.status-indicator{width:28px;text-align:center;margin-right:6px;font-size:28px;}
.timer{font-size:16px;margin-top:4px;}
.current-group{border:2px solid #00ff00;background:#262626;}
.progress-bar{height:6px;background:#555;border-radius:3px;margin-top:4px;position:relative;overflow:hidden;}
.progress{height:100%;background:#00ff00;width:0%;transition:width 1s linear;}
.footer{margin-top:14px;text-align:center;font-size:14px;opacity:0.7;}
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

<div class="statusCard" id="statusCard"></div>
<div class="footer">Автооновлення • Демо</div>
</div>

<script>
// ==== Оновлення та перегляди ====
const updatedAt = new Date("2026-01-21T22:18:00");
const viewsEl = document.getElementById("views");
const lastUpdate = document.getElementById("lastUpdate");

// ==== Дні тижня ====
const days = ["mon","tue","wed","thu","fri","sat","sun"];
const dayNames = {mon:"Понеділок",tue:"Вівторок",wed:"Середа",thu:"Четвер",fri:"Пʼятниця",sat:"Субота",sun:"Неділя"};

// ==== Словник графіків ====
const schedules = {mon:{},tue:{},wed:{},thu:{},fri:{},sat:{},sun:{}};

// ==== Четвер ====
schedules.thu = {
"1.1":[["00:00","03:00","off"],["03:00","13:30","on"],["13:30","17:00","off"],["17:00","24:00","on"]],
"1.2":[["00:00","06:30","off"],["06:30","10:00","on"],["10:00","13:30","off"],["13:30","17:00","on"],["17:00","24:00","on"]],
"2.1":[["00:00","08:00","off"],["08:00","13:30","on"],["13:30","17:00","off"],["17:00","22:00","on"],["22:00","24:00","on"]],
"2.2":[["00:00","10:00","on"],["10:00","13:30","off"],["13:30","17:00","on"],["17:00","22:00","off"],["22:00","24:00","on"]],
"3.1":[["00:00","03:00","off"],["03:00","17:00","on"],["17:00","22:00","off"],["22:00","24:00","on"]],
"3.2":[["00:00","10:00","on"],["10:00","13:30","off"],["13:30","17:00","on"],["17:00","22:00","off"],["22:00","24:00","on"]],
"4.1":[["00:00","06:30","off"],["06:30","10:00","on"],["10:00","13:30","off"],["13:30","17:00","on"],["17:00","24:00","on"]],
"4.2":[["00:00","03:00","off"],["03:00","06:30","on"],["06:30","20:30","off"],["20:30","24:00","on"]],
"5.1":[["00:00","08:00","off"],["08:00","13:30","on"],["13:30","17:00","off"],["17:00","20:30","on"],["20:30","24:00","on"]],
"5.2":[["00:00","13:30","off"],["13:30","17:00","on"],["17:00","20:30","off"],["20:30","24:00","on"]],
"6.1":[["00:00","03:00","off"],["03:00","06:30","on"],["06:30","13:30","off"],["13:30","17:00","on"],["17:00","20:30","off"],["20:30","24:00","on"]],
"6.2":[["00:00","08:00","off"],["08:00","13:30","on"],["13:30","17:00","off"],["17:00","20:30","on"],["20:30","24:00","on"]]
};

// Інші дні графік формується
["mon","tue","wed","fri","sat","sun"].forEach(d=>{
  for(let g=1;g<=6;g++){
    ["1","2"].forEach(s=>{
      schedules[d][`${g}.${s}`]=null;
    });
  }
});

// ==== Елементи ====
const daySel = document.getElementById("day");
days.forEach(d=>daySel.innerHTML+=`<option value="${d}">${dayNames[d]}</option>`);

const groupSel = document.getElementById("group");
for(let g=1;g<=6;g++){["1","2"].forEach(s=>groupSel.innerHTML+=`<option value="${g}.${s}">${g}.${s}</option>`);}
groupSel.value = localStorage.getItem("group") || "1.1";

const statusCard = document.getElementById("statusCard");

// ==== Функції ====
function toMin(t){let[h,m]=t.split(":");return +h*60+ +m;}
function nowMin(){let d=new Date();return d.getHours()*60+d.getMinutes();}
function pad(n){return n<10?"0"+n:n;}

function render(){
  const day = daySel.value;
  const daySchedule = schedules[day];
  let html="";

  Object.keys(daySchedule).forEach(group=>{
    const gSched = daySchedule[group];
    let current=null;
    const n = nowMin();

    if(gSched){
      gSched.forEach(s=>{
        const from=toMin(s[0]), to=toMin(s[1]);
        if(n>=from && n<to) current={from,to,state:s[2],start:s[0],end:s[1]};
      });
    }

    const isCurrent = group===groupSel.value;
    html+=`<div class="group-card ${isCurrent?'current-group':''}">`;
    html+=`<div class="group-name">Група ${group}</div>`;

    if(!gSched){
      html+="⏳ Графік ще формується</div>";
      return;
    }

    gSched.forEach(s=>{
      const indicator = s[2]==="on"?"🟢":"⚫";
      html+=`<div class="status-line"><span class="status-indicator">${indicator}</span>${s[0]}-${s[1]}</div>`;
    });

    if(current){
      let left=current.to-n;
      let h=Math.floor(left/60), m=left%60;
      let percent = Math.floor((n-current.from)/(current.to-current.from)*100);
      if(percent<0) percent=0; if(percent>100) percent=100;
      html+=`<div class="timer">⏱ До ${current.state==="on"?"вимкнення":"увімкнення"}: ${h} год ${m} хв</div>`;
      html+=`<div class="timer">${current.state==="on"?"🟢 ЗАРАЗ Є СВІТЛО":"⚫ ЗАРАЗ НЕМАЄ СВІТЛА"}</div>`;
      html+=`<div class="progress-bar"><div class="progress" style="width:${percent}%"></div></div>`;
    }

    html+="</div>";
  });

  statusCard.innerHTML = html;
}

function pinGroup(){localStorage.setItem("group",groupSel.value); render();}

function updateMeta(){
  let diff=Math.floor((Date.now()-updatedAt)/60000);
  let t="щойно";
  if(diff>=1) t=diff+" хв тому";
  if(diff>=60){let h=Math.floor(diff/60), m=diff%60; t=h+" год "+m+" хв тому";}
  lastUpdate.textContent="Останнє оновлення: "+t;
  viewsEl.textContent=Math.floor(975+Math.random()*700000);
}

// ==== Ініціалізація ====
render();
updateMeta();
daySel.onchange=render;
groupSel.onchange=render;
setInterval(()=>{render();updateMeta();},1000);

// Авто-перемикання дня опівночі
setInterval(()=>{
  const d = new Date();
  if(d.getHours()===0 && d.getMinutes()===0 && d.getSeconds()===0){
    const nd = days[d.getDay()-1>=0?d.getDay()-1:0];
    daySel.value=nd;
    render();
  }
},1000);
</script>
</body>
</html>
