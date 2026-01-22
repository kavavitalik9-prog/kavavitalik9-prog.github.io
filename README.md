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
  animation:blink 3s ease-in-out infinite;
}

@keyframes blink{
  0%,100%{opacity:1}
  50%{opacity:.7}
}

.timer{font-size:14px;color:#ccc;margin-top:6px}

/* ===== TIME SCALE ===== */
.timeNumbers{
  display:flex;
  justify-content:space-between;
  font-size:12px;
  color:#aaa;
  margin-top:8px;
}

.timeline{
  position:relative;
  height:22px;
  border-radius:11px;
  overflow:hidden;
  margin-top:4px;
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
  <select id="daySelect"></select>
  <select id="groupSelect"></select>
</div>

<div id="groups"></div>

</div>

<script>
/* ===== DATA ===== */
const days=["Понеділок","Вівторок","Середа","Четвер","Пʼятниця","Субота","Неділя"];
const groups=[
"1.1","1.2","2.1","2.2","3.1","3.2",
"4.1","4.2","5.1","5.2","6.1","6.2"
];

let schedules={};
days.forEach(d=>{
  schedules[d]={};
  groups.forEach(g=>{
    schedules[d][g]=[["00:00","24:00","on"]];
  });
});

/* ===== AUTO DAY ===== */
const today=new Date().getDay(); // 0 = Sunday
let currentDay=days[(today+6)%7];

let selectedGroup=localStorage.getItem("group")||"ALL";
let lastUpdate=new Date();

/* ===== UTIL ===== */
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

/* ===== VIEWERS ===== */
function fakeViewers(){
  document.getElementById("viewers").innerText=
   "👀 "+(Math.floor(Math.random()*40)+60)+" онлайн";
}
setInterval(fakeViewers,5000);
fakeViewers();

/* ===== SELECTS ===== */
const daySel=document.getElementById("daySelect");
days.forEach(d=>daySel.innerHTML+=`<option>${d}</option>`);
daySel.value=currentDay;
daySel.onchange=()=>{currentDay=daySel.value;render();};

const groupSel=document.getElementById("groupSelect");
groupSel.innerHTML="<option value='ALL'>Усі групи</option>";
groups.forEach(g=>groupSel.innerHTML+=`<option>${g}</option>`);
groupSel.value=selectedGroup;
groupSel.onchange=()=>{
  selectedGroup=groupSel.value;
  localStorage.setItem("group",selectedGroup);
  render();
};

/* ===== RENDER ===== */
function render(){
  document.getElementById("updated").innerText=
   "Останнє оновлення: "+
   format(Math.floor((Date.now()-lastUpdate)/60000))+" тому";

  const box=document.getElementById("groups");
  box.innerHTML="";
  const now=nowMin();

  groups.forEach(g=>{
    if(selectedGroup!=="ALL" && selectedGroup!==g) return;

    let state="off",next=0;
    for(const s of schedules[currentDay][g]){
      const a=toMin(s[0]),b=toMin(s[1]);
      if(now>=a && now<b){
        state=s[2];
        next=b-now;
        break;
      }
    }

    let segments="";
    schedules[currentDay][g].forEach(s=>{
      const left=toMin(s[0])/1440*100;
      const width=(toMin(s[1])-toMin(s[0]))/1440*100;
      segments+=`<div class="segment ${s[2]==="on"?"onSeg":"offSeg"}"
        style="left:${left}%;width:${width}%"></div>`;
    });

    box.innerHTML+=`
    <div class="group ${state==="on"?"onBox":"offBox"}">
      <div class="group-title">Група ${g}</div>

      <div class="timeNumbers">
        <span>00</span><span>06</span><span>12</span><span>18</span><span>24</span>
      </div>

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
