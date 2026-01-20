<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<title>Графіки відключення світла — Львівська область</title>

<style>
body{
  margin:0;
  background:#0b0b0b;
  color:#ffffff;
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
  gap:6px;
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

.empty{
  text-align:center;
  padding:30px;
  font-size:18px;
  opacity:0.8;
}

.footer{
  margin-top:20px;
  opacity:.6;
  text-align:center;
  font-size:14px;
}
</style>
</head>

<body>

<div class="container">
  <h1>⚡ Графіки відключення світла<br>Львівська область</h1>

  <div class="days" id="days"></div>
  <div id="schedule"></div>

  <div class="footer">
    Інформація має ознайомчий характер
  </div>
</div>

<script>
// ===== ГРУПИ =====
const groups = [
  "1.1","1.2","2.1","2.2",
  "3.1","3.2","4.1","4.2",
  "5.1","5.2","6.1","6.2"
];

// ===== ГРАФІКИ =====
// Якщо день порожній {} → буде "Графік ще формується"
const week = {
  "Понеділок": {},
  "Вівторок": {},
  "Середа": {},
  "Четвер": {},
  "Пʼятниця": {},
  "Субота": {},
  "Неділя": {}
};

// приклад: заповнимо ПОНЕДІЛОК
groups.forEach(g=>{
  week["Понеділок"][`Група ${g}`] = [
    ["00:00–04:00","off"],
    ["04:00–08:00","on"],
    ["08:00–12:00","off"],
    ["12:00–16:00","on"],
    ["16:00–20:00","off"],
    ["20:00–24:00","on"]
  ];
});

// ===== ЛОГІКА =====
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

  // ❗ якщо графіків немає
  if(Object.keys(dayData).length === 0){
    scheduleDiv.innerHTML =
      `<div class="empty">⏳ Графік ще формується</div>`;
    return;
  }

  for(const group in dayData){
    const box = document.createElement("div");
    box.className = "group";
    box.innerHTML = `<h3>${group}</h3>`;

    dayData[group].forEach(s=>{
      const row = document.createElement("div");
      row.className = "slot " + (s[1]==="off"?"off":"on");
      row.innerHTML =
        `<span>${s[0]}</span>
         <span>${s[1]==="off"?"Немає світла":"Є світло"}</span>`;
      box.appendChild(row);
    });

    scheduleDiv.appendChild(box);
  }
}

render();
</script>

</body>
</html>
