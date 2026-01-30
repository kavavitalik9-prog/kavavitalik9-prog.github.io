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
h1{font-size:18px;margin:0 0 8px}

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
.row-gap{height:4px}
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
const groups=["1.1","1.2","2.1","3.1","3.2","4.1","4.2","5.1","5.2","6.1","6.2"];

const todayData={
  "1.1":[[19,22]],
  "1.2":[],
  "2.1":[[19,21]],
  "3.1":[[21,24]],
  "3.2":[],
  "4.1":[],
  "4.2":[],
  "5.1":[[0,3]],
  "5.2":[[19,21]],
  "6.1":[],
  "6.2":[[21,24]]
};

let mode="today";

function getStatus(g,h){
  return (todayData[g]||[]).some(p=>h>=p[0]&&h<p[1]);
}

function renderCards(){
  cards.innerHTML="";
  if(mode==="tomorrow"){
    cards.innerHTML=`<div class="card">⏳ <b>ГРАФИК ЕЩЕ ФОРМИРУЕТСЯ</b></div>`;
    return;
  }

  const now=new Date();
  const h=now.getHours();
  const m=now.getMinutes();

  groups.forEach(g=>{
    const offRanges=todayData[g]||[];
    let status=getStatus(g,h);
    let next=null;

    offRanges.forEach(p=>{
      if(status && h<p[1]) next=p[1];
      if(!status && h<p[0]) next=p[0];
    });

    const c=document.createElement("div");
    c.className="card";
    c.innerHTML=`<h3>${g}</h3>`;

    offRanges.forEach(p=>{
      c.innerHTML+=`<div class="line off">🔴 ${p[0]}:00–${p[1]}:00</div>`;
    });
    if(!offRanges.length){
      c.innerHTML+=`<div class="line on">🟢 00:00–24:00</div>`;
    }

    if(next!==null){
      const diff=(next*60)-(h*60+m);
      const hh=Math.floor(diff/60);
      const mm=diff%60;
      c.innerHTML+=`<div class="timer">⏱ ${hh}ч ${mm}м</div>`;
    }

    cards.appendChild(c);
  });
}

function renderTable(){
  table.innerHTML="";

  // верхнее время
  table.innerHTML+=`<div></div>`;
  for(let i=0;i<24;i++) table.innerHTML+=`<div class="head">${i}</div>`;
  table.innerHTML+=`<div></div>`;

  groups.forEach(g=>{
    table.innerHTML+=`<div class="side">${g}</div>`;
    for(let i=0;i<24;i++){
      const off=mode==="today" && getStatus(g,i);
      table.innerHTML+=`<div class="cell ${off?"off":""}"></div>`;
    }
    table.innerHTML+=`<div class="side">${g}</div>`;
  });

  // нижнее время
  table.innerHTML+=`<div></div>`;
  for(let i=0;i<24;i++) table.innerHTML+=`<div class="head">${i}</div>`;
  table.innerHTML+=`<div></div>`;
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
setInterval(renderCards,60000);
</script>
</body>
</html>
