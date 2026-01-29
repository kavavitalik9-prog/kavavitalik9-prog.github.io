<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>График отключений — Львовская область</title>

<script src="https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js"></script>

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
.top{
  display:flex;
  justify-content:space-between;
  align-items:center;
  margin-bottom:6px;
}
h1{font-size:18px;margin:0}
.lock{font-size:20px;cursor:pointer}

.date{
  display:flex;
  gap:8px;
  margin:8px 0;
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

.updated{
  font-size:11px;
  opacity:.7;
  margin-bottom:6px;
}

/* ===== ТАБЛИЦА ===== */
.graph{
  background:#020617;
  border:1px solid #334155;
  border-radius:10px;
  padding:8px;
}

.hours{
  display:grid;
  grid-template-columns:48px repeat(24, 1fr);
  font-size:10px;
  margin-bottom:6px;
}
.hours div{
  text-align:center;
  opacity:.6;
}

.row{
  display:grid;
  grid-template-columns:48px repeat(24, 1fr);
  margin-bottom:3px;
}
.group{
  display:flex;
  align-items:center;
  justify-content:center;
  font-size:12px;
  background:#020617;
}
.cell{
  height:18px;
  border-radius:3px;
  background:#22c55e;
}
.cell.off{
  background:#ef4444;
}

/* ===== КНОПКИ ===== */
.actions{
  display:flex;
  gap:8px;
  margin-top:8px;
}
.actions button{
  flex:1;
  padding:8px;
  border:none;
  border-radius:6px;
  background:#1e293b;
  color:#fff;
  font-size:13px;
}

/* ===== АДМИН ===== */
.admin{
  display:none;
  margin-top:10px;
  padding:10px;
  border:1px solid #334155;
  border-radius:10px;
  background:#020617;
}
.admin input,.admin textarea,.admin button{
  width:100%;
  margin-top:6px;
  padding:8px;
  background:#020617;
  border:1px solid #334155;
  color:#fff;
  border-radius:6px;
  font-size:13px;
}
.admin label{
  font-size:12px;
  opacity:.7;
}

.footer{
  font-size:11px;
  opacity:.6;
  margin-top:6px;
}
</style>
</head>

<body>
<div class="app">

<div class="top">
  <h1>⚡ Львовская область</h1>
  <div class="lock" id="lock">🔒</div>
</div>

<div class="date">
  <button id="today">Сегодня</button>
  <button id="tomorrow">Завтра</button>
</div>

<div class="updated" id="updated"></div>

<!-- ===== ГРАФИК ===== -->
<div class="graph" id="graph"></div>

<div class="actions">
  <button id="export">📸 Экспорт PNG</button>
</div>

<!-- ===== АДМИН ===== -->
<div class="admin" id="admin">
  <b>Админка</b>
  <input type="password" id="pass" placeholder="Пароль">

  <div id="fields"></div>

  <button id="save">Сохранить</button>
</div>

<div class="footer">
🔴 нет света · 🟢 есть свет
</div>

</div>

<script>
const groups=["1.1","1.2","2.1","2.2","3.1","3.2","4.1","4.2","5.1","5.2","6.1","6.2"];
let data=JSON.parse(localStorage.getItem("power"))||{today:{},tomorrow:{}};
let last=localStorage.getItem("last")||Date.now();
let day="today";

function parse(text){
  if(!text.trim()) return [];
  return text.split("\n").map(l=>{
    const [a,b]=l.split("-");
    const [h1,m1]=a.split(":").map(Number);
    const [h2,m2]=b.split(":").map(Number);
    return [h1+(m1||0)/60,h2+(m2||0)/60];
  });
}

function timeAgo(){
  const d=(Date.now()-last)/1000;
  if(d<60) return "только что";
  if(d<3600) return Math.floor(d/60)+" мин назад";
  if(d<86400) return Math.floor(d/3600)+" ч назад";
  return Math.floor(d/86400)+" дн назад";
}

function render(){
  updated.textContent="Обновлено: "+timeAgo();
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

/* ===== ДАТА ===== */
today.onclick=()=>{day="today";today.classList.add("active");tomorrow.classList.remove("active");render();}
tomorrow.onclick=()=>{day="tomorrow";tomorrow.classList.add("active");today.classList.remove("active");render();}
today.click();

/* ===== АДМИН ===== */
lock.onclick=()=>admin.style.display=admin.style.display==="block"?"none":"block";

fields.innerHTML=groups.map(g=>`
<label>${g} — нет света</label>
<textarea id="f_${g}" rows="2" placeholder="10:00-13:00"></textarea>
`).join("");

save.onclick=()=>{
  if(pass.value!=="3709"){alert("Неверный пароль");return;}
  data[day]={};
  groups.forEach(g=>{
    const v=document.getElementById("f_"+g).value;
    const p=parse(v);
    if(p.length) data[day][g]=p;
  });
  last=Date.now();
  localStorage.setItem("power",JSON.stringify(data));
  localStorage.setItem("last",last);
  render();
  alert("Сохранено");
};

/* ===== ЭКСПОРТ ===== */
export.onclick=()=>{
  html2canvas(graph,{backgroundColor:"#020617",scale:2}).then(c=>{
    const a=document.createElement("a");
    a.download="grafik_lviv.png";
    a.href=c.toDataURL();
    a.click();
  });
};

render();
</script>
</body>
</html>
