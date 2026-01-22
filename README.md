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
  color:#fff;
  display:flex;
  justify-content:center;
}
.phone{
  max-width:420px;
  width:100%;
  min-height:100vh;
  padding:12px 12px 80px;
}
.header{
  background:#151a26;
  border-radius:18px;
  padding:14px;
  margin-bottom:12px;
}
.group{
  background:#151a26;
  border-radius:18px;
  padding:14px;
  margin-bottom:12px;
}
.timeline{
  height:20px;
  background:#222;
  border-radius:10px;
  overflow:hidden;
  margin-top:6px;
  position:relative;
}
.segOn{background:#2cff9a;height:100%;position:absolute}
.segOff{background:#ff5c5c;height:100%;position:absolute}
.now{
  position:absolute;
  width:2px;
  height:26px;
  background:#fff;
  top:-3px;
}
.on{color:#2cff9a}
.off{color:#ff5c5c}
.blink{animation:blink 3s infinite}
@keyframes blink{
  0%,100%{opacity:1}
  50%{opacity:.7}
}

/* ADMIN BUTTON */
.adminBtn{
  position:fixed;
  right:0;
  top:40%;
  background:#222;
  color:#fff;
  border:none;
  border-radius:14px 0 0 14px;
  padding:12px;
  font-size:18px;
}

/* MODAL */
.modal{
  position:fixed;
  inset:0;
  background:#000a;
  display:none;
  justify-content:center;
  align-items:center;
}
.modalBox{
  background:#151a26;
  padding:16px;
  border-radius:16px;
  width:90%;
  max-width:380px;
}
input,select,textarea,button{
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
    <b>⚡ Львівська область</b><br>
    <span id="updated"></span>
  </div>

  <div id="list"></div>
</div>

<button class="adminBtn" onclick="openAdmin()">⚙</button>

<!-- ADMIN -->
<div class="modal" id="admin">
  <div class="modalBox">
    <h3>Адмін панель</h3>
    <input id="pass" placeholder="Пароль">
    <select id="day"></select>
    <select id="group"></select>
    <textarea id="offInput" rows="5"
      placeholder="Періоди БЕЗ світла
10:00-12:00
18:30-20:00"></textarea>
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

const daySel=document.getElementById("day");
const groupSel=document.getElementById("group");
days.forEach(d=>daySel.innerHTML+=`<option>${d}</option>`);
groups.forEach(g=>groupSel.innerHTML+=`<option>${g}</option>`);

function openAdmin(){document.getElementById("admin").style.display="flex"}
function closeAdmin(){document.getElementById("admin").style.display="none"}

function toMin(t){
  let[h,m]=t.split(":").map(Number);
  return h*60+m;
}

function formatTime(min){
  let h=Math.floor(min/60);
  let m=min%60;
  let r=[];
  if(h>0)r.push(h+" год");
  if(m>0)r.push(m+" хв");
  return r.join(" ");
}

function build(off){
  let p=[0,1440];
  off.forEach(r=>p.push(r[0],r[1]));
  p=[...new Set(p)].sort((a,b)=>a-b);
  let r=[];
  for(let i=0;i<p.length-1;i++){
    let a=p[i],b=p[i+1];
    let offed=off.some(o=>a>=o[0]&&b<=o[1]);
    r.push([a,b,offed?"off":"on"]);
  }
  return r;
}

function save(){
  if(pass.value!==PASS){alert("Невірний пароль");return;}
  let off=offInput.value.split("\n").filter(Boolean)
    .map(l=>l.split("-").map(toMin));
  schedules[day.value][group.value]=build(off);
  lastUpdate=new Date();
  closeAdmin();
  render();
}

function render(){
  updated.innerText="Останнє оновлення: "+lastUpdate.toLocaleString();
  let now=new Date();
  let m=now.getHours()*60+now.getMinutes();
  let d=days[(now.getDay()+6)%7];
  list.innerHTML="";

  groups.forEach(g=>{
    let segs="",state="off",next=0;
    schedules[d][g].forEach(s=>{
      let[a,b,t]=s;
      let l=a/1440*100,w=(b-a)/1440*100;
      segs+=`<div class="${t=="on"?"segOn":"segOff"}" style="left:${l}%;width:${w}%"></div>`;
      if(m>=a&&m<b){state=t;next=b-m}
    });

    list.innerHTML+=`
    <div class="group">
      <b>Група ${g}</b>
      <div class="timeline">
        ${segs}
        <div class="now" style="left:${m/1440*100}%"></div>
      </div>
      <div class="${state=="on"?"on blink":"off"}">
        ${state=="on"?"🟢 ЗАРАЗ Є СВІТЛО":"⚫ ЗАРАЗ НЕМА СВІТЛА"}
      </div>
      <div>До зміни: ${formatTime(next)}</div>
    </div>`;
  });
}

render();
setInterval(render,60000);
</script>
</body>
</html>
