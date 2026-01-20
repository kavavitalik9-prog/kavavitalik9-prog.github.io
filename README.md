<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Графики отключения света — Львовская область</title>

<style>
body{
  margin:0;
  background:#0b0b0b;
  color:#fff;
  font-family:Arial, sans-serif;
}
.container{
  max-width:1000px;
  margin:auto;
  padding:15px;
}
h1{
  text-align:center;
}
.days{
  display:flex;
  gap:5px;
  overflow-x:auto;
  margin-bottom:15px;
}
.days button{
  background:#111;
  color:#fff;
  border:1px solid #333;
  padding:8px 12px;
  cursor:pointer;
}
.days button.active{
  background:#fff;
  color:#000;
  font-weight:bold;
}
.group{
  margin-bottom:15px;
  border:1px solid #333;
  padding:10px;
}
.group h3{
  margin:0 0 8px 0;
}
.slot{
  display:flex;
  justify-content:space-between;
  padding:6px;
  border-bottom:1px solid #222;
}
.off{ color:#ff4d4d; }
.on{ color:#4dff88; }
.footer{
  margin-top:20px;
  opacity:.6;
  text-align:center;
}
</style>
</head>

<body>

<div class="container">
  <h1>⚡ Графики отключения света<br>Львовская область</h1>

  <div class="days" id="days"></div>
  <div id="schedule"></div>

  <div class="footer">
    Данные ознакомительные • Львовская область
  </div>
</div>

<script>
// ===== ГРУППЫ =====
const groups = [
  "1.1","1.2","2.1","2.2",
  "3.1","3.2","4.1","4.2",
  "5.1","5.2","6.1","6.2"
];

// ===== ДАННЫЕ (ШАБЛОН) =====
const week = {
  "Понедельник": {},
  "Вторник": {},
  "Среда": {},
  "Четверг": {},
  "Пятница": {},
  "Суббота": {},
  "Воскресенье": {}
};

// заполняем одинаковым шаблоном все группы
for(const day in week){
  groups.forEach(g=>{
    week[day][`Группа ${g}`] = [
      ["00:00–04:00","off"],
      ["04:00–08:00","on"],
      ["08:00–12:00","off"],
      ["12:00–16:00","on"],
      ["16:00–20:00","off"],
      ["20:00–24:00","on"]
    ];
  });
}

// ===== ЛОГИКА =====
const daysDiv = document.getElementById("days");
const scheduleDiv = document.getElementById("schedule");

const days = Object.keys(week);
let currentDay = days[0];

days.forEach(day=>{
  const btn = document.createElement("button");
  btn.textContent = day;
  btn.onclick = ()=>{
    currentDay = day;
    document.querySelectorAll(".days button")
      .forEach(b=>b.classList.remove("active"));
    btn.classList.add("active");
    render();
  };
  if(day === currentDay) btn.classList.add("active");
  daysDiv.appendChild(btn);
});

function render(){
  scheduleDiv.innerHTML = "";
  const dayData = week[currentDay];

  for(const group in dayData){
    const box = document.createElement("div");
    box.className = "group";
    box.innerHTML = `<h3>${group}</h3>`;

    dayData[group].forEach(s=>{
      const row = document.createElement("div");
      row.className = "slot " + (s[1]==="off"?"off":"on");
      row.innerHTML =
        `<span>${s[0]}</span>
         <span>${s[1]==="off"?"Нет света":"Есть свет"}</span>`;
      box.appendChild(row);
    });

    scheduleDiv.appendChild(box);
  }
}

render();
</script>

</body>
</html>
