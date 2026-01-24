<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<title>Мій прогноз погоди</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
body{
  margin:0;
  font-family:system-ui,Arial;
  background:linear-gradient(180deg,#0f172a,#020617);
  color:#fff;
}
.container{
  max-width:900px;
  margin:auto;
  padding:15px;
}
h1,h2{margin:10px 0}
.card{
  background:rgba(255,255,255,.08);
  border-radius:14px;
  padding:15px;
  margin-bottom:15px;
}
.now{
  font-size:42px;
  text-align:center;
}
.hourly{
  display:flex;
  gap:10px;
  overflow-x:auto;
}
.hour{
  min-width:100px;
  background:rgba(255,255,255,.12);
  padding:10px;
  border-radius:12px;
  text-align:center;
}
.daily{
  display:grid;
  grid-template-columns:repeat(auto-fill,minmax(120px,1fr));
  gap:10px;
}
.day{
  background:rgba(255,255,255,.12);
  padding:10px;
  border-radius:12px;
  text-align:center;
}
.sun{
  display:flex;
  justify-content:space-between;
  text-align:center;
}
#updated{opacity:.7;font-size:14px}

#adminBtn{
  position:fixed;
  bottom:16px;
  right:16px;
  width:52px;
  height:52px;
  border-radius:50%;
  border:none;
  font-size:22px;
  background:#2563eb;
  color:#fff;
  cursor:pointer;
}

#adminModal{
  display:none;
  position:fixed;
  inset:0;
  background:rgba(0,0,0,.7);
}
#adminBox{
  max-width:520px;
  margin:60px auto;
  background:#020617;
  padding:15px;
  border-radius:14px;
}
input,textarea,button{
  width:100%;
  padding:8px;
  margin:5px 0;
  border-radius:8px;
  border:none;
}
textarea{min-height:70px}
.close{
  text-align:right;
  cursor:pointer;
}
</style>
</head>
<body>

<div class="container">
  <h1>🌦 Мій прогноз погоди</h1>

  <div class="card now" id="now">--</div>

  <div class="card">
    <h2>⏰ Погодинно (24 години)</h2>
    <div class="hourly" id="hourly"></div>
  </div>

  <div class="card">
    <h2>📅 7 днів</h2>
    <div class="daily" id="daily"></div>
  </div>

  <div class="card sun">
    <div>
      🌅<br><b id="sunrise">—</b><br><small>Схід</small>
    </div>
    <div>
      🌇<br><b id="sunset">—</b><br><small>Захід</small>
    </div>
  </div>

  <div id="updated">—</div>
</div>

<button id="adminBtn">⚙</button>

<div id="adminModal">
  <div id="adminBox">
    <div class="close" onclick="closeAdmin()">❌</div>

    <div id="loginBox">
      <h3>Адмін доступ</h3>
      <input type="password" id="pass" placeholder="Пароль">
      <button onclick="login()">Увійти</button>
    </div>

    <div id="panel" style="display:none">
      <h3>Редагування</h3>

      <label>Погода зараз</label>
      <input id="nowInput">

      <label>Погодинно (00–23, кожен рядок)</label>
      <textarea id="hourlyInput"></textarea>

      <label>7 днів (кожен рядок)</label>
      <textarea id="dailyInput"></textarea>

      <label>Схід / Захід (напр. 07:48|16:32)</label>
      <input id="sunInput">

      <button onclick="save()">💾 Зберегти</button>
    </div>
  </div>
</div>

<script>
let data = JSON.parse(localStorage.getItem("weatherData")) || {
  now:"10° ☀️",
  hourly:Array(24).fill("10° ☀️"),
  daily:[
    "Пт 23.01: 12° / 5° ☀️",
    "Сб 24.01: 11° / 4° 🌤",
    "Нд 25.01: 9° / 2° ☁️",
    "Пн 26.01: 8° / 1° 🌧",
    "Вт 27.01: 7° / 0° 🌧",
    "Ср 28.01: 6° / -1° ❄️",
    "Чт 29.01: 5° / -2° ❄️"
  ],
  sun:"07:48|16:32",
  updated:Date.now()
};

function render(){
  now.textContent=data.now;

  hourly.innerHTML="";
  let start=new Date().getHours();
  for(let i=0;i<24;i++){
    let h=(start+i)%24;
    let el=document.createElement("div");
    el.className="hour";
    el.innerHTML=`<b>${String(h).padStart(2,"0")}:00</b><br>${data.hourly[h]||""}`;
    hourly.appendChild(el);
  }

  daily.innerHTML="";
  data.daily.slice(0,7).forEach(d=>{
    let el=document.createElement("div");
    el.className="day";
    el.textContent=d;
    daily.appendChild(el);
  });

  let [r,s]=data.sun.split("|");
  sunrise.textContent=r||"—";
  sunset.textContent=s||"—";

  let diff=Math.floor((Date.now()-data.updated)/60000);
  updated.textContent=
    diff<1?"Оновлено щойно":
    diff<60?`Оновлено ${diff} хв тому`:
    diff<1440?`Оновлено ${Math.floor(diff/60)} год тому`:
    `Оновлено ${Math.floor(diff/1440)} дн тому`;
}

render();
setInterval(render,60000);

adminBtn.onclick=()=>adminModal.style.display="block";
function closeAdmin(){adminModal.style.display="none";}

function login(){
  if(pass.value==="3709"){
    loginBox.style.display="none";
    panel.style.display="block";
    nowInput.value=data.now;
    hourlyInput.value=data.hourly.join("\n");
    dailyInput.value=data.daily.join("\n");
    sunInput.value=data.sun;
  }
}

function save(){
  data.now=nowInput.value;
  data.hourly=hourlyInput.value.split("\n");
  data.daily=dailyInput.value.split("\n");
  data.sun=sunInput.value;
  data.updated=Date.now();
  localStorage.setItem("weatherData",JSON.stringify(data));
  closeAdmin();
  render();
}
</script>

</body>
</html>
