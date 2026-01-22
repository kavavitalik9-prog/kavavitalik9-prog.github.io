<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>⚡ Графік світла — Львівська область</title>
<style>
body{
  margin:0;
  font-family:'Segoe UI', sans-serif;
  background:#121212;
  color:#fff;
}
.container{
  max-width:420px;
  margin:auto;
  padding:12px;
}
.header{
  background:#1e1e1e;
  border-radius:16px;
  padding:16px;
  text-align:center;
  position:relative;
  box-shadow:0 0 20px rgba(255,196,0,.3);
}
.header h1{
  margin:0;
  color:#ffc400;
  font-size:22px;
}
.update{
  font-size:14px;
  opacity:.8;
  margin-top:6px;
}
.views{
  margin-top:6px;
  font-size:14px;
  padding:6px 10px;
  border:2px solid #ffc400;
  border-radius:14px;
  text-align:center;
  background:#222;
}
.big-status{
  margin-top:14px;
  padding:18px;
  border-radius:20px;
  font-size:22px;
  font-weight:700;
  text-align:center;
  box-shadow:0 0 15px rgba(0,255,0,.3);
}
.big-status.on{
  background: rgba(0,255,0,0.15);
  color:#6cff8f;
  animation: blink 2s infinite;
  box-shadow:0 0 20px rgba(0,255,0,.5);
}
.big-status.off{
  background: rgba(255,0,0,0.15);
  color:#ff6c6c;
  box-shadow:0 0 20px rgba(255,0,0,.5);
}
@keyframes blink{
  0%,50%,100% {opacity:1;}
  25%,75% {opacity:0.5;}
}
.selects{
  display:flex;
  gap:6px;
  flex-wrap:wrap;
  margin-top:12px;
}
.selects select, .selects button{
  flex:1 1 48%;
  padding:12px;
  border-radius:12px;
  border:none;
  background:#2a2a2a;
  color:#fff;
  font-size:16px;
  cursor:pointer;
}
.group-card{
  background:#1c1c1c;
  margin-top:12px;
  padding:14px;
  border-radius:16px;
  box-shadow:0 0 12px rgba(0,0,0,.5);
}
.line{
  display:flex;
  align-items:center;
  gap:10px;
  padding:6px 10px;
  border-radius:12px;
  margin:6px 0;
}
.ind{
  width:28px;
  font-size:20px;
}
.timer-line{
  margin-left:auto;
  font-size:14px;
  opacity:.85;
}
.line.on{background:rgba(0,255,0,0.2);}
.line.off{background:rgba(255,0,0,0.2);}
.admin-btn{
  position:absolute;
  top:12px;
  right:12px;
  background:#333;
  width:38px;
  height:38px;
  border-radius:50%;
  display:flex;
  align-items:center;
  justify-content:center;
  cursor:pointer;
}
.admin-panel{
  display:none;
  margin-top:14px;
  background:#111;
  border-radius:16px;
  padding:12px;
  box-shadow:0 0 18px rgba(255,196,0,.5);
}
textarea{
  width:100%;
  height:100px;
  resize:none;
  border-radius:12px;
  background:#222;
  color:#fff;
  padding:8px;
  margin-top:6px;
}
.group-edit label{font-weight:600;margin-top:6px;display:block}
</style>
</head>
<body>

<div class="container">

<div class="header">
  <h1>⚡ Львівська область</h1>
  <div class="update" id="updateText">Останнє оновлення: —</div>
  <div class="views" id="viewers">Переглядають: —</div>
  <div class="admin-btn" onclick="openAdmin()">🔒</div>
</div>

<div id="status" class="big-status">—</div>

<div class="selects">
  <select id="day"></select>
  <select id="groupSelect">
    <option value="all">Всі групи</option>
    <option value="1.1">1.1</option>
    <option value="1.2">1.2</option>
    <option value="2.1">2.1</option>
    <option value="2.2">2.2</option>
    <option value="3.1">3.1</option>
    <option value="3.2">3.2</option>
    <option value="4.1">4.1</option>
    <option value="4.2">4.2</option>
    <option value="5.1">5.1</option>
    <option value="5.2">5.2</option>
    <option value="6.1">6.1</option>
    <option value="6.2">6.2</option>
  </select>
  <button onclick="pinGroup()">Закріпити групу</button>
  <button onclick="showAllGroups()">Показати всі групи</button>
</div>

<div id="content"></div>

<div class="admin-panel" id="admin">
<h3>🔧 Адмін-панель</h3>
<label>День:</label>
<select id="adminDay">
  <option value="current">Поточний</option>
  <option value="all">Весь тиждень</option>
</select>
<div id="allGroupsEditor"></div>
<button onclick="saveAll()">💾 Зберегти усі групи</button>
</div>

</div>

<script>
const PASS="3709";
const days=[["mon","Понеділок"],["tue","Вівторок"],["wed","Середа"],["thu","Четвер"],["fri","Пʼятниця"],["sat","Субота"],["sun","Неділя"]];
const groups=[...Array(6)].flatMap((_,i)=>[`${i+1}.1`,`${i+1}.2`]);

const daySel=document.getElementById("day");
days.forEach(d=>daySel.innerHTML+=`<option value="${d[0]}">${d[1]}</option>`);
daySel.value=["sun","mon","tue","wed","thu","fri","sat"][new Date().getDay()];

let data=JSON.parse(localStorage.getItem("data")||"{}");
let last=+localStorage.getItem("last")||null;
let pinnedGroup = localStorage.getItem("pinnedGroup") || "all";
document.getElementById("groupSelect").value = pinnedGroup;

function toMin(t){let[a,b]=t.split(":");return a*60+ +b}
function normalize(off){
 let r=[],p=0;
 off.sort((a,b)=>toMin(a[0])-toMin(b[0]));
 off.forEach(o=>{
  let f=toMin(o[0]),t=toMin(o[1]);
  if(p<f) r.push([p,f,"on"]);
  r.push([f,t,"off"]); p=t;
 });
 if(p<1440) r.push([p,1440,"on"]);
 return r;
}

function human(){
 if(!last) return "—";
 let s=(Date.now()-last)/1000;
 if(s<60) return "щойно";
 if(s<3600) return Math.floor(s/60)+" хв тому";
 if(s<86400) return Math.floor(s/3600)+" год тому";
 return Math.floor(s/86400)+" дн тому";
}

function render(){
 // фейкові глядачі
 const viewCount=Math.floor(Math.random()*(700000-975+1)+975);
 document.getElementById("viewers").innerText=`Переглядають: ${viewCount}`;

 document.getElementById("updateText").innerText="Останнє оновлення: "+human();
 let c=document.getElementById("content");
 c.innerHTML="";
 let now=new Date();
 let m=now.getHours()*60+now.getMinutes();
 let st=document.getElementById("status");

 groups.forEach((g,i)=>{
  if(pinnedGroup!=="all" && g!==pinnedGroup) return;
  if(!data[daySel.value]||!data[daySel.value][g]){
   c.innerHTML+=`<div class="center">${g}: ⏳ графік формується</div>`;
   return;
  }
  let seg=normalize(data[daySel.value][g]);
  let cur=seg.find(s=>m>=s[0]&&m<s[1]);
  if(i===0 || pinnedGroup===g){
    st.className="big-status "+cur[2];
    st.innerText=cur[2]=="on"?"🟢 ЗАРАЗ Є СВІТЛО":"⚫ ЗАРАЗ НЕМАЄ СВІТЛА";
  }
  let html=`<div class="group-card"><b>${g}</b>`;
  seg.forEach(s=>{
   let timerText="";
   if(cur===s){
     let d=s[1]-m;
     let h=Math.floor(d/60);
     let min=d%60;
     timerText=(s[2]=="on"?"До вимкнення: ":"До увімкнення: ")+`${h}г ${min}хв`;
   }
   let lineClass = (cur===s) ? (s[2]=="on"?"on":"off") : "";
   html+=`<div class="line ${s[2]} ${lineClass}">
   <div class="ind">${s[2]=="on"?"🟢":"⚫"}</div>
   ${String(Math.floor(s[0]/60)).padStart(2,"0")}:${String(s[0]%60).padStart(2,"0")} – ${String(Math.floor(s[1]/60)).padStart(2,"0")}:${String(s[1]%60).padStart(2,"0")}
   <div class="timer-line">${timerText}</div>
   </div>`;
  });
  html+="</div>";
  c.innerHTML+=html;
 });
}

/* Адмін */
function openAdmin(){
 if(prompt("Пароль")!==PASS){alert("Невірно"); return;}
 document.getElementById("admin").style.display="block";
 renderAdmin();
}

function renderAdmin(){
 const cont=document.getElementById("allGroupsEditor");
 cont.innerHTML="";
 groups.forEach(g=>{
  let val=data[daySel.value]?.[g]?.map(a=>a.join("-")).join("\n")||"";
  cont.innerHTML+=`<div class="group-edit"><label>${g}</label><textarea id="ta_${g}">${val}</textarea></div>`;
 });
}

function saveAll(){
 let dayTarget=document.getElementById("adminDay").value;
 let daysTarget=dayTarget=="all"?days.map(d=>d[0]):[daySel.value];
 groups.forEach(g=>{
  let val=document.getElementById(`ta_${g}`).value.trim().split("\n").filter(Boolean).map(l=>l.split("-"));
  daysTarget.forEach(d=>{
    if(!data[d]) data[d]={};
    data[d][g]=val;
  });
 });
 last=Date.now();
 localStorage.setItem("data",JSON.stringify(data));
 localStorage.setItem("last",last);
 render();
}

function pinGroup() {
  pinnedGroup = document.getElementById("groupSelect").value;
  localStorage.setItem("pinnedGroup", pinnedGroup);
  render();
}

function showAllGroups() {
  pinnedGroup = "all";
  document.getElementById("groupSelect").value = "all";
  localStorage.setItem("pinnedGroup", pinnedGroup);
  render();
}

daySel.onchange=()=>{render(); renderAdmin();}
setInterval(render,1000);
render();
</script>
</body>
</html>
