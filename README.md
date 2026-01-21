<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>⚡Графік світла — Львівська область</title>
<style>
body{margin:0;font-family:Arial,sans-serif;background:#0f0f0f;color:#fff;}
.container{max-width:480px;margin:auto;padding:10px;}
.header{display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;}
.status{font-size:20px;font-weight:bold;margin:8px 0;}
.timer{font-size:14px;opacity:.8;margin-bottom:10px;}
select,button{width:100%;padding:8px;border-radius:8px;border:none;margin:6px 0;font-size:15px;background:#1c1c1c;color:#fff;}
.box{background:#151515;border-radius:12px;padding:10px;margin-top:10px;}
.group{font-size:14px;border-bottom:1px solid #222;padding:6px 0;}
.reactions{display:flex;gap:10px;margin-top:10px;}
.likes,.dislikes{border:1px solid #333;padding:6px 10px;border-radius:8px;font-size:14px;}
.dislikes{opacity:.5;}
.footer{margin-top:12px;font-size:12px;opacity:.6;text-align:center;}
</style>
</head>
<body>
<div class="container">

<div class="header">
  <h2>⚡Львівська область</h2>
  <div>👁 <span id="views">...</span></div>
</div>

<div class="status" id="status">...</div>
<div class="timer" id="timer">...</div>

<select id="daySelect"></select>
<select id="groupSelect"></select>

<button onclick="pinGroup()">📌 Закріпити групу</button>
<button onclick="toggleAll()">📊 Показати всі групи</button>

<div class="box" id="myGroup"></div>
<div class="box" id="allGroups" style="display:none"></div>

<div class="reactions">
  <div class="likes">❤️ Лайки: <span id="likes">0</span></div>
  <div class="dislikes">👎 Дизлайки: <span>0</span></div>
</div>

<div class="footer">Графік формується • Автооновлення</div>

</div>

<script>
// ================== Fake Views ==================
function rand(min,max){return Math.floor(Math.random()*(max-min)+min);}
document.getElementById("views").textContent = rand(975,700000).toLocaleString("uk-UA");

// ================== Liked Counter ==================
const startLikes = new Date("2026-01-21T19:40:00");
function updateLikes(){
  const diffSec = Math.floor((Date.now()-startLikes)/1000);
  document.getElementById("likes").textContent = (diffSec>0?diffSec:0).toLocaleString("uk-UA");
}
setInterval(updateLikes,1000);
updateLikes();

// ================== Days & Groups ==================
const days=["Понеділок","Вівторок","Середа","Четвер","Пʼятниця","Субота","Неділя"];
const daySelect = document.getElementById("daySelect");
days.forEach(d=>{
  const o=document.createElement("option"); o.value=d; o.textContent=d;
  daySelect.appendChild(o);
});

const groups=["1.1","1.2","2.1","2.2","3.1","3.2","4.1","4.2","5.1","5.2","6.1","6.2"];
const groupSelect = document.getElementById("groupSelect");
groups.forEach(g=>{
  const o=document.createElement("option"); o.value=g; o.textContent=g;
  groupSelect.appendChild(o);
});
if(localStorage.group) groupSelect.value = localStorage.group;

// ================== Schedules ==================
// Only “times when NO LIGHT”
const schedules = {
  "Середа":{
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
  "Четвер":{
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

// ================== Helpers ==================
function inNoLight(list){
  const now=new Date();
  const cur = now.getHours()*60+now.getMinutes();
  for(const [s,e] of list){
    const sm=parseInt(s.split(":")[0])*60 + parseInt(s.split(":")[1]);
    const em = (e==="24:00"?1440:(parseInt(e.split(":")[0])*60 + parseInt(e.split(":")[1])));
    if(cur>=sm && cur<em) return true;
  }
  return false;
}

function minutesToNext(list){
  const now=new Date();
  const cur = now.getHours()*60 + now.getMinutes();
  let best=null;
  // times when light changes
  const points = [];

  // all off intervals
  list.forEach(([s,e])=>{
    const sm=parseInt(s.split(":")[0])*60+parseInt(s.split(":")[1]);
    const em=(e==="24:00"?1440:(parseInt(e.split(":")[0])*60+parseInt(e.split(":")[1]));
    points.push(sm,em);
  });

  points.sort((a,b)=>a-b);
  for(const p of points){
    if(p>cur){ best=p; break;}
  }
  if(best===null) best=1440; // next day
  return best-cur;
}

// ================== Render ==================
function render(){
  const day = daySelect.value;
  const group = groupSelect.value;
  const noLight = (schedules[day] && schedules[day][group])?inNoLight(schedules[day][group]):false;
  const lightNow = !noLight;

  document.getElementById("status").textContent = lightNow?"🟢 ЗАРАЗ Є СВІТЛО":"⚫ ЗАРАЗ НЕМАЄ СВІТЛА";

  const minsNext = (schedules[day] && schedules[day][group])?minutesToNext(schedules[day][group]):0;
  const hrs = Math.floor(minsNext/60), mins = minsNext%60;
  document.getElementById("timer").textContent =
    "⏱ До зміни: "+(hrs>0?hrs+"г ":"")+mins+"хв";

  // Show own group
  document.getElementById("myGroup").innerHTML =
    `<b>${day} — група ${group}</b><br>`+
    (lightNow?"Світло є":"Світла нема");

  // All groups
  if(schedules[day]){
    document.getElementById("allGroups").innerHTML =
      Object.entries(schedules[day]).map(([g,list])=>{
        const noL = inNoLight(list);
        return `<div class="group">${g}: ${!noL?"🟢 є":"⚫ нема"}</div>`;
      }).join("");
  } else {
    document.getElementById("allGroups").innerHTML="Графік ще формується";
  }
}

function pinGroup(){ localStorage.group=groupSelect.value; }
function toggleAll(){
  const el=document.getElementById("allGroups");
  el.style.display = el.style.display==="none"?"block":"none";
}

daySelect.onchange=render;
groupSelect.onchange=render;

// init
days.forEach(d=>{}); // fill days
daySelect.value = days[new Date().getDay()-1]||"Середа";
render();

// auto update
setInterval(render,30000);

</script>
</body>
</html>
