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
  background:linear-gradient(180deg,#0b1d2b,#163a52,#1f5c7a);
  font-family:system-ui;
  color:#fff;
}
.app{max-width:420px;margin:auto;padding:14px 14px 80px}
.glass{
  background:rgba(255,255,255,.14);
  backdrop-filter:blur(12px);
  border-radius:22px;
  padding:16px;
  margin-bottom:14px;
}
.now{text-align:center}
.now .temp{font-size:50px;font-weight:800}
.now .desc{font-size:18px}
.time{opacity:.6;font-size:13px}
.title{font-size:16px;margin-bottom:8px}
.scroll{display:flex;gap:10px;overflow-x:auto}
.card{
  min-width:90px;
  background:rgba(255,255,255,.18);
  border-radius:16px;
  padding:10px;
  text-align:center;
  font-size:13px;
}
.icon{font-size:22px;margin:4px 0}

.admin-btn{
  position:fixed;
  bottom:18px;
  right:18px;
  width:50px;
  height:50px;
  border-radius:50%;
  background:rgba(0,0,0,.5);
  display:flex;
  align-items:center;
  justify-content:center;
  font-size:22px;
  cursor:pointer;
}

.modal{
  position:fixed;
  inset:0;
  background:rgba(0,0,0,.6);
  display:none;
  align-items:center;
  justify-content:center;
}
.box{
  background:#0f2533;
  border-radius:20px;
  padding:16px;
  width:92%;
  max-width:420px;
}
textarea,input,button{
  width:100%;
  margin-top:10px;
  padding:10px;
  border-radius:10px;
  border:none;
  font-size:14px;
}
textarea,input{background:#123448;color:#fff}
button{background:#2ecc71;font-weight:700}
.close{background:#ff4d4d}
small{opacity:.6}
</style>
</head>

<body>

<div class="app">

<!-- ЗАРАЗ -->
<div class="glass now">
  <div class="temp" id="nowTemp">--</div>
  <div class="desc" id="nowDesc">Зараз</div>
  <div class="time" id="clock"></div>
</div>

<!-- ПОГОДИННО -->
<div class="glass">
  <div class="title">⏰ Погодні години</div>
  <div class="scroll" id="hourly"></div>
</div>

<!-- 7 ДНІВ -->
<div class="glass">
  <div class="title">📅 7 днів</div>
  <div class="scroll" id="daily"></div>
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

// ===== ПОГОДА ПО ДНЯХ (24 ГОДИНИ) =====
let hourlyByDay=`
[2026-01-23]
00:00 10° 🌙
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
23:00 10° 🌙
`;

// ===== 7 ДНІВ =====
let dailyText=`23.01 Пт 12°/5° ☀️
24.01 Сб 10°/3° 🌧
25.01 Нд 8°/2° ❄️
26.01 Пн 9°/3° 🌤
27.01 Вт 11°/4° ☀️
28.01 Ср 13°/5° ☀️
29.01 Чт 12°/6° 🌧
30.01 Пт 14°/7° ☀️`;

function parseHourly(){
  const blocks=hourlyByDay.split(/\n(?=\[)/);
  const map={};
  blocks.forEach(b=>{
    const lines=b.trim().split("\n");
    if(!lines[0]) return;
    const date=lines[0].replace(/\[|\]/g,"");
    map[date]=lines.slice(1);
  });
  return map;
}

function render(){
  const now=new Date();
  const today=now.toISOString().slice(0,10);
  const hour=now.getHours();
  const data=parseHourly();
  const todayHours=data[today]||[];

  hourly.innerHTML="";
  todayHours.forEach(l=>{
    const h=parseInt(l.slice(0,2));
    if(h>=hour){
      const p=l.split(" ");
      hourly.innerHTML+=`
        <div class="card">
          <div>${p[0]}</div>
          <div class="icon">${p[2]}</div>
          <div>${p[1]}</div>
        </div>`;
      if(h===hour){
        nowTemp.textContent=`${p[2]} ${p[1]}`;
        nowDesc.textContent="Зараз";
      }
    }
  });

  daily.innerHTML="";
  let shown=0;
  dailyText.split("\n").forEach(l=>{
    if(shown>=7) return;
    const [d,m]=l.split(" ")[0].split(".");
    const date=new Date(now.getFullYear(),m-1,d);
    if(date>=new Date(now.getFullYear(),now.getMonth(),now.getDate())){
      daily.innerHTML+=`
        <div class="card">
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
tick();
setInterval(()=>{tick();render()},60000);

// ===== АДМІН =====
function openAdmin(){modal.style.display="flex"}
function closeAdmin(){modal.style.display="none"}

function login(){
  if(pass.value!==PASSWORD) return alert("Невірний пароль");
  box.innerHTML=`
    <h3>Редагування</h3>
    <small>⏰ Погодинно по днях</small>
    <textarea id="hEdit" rows="12">${hourlyByDay}</textarea>
    <small>📅 Дні</small>
    <textarea id="dEdit" rows="6">${dailyText}</textarea>
    <button onclick="save()">Зберегти</button>
    <button class="close" onclick="closeAdmin()">Закрити</button>`;
}

function save(){
  hourlyByDay=hEdit.value.trim();
  dailyText=dEdit.value.trim();
  render();
  closeAdmin();
}
</script>

</body>
</html>
