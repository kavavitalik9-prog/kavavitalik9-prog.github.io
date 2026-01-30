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
  display:flex;
  justify-content:center;
}
.app{
  width:100%;
  max-width:430px;
  min-height:100vh;
  padding:12px;
}
h1{
  font-size:18px;
  margin:0 0 8px 0;
}
.date{
  display:flex;
  gap:8px;
  margin-bottom:10px;
}
.date button{
  flex:1;
  padding:8px;
  border:none;
  border-radius:6px;
  background:#1e293b;
  color:#fff;
}
.date button.active{background:#2563eb}

/* ===== ТАБЛИЦА ===== */
.graph{
  border:1px solid #334155;
  border-radius:10px;
  padding:8px;
}
.hours{
  display:grid;
  grid-template-columns:48px repeat(24,1fr);
  font-size:10px;
  opacity:.6;
  margin-bottom:6px;
}
.hours div{text-align:center}
.row{
  display:grid;
  grid-template-columns:48px repeat(24,1fr);
  margin-bottom:3px;
}
.group{
  display:flex;
  align-items:center;
  justify-content:center;
  font-size:12px;
}
.cell{
  height:18px;
  border-radius:3px;
  background:#22c55e; /* свет есть */
}
.cell.off{
  background:#ef4444; /* света нет */
}
.footer{
  font-size:11px;
  opacity:.6;
  margin-top:8px;
}
</style>
</head>

<body>
<div class="app">

<h1>⚡ Львовская область</h1>

<div class="date">
  <button id="today">Сегодня</button>
  <button id="tomorrow">Завтра</button>
</div>

<div class="graph" id="graph"></div>

<div class="footer">
🔴 нет света · 🟢 есть свет
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
tomorrow:{}
};

let day="today";

function render(){
  graph.innerHTML="";
  const h=document.createElement("div");
  h.className="hours";
  h.innerHTML="<div></div>";
  for(let i=0;i<24;i++) h.innerHTML+=`<div>${i}</div>`;
  graph.appendChild(h);

  groups.forEach(g=>{
    const row=document.createElement("div");
    row.className="row";
    row.innerHTML=`<div class="group">${g}</div>`;
    for(let i=0;i<24;i++){
      let off=false;
      (data[day][g]||[]).forEach(p=>{
        if(i>=p[0] && i<p[1]) off=true;
      });
      row.innerHTML+=`<div class="cell ${off?"off":""}"></div>`;
    }
    graph.appendChild(row);
  });
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
