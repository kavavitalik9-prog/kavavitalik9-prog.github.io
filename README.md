<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Графики отключения света — Львовская область</title>

<style>
*{box-sizing:border-box}
body{
  margin:0;
  font-family:system-ui, sans-serif;
  background:#0f172a;
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
}
h1{
  font-size:18px;
  margin:0;
}
.lock{
  font-size:20px;
  cursor:pointer;
}
.date-select{
  display:flex;
  gap:8px;
  margin:10px 0;
}
.date-select button{
  flex:1;
  padding:8px;
  background:#1e293b;
  color:#fff;
  border:none;
  border-radius:6px;
}
.date-select button.active{
  background:#2563eb;
}
.updated{
  font-size:11px;
  opacity:.7;
  margin-bottom:6px;
}
.table{
  overflow-x:auto;
}
.row{
  display:grid;
  grid-template-columns:52px repeat(24, 1fr);
  margin-bottom:4px;
}
.group{
  font-size:12px;
  display:flex;
  align-items:center;
  justify-content:center;
  background:#020617;
}
.hour{
  height:18px;
  background:#22c55e;
}
.hour.off{
  background:#ef4444;
}
.hours{
  display:grid;
  grid-template-columns:52px repeat(24, 1fr);
  font-size:10px;
  margin-bottom:6px;
}
.hours div{
  text-align:center;
  opacity:.6;
}
.admin{
  display:none;
  background:#020617;
  border:1px solid #334155;
  border-radius:8px;
  padding:10px;
  margin-top:10px;
}
.admin input, .admin textarea, .admin button{
  width:100%;
  margin-top:6px;
  padding:8px;
  background:#0f172a;
  color:#fff;
  border:1px solid #334155;
  border-radius:6px;
  font-size:13px;
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

<div class="top">
  <h1>⚡ Львовская область</h1>
  <div class="lock" id="lockBtn">🔒</div>
</div>

<div class="date-select">
  <button id="todayBtn">Сегодня</button>
  <button id="tomorrowBtn">Завтра</button>
</div>

<div class="updated" id="updated"></div>

<div class="table" id="table"></div>

<div class="admin" id="admin">
  <b>Админка</b>
  <input type="password" id="pass" placeholder="Пароль">

  <textarea id="editor" rows="8"
placeholder="Формат:
1.1=7-11,14-18,21-24
1.2=0-4,7-11
..."></textarea>

  <button id="saveAdmin">Сохранить график</button>
</div>

<div class="footer">
🔴 нет света · 🟢 есть свет
</div>

</div>

<script>
// ===== ДАННЫЕ =====
const defaultData={
today:{
"1.1":[[7,11],[14,18],[21,24]],
"1.2":[[0,4],[7,11],[14,18],[21,24]],
"2.1":[[4,7],[11,14],[18,22]],
"2.2":[[0,4],[7,11],[14,18],[21,24]],
"3.1":[[0,4],[11,14],[18,21]],
"3.2":[[4,7],[11,14],[18,21]],
"4.1":[[4,7],[11,14],[18,21]],
"4.2":[[0,4],[7,11],[14,18],[21,24]],
"5.1":[[0,4],[7,11],[14,18]],
"5.2":[[4,7],[11,14],[18,21]],
"6.1":[[4,7],[11,14],[18,21]],
"6.2":[[6,11],[14,18],[21,24]]
},
tomorrow:{}
};

let data=JSON.parse(localStorage.getItem("powerData"))||defaultData;
let lastUpdate=localStorage.getItem("lastUpdate")||Date.now();
let day="today";

// ===== ВРЕМЯ =====
function timeAgo(){
  const diff=Math.floor((Date.now()-lastUpdate)/1000);
  if(diff<60) return "только что";
  if(diff<3600) return Math.floor(diff/60)+" мин назад";
  if(diff<86400) return Math.floor(diff/3600)+" ч назад";
  return Math.floor(diff/86400)+" дн назад";
}

// ===== РЕНДЕР =====
const table=document.getElementById("table");
function render(){
  table.innerHTML="";
  document.getElementById("updated").textContent="Обновлено: "+timeAgo();

  const hours=document.createElement("div");
  hours.className="hours";
  hours.innerHTML="<div></div>";
  for(let i=0;i<24;i++) hours.innerHTML+=`<div>${i}</div>`;
  table.appendChild(hours);

  Object.keys(data[day]).forEach(g=>{
    const row=document.createElement("div");
    row.className="row";
    row.innerHTML=`<div class="group">${g}</div>`;
    for(let h=0;h<24;h++){
      let off=false;
      data[day][g].forEach(p=>{
        if(h>=p[0] && h<p[1]) off=true;
      });
      row.innerHTML+=`<div class="hour ${off?"off":""}"></div>`;
    }
    table.appendChild(row);
  });
}

// ===== ДАТЫ =====
todayBtn.onclick=()=>{
day="today";
todayBtn.classList.add("active");
tomorrowBtn.classList.remove("active");
render();
};
tomorrowBtn.onclick=()=>{
day="tomorrow";
tomorrowBtn.classList.add("active");
todayBtn.classList.remove("active");
render();
};
todayBtn.click();

// ===== АДМИН =====
lockBtn.onclick=()=>{
document.getElementById("admin").style.display=
document.getElementById("admin").style.display==="block"?"none":"block";
};

saveAdmin.onclick=()=>{
if(pass.value!=="3709"){alert("Неверный пароль");return;}
const lines=editor.value.trim().split("\n");
const obj={};
lines.forEach(l=>{
const [g,v]=l.split("=");
if(!g||!v) return;
obj[g]=v.split(",").map(p=>p.split("-").map(Number));
});
data[day]=obj;
lastUpdate=Date.now();
localStorage.setItem("powerData",JSON.stringify(data));
localStorage.setItem("lastUpdate",lastUpdate);
render();
alert("Сохранено");
};
render();
</script>
</body>
</html>
