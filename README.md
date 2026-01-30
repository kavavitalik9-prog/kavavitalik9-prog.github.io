<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>График отключений — Львовская область</title>

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
.cards{
  display:grid;
  gap:8px;
}
.card{
  border:1px solid #334155;
  border-radius:10px;
  padding:8px;
}
.card h3{
  margin:0 0 4px;
  font-size:14px;
}
.line{
  font-size:13px;
}
.on{color:#22c55e}
.off{color:#ef4444}

/* ===== ТАБЛИЦА ===== */
.table-wrap{
  margin-top:16px;
  overflow-x:auto;
}
.table{
  border:1px solid #334155;
  border-radius:10px;
  padding:8px;
  min-width:600px;
}
.header, .row{
  display:grid;
  grid-template-columns:40px repeat(11,1fr);
  font-size:10px;
}
.header div{
  text-align:center;
  opacity:.6;
}
.row{
  margin-top:2px;
}
.hour{
  text-align:center;
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
  margin-top:10px;
}
</style>
</head>

<body>
<div class="app">

<h1>⚡ Львовская область</h1>

<div class="tabs">
  <button id="today">Сегодня</button>
  <button id="tomorrow">Завтра</button>
</div>

<div class="cards" id="cards"></div>

<div class="table-wrap">
  <div class="table" id="table"></div>
</div>

<div class="footer">
🟢 есть свет · 🔴 нет света
</div>

</div>

<script>
const groups=["1.1","1.2","2.1","3.1","3.2","4.1","4.2","5.1","5.2","6.1","6.2"];

const data={
today:{
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
},
tomorrow:{} // можно будет заполнить
};

let day="today";

function ranges(off){
  let res=[],last=0;
  off.forEach(p=>{
    if(last<p[0]) res.push({t:"on",a:last,b:p[0]});
    res.push({t:"off",a:p[0],b:p[1]});
    last=p[1];
  });
  if(last<24) res.push({t:"on",a:last,b:24});
  return res;
}

function renderCards(){
  cards.innerHTML="";
  groups.forEach(g=>{
    const c=document.createElement("div");
    c.className="card";
    c.innerHTML=`<h3>${g}</h3>`;
    ranges(data[day][g]||[]).forEach(r=>{
      c.innerHTML+=`
        <div class="line ${r.t}">
          ${r.t==="on"?"🟢":"🔴"} ${String(r.a).padStart(2,"0")}:00–${String(r.b).padStart(2,"0")}:00
        </div>`;
    });
    cards.appendChild(c);
  });
}

function renderTable(){
  table.innerHTML="";
  const h=document.createElement("div");
  h.className="header";
  h.innerHTML="<div></div>";
  groups.forEach(g=>h.innerHTML+=`<div>${g}</div>`);
  table.appendChild(h);

  for(let i=0;i<24;i++){
    const r=document.createElement("div");
    r.className="row";
    r.innerHTML=`<div class="hour">${i}</div>`;
    groups.forEach(g=>{
      let off=false;
      (data[day][g]||[]).forEach(p=>{
        if(i>=p[0] && i<p[1]) off=true;
      });
      r.innerHTML+=`<div class="cell ${off?"off":""}"></div>`;
    });
    table.appendChild(r);
  }
}

function render(){
  renderCards();
  renderTable();
}

today.onclick=()=>{
  day="today";
  today.classList.add("active");
  tomorrow.classList.remove("active");
  render();
};
tomorrow.onclick=()=>{
  day="tomorrow";
  tomorrow.classList.add("active");
  today.classList.remove("active");
  render();
};

today.click();
</script>
</body>
</html>
