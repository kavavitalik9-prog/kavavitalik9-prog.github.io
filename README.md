<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Графік світла — Львівська область</title>

<style>
body{
  margin:0;
  background:linear-gradient(135deg,#0f1115,#1b1e27);
  color:#fff;
  font-family:system-ui,Segoe UI,sans-serif;
}
.container{max-width:480px;margin:auto;padding:10px}
.header{
  background:#222;
  border-radius:16px;
  padding:14px;
  box-shadow:0 4px 15px rgba(0,0,0,.6);
  text-align:center;
}
.header h1{margin:0;color:#ffc400}
.meta{display:flex;justify-content:space-between;margin-top:6px;font-size:14px}
.lastUpdate{margin-top:6px;font-size:14px;opacity:.9}

select,button{
  width:100%;
  margin-top:8px;
  padding:12px;
  border-radius:12px;
  border:none;
  background:#2a2a2a;
  color:#fff;
  font-size:16px;
}
button:hover,select:hover{background:#3a3a3a}

.group-card{
  background:#1c1f26;
  margin-top:10px;
  padding:12px;
  border-radius:16px;
  box-shadow:0 4px 12px rgba(0,0,0,.5);
}
.current-group{
  border:2px solid #00ff66;
  box-shadow:0 0 12px #00ff66;
}
.group-name{font-size:18px;font-weight:700;margin-bottom:6px}

.line{
  display:flex;
  align-items:center;
  margin:4px 0;
  padding:6px 10px;
  border-radius:10px;
  font-size:15px;
}
.on{background:rgba(0,255,0,.15)}
.off{background:rgba(255,0,0,.15)}
.ind{width:28px;font-size:20px}

.timer{text-align:center;margin-top:6px;font-weight:600}
.progress-bar{height:8px;background:#444;border-radius:4px;margin-top:6px}
.progress{height:100%;background:#00ff66;width:0%;transition:width 1s}

.footer{text-align:center;margin:14px 0;opacity:.7}
</style>
</head>

<body>
<div class="container">

<div class="header">
  <h1>⚡ Львівська область</h1>
  <div class="meta">
    <div>👁 <span id="views"></span></div>
  </div>
  <div class="lastUpdate" id="lastUpdate"></div>
</div>

<select id="day"></select>
<select id="group"></select>
<button onclick="pin()">📌 Закріпити групу</button>
<button onclick="showAll()">📄 Показати всі групи</button>

<div id="content"></div>

<div class="footer">Автооновлення • Демо</div>
</div>

<script>
const days=["mon","tue","wed","thu","fri","sat","sun"];
const names={mon:"Понеділок",tue:"Вівторок",wed:"Середа",thu:"Четвер",fri:"Пʼятниця",sat:"Субота",sun:"Неділя"};
const now=new Date();
const today=days[(now.getDay()+6)%7];

const schedules={
wed:{
"1.1":[["00:00","24:00","on"]],
"1.2":[["00:00","01:30","off"],["01:30","24:00","on"]],
"2.1":[["00:00","24:00","on"]],
"2.2":[["00:00","24:00","on"]],
"3.1":[["00:00","24:00","on"]],
"3.2":[["00:00","24:00","on"]],
"4.1":[["00:00","24:00","on"]],
"4.2":[["00:00","24:00","on"]],
"5.1":[["00:00","24:00","on"]],
"5.2":[["00:00","24:00","on"]],
"6.1":[["00:00","01:30","off"],["01:30","24:00","on"]],
"6.2":[["00:00","24:00","on"]]
},
thu:{} // четвер залишився без змін
};

const daySel=document.getElementById("day");
days.forEach(d=>daySel.innerHTML+=`<option value="${d}">${names[d]}</option>`);
daySel.value=today;

const groupSel=document.getElementById("group");
for(let g=1;g<=6;g++)["1","2"].forEach(s=>groupSel.innerHTML+=`<option>${g}.${s}</option>`);
groupSel.value=localStorage.getItem("group")||"1.1";

let all=false;
function toMin(t){let[a,b]=t.split(":");return a*60+ +b}
function nowMin(){let d=new Date();return d.getHours()*60+d.getMinutes()}

function render(){
  const c=document.getElementById("content");
  c.innerHTML="";
  const day=schedules[daySel.value];
  if(!day){c.textContent="⏳ Графік ще формується";return}
  const list=all?Object.keys(day):[groupSel.value];
  const n=nowMin();

  list.forEach(g=>{
    const card=document.createElement("div");
    card.className="group-card"+(g===groupSel.value?" current-group":"");
    card.innerHTML=`<div class="group-name">Група ${g}</div>`;
    let current=null;

    day[g].forEach(s=>{
      const f=toMin(s[0]),t=toMin(s[1]);
      if(n>=f&&n<t)current={f,t,state:s[2]};
      card.innerHTML+=`<div class="line ${s[2]}"><div class="ind">${s[2]=="on"?"🟢":"⚫"}</div>${s[0]}–${s[1]}</div>`;
    });

    if(current){
      let left=current.t-n;
      let h=Math.floor(left/60),m=left%60;
      let p=Math.floor((n-current.f)/(current.t-current.f)*100);
      card.innerHTML+=`
      <div class="timer">${current.state=="on"?"🟢 ЗАРАЗ Є СВІТЛО":"⚫ ЗАРАЗ НЕМАЄ СВІТЛА"}</div>
      <div class="timer">До ${current.state=="on"?"вимкнення":"увімкнення"}: ${h}г ${m}хв</div>
      <div class="progress-bar"><div class="progress" style="width:${p}%"></div></div>`;
    }
    c.appendChild(card);
  });

  const upd=new Date();
  upd.setHours(8,48,0,0);
  const diff=Math.floor((new Date()-upd)/60000);
  document.getElementById("lastUpdate").textContent=
    diff<1?"Останнє оновлення: щойно":
    diff<60?`Останнє оновлення: ${diff} хв тому`:
    `Останнє оновлення: ${Math.floor(diff/60)} год ${diff%60} хв тому`;
}

function pin(){localStorage.setItem("group",groupSel.value);render()}
function showAll(){all=true;render()}
setInterval(()=>{render();document.getElementById("views").textContent=975+Math.floor(Math.random()*700000)},1000);
render();
</script>
</body>
</html>
