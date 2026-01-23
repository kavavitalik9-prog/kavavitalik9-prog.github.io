<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<title>🌦 Мій прогноз</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
*{box-sizing:border-box}
body{
  margin:0;
  min-height:100vh;
  background:linear-gradient(180deg,#0b1d2b,#163a52,#1f5c7a);
  font-family:system-ui,-apple-system;
  color:#fff;
}

.app{
  max-width:420px;
  margin:auto;
  padding:14px 14px 80px;
}

.glass{
  background:rgba(255,255,255,.12);
  backdrop-filter:blur(10px);
  border-radius:22px;
  padding:16px;
  margin-bottom:14px;
  box-shadow:0 10px 30px rgba(0,0,0,.25);
}

.now{
  text-align:center;
}
.now .title{
  font-size:14px;
  opacity:.7;
}
.now .temp{
  font-size:52px;
  font-weight:800;
  margin:4px 0;
}
.now .desc{
  font-size:18px;
}
.now .time{
  font-size:13px;
  opacity:.6;
  margin-top:6px;
}

.section-title{
  font-size:16px;
  margin-bottom:8px;
  opacity:.9;
}

.scroll{
  display:flex;
  gap:10px;
  overflow-x:auto;
  padding-bottom:4px;
}

.hour-card, .day-card{
  min-width:88px;
  background:rgba(255,255,255,.15);
  border-radius:16px;
  padding:10px 8px;
  text-align:center;
  font-size:13px;
}

.hour-card .icon,
.day-card .icon{
  font-size:22px;
  margin:4px 0;
}

.hour-card .hour{
  opacity:.7;
}

.admin-btn{
  position:fixed;
  bottom:18px;
  right:18px;
  width:50px;
  height:50px;
  border-radius:50%;
  background:rgba(0,0,0,.5);
  backdrop-filter:blur(8px);
  display:flex;
  align-items:center;
  justify-content:center;
  font-size:22px;
  cursor:pointer;
  box-shadow:0 8px 20px rgba(0,0,0,.4);
}

.modal{
  position:fixed;
  inset:0;
  background:rgba(0,0,0,.6);
  display:none;
  align-items:center;
  justify-content:center;
  z-index:10;
}

.modal-box{
  width:92%;
  max-width:420px;
  background:#0f2533;
  border-radius:20px;
  padding:16px;
}

input,textarea,button{
  width:100%;
  margin-top:10px;
  padding:10px;
  border-radius:10px;
  border:none;
  font-size:14px;
}

textarea{
  background:#122f40;
  color:#fff;
}

input{
  background:#122f40;
  color:#fff;
}

button{
  background:#2ecc71;
  font-weight:700;
}

.close{
  background:#ff4d4d;
}
small{opacity:.6}
</style>
</head>

<body>

<div class="app">

  <!-- ЗАРАЗ -->
  <div class="glass now">
    <div class="title">Фейковий прогноз</div>
    <div class="temp">☀️ +10°</div>
    <div class="desc">Сонячно</div>
    <div class="time" id="clock"></div>
  </div>

  <!-- ПОГОДИННО -->
  <div class="glass">
    <div class="section-title">⏰ Погодинно</div>
    <div class="scroll" id="hourly"></div>
  </div>

  <!-- 7 ДНІВ -->
  <div class="glass">
    <div class="section-title">📅 7 днів</div>
    <div class="scroll" id="daily"></div>
  </div>

</div>

<div class="admin-btn" onclick="openAdmin()">🔒</div>

<div class="modal" id="modal">
  <div class="modal-box" id="box">
    <h3>Адмін доступ</h3>
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
29.01 Чт 12°/6° 🌧
30.01 Пт 14°/7° ☀️`;

function render(){
  const now=new Date();
  const hNow=now.getHours();

  hourly.innerHTML="";
  hourlyText.split("\n").forEach(l=>{
    const h=parseInt(l.slice(0,2));
    if(h>=hNow){
      const parts=l.split(" ");
      hourly.innerHTML+=`
        <div class="hour-card">
          <div class="hour">${parts[0]}</div>
          <div class="icon">${parts[2]}</div>
          <div>${parts[1]}</div>
        </div>`;
    }
  });

  daily.innerHTML="";
  let shown=0;
  dailyText.split("\n").forEach(l=>{
    if(shown>=7) return;
    const d=l.split(" ")[0];
    const [dd,mm]=d.split(".");
    const date=new Date(now.getFullYear(),mm-1,dd);
    if(date>=new Date(now.getFullYear(),now.getMonth(),now.getDate())){
      daily.innerHTML+=`
        <div class="day-card">
          <div>${l.split(" ").slice(0,2).join(" ")}</div>
          <div class="icon">${l.split(" ").pop()}</div>
          <div>${l.split(" ")[2]}</div>
        </div>`;
      shown++;
    }
  });
}
render();

function tick(){
  clock.textContent="Зараз: "+new Date().toLocaleTimeString("uk-UA");
}
tick(); setInterval(tick,1000);
setInterval(render,60000);

// АДМІН
function openAdmin(){modal.style.display="flex"}
function closeAdmin(){modal.style.display="none"}

function login(){
  if(pass.value!==PASSWORD) return alert("Невірний пароль");
  box.innerHTML=`
    <h3>Редагування</h3>
    <small>⏰ Погодинно</small>
    <textarea id="hEdit" rows="8">${hourlyText}</textarea>
    <small>📅 Дні</small>
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
