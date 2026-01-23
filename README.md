<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<title>🌦 Мій прогноз погоди</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
  margin:0;
  background:linear-gradient(180deg,#0f2027,#203a43,#2c5364);
  font-family:system-ui;
  color:#fff;
}
.app{max-width:390px;margin:auto;padding:14px}
.card{
  background:rgba(0,0,0,.25);
  border-radius:18px;
  padding:14px;
  margin-bottom:14px;
}
.now{text-align:center}
.temp{font-size:48px;font-weight:700}
.desc{opacity:.85}
.hourly,.daily{display:flex;gap:10px;overflow-x:auto}
.item{
  min-width:64px;
  text-align:center;
  background:rgba(255,255,255,.08);
  padding:8px;
  border-radius:12px;
  font-size:13px;
}
.icon{font-size:22px}
.time{opacity:.7;font-size:13px}

.admin-btn{
  position:fixed;
  bottom:16px;
  right:16px;
  font-size:22px;
  background:#0008;
  border-radius:50%;
  width:48px;
  height:48px;
  display:flex;
  align-items:center;
  justify-content:center;
  cursor:pointer;
}

.modal{
  position:fixed;
  inset:0;
  background:#000a;
  display:none;
  align-items:center;
  justify-content:center;
}
.modal-box{
  background:#1c1f26;
  padding:16px;
  border-radius:14px;
  width:90%;
  max-width:360px;
}
input,textarea,button{
  width:100%;
  margin-top:8px;
  padding:8px;
  border-radius:8px;
  border:none;
  font-size:14px;
}
button{background:#2ecc71;color:#000;font-weight:600}
.close{background:#ff4d4d}
</style>
</head>

<body>

<div class="app">

<div class="card now">
  <div class="icon" id="nowIcon">☀️</div>
  <div class="temp" id="nowTemp">+18°</div>
  <div class="desc" id="nowDesc">Сонячно</div>
  <div class="time" id="timeNow"></div>
</div>

<div class="card">
  <h3>⏰ Погодинно</h3>
  <div class="hourly" id="hourly"></div>
</div>

<div class="card">
  <h3>📅 7 днів</h3>
  <div class="daily" id="daily"></div>
</div>

</div>

<!-- 🔒 КНОПКА АДМІНА -->
<div class="admin-btn" onclick="openLogin()">🔒</div>

<!-- 🔑 МОДАЛЬ -->
<div class="modal" id="modal">
  <div class="modal-box" id="modalBox">
    <h3>Адмін доступ</h3>
    <input id="pass" placeholder="Пароль">
    <button onclick="login()">Увійти</button>
    <button class="close" onclick="closeModal()">Закрити</button>
  </div>
</div>

<script>
// ===== ПАРОЛЬ =====
let ADMIN_PASSWORD = "3709";

// ===== ДАНІ =====
let hourlyData = [
 {h:"00:00",t:"+12°",i:"🌙"},
 {h:"06:00",t:"+14°",i:"🌤"},
 {h:"12:00",t:"+19°",i:"☀️"},
 {h:"18:00",t:"+17°",i:"🌤"}
];

let dailyData = [
 {d:"Пн",t:"+18°",i:"☀️"},
 {d:"Вт",t:"+16°",i:"🌧"},
 {d:"Ср",t:"+14°",i:"🌧"},
 {d:"Чт",t:"+17°",i:"🌤"},
 {d:"Пт",t:"+20°",i:"☀️"},
 {d:"Сб",t:"+22°",i:"☀️"},
 {d:"Нд",t:"+19°",i:"⛅"}
];

// ===== РЕНДЕР =====
function render(){
  hourly.innerHTML="";
  hourlyData.forEach(x=>{
    hourly.innerHTML+=`
    <div class="item">
      <div>${x.h}</div>
      <div class="icon">${x.i}</div>
      <div>${x.t}</div>
    </div>`;
  });

  daily.innerHTML="";
  dailyData.forEach(x=>{
    daily.innerHTML+=`
    <div class="item">
      <div>${x.d}</div>
      <div class="icon">${x.i}</div>
      <div>${x.t}</div>
    </div>`;
  });
}
render();

// ===== ЧАС =====
function updateTime(){
  const n=new Date();
  timeNow.textContent="Зараз: "+n.toLocaleTimeString("uk-UA");
}
updateTime();
setInterval(updateTime,1000);

// ===== АДМІН =====
function openLogin(){modal.style.display="flex"}
function closeModal(){modal.style.display="none"}

function login(){
  if(pass.value!==ADMIN_PASSWORD) return alert("Невірний пароль");
  modalBox.innerHTML=`
  <h3>Редагування</h3>
  <textarea id="hEdit" rows="4">${JSON.stringify(hourlyData,null,1)}</textarea>
  <textarea id="dEdit" rows="4">${JSON.stringify(dailyData,null,1)}</textarea>
  <button onclick="save()">Зберегти</button>
  <button class="close" onclick="closeModal()">Закрити</button>`;
}

function save(){
  try{
    hourlyData=JSON.parse(hEdit.value);
    dailyData=JSON.parse(dEdit.value);
    render();
    closeModal();
  }catch{
    alert("Помилка формату");
  }
}
</script>

</body>
</html>
