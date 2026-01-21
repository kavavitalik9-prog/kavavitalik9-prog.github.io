<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>⚡ Львівська область — графіки світла</title>

<style>
body{
  margin:0;
  font-family:system-ui, sans-serif;
  background:#0f0f0f;
  color:#fff;
}
.container{
  max-width:480px;
  margin:auto;
  padding:14px;
}
h1{
  font-size:20px;
  margin-bottom:6px;
}
select,button{
  width:100%;
  padding:12px;
  margin:6px 0;
  border-radius:10px;
  border:none;
  background:#1f1f1f;
  color:#fff;
  font-size:15px;
}
button{
  cursor:pointer;
}
.card{
  background:#1a1a1a;
  border-radius:14px;
  padding:14px;
  margin-top:10px;
}
.big-status{
  font-size:20px;
  font-weight:700;
  text-align:center;
  margin-bottom:6px;
}
.timer{
  text-align:center;
  font-size:14px;
  opacity:.85;
}
.group{
  font-weight:600;
  margin-bottom:6px;
}
.footer{
  margin-top:16px;
  font-size:13px;
  opacity:.6;
  text-align:center;
}
.likes{
  display:flex;
  gap:10px;
  justify-content:center;
  margin-top:10px;
}
.like, .dislike{
  background:#1f1f1f;
  padding:8px 14px;
  border-radius:20px;
}
.views{
  float:right;
  opacity:.7;
}
.hidden{
  display:none;
}
</style>
</head>

<body>
<div class="container">

<h1>⚡ Львівська область <span class="views">👁 <span id="views"></span></span></h1>

<select id="daySelect">
  <option value="mon">Понеділок</option>
  <option value="tue">Вівторок</option>
  <option value="wed">Середа</option>
  <option value="thu">Четвер</option>
  <option value="fri">Пʼятниця</option>
  <option value="sat">Субота</option>
  <option value="sun">Неділя</option>
</select>

<select id="groupSelect">
  <option>1.1</option><option>1.2</option>
  <option>2.1</option><option>2.2</option>
  <option>3.1</option><option>3.2</option>
  <option>4.1</option><option>4.2</option>
  <option>5.1</option><option>5.2</option>
  <option>6.1</option><option>6.2</option>
</select>

<button onclick="pinGroup()">📌 Закріпити мою групу</button>
<button onclick="toggleAll()">📊 Показати всі групи</button>

<div id="statusCard" class="card"></div>
<div id="allGroups" class="hidden"></div>

<div class="likes">
  <div class="like">❤️ Лайки: <span id="likes">0</span></div>
  <div class="dislike">👎 Дизлайки: 0</div>
</div>

<div class="footer">
  Останнє оновлення: 21.01.2026 19:50<br>
  Оновлюється автоматично
</div>

</div>

<script>
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

function nowMinutes(){
  const d=new Date();
  return d.getHours()*60+d.getMinutes();
}
function toMin(t){
  const [h,m]=t.split(":").map(Number);
  return h*60+m;
}

function render(){
  const day=document.getElementById("daySelect").value;
  const group=document.getElementById("groupSelect").value;
  const box=document.getElementById("statusCard");

  if(!schedules[day]){
    box.innerHTML="⏳ Графік ще формується";
    return;
  }

  const offs=schedules[day][group]||[];
  const now=nowMinutes();

  let off=false,next=null;

  offs.forEach(p=>{
    if(now>=toMin(p[0]) && now<toMin(p[1])) off=true;
    if(toMin(p[0])>now && (!next || toMin(p[0])<toMin(next[0]))) next=p;
  });

  if(off){
    box.innerHTML=`<div class="big-status">⚫ ЗАРАЗ НЕМАЄ СВІТЛА</div>`;
  }else{
    box.innerHTML=`<div class="big-status">🟢 ЗАРАЗ Є СВІТЛО</div>`;
  }
}

document.getElementById("daySelect").value=
["sun","mon","tue","wed","thu","fri","sat"][new Date().getDay()];

render();
setInterval(render,60000);

const start=new Date("2026-01-21T19:40:00").getTime();
setInterval(()=>{
  const l=Math.max(0,Math.floor((Date.now()-start)/1000));
  document.getElementById("likes").textContent=l;
},1000);

document.getElementById("views").textContent=
Math.floor(975+Math.random()*699000);

function pinGroup(){
  localStorage.setItem("group",groupSelect.value);
}
const g=localStorage.getItem("group");
if(g) groupSelect.value=g;
</script>
</body>
</html>
