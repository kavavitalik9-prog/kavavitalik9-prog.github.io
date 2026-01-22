<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>⚡ Графік світла — Львівська область</title>

<style>
body{
  margin:0;
  background:linear-gradient(135deg,#0d0f14,#1b1f2a);
  font-family:system-ui,Segoe UI,sans-serif;
  color:#fff;
}
.container{max-width:520px;margin:auto;padding:12px}
.header{
  background:#222;
  border-radius:18px;
  padding:16px;
  text-align:center;
  position:relative;
  box-shadow:0 0 20px rgba(255,196,0,.3);
}
.header h1{margin:0;color:#ffc400}
.update{font-size:14px;opacity:.85;margin-top:6px}

.big-status{
  margin-top:12px;
  padding:14px;
  border-radius:16px;
  font-size:20px;
  font-weight:700;
}
.on{background:rgba(0,255,0,.15);color:#6cff8f}
.off{background:rgba(255,0,0,.15);color:#ff6c6c}

select,button,textarea{
  width:100%;
  margin-top:10px;
  padding:12px;
  border-radius:14px;
  border:none;
  background:#2a2a2a;
  color:#fff;
  font-size:16px;
}
button{cursor:pointer}

.group-card{
  background:#1b1f2a;
  margin-top:10px;
  padding:12px;
  border-radius:16px;
}
.line{
  display:flex;
  align-items:center;
  gap:8px;
  padding:6px 10px;
  border-radius:10px;
  margin:4px 0;
}
.ind{width:26px;font-size:20px}

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

textarea{height:160px;resize:none}
.center{text-align:center;padding:20px;opacity:.8}
.timer{margin-top:8px;font-size:16px;opacity:.9}
.pin-btn{background:#444}
</style>
</head>

<body>
<div class="container">

<div class="header">
  <h1>⚡ Львівська область</h1>
  <div class="update" id="updateText">Останнє оновлення: —</div>
  <div class="admin-btn" onclick="openAdmin()">🔒</div>
</div>

<div id="status" class="big-status">—</div>
<div class="timer" id="timer"></div>

<select id="day"></select>
<select id="group"></select>
<button class="pin-btn" onclick="pinGroup()">📌 Закріпити групу</button>
<button onclick="toggleAll()">👥 Показати всі групи</button>

<div id="content"></div>

<div class="admin-panel" id="admin">
<h3>🔧 Адмін-панель</h3>
<label>Режим:</label>
<select id="mode">
  <option value="one">Одна група</option>
  <option value="all">Всі групи</option>
</select>

<label>День:</label>
<select id="adminDay">
  <option value="current">Поточний</option>
  <option value="all">Весь тиждень</option>
</select>

<textarea id="editor" placeholder="Періоди БЕЗ світла
Напр:
01:30-06:00
18:00-20:00"></textarea>

<button onclick="save()">💾 Зберегти</button>
</div>

</div>

<script>
const PASS="3709";
const days=[["mon","Понеділок"],["tue","Вівторок"],["wed","Середа"],["thu","Четвер"],["fri","Пʼятниця"],["sat","Субота"],["sun","Неділя"]];
const groups=[...Array(6)].flatMap((_,i)=>[`${i+1}.1`,`${i+1}.2`]);

const daySel=day,groupSel=group;
days.forEach(d=>daySel.innerHTML+=`<option value="${d[0]}">${d[1]}</option>`);
groups.forEach(g=>groupSel.innerHTML+=`<option>${g}</option>`);

daySel.value=["sun","mon","tue","wed","thu","fri","sat"][new Date().getDay()];
groupSel.value=localStorage.getItem("pinned")||"1.1";

let data=JSON.parse(localStorage.getItem("data")||"{}");
let last=+localStorage.getItem("last")||null;
let showAll=false;

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
 updateText.innerText="Останнє оновлення: "+human();
 content.innerHTML="";
 let now=new Date();
 let m=now.getHours()*60+now.getMinutes();

 let list=showAll?groups:[groupSel.value];
 list.forEach(g=>{
  if(!data[daySel.value]||!data[daySel.value][g]){
   content.innerHTML+=`<div class="center">${g}: ⏳ графік формується</div>`;
   return;
  }
  let seg=normalize(data[daySel.value][g]);
  let cur=seg.find(s=>m>=s[0]&&m<s[1]);
  if(g===groupSel.value){
    status.className="big-status "+cur[2];
    status.innerText=cur[2]=="on"?"🟢 ЗАРАЗ Є СВІТЛО":"⚫ ЗАРАЗ НЕМАЄ СВІТЛА";
    let d=cur[1]-m;
    timer.innerText=(cur[2]=="on"?"До вимкнення: ":"До увімкнення: ")+
      `${String(Math.floor(d/60)).padStart(2,"0")}:${String(d%60).padStart(2,"0")}`;
  }
  let html=`<div class="group-card"><b>${g}</b>`;
  seg.forEach(s=>{
   html+=`<div class="line ${s[2]}">
   <div class="ind">${s[2]=="on"?"🟢":"⚫"}</div>
   ${String(Math.floor(s[0]/60)).padStart(2,"0")}:${String(s[0]%60).padStart(2,"0")}
   –
   ${String(Math.floor(s[1]/60)).padStart(2,"0")}:${String(s[1]%60).padStart(2,"0")}
   </div>`;
  });
  html+="</div>";
  content.innerHTML+=html;
 });
}

function pinGroup(){
 localStorage.setItem("pinned",groupSel.value);
 alert("Групу закріплено");
}

function toggleAll(){
 showAll=!showAll;
 render();
}

function openAdmin(){
 if(prompt("Пароль")!==PASS) return alert("❌ Невірно");
 admin.style.display="block";
}

function save(){
 let lines=editor.value.trim().split("\n").filter(Boolean).map(l=>l.split("-"));
 let daysTarget=adminDay.value=="all"?days.map(d=>d[0]):[daySel.value];
 let groupsTarget=mode.value=="all"?groups:[groupSel.value];
 daysTarget.forEach(d=>{
  if(!data[d]) data[d]={};
  groupsTarget.forEach(g=>data[d][g]=lines);
 });
 last=Date.now();
 localStorage.setItem("data",JSON.stringify(data));
 localStorage.setItem("last",last);
 render();
}

setInterval(render,1000);
render();
</script>
</body>
</html>
