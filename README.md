<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Графики отключения света — Львовская область</title>

<style>
body{
  margin:0;
  font-family:system-ui, sans-serif;
  background:#0f172a;
  color:#fff;
  display:flex;
  justify-content:center;
}
.app{
  width:390px;
  min-height:100vh;
  padding:12px;
}
h1{
  font-size:18px;
  margin:0 0 8px;
}
.date-select{
  display:flex;
  gap:8px;
  margin-bottom:10px;
}
.date-select button{
  flex:1;
  padding:8px;
  background:#1e293b;
  color:#fff;
  border:none;
  border-radius:6px;
  cursor:pointer;
}
.date-select button.active{
  background:#2563eb;
}
.table{
  overflow-x:auto;
}
.row{
  display:grid;
  grid-template-columns:50px repeat(24, 1fr);
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
  grid-template-columns:50px repeat(24, 1fr);
  font-size:10px;
  margin-bottom:6px;
}
.hours div{
  text-align:center;
  opacity:.6;
}
.footer{
  margin-top:10px;
  font-size:11px;
  opacity:.6;
}
</style>
</head>

<body>
<div class="app">

<h1>⚡ Львовская область</h1>

<div class="date-select">
  <button id="todayBtn">Сегодня</button>
  <button id="tomorrowBtn">Завтра</button>
</div>

<div class="table" id="table"></div>

<div class="footer">
  🔴 нет света · 🟢 есть свет
</div>

</div>

<script>
// ===== ДАННЫЕ =====
const schedules = {
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
  tomorrow:{
    "1.1":[[8,12],[15,19]],
    "1.2":[[1,5],[8,12],[15,19]],
    "2.1":[[5,8],[12,15],[19,23]],
    "2.2":[[1,5],[8,12],[15,19]],
    "3.1":[[1,5],[12,15],[19,22]],
    "3.2":[[5,8],[12,15],[19,22]],
    "4.1":[[5,8],[12,15],[19,22]],
    "4.2":[[1,5],[8,12],[15,19]],
    "5.1":[[1,5],[8,12],[15,19]],
    "5.2":[[5,8],[12,15],[19,22]],
    "6.1":[[5,8],[12,15],[19,22]],
    "6.2":[[7,12],[15,19]]
  }
};

const table=document.getElementById("table");
const todayBtn=document.getElementById("todayBtn");
const tomorrowBtn=document.getElementById("tomorrowBtn");

// ===== РЕНДЕР =====
function render(day){
  table.innerHTML="";

  // Часы
  const hours=document.createElement("div");
  hours.className="hours";
  hours.innerHTML="<div></div>";
  for(let i=0;i<24;i++){
    hours.innerHTML+=`<div>${i}</div>`;
  }
  table.appendChild(hours);

  Object.keys(schedules[day]).forEach(group=>{
    const row=document.createElement("div");
    row.className="row";
    row.innerHTML=`<div class="group">${group}</div>`;

    for(let h=0;h<24;h++){
      let off=false;
      schedules[day][group].forEach(p=>{
        if(h>=p[0] && h<p[1]) off=true;
      });
      row.innerHTML+=`<div class="hour ${off?"off":""}"></div>`;
    }
    table.appendChild(row);
  });
}

// ===== КНОПКИ =====
todayBtn.onclick=()=>{
  todayBtn.classList.add("active");
  tomorrowBtn.classList.remove("active");
  render("today");
};
tomorrowBtn.onclick=()=>{
  tomorrowBtn.classList.add("active");
  todayBtn.classList.remove("active");
  render("tomorrow");
};

// ===== АВТО СЕГОДНЯ =====
todayBtn.click();
</script>

</body>
</html>
