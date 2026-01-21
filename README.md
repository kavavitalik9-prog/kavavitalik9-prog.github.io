<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>⚡ Львівська область — графіки світла</title>

<style>
body{
  margin:0;
  font-family:Arial, sans-serif;
  background:#0e0e0e;
  color:#fff;
}

.container{
  max-width:480px;
  margin:auto;
  padding:10px;
}

.header{
  border:1px solid #333;
  border-radius:12px;
  padding:10px;
  margin-bottom:10px;
}

.big-status{
  font-size:20px;
  font-weight:bold;
  margin:6px 0;
}

.timer{
  font-size:14px;
  opacity:.9;
}

select, button{
  width:100%;
  padding:8px;
  margin-top:6px;
  border-radius:8px;
  border:none;
  background:#1c1c1c;
  color:#fff;
}

.group-box{
  margin-top:10px;
  border:1px solid #333;
  border-radius:12px;
  padding:10px;
}

.all-groups{
  margin-top:10px;
}

.group{
  border-bottom:1px solid #222;
  padding:6px 0;
  font-size:14px;
}

.reactions{
  display:flex;
  gap:10px;
  margin-top:10px;
}

.likes{
  border:1px solid #333;
  padding:6px 10px;
  border-radius:8px;
  cursor:pointer;
  user-select:none;
}

.dislikes{
  border:1px solid #555;
  padding:6px 10px;
  border-radius:8px;
  opacity:.5;
  cursor:not-allowed;
}

.footer{
  margin-top:10px;
  font-size:12px;
  opacity:.7;
  text-align:center;
}
</style>
</head>

<body>
<div class="container">

<div class="header">
  <div>⚡ <b>Львівська область</b></div>
  <div>👀 Переглядають: <span id="views">...</span></div>

  <div class="big-status" id="status">...</div>
  <div class="timer" id="timer">...</div>

  <div class="timer" id="lastUpdate"></div>
</div>

<select id="groupSelect"></select>

<button onclick="toggleAll()">Показати всі групи</button>

<div class="group-box" id="myGroup"></div>

<div class="all-groups" id="allGroups" style="display:none"></div>

<div class="reactions">
  <div class="likes" id="likeBtn">❤️ Лайки: <span id="likeCount">0</span></div>
  <div class="dislikes">👎 Дизлайки: <span>0</span></div>
</div>

<div class="footer">Дані тестові • Не офіційно</div>

</div>

<script>
// ================= ДАНІ =================
const lastUpdateTime = new Date("2026-01-21T19:50:00");

const schedules = {
  "Середа": {
    "1.1": [],
    "1.2": [["00:00","01:30"]],
    "2.1": [["22:00","24:00"]],
    "2.2": [],
    "3.1": [["22:00","24:00"]],
    "3.2": [],
    "4.1": [],
    "4.2": [],
    "5.1": [],
    "5.2": [],
    "6.1": [["00:00","01:30"]],
    "6.2": []
  },
  "Четвер": {
    "1.1": [["00:00","03:00"],["13:30","17:00"]],
    "1.2": [["06:30","10:00"],["13:30","17:00"]],
    "2.1": [["08:00","13:30"]],
    "2.2": [["10:00","13:30"],["17:00","22:00"]],
    "3.1": [["00:00","03:00"],["17:00","22:00"]],
    "3.2": [["10:00","13:30"],["17:00","22:00"]],
    "4.1": [["06:30","10:00"],["13:30","17:00"]],
    "4.2": [["03:00","06:30"],["20:30","24:00"]],
    "5.1": [["08:00","13:30"],["17:00","20:30"]],
    "5.2": [["13:30","17:00"],["20:30","24:00"]],
    "6.1": [["03:00","06:30"],["13:30","17:00"]],
    "6.2": [["08:00","13:30"],["17:00","20:30"]]
  }
};

const groups = Object.keys(schedules["Середа"]);

// ================= ІНІЦ =================
const select = document.getElementById("groupSelect");
groups.forEach(g=>{
  const o=document.createElement("option");
  o.value=g; o.textContent="Група "+g;
  select.appendChild(o);
});

select.value = localStorage.getItem("group") || "1.1";
select.onchange=()=>{
  localStorage.setItem("group",select.value);
  render();
};

// ================= ФУНКЦІЇ =================
function nowMin(){
  const d=new Date();
  return d.getHours()*60+d.getMinutes();
}

function inOut(list){
  for(const [s,e] of list){
    const sm=parseInt(s.split(":")[0])*60+parseInt(s.split(":")[1]);
    const em=(e==="24:00"?1440:parseInt(e.split(":")[0])*60+parseInt(e.split(":")[1]));
    if(nowMin()>=sm && nowMin()<em) return false;
  }
  return true;
}

function render(){
  const day=(new Date().getDay()==4)?"Четвер":"Середа";
  const group=select.value;
  const offs=schedules[day][group];
  const hasLight=inOut(offs);

  document.getElementById("status").textContent =
    hasLight ? "🟢 ЗАРАЗ Є СВІТЛО" : "⚫ ЗАРАЗ НЕМАЄ СВІТЛА";

  document.getElementById("myGroup").innerHTML =
    `<b>Твоя група ${group}</b><br>${hasLight?"Світло є":"Світла нема"}`;

  document.getElementById("allGroups").innerHTML =
    groups.map(g=>{
      const l=inOut(schedules[day][g]);
      return `<div class="group">${g}: ${l?"🟢 є":"⚫ нема"}</div>`;
    }).join("");
}

function toggleAll(){
  const el=document.getElementById("allGroups");
  el.style.display=el.style.display==="none"?"block":"none";
}

// ================= ЛАЙКИ =================
let likes=0;
document.getElementById("likeBtn").onclick=()=>{
  likes++;
  document.getElementById("likeCount").textContent=likes;
};

// ================= ПЕРЕГЛЯДИ =================
document.getElementById("views").textContent =
  Math.floor(975+Math.random()*699000);

// ================= ОСТАННЄ ОНОВЛЕННЯ =================
function updateLast(){
  const diff=Math.floor((Date.now()-lastUpdateTime)/60000);
  document.getElementById("lastUpdate").textContent =
    "Останнє оновлення: "+(diff<60?diff+" хв тому":Math.floor(diff/60)+" год тому");
}

setInterval(()=>{render();updateLast();},1000);
render();updateLast();
</script>

</body>
</html>
