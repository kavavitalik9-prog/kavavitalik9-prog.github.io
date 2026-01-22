<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Графік світла</title>

<style>
body{
  margin:0;
  background:#0b0d13;
  font-family:system-ui,-apple-system;
  display:flex;
  justify-content:center;
  color:#fff;
}

.phone{
  width:100%;
  max-width:420px;
  min-height:100vh;
  padding:12px;
}

.header{
  background:#151a26;
  border-radius:18px;
  padding:14px;
  margin-bottom:10px;
}

.title{font-size:18px;font-weight:600}
.updated{font-size:13px;color:#aaa;margin-top:4px}
.viewers{font-size:13px;color:#7aff9e;margin-top:6px}

.controls{
  display:flex;
  gap:8px;
  margin-bottom:10px;
}

select,button,input{
  flex:1;
  background:#151a26;
  color:#fff;
  border:none;
  border-radius:14px;
  padding:10px;
  font-size:14px;
}

button{cursor:pointer}

.group{
  background:#151a26;
  border-radius:18px;
  padding:14px;
  margin-bottom:12px;
}

.onBox{box-shadow:0 0 0 2px #2cff9a55}
.offBox{box-shadow:0 0 0 2px #ff4c4c55}

.group-title{font-size:16px;font-weight:600}
.status{margin-top:6px;font-weight:600}
.on{color:#2cff9a}
.off{color:#ff5c5c}

.blink{
  animation:blink 1.8s ease-in-out infinite;
}

@keyframes blink{
  0%,100%{opacity:1}
  50%{opacity:.6}
}

.timer{font-size:14px;color:#ccc;margin-top:6px}

/* ===== TIME SCALE ===== */
.timeline{
  position:relative;
  height:22px;
  border-radius:11px;
  overflow:hidden;
  margin-top:10px;
  background:#222;
}

.segment{
  position:absolute;
  top:0;
  height:100%;
}

.onSeg{background:#2cff9a}
.offSeg{background:#ff5c5c}

.nowLine{
  position:absolute;
  top:-4px;
  width:2px;
  height:30px;
  background:#fff;
  opacity:.9;
}

/* ===== ADMIN ===== */
.admin{
  display:none;
  background:#111522;
  border-radius:18px;
  padding:14px;
  margin-top:12px;
}
</style>
</head>

<body>
<div class="phone">

<div class="header">
  <div class="title">⚡ Графік відключень</div>
  <div class="updated" id="updated"></div>
  <div class="viewers" id="viewers"></div>
</div>

<div class="controls">
  <select id="groupSelect"></select>
  <button onclick="toggleAdmin()">🔐</button>
</div>

<div id="groups"></div>

<div class="admin" id="admin">
  <input id="adminPass" placeholder="Пароль">
  <button onclick="login()">Увійти</button>
  <textarea id="adminText"
   style="width:100%;height:100px;margin-top:8px;border-radius:14px;
   background:#151a26;color:#fff;padding:8px"></textarea>
  <button onclick="saveAll()">💾 Оновити всі групи</button>
</div>

</div>

<script>
const PASSWORD="3709";
let admin=false;

const allGroups=[
"1.1","1.2","2.1","2.2","3.1","3.2",
"4.1","4.2","5.1","5.2","6.1","6.2"
];

let selected=localStorage.getItem("group")||"ALL";
let lastUpdate=new Date();

let schedule={};
allGroups.forEach(g=>{
  schedule[g]=[["00:00","24:00","on"]];
});

// ===== UTIL =====
function toMin(t){
  const [h,m]=t.split(":").map(Number);
  return h*60+m;
}
function nowMin(){
  const d=new Date();
  return d.getHours()*60+d.getMinutes();
}
function format(min){
  if(min<=0) return "0 хв";
  let d=Math.floor(min/1440);
  let h=Math.floor((min%1440)/60);
  let m=min%60;
  let r=[];
  if(d) r.push(d+" д");
  if(h) r.push(h+" год");
  if(m) r.push(m+" хв");
  return r.join(" ");
}

// ===== VIEWERS =====
function fakeViewers(){
  document.getElementById("viewers").innerText=
   "👀 "+(Math.floor(Math.random()*40)+60)+" онлайн";
}
setInterval(fakeViewers,5000);
fakeViewers();

// ===== SELECT =====
const sel=document.getElementById("groupSelect");
sel.innerHTML="<option value='ALL'>Усі групи</option>";
allGroups.forEach(g=>sel.innerHTML+=`<option>${g}</option>`);
sel.value=selected;
sel.onchange=()=>{
  selected=sel.value;
  localStorage.setItem("group",selected);
  render();
};

// ===== ADMIN =====
function toggleAdmin(){
  document.getElementById("admin").style.display="block";
}
function login(){
  if(document.getElementById("adminPass").value===PASSWORD){
    admin=true;
    document.getElementById("adminText").value=
      JSON.stringify(schedule,null,2);
    alert("Адмін режим увімкнено");
  }
}
function saveAll(){
  if(!admin) return;
  schedule=JSON.parse(document.getElementById("adminText").value);
  lastUpdate=new Date();
  render();
}

// ===== RENDER =====
function render(){
  document.getElementById("updated").innerText=
   "Останнє оновлення: "+
   format(Math.floor((Date.now()-lastUpdate)/60000))+" тому";

  const box=document.getElementById("groups");
  box.innerHTML="";
  const now=nowMin();

  allGroups.forEach(g=>{
    if(selected!=="ALL" && selected!==g) return;

    let state="off",next=0;
    for(const s of schedule[g]){
      const a=toMin(s[0]),b=toMin(s[1]);
      if(now>=a && now<b){
        state=s[2];
        next=b-now;
        break;
      }
    }

    let segments="";
    schedule[g].forEach(s=>{
      const left=toMin(s[0])/1440*100;
      const width=(toMin(s[1])-toMin(s[0]))/1440*100;
      segments+=`<div class="segment ${s[2]==="on"?"onSeg":"offSeg"}"
        style="left:${left}%;width:${width}%"></div>`;
    });

    box.innerHTML+=`
    <div class="group ${state==="on"?"onBox":"offBox"}">
      <div class="group-title">Група ${g}</div>

      <div class="timeline">
        ${segments}
        <div class="nowLine" style="left:${now/1440*100}%"></div>
      </div>

      <div class="status ${state==="on"?"on blink":"off"}">
        ${state==="on"?"🟢 ЗАРАЗ Є СВІТЛО":"⚫ ЗАРАЗ НЕМА СВІТЛА"}
      </div>

      <div class="timer">
        ${state==="on"?"До вимкнення: ":"До увімкнення: "}
        ${format(next)}
      </div>
    </div>`;
  });
}

render();
setInterval(render,60000);
</script>
</body>
</html>
