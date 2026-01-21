<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>⚡ Львівська область — графік світла</title>

<style>
body{
  margin:0;
  background:#0f0f0f;
  color:#fff;
  font-family:Arial,sans-serif;
}
.container{
  max-width:420px;
  margin:auto;
  padding:10px;
}
.box{
  border:1px solid #333;
  border-radius:12px;
  padding:10px;
  margin-bottom:10px;
}
.status{
  font-size:20px;
  font-weight:bold;
  margin:6px 0;
}
select{
  width:100%;
  padding:8px;
  border-radius:8px;
  border:none;
  background:#1c1c1c;
  color:#fff;
}
.group{
  font-size:14px;
  padding:4px 0;
  border-bottom:1px solid #222;
}
.reactions{
  display:flex;
  gap:10px;
}
.likes,.dislikes{
  border:1px solid #333;
  padding:6px 10px;
  border-radius:8px;
  font-size:14px;
}
.dislikes{
  opacity:.5;
}
</style>
</head>

<body>
<div class="container">

<div class="box">
  <div>⚡ <b>Львівська область</b></div>
  <div class="status" id="status">...</div>
</div>

<select id="groupSelect"></select>

<div class="box" id="myGroup"></div>
<div class="box" id="allGroups"></div>

<div class="reactions">
  <div class="likes">❤️ Лайки: <span id="likes">0</span></div>
  <div class="dislikes">👎 Дизлайки: <span id="dislikes">0</span></div>
</div>

</div>

<script>
const startLikes = new Date("2026-01-21T20:40:00");

const schedules={
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

const groups=Object.keys(schedules["Середа"]);
const select=document.getElementById("groupSelect");
groups.forEach(g=>{
  const o=document.createElement("option");
  o.value=g;o.textContent="Група "+g;
  select.appendChild(o);
});
select.value=localStorage.getItem("group")||"1.1";
select.onchange=()=>{localStorage.setItem("group",select.value);render();};

function nowMin(){
  const d=new Date();
  return d.getHours()*60+d.getMinutes();
}

function hasLight(list){
  for(const [s,e] of list){
    const sm=+s.split(":")[0]*60+ +s.split(":")[1];
    const em=e==="24:00"?1440:+e.split(":")[0]*60+ +e.split(":")[1];
    if(nowMin()>=sm && nowMin()<em) return false;
  }
  return true;
}

function render(){
  const day=new Date().getDay()==4?"Четвер":"Середа";
  const g=select.value;
  const light=hasLight(schedules[day][g]);

  document.getElementById("status").textContent=
    light?"🟢 ЗАРАЗ Є СВІТЛО":"⚫ ЗАРАЗ НЕМАЄ СВІТЛА";

  document.getElementById("myGroup").innerHTML=
    `<b>${day} — група ${g}</b><br>${light?"Світло є":"Світла нема"}`;

  document.getElementById("allGroups").innerHTML=
    groups.map(x=>`<div class="group">${x}: ${hasLight(schedules[day][x])?"🟢 є":"⚫ нема"}</div>`).join("");
}

// лайки + дизлайки авто
setInterval(()=>{
  const diff=Math.floor((Date.now()-startLikes)/1000);
  if(diff>0){
    document.getElementById("likes").textContent=diff;
    document.getElementById("dislikes").textContent=Math.floor(diff/3);
  }
},1000);

render();
setInterval(render,60000);
</script>

</body>
</html>
