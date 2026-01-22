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
  font-family:system-ui;
  display:flex;
  justify-content:center;
  color:#fff;
}
.phone{max-width:420px;width:100%;min-height:100vh;padding:12px}
.header{
  background:#151a26;
  border-radius:18px;
  padding:14px;
  margin-bottom:10px;
  position:relative;
}
.adminBtn{
  position:absolute;
  right:12px;
  top:12px;
  background:#222;
  border:none;
  color:#fff;
  border-radius:10px;
  padding:6px 10px;
}
.group{background:#151a26;border-radius:18px;padding:14px;margin-bottom:12px}
.on{color:#2cff9a}
.off{color:#ff5c5c}
.blink{animation:blink 3s infinite}
@keyframes blink{0%,100%{opacity:1}50%{opacity:.7}}

.timeline{height:20px;background:#222;border-radius:10px;overflow:hidden;margin-top:6px;position:relative}
.segOn{background:#2cff9a;height:100%;position:absolute}
.segOff{background:#ff5c5c;height:100%;position:absolute}
.now{position:absolute;width:2px;height:26px;background:#fff;top:-3px}

.modal{
  position:fixed;inset:0;background:#000a;
  display:none;justify-content:center;align-items:center
}
.modalBox{
  background:#151a26;
  padding:16px;
  border-radius:16px;
  width:90%;
  max-width:380px;
}
textarea,input,select,button{
  width:100%;
  margin-top:8px;
  padding:10px;
  border-radius:10px;
  border:none;
  background:#222;
  color:#fff;
}
</style>
</head>

<body>
<div class="phone">

<div class="header">
  <b>⚡ Львівська область</b>
  <div id="updated"></div>
  <button class="adminBtn" onclick="openAdmin()">⚙ Адмін</button>
</div>

<div id="list"></div>

</div>

<!-- ADMIN -->
<div class="modal" id="admin">
  <div class="modalBox">
    <h3>Адмін доступ</h3>
    <input id="pass" placeholder="Пароль">
    <select id="day"></select>
    <textarea id="offInput" rows="5"
      placeholder="Впиши періоди БЕЗ світла
Напр:
10:00-12:00
18:00-20:30"></textarea>
    <button onclick="save()">Зберегти</button>
    <button onclick="closeAdmin()">Закрити</button>
  </div>
</div>

<script>
const PASS="3709";
const groups=["1.1","1.2","2.1","2.2","3.1","3.2","4.1","4.2","5.1","5.2","6.1","6.2"];
const days=["Понеділок","Вівторок","Середа","Четвер","Пʼятниця","Субота","Неділя"];

let schedules={};
days.forEach(d=>{
  schedules[d]={};
  groups.forEach(g=>schedules[d][g]=[]);
});

let lastUpdate=new Date();

function openAdmin(){
  document.getElementById("admin").style.display="flex";
}
function closeAdmin(){
  document.getElementById("admin").style.display="none";
}

const daySel=document.getElementById("day");
days.forEach(d=>daySel.innerHTML+=`<option>${d}</option>`);

function toMin(t){
  let [h,m]=t.split(":").map(Number);
  return h*60+m;
}

function buildSchedule(offRanges){
  let points=[0,1440];
  offRanges.forEach(r=>{
    let[a,b]=r;
    points.push(a,b);
  });
  points=[...new Set(points)].sort((a,b)=>a-b);

  let res=[];
  for(let i=0;i<points.length-1;i++){
    let a=points[i],b=points[i+1];
    let off=offRanges.some(r=>a>=r[0] && b<=r[1]);
    res.push([a,b,off?"off":"on"]);
  }
  return res;
}

function save(){
  if(document.getElementById("pass").value!==PASS){
    alert("Невірний пароль");
    return;
  }
  let off=document.getElementById("offInput").value
    .split("\n")
    .filter(Boolean)
    .map(l=>{
      let[t1,t2]=l.split("-");
      return [toMin(t1),toMin(t2)];
    });

  let sched=buildSchedule(off);
  let d=daySel.value;
  groups.forEach(g=>schedules[d][g]=sched);
  lastUpdate=new Date();
  closeAdmin();
  render();
}

function render(){
  document.getElementById("updated").innerText=
    "Останнє оновлення: "+lastUpdate.toLocaleTimeString();

  let now=new Date();
  let m=now.getHours()*60+now.getMinutes();
  let box=document.getElementById("list");
  box.innerHTML="";

  let d=days[(now.getDay()+6)%7];

  groups.forEach(g=>{
    let segs="";
    let state="off";
    let next=0;
    schedules[d][g].forEach(s=>{
      let[a,b,t]=s;
      let l=a/1440*100,w=(b-a)/1440*100;
      segs+=`<div class="${t==="on"?"segOn":"segOff"}"
        style="left:${l}%;width:${w}%"></div>`;
      if(m>=a && m<b){state=t;next=b-m}
    });

    box.innerHTML+=`
    <div class="group">
      <b>Група ${g}</b>
      <div class="timeline">
        ${segs}
        <div class="now" style="left:${m/1440*100}%"></div>
      </div>
      <div class="${state==="on"?"on blink":"off"}">
        ${state==="on"?"🟢 ЗАРАЗ Є СВІТЛО":"⚫ ЗАРАЗ НЕМА СВІТЛА"}
      </div>
      <div>До зміни: ${next} хв</div>
    </div>`;
  });
}

render();
setInterval(render,60000);
</script>
</body>
</html>
