<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Графік світла — Львівська область</title>
<style>
body {
  margin:0;
  background: linear-gradient(135deg, #0f1115, #1c1f28);
  color:#fff;
  font-family:'Segoe UI', system-ui, sans-serif;
}
.container{
  max-width:480px;
  margin:auto;
  padding:10px;
}
.header{
  border-radius:16px;
  padding:16px;
  text-align:center;
  background: linear-gradient(90deg,#222,#333);
  box-shadow: 0 4px 15px rgba(0,0,0,0.5);
}
.header h1{
  margin:0;
  font-size:24px;
  color:#ffc400;
  text-shadow: 1px 1px 5px #000;
}
.meta{
  margin-top:8px;
  font-size:14px;
  display:flex;
  justify-content:space-between;
  flex-wrap:wrap;
}
select, button{
  width:100%;
  padding:12px;
  margin-top:8px;
  border-radius:12px;
  border:none;
  background:#2a2a2a;
  color:#fff;
  font-size:16px;
  transition:0.3s;
}
select:hover,button:hover{background:#3a3a3a; cursor:pointer;}
.statusCard{
  margin-top:14px;
}
.group-card{
  background:#1b1d24;
  border-radius:16px;
  padding:14px;
  margin:8px 0;
  transition:all 0.3s ease;
  box-shadow:0 4px 12px rgba(0,0,0,0.5);
  position:relative;
}
.group-card.current-group{
  border:2px solid #00ff00;
  background:#122011;
  box-shadow:0 0 15px #00ff00;
}
.group-name{
  font-weight:700;
  font-size:18px;
  margin-bottom:10px;
}
.status-line{
  display:flex;
  align-items:center;
  margin:4px 0;
  justify-content:space-between;
  font-size:16px;
  padding:6px 10px;
  border-radius:12px;
}
.status-line.on{
  background: rgba(0,255,0,0.15);
}
.status-line.off{
  background: rgba(255,0,0,0.15);
}
.status-line span.status-indicator{
  width:28px;
  text-align:center;
  margin-right:6px;
  font-size:22px;
}
.timer{
  font-size:15px;
  margin-top:6px;
  font-weight:600;
  text-align:center;
}
.progress-bar{
  height:8px;
  background:#555;
  border-radius:4px;
  margin-top:6px;
  position:relative;
  overflow:hidden;
}
.progress{
  height:100%;
  width:0%;
  background: linear-gradient(90deg,#00ff00,#00aa00);
  transition:width 1s linear;
}
.footer{
  margin-top:16px;
  text-align:center;
  font-size:14px;
  opacity:0.8;
}
.lastUpdate{
  margin-top:6px;
  font-size:14px;
  text-align:center;
  opacity:0.9;
}
.showAllBtn{
  margin-top:8px;
}
@media(max-width:480px){
  .group-name{font-size:16px;}
  .status-line span.status-indicator{font-size:20px;}
  .timer{font-size:14px;}
}
</style>
</head>
<body>
<div class="container">
<div class="header">
<h1>⚡ Львівська область</h1>
<div class="meta">
<div>👁 <span id="views"></span></div>
</div>
<div class="lastUpdate" id="lastUpdate"></div>
</div>

<select id="daySelect"></select>
<select id="group"></select>
<button class="showAllBtn" onclick="showAll()">📄 Переглянути усі групи</button>
<button onclick="pinGroup()">📌 Закріпити мою групу</button>

<div class="statusCard" id="statusCard"></div>
<div class="footer">Автооновлення • Демо</div>
</div>

<script>
// ==== Дні тижня ====
const days = ["mon","tue","wed","thu","fri","sat","sun"];
const dayNames = {mon:"Понеділок",tue:"Вівторок",wed:"Середа",thu:"Четвер",fri:"Пʼятниця",sat:"Субота",sun:"Неділя"};
let now = new Date();
let today = days[now.getDay()===0?6:now.getDay()-1];

// ==== Графіки (приклад) ====
const schedules = {
wed: {
"1.1":[["00:00","24:00","on"]],
"1.2":[["00:00","01:30","off"],["01:30","24:00","on"]],
"2.1":[["00:00","23:00","on"],["23:00","24:00","off"]],
"2.2":[["00:00","24:00","on"]],
"3.1":[["00:00","23:00","on"],["23:00","24:00","off"]],
"3.2":[["00:00","24:00","on"]],
"4.1":[["00:00","24:00","on"]],
"4.2":[["00:00","24:00","on"]],
"5.1":[["00:00","24:00","on"]],
"5.2":[["00:00","24:00","on"]],
"6.1":[["00:00","01:30","off"],["01:30","24:00","on"]],
"6.2":[["00:00","24:00","on"]]
},
thu:{
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
}
};

// ==== Селект днів і груп ====
const daySelect = document.getElementById("daySelect");
days.forEach(d=>daySelect.innerHTML+=`<option value="${d}">${dayNames[d]}</option>`);
daySelect.value = today;

const groupSel = document.getElementById("group");
for(let g=1;g<=6;g++){["1","2"].forEach(s=>groupSel.innerHTML+=`<option value="${g}.${s}">${g}.${s}</option>`);}
groupSel.value = localStorage.getItem("group") || "1.1";

// ==== Фейкові глядачі ====
const viewsEl = document.getElementById("views");
function updateViews(){viewsEl.textContent = Math.floor(975+Math.random()*700000);}

// ==== Час ====
function nowMin(){let d=new Date();return d.getHours()*60+d.getMinutes();}
function toMin(t){let[h,m]=t.split(":");return +h*60+ +m;}

// ==== Рендер ====
const statusCard = document.getElementById("statusCard");
let showAllGroups=false;

function render(){
  const dayKey = daySelect.value;
  const daySchedule = schedules[dayKey];
  if(!daySchedule){statusCard.innerHTML="⏳ Графік ще формується"; return;}

  let html="";
  const groups = showAllGroups?Object.keys(daySchedule):[groupSel.value];
  const n = nowMin();

  groups.forEach(group=>{
    const gSched = daySchedule[group];
    let current=null;

    gSched.forEach(s=>{
      const from=toMin(s[0]), to=toMin(s[1]);
      if(n>=from && n<to) current={from,to,state:s[2],start:s[0],end:s[1]};
    });

    const isCurrent = group===groupSel.value;
    html+=`<div class="group-card ${isCurrent?'current-group':''}">`;
    html+=`<div class="group-name">Група ${group}</div>`;

    gSched.forEach(s=>{
      const indicator = s[2]==="on"?"🟢":"⚫";
      html+=`<div class="status-line ${s[2]}"><span class="status-indicator">${indicator}</span>${s[0]}-${s[1]}</div>`;
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

  // ==== Останнє оновлення ====
  let lastUpdate = new Date("2026-01-21T22:18:00"); 
  let diff = Math.floor((new Date() - lastUpdate)/1000);
  let text="";
  if(diff<60) text="Останнє оновлення: щойно";
  else if(diff<3600) text=`Останнє оновлення: ${Math.floor(diff/60)} хв тому`;
  else text=`Останнє оновлення: ${Math.floor(diff/3600)} год ${Math.floor(diff/60%60)} хв тому`;
  document.getElementById("lastUpdate").textContent=text;
}

// ==== Закріпити групу ====
function pinGroup(){localStorage.setItem("group",groupSel.value); render();}
function showAll(){showAllGroups=true;render();}

// ==== Автооновлення і перегляди ====
setInterval(()=>{render();updateViews();},1000);

// ==== Ініціалізація ====
render();
updateViews();
</script>
</body>
</html>
