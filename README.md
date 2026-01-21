<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Графіки світла — Львівська область</title>

<style>
body{
  margin:0;
  background:#0e0e0e;
  color:#fff;
  font-family:system-ui,sans-serif;
}
.container{
  max-width:520px;
  margin:auto;
  padding:14px;
}
.header{
  border:2px solid #ffc400;
  border-radius:14px;
  padding:12px;
  margin-bottom:10px;
}
.header-top{
  display:flex;
  justify-content:space-between;
  align-items:center;
}
h1{
  font-size:20px;
  margin:0;
}
.views{
  font-size:14px;
  opacity:.85;
}
select,button{
  width:100%;
  padding:12px;
  margin-top:8px;
  border-radius:10px;
  border:none;
  background:#1f1f1f;
  color:#fff;
  font-size:15px;
}
button{cursor:pointer;}
.card{
  background:#1a1a1a;
  border-radius:14px;
  padding:14px;
  margin-top:10px;
}
.big{
  font-size:22px;
  font-weight:700;
  text-align:center;
}
.timer{
  text-align:center;
  margin-top:6px;
  opacity:.85;
}
.group-title{
  font-weight:600;
  margin-bottom:6px;
}
.schedule{
  font-size:14px;
  opacity:.9;
}
.hidden{display:none;}
.footer{
  margin-top:12px;
  font-size:13px;
  opacity:.6;
  text-align:center;
}
</style>
</head>

<body>
<div class="container">

<div class="header">
  <div class="header-top">
    <h1>⚡ Львівська область</h1>
    <div class="views">👁 <span id="views"></span></div>
  </div>
  <div id="lastUpdate"></div>
</div>

<select id="day"></select>
<select id="group"></select>

<button onclick="pinGroup()">📌 Закріпити мою групу</button>
<button onclick="toggleAll()">📊 Показати всі групи</button>

<div id="mainStatus" class="card"></div>
<div id="allGroups" class="card hidden"></div>

<div class="footer">
  Автооновлення • Демонстрація
</div>

</div>

<script>
// ===== ДАНІ =====
const updatedAt = new Date("2026-01-21T19:50:00");

const schedules = {
  wed:{
    "1.1":[],
    "1.2":[["00:00","01:30"]],
    "2.1":[["22:00","24:00"]],
    "2.2":[],
    "3.1":[["22:00","24:00"]],
    "3.2":[],
    "4.1":[],
    "4.2":[],
    "5.1":[],
    "5.2":[],
    "6.1":[["00:00","01:30"]],
    "6.2":[]
  },
  thu:{
    "1.1":[["00:00","03:00"],["13:30","17:00"]],
    "1.2":[["06:30","10:00"],["13:30","17:00"]],
    "2.1":[["08:00","13:30"]],
    "2.2":[["10:00","13:30"],["17:00","22:00"]],
    "3.1":[["00:00","03:00"],["17:00","22:00"]],
    "3.2":[["10:00","13:30"],["17:00","22:00"]],
    "4.1":[["06:30","10:00"],["13:30","17:00"]],
    "4.2":[["03:00","06:30"],["20:30","24:00"]],
    "5.1":[["08:00","13:30"],["17:00","20:30"]],
    "5.2":[["13:30","17:00"],["20:30","24:00"]],
    "6.1":[["03:00","06:30"],["13:30","17:00"]],
    "6.2":[["08:00","13:30"],["17:00","20:30"]]
  }
};

// ===== ІНІЦІАЛІЗАЦІЯ =====
const days = ["mon","tue","wed","thu","fri","sat","sun"];
const names = {
  mon:"Понеділок", tue:"Вівторок", wed:"Середа",
  thu:"Четвер", fri:"Пʼятниця", sat:"Субота", sun:"Неділя"
};

days.forEach(d=>{
  day.innerHTML+=`<option value="${d}">${names[d]}</option>`;
});
["1.1","1.2","2.1","2.2","3.1","3.2","4.1","4.2","5.1","5.2","6.1","6.2"]
.forEach(g=>group.innerHTML+=`<option>${g}</option>`);

day.value = days[new Date().getDay()-1] || "mon";
group.value = localStorage.getItem("group") || "1.1";

// ===== ФУНКЦІЇ =====
function min(t){let[h,m]=t.split(":");return +h*60+ +m;}
function now(){let d=new Date();return d.getHours()*60+d.getMinutes();}

function statusFor(d,g){
  if(!schedules[d]) return null;
  let offs=schedules[d][g]||[];
  let n=now();
  for(let o of offs){
    if(n>=min(o[0]) && n<min(o[1])) return false;
  }
  return true;
}

function render(){
  let d=day.value, g=group.value;
  let box=mainStatus;

  if(!schedules[d]){
    box.innerHTML="<div class='big'>⏳ Графік ще формується</div>";
    return;
  }

  let on=statusFor(d,g);
  box.innerHTML=`
    <div class="group-title">Група ${g}</div>
    <div class="big">${on?"🟢 ЗАРАЗ Є СВІТЛО":"⚫ ЗАРАЗ НЕМАЄ СВІТЛА"}</div>
  `;
}

function toggleAll(){
  let box=allGroups;
  box.classList.toggle("hidden");
  box.innerHTML="";
  let d=day.value;
  if(!schedules[d]){box.innerHTML="⏳ Немає графіка";return;}
  for(let g in schedules[d]){
    let on=statusFor(d,g);
    box.innerHTML+=`
      <div class="schedule">
        <b>${g}</b> — ${on?"🟢 є":"⚫ нема"}
      </div>`;
  }
}

function pinGroup(){
  localStorage.setItem("group",group.value);
}

// ===== ОНОВЛЕННЯ =====
function updateLast(){
  let diff=Math.floor((Date.now()-updatedAt)/60000);
  let t="щойно";
  if(diff>=1) t=diff+" хв тому";
  if(diff>=60){
    let h=Math.floor(diff/60);
    let m=diff%60;
    t=h+" год "+m+" хв тому";
  }
  lastUpdate.textContent="Останнє оновлення: "+t;
}

views.textContent=Math.floor(975+Math.random()*699000);
render();updateLast();
setInterval(()=>{render();updateLast();},60000);
</script>
</body>
</html>
