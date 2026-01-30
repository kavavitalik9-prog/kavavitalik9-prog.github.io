<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>График отключений</title>

<style>
*{box-sizing:border-box}
body{
  margin:0;
  background:#020617;
  font-family:system-ui,sans-serif;
  color:#fff;
}
.app{
  max-width:430px;
  margin:auto;
  padding:12px;
}
h1{font-size:18px;margin:0 0 4px}

.updated{
  font-size:12px;
  opacity:.7;
  margin-bottom:8px;
}

.tabs{
  display:flex;
  gap:8px;
  margin-bottom:10px;
}
.tabs button{
  flex:1;
  padding:8px;
  border:none;
  border-radius:6px;
  background:#1e293b;
  color:#fff;
}
.tabs button.active{background:#2563eb}

/* ===== КАРТОЧКИ ===== */
.cards{display:grid;gap:8px;margin-bottom:12px}
.card{
  border:1px solid #334155;
  border-radius:10px;
  padding:8px;
}
.card h3{margin:0 0 4px;font-size:14px}
.line{font-size:13px}
.on{color:#22c55e}
.off{color:#ef4444}
.timer{font-size:12px;opacity:.8;margin-top:4px}

/* ===== ТАБЛИЦА ===== */
.wrap{overflow:auto}
.table{
  display:grid;
  grid-template-columns:50px repeat(24,1fr) 50px;
  min-width:900px;
}
.head,.side{
  font-size:10px;
  text-align:center;
  opacity:.7;
}
.cell{
  height:14px;
  border-radius:3px;
  background:#22c55e;
}
.cell.off{background:#ef4444}

.footer{
  font-size:11px;
  opacity:.6;
  margin-top:8px;
}
</style>
</head>

<body>
<div class="app">

<h1>⚡ График отключений</h1>
<div class="updated" id="updated"></div>

<div class="tabs">
  <button id="today">Сегодня</button>
  <button id="tomorrow">Завтра</button>
</div>

<div class="cards" id="cards"></div>

<div class="wrap">
  <div class="table" id="table"></div>
</div>

<div class="footer">🟢 есть свет · 🔴 нет света</div>

</div>

<script>
const groups=["1.1","1.2","2.1","2.2","3.1","3.2","4.1","4.2","5.1","5.2","6.1","6.2"];

/* ===== ДАННЫЕ (в минутах) ===== */
const todayData={
 "1.1":[],
 "1.2":[],
 "2.1":[[19*60,20*60]],
 "2.2":[[0,3*60]],
 "3.1":[[22*60,24*60]],
 "3.2":[],
 "4.1":[],
 "4.2":[],
 "5.1":[[0,3*60]],
 "5.2":[[19*60,20*60]],
 "6.1":[],
 "6.2":[[22*60,24*60]]
};

const tomorrowData={
 "1.1":[[8*60,11*60]],
 "1.2":[[14*60+30,18*60]],
 "2.1":[[0,4*60]],
 "2.2":[[11*60,14*60+30]],
 "3.1":[[21*60+30,24*60]],
 "3.2":[[18*60,21*60+30]],
 "4.1":[[21*60+30,24*60]],
 "4.2":[[4*60,7*60+30]],
 "5.1":[[14*60+30,18*60]],
 "5.2":[[7*60+30,11*60]],
 "6.1":[[11*60,14*60+30]],
 "6.2":[[18*60,21*60+30]]
};

let mode="today";

/* ===== ПОСЛЕДНЕЕ ОБНОВЛЕНИЕ ===== */
const lastUpdate=new Date();
lastUpdate.setHours(19,56,0,0);

function updateText(){
  const now=new Date();
  let diff=Math.floor((now-lastUpdate)/1000);
  if(diff<0) diff=0;

  const d=Math.floor(diff/86400);
  diff%=86400;
  const h=Math.floor(diff/3600);
  diff%=3600;
  const m=Math.floor(diff/60);

  let t="Последнее обновление: ";
  if(d>0) t+=`${d} дн `;
  if(h>0||d>0) t+=`${h} ч `;
  t+=`${m} мин назад`;
  updated.textContent=t;
}

function dataSet(){
  return mode==="today"?todayData:tomorrowData;
}

function isOff(g,min){
  return (dataSet()[g]||[]).some(p=>min>=p[0]&&min<p[1]);
}

/* ===== КАРТОЧКИ ===== */
function renderCards(){
  cards.innerHTML="";
  const now=new Date();
  const cur=now.getHours()*60+now.getMinutes();

  groups.forEach(g=>{
    const off=dataSet()[g]||[];
    const c=document.createElement("div");
    c.className="card";
    c.innerHTML=`<h3>${g}</h3>`;

    if(off.length===0){
      c.innerHTML+=`<div class="line on">🟢 00:00–23:59</div>`;
    }else{
      let last=0;
      off.forEach(p=>{
        if(last<p[0])
          c.innerHTML+=`<div class="line on">🟢 ${fmt(last)}–${fmt(p[0])}</div>`;
        c.innerHTML+=`<div class="line off">🔴 ${fmt(p[0])}–${fmt(p[1])}</div>`;
        last=p[1];
      });
      if(last<1440)
        c.innerHTML+=`<div class="line on">🟢 ${fmt(last)}–23:59</div>`;
    }

    if(mode==="today"){
      let next=null;
      off.forEach(p=>{
        if(cur<p[0]) next=p[0];
        if(cur>=p[0]&&cur<p[1]) next=p[1];
      });
      if(next!==null){
        const diff=next-cur;
        c.innerHTML+=`<div class="timer">⏱ ${Math.floor(diff/60)} ч ${diff%60} мин</div>`;
      }
    }

    cards.appendChild(c);
  });
}

/* ===== ТАБЛИЦА (по часам) ===== */
function renderTable(){
  table.innerHTML="";
  table.innerHTML+=`<div></div>`;
  for(let i=0;i<24;i++) table.innerHTML+=`<div class="head">${i}</div>`;
  table.innerHTML+=`<div></div>`;

  groups.forEach(g=>{
    table.innerHTML+=`<div class="side">${g}</div>`;
    for(let h=0;h<24;h++){
      const off=isOff(g,h*60+30);
      table.innerHTML+=`<div class="cell ${off?"off":""}"></div>`;
    }
    table.innerHTML+=`<div class="side">${g}</div>`;
  });

  table.innerHTML+=`<div></div>`;
  for(let i=0;i<24;i++) table.innerHTML+=`<div class="head">${i}</div>`;
  table.innerHTML+=`<div></div>`;
}

function fmt(m){
  return String(Math.floor(m/60)).padStart(2,"0")+":"+
         String(m%60).padStart(2,"0");
}

function render(){
  renderCards();
  renderTable();
}

today.onclick=()=>{
  mode="today";
  today.classList.add("active");
  tomorrow.classList.remove("active");
  render();
};
tomorrow.onclick=()=>{
  mode="tomorrow";
  tomorrow.classList.add("active");
  today.classList.remove("active");
  render();
};

today.click();
updateText();
setInterval(updateText,60000);
</script>
</body>
</html>
