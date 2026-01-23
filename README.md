<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<title>🌦 Мій прогноз погоди</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
body{
  margin:0;
  background:#0f2027;
  font-family:system-ui;
  color:#fff;
}
.app{max-width:390px;margin:auto;padding:12px}
.card{
  background:#1b2a33;
  border-radius:16px;
  padding:12px;
  margin-bottom:12px;
}
h3{margin:0 0 8px}
.now{text-align:center}
.temp{font-size:42px;font-weight:700}
.time{opacity:.7;font-size:13px}

.list{
  display:flex;
  gap:8px;
  overflow-x:auto;
}
.item{
  min-width:90px;
  background:#243843;
  border-radius:12px;
  padding:8px;
  text-align:center;
  font-size:13px;
}

.admin-btn{
  position:fixed;
  bottom:14px;
  right:14px;
  background:#000a;
  width:46px;
  height:46px;
  border-radius:50%;
  display:flex;
  align-items:center;
  justify-content:center;
  font-size:22px;
  cursor:pointer;
}

.modal{
  position:fixed;
  inset:0;
  background:#000b;
  display:none;
  align-items:center;
  justify-content:center;
}
.box{
  background:#162229;
  padding:14px;
  border-radius:14px;
  width:92%;
}
textarea,input,button{
  width:100%;
  margin-top:8px;
  padding:8px;
  border-radius:8px;
  border:none;
}
button{background:#2ecc71;font-weight:600}
.close{background:#ff4d4d}
</style>
</head>

<body>

<div class="app">

<div class="card now">
  <div class="temp" id="nowTemp">+10° ☀️</div>
  <div id="nowDesc">Сонячно</div>
  <div class="time" id="clock"></div>
</div>

<div class="card">
  <h3>⏰ Погодинно</h3>
  <div class="list" id="hourly"></div>
</div>

<div class="card">
  <h3>📅 Прогноз</h3>
  <div class="list" id="daily"></div>
</div>

</div>

<div class="admin-btn" onclick="openAdmin()">🔒</div>

<div class="modal" id="modal">
  <div class="box" id="box">
    <h3>Адмін</h3>
    <input id="pass" placeholder="Пароль">
    <button onclick="login()">Увійти</button>
    <button class="close" onclick="closeAdmin()">Закрити</button>
  </div>
</div>

<script>
const PASSWORD="3709";

let hourlyText=`00:00 10° ☀️
01:00 9° 🌙
02:00 9° 🌙
03:00 8° 🌙
04:00 8° 🌙
05:00 9° 🌤
06:00 10° 🌤
07:00 12° ☀️
08:00 14° ☀️
09:00 16° ☀️
10:00 18° ☀️
11:00 19° ☀️
12:00 20° ☀️
13:00 20° ☀️
14:00 19° 🌤
15:00 18° 🌤
16:00 17° 🌤
17:00 16° 🌤
18:00 15° 🌙
19:00 14° 🌙
20:00 13° 🌙
21:00 12° 🌙
22:00 11° 🌙
23:00 10° 🌙`;

let dailyText=`23.01 Пт 12°/5° ☀️
24.01 Сб 10°/3° 🌧
25.01 Нд 8°/2° ❄️
26.01 Пн 9°/3° 🌤
27.01 Вт 11°/4° ☀️
28.01 Ср 13°/5° ☀️
29.01 Чт 12°/6° 🌧`;

function render(){
  hourly.innerHTML="";
  hourlyText.split("\n").forEach(l=>{
    hourly.innerHTML+=`<div class="item">${l}</div>`;
  });
  daily.innerHTML="";
  dailyText.split("\n").forEach(l=>{
    daily.innerHTML+=`<div class="item">${l}</div>`;
  });
}
render();

// ЧАС
function clockTick(){
  clock.textContent="Зараз: "+new Date().toLocaleTimeString("uk-UA");
}
clockTick();
setInterval(clockTick,1000);

// АДМІН
function openAdmin(){modal.style.display="flex"}
function closeAdmin(){modal.style.display="none"}

function login(){
  if(pass.value!==PASSWORD) return alert("Невірний пароль");
  box.innerHTML=`
    <h3>Редагування</h3>
    <small>Погодинно (1 рядок = 1 година)</small>
    <textarea id="hEdit" rows="8">${hourlyText}</textarea>
    <small>Дні</small>
    <textarea id="dEdit" rows="6">${dailyText}</textarea>
    <button onclick="save()">Зберегти</button>
    <button class="close" onclick="closeAdmin()">Закрити</button>`;
}

function save(){
  hourlyText=hEdit.value.trim();
  dailyText=dEdit.value.trim();
  render();
  closeAdmin();
}
</script>

</body>
</html>
