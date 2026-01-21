<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
<title>⚡ Графік світла — Львівська область</title>
<style>
body{margin:0;font-family:system-ui,Arial;background:#0f0f0f;color:#fff;}
.container{width:100%;padding:12px;}
.header{display:flex;justify-content:space-between;align-items:center;gap:10px;flex-wrap:wrap;}
h2{font-size:18px;margin:0;}
.viewers{border:1px solid #ffd000;padding:6px 10px;border-radius:10px;background:#111;font-size:13px;}
select,button{width:100%;padding:12px;border-radius:10px;border:none;margin:6px 0;font-size:16px;background:#1c1c1c;color:#fff;}
button{background:#222;}
.card{background:#151515;border-radius:14px;padding:12px;margin-top:10px;}
.groupCard{border:1px solid #222;border-radius:12px;padding:10px;margin-bottom:12px;}
.row{display:flex;justify-content:space-between;align-items:center;border-bottom:1px solid #222;padding:8px 0;font-size:14px;}
.row:last-child{border:none;}
.on{color:#4cff4c;}
.off{color:#ff4c4c;}
.now{background:#222;border-radius:8px;padding:6px;}
.timer{margin-top:4px;font-size:13px;opacity:.9;}
.pin{color:#ffd000;font-size:14px;}
.status{font-size:20px;font-weight:bold;text-align:center;margin:12px 0;padding:8px;border-radius:12px;}
footer{text-align:center;opacity:.5;margin:20px 0 10px;font-size:12px;}
@media(min-width:768px){.groupCard{font-size:16px;}}
</style>
</head>
<body>
<div class="container">

<div class="header">
  <h2>⚡ Львівська область</h2>
  <div class="viewers">👁 <span id="viewers"></span></div>
</div>

<div id="status" class="status">Завантаження...</div>

<select id="day"></select>
<select id="group"></select>

<button onclick="pinGroup()">📌 Закріпити мою групу</button>
<button onclick="toggleAll()">📊 Показати всі групи</button>

<div id="content" class="card"></div>

<footer>Оновлюється автоматично • Демонстрація</footer>

<script>
// =================== Глядачі ===================
let viewers=Math.floor(Math.random()*(700000-975)+975);
const v=document.getElementById("viewers");
function updView(){viewers+=Math.floor(Math.random()*4000-2000);viewers=Math.max(975,Math.min(700000,viewers));v.textContent=viewers.toLocaleString("uk-UA");}
updView(); setInterval(updView,3000);

// =================== Дні та групи ===================
const days=["Понеділок","Вівторок","Середа","Четвер","Пʼятниця","Субота","Неділя"];
const daySelect=document.getElementById("day");
days.forEach(d=>{let o=document.createElement("option");o.textContent=d;o.value=d;daySelect.appendChild(o);});

const groups=["1.1","1.2","2.1","2.2","3.1","3.2","4.1","4.2","5.1","5.2","6.1","6.2"];
const groupSelect=document.getElementById("group");
groups.forEach(g=>{let o=document.createElement("option");o.textContent=g;groupSelect.appendChild(o);});
if(localStorage.group) groupSelect.value=localStorage.group;

let showAll=true;

// =================== Графіки ===================
const schedule={
"Середа": {
"1.1":[["00:00","18:00","on"],["18:00","20:00","off"],["20:00","23:59","on"]],
"1.2":[["00:00","01:30","off"],["01:30","23:59","on"]],
"2.1":[["00:00","20:00","on"],["20:00","23:59","off"]],
"2.2":[["00:00","23:59","on"]],
"3.1":[["00:00","20:00","on"],["20:00","23:59","off"]],
"3.2":[["00:00","23:59","on"]],
"4.1":[["00:00","20:00","on"],["20:00","22:00","off"],["22:00","23:59","on"]],
"4.2":[["00:00","18:00","on"],["18:00","20:00","off"],["20:00","23:59","on"]],
"5.1":[["00:00","18:00","on"],["18:00","20:00","off"],["20:00","23:59","on"]],
"5.2":[["00:00","23:59","on"]],
"6.1":[["00:00","01:30","off"],["01:30","23:59","on"]],
"6.2":[["00:00","23:59","on"]],
}
// інші дні поки що порожні, графік формується
};

// =================== Функції ===================
function min(t){let[a,b]=t.split(":");return +a*60+ +b;}
function toggleAll(){showAll=!showAll;render();}
function pinGroup(){localStorage.group=groupSelect.value;alert("Групу закріплено ✅");}

function autoSelectDay(){
  const today=new Date().getDay();
  const map={1:"Понеділок",2:"Вівторок",3:"Середа",4:"Четвер",5:"Пʼятниця",6:"Субота",0:"Неділя"};
  daySelect.value=map[today];
}

// =================== Рендер ===================
function render(){
  const box=document.getElementById("content");
  const statusDiv=document.getElementById("status");
  box.innerHTML="";
  const day=daySelect.value;
  
  const now=new Date();
  const curM=now.getHours()*60+now.getMinutes();
  
  if(!schedule[day]){
    statusDiv.textContent="⏳ Графік ще формується";
    return;
  }
  
  const daySched=schedule[day];
  let currentStatus;
  
  function renderGroup(g){
    const s=daySched[g]; if(!s) return;
    let html=`<div class="groupCard"><b>Група ${g}</b>${localStorage.group===g?` <span class="pin">📌</span>`:""}`;
    let nextStatus="";
    s.forEach(r=>{
      const isNow=curM>=min(r[0])&&curM<=min(r[1]);
      if(isNow) nextStatus=r[2];
      html+=`<div class="row ${isNow?"now":""}"><span>${r[0]}–${r[1]}</span><span class="${r[2]}">${r[2]==="on"?"⚡ світло є":"⛔ немає"}</span></div>`;
    });
    
    // таймер
    let remaining=0;
    for(let r of s){
      const start=min(r[0]), end=min(r[1]);
      if(curM>=start&&curM<=end){remaining=end-curM; nextStatus=r[2]; break;}
      if(curM<start){remaining=start-curM; nextStatus=r[2]; break;}
    }
    const h=Math.floor(remaining/60), m=remaining%60;
    html+=`<div class="timer">⏱ До зміни: ${h}г ${m}хв</div>`;
    html+="</div>";
    box.innerHTML+=html;
    return nextStatus;
  }
  
  if(showAll){
    groups.forEach(g=>{currentStatus=renderGroup(g)});
  }else{
    currentStatus=renderGroup(groupSelect.value);
  }
  
  statusDiv.textContent=currentStatus==="on"?"🟢 ЗАРАЗ Є СВІТЛО":"⚫ ЗАРАЗ НЕМАЄ СВІТЛА";
}

// =================== Автооновлення ==================
daySelect.onchange=render;
groupSelect.onchange=render;

autoSelectDay();
render();
setInterval(render,1000); // щосекунди
</script>
</body>
</html>
