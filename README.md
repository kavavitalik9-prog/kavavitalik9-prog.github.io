<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<title>Погода Телефон</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
*{box-sizing:border-box;margin:0;padding:0;}
body{
  display:flex;
  justify-content:center;
  align-items:center;
  min-height:100vh;
  background:#111827;
  font-family:system-ui,-apple-system,Segoe UI,Roboto;
  color:#fff;
}

/* === Телефонна рамка === */
.phone-frame{
  width:390px;
  max-width:100%;
  height:800px;
  border-radius:36px;
  border:12px solid #1f2937;
  box-shadow:0 0 40px rgba(0,0,0,0.5);
  overflow:hidden;
  background:linear-gradient(180deg,#0f172a,#020617);
  display:flex;
  flex-direction:column;
  position:relative;
}

/* === Контент всередині телефону === */
.phone-screen{
  flex:1;
  overflow-y:auto;
  padding:16px;
}

h1,h2{margin:10px 0;text-align:center;}
.card{
  background:rgba(255,255,255,0.08);
  border-radius:16px;
  padding:14px;
  margin-bottom:14px;
}
.now{
  font-size:40px;
  text-align:center;
}

/* hourly */
.hourly{
  display:flex;
  gap:10px;
  overflow-x:auto;
}
.hour{
  min-width:92px;
  background:rgba(255,255,255,.12);
  border-radius:14px;
  padding:10px;
  text-align:center;
}

/* daily */
.daily{
  display:grid;
  grid-template-columns:repeat(2,1fr);
  gap:10px;
}
.day{
  background:rgba(255,255,255,.12);
  border-radius:14px;
  padding:10px;
  text-align:center;
}

/* sun */
.sun{
  display:flex;
  justify-content:space-between;
}

/* оновлення */
#updated{
  font-size:13px;
  opacity:0.6;
  text-align:center;
}

/* адмінка */
#adminBtn{
  position:absolute;
  bottom:16px;
  right:16px;
  width:50px;
  height:50px;
  border-radius:50%;
  border:none;
  background:#2563eb;
  font-size:22px;
  color:#fff;
  cursor:pointer;
  z-index:999;
}

#adminModal{
  display:none;
  position:fixed;
  inset:0;
  background:rgba(0,0,0,0.7);
}
#adminBox{
  background:#020617;
  max-width:360px;
  margin:80px auto;
  padding:15px;
  border-radius:16px;
}
input,textarea,button{
  width:100%;
  padding:8px;
  margin:5px 0;
  border-radius:10px;
  border:none;
}
textarea{min-height:70px}
.close{text-align:right;cursor:pointer}
</style>
</head>
<body>

<div class="phone-frame">
  <div class="phone-screen">
    <h1>🌦 Погода</h1>
    <div class="card now" id="now">—</div>

    <div class="card">
      <h2>⏰ Погодинно</h2>
      <div class="hourly" id="hourly"></div>
    </div>

    <div class="card">
      <h2>📅 7 днів</h2>
      <div class="daily" id="daily"></div>
    </div>

    <div class="card sun">
      <div>🌅 <b id="sunrise">—</b></div>
      <div>🌇 <b id="sunset">—</b></div>
    </div>

    <div id="updated">—</div>
  </div>

  <button id="adminBtn">⚙</button>
</div>

<div id="adminModal">
  <div id="adminBox">
    <div class="close" onclick="closeAdmin()">✖</div>
    <div id="loginBox">
      <input type="password" id="pass" placeholder="Пароль">
      <button onclick="login()">Увійти</button>
    </div>
    <div id="panel" style="display:none">
      <label>Зараз</label>
      <input id="nowInput">
      <label>Погодинно (24 рядки)</label>
      <textarea id="hourlyInput"></textarea>
      <label>7 днів</label>
      <textarea id="dailyInput"></textarea>
      <label>Схід|Захід</label>
      <input id="sunInput">
      <button onclick="save()">💾 Зберегти</button>
    </div>
  </div>
</div>

<script>
const PASS="3709";

let data=JSON.parse(localStorage.getItem("weatherData"))||{
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
  document.getElementById("now").textContent=data.now;

  const hourlyEl=document.getElementById("hourly");
  hourlyEl.innerHTML="";
  let start=new Date().getHours();
  for(let i=0;i<24;i++){
    const h=(start+i)%24;
    hourlyEl.innerHTML+=`<div class="hour"><b>${String(h).padStart(2,"0")}:00</b><br>${data.hourly[h]||"—"}</div>`;
  }

  document.getElementById("daily").innerHTML=data.daily.slice(0,7).map(d=>`<div class="day">${d}</div>`).join("");

  const [sr,ss]=data.sun.split("|");
  sunrise.textContent=sr;
  sunset.textContent=ss;

  const min=Math.floor((Date.now()-data.updated)/60000);
  updated.textContent=min<1?"Оновлено щойно":min<60?`Оновлено ${min} хв тому`:`Оновлено ${Math.floor(min/60)} год тому`;
}

render();
setInterval(render,60000);

adminBtn.onclick=()=>adminModal.style.display="block";
function closeAdmin(){adminModal.style.display="none";}
function login(){
  if(pass.value===PASS){
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
