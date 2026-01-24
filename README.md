<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<title>Погода Телефон</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
*{box-sizing:border-box;margin:0;padding:0}
body{
  display:flex;
  justify-content:center;
  align-items:center;
  min-height:100vh;
  background:#111827;
  font-family:system-ui,-apple-system,Segoe UI,Roboto;
  color:#fff;
}
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
.sun{
  display:flex;
  justify-content:space-between;
  flex-direction:column;
  gap:5px;
}
#updated{
  font-size:13px;
  opacity:0.6;
  text-align:center;
}
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
      <div>
        🌅 <b id="sunrise">—</b> <span id="toSunrise">—</span>
      </div>
      <div>
        🌇 <b id="sunset">—</b> <span id="toSunset">—</span>
      </div>
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
      <label>Додати погодинну погоду [YYYY-MM-DD]</label>
      <textarea id="hourlyInput" placeholder="00:00: 10° ☀️"></textarea>
      <label>7 днів (дата: min/max 🌤)</label>
      <textarea id="dailyInput" placeholder="2026-01-24: 12°/5° ☀️"></textarea>
      <label>Схід|Захід (дата: HH:MM HH:MM)</label>
      <textarea id="sunInput" placeholder="24.01: 06:00 16:00"></textarea>
      <button onclick="save()">💾 Зберегти</button>
    </div>
  </div>
</div>

<script>
const PASS="3709";
let data=JSON.parse(localStorage.getItem("weatherData"))||{
  now:"",
  hourlyDays:{},
  daily:[],
  sunDays:{},
  updated:Date.now()
};

function formatTimeDiff(ms){
  const totalMin=Math.max(0,Math.floor(ms/60000));
  const h=Math.floor(totalMin/60);
  const m=totalMin%60;
  return h>0?`${h} год ${m} хв`:`${m} хв`;
}

function render(){
  const nowDate=new Date();
  const dateStr=nowDate.toISOString().slice(0,10);
  const hour=nowDate.getHours();

  let hours=data.hourlyDays[dateStr]||Array(24).fill("—");
  document.getElementById("now").textContent=hours[hour]||"—";

  const hourlyEl=document.getElementById("hourly");
  hourlyEl.innerHTML="";
  for(let i=0;i<24;i++){
    hourlyEl.innerHTML+=`<div class="hour"><b>${String(i).padStart(2,"0")}:00</b><br>${hours[i]||"—"}</div>`;
  }

  document.getElementById("daily").innerHTML=data.daily.slice(0,7).map(d=>`<div class="day">${d}</div>`).join("");

  const todayKey=nowDate.toISOString().slice(5,10).replace("-",".");
  const sun=data.sunDays[todayKey]||"—|—";
  const [sr,ss]=sun.split("|");
  sunrise.textContent=sr;
  sunset.textContent=ss;

  const [srH,srM]=sr.split(":").map(Number);
  const [ssH,ssM]=ss.split(":").map(Number);
  const sunriseDate=new Date(nowDate); sunriseDate.setHours(srH,srM,0,0);
  const sunsetDate=new Date(nowDate); sunsetDate.setHours(ssH,ssM,0,0);

  const toSR=srH>=0?formatTimeDiff(sunriseDate-nowDate):"—";
  const toSS=ssH>=0?formatTimeDiff(sunsetDate-nowDate):"—";

  document.getElementById("toSunrise").textContent=toSR!=="0 хв"?`(${toSR})`:"(Зараз!)";
  document.getElementById("toSunset").textContent=toSS!=="0 хв"?`(${toSS})`:"(Зараз!)";

  const min=Math.floor((Date.now()-data.updated)/60000);
  updated.textContent=min<1?"Оновлено щойно":min<60?`Оновлено ${min} хв тому`:`Оновлено ${Math.floor(min/60)} год тому`;
}

render();
setInterval(render,60000);

// адмінка
adminBtn.onclick=()=>adminModal.style.display="block";
function closeAdmin(){adminModal.style.display="none";}
function login(){
  if(pass.value===PASS){
    loginBox.style.display="none";
    panel.style.display="block";
    hourlyInput.value=Object.entries(data.hourlyDays).map(([d,h])=>`${d}\n${h.join("\n")}`).join("\n\n");
    dailyInput.value=data.daily.join("\n");
    sunInput.value=Object.entries(data.sunDays).map(([d,v])=>`${d}: ${v.replace("|"," ")}`).join("\n");
  } else {
    alert("Невірний пароль");
  }
}

function save(){
  const lines=hourlyInput.value.split("\n");
  let currentDate="";
  data.hourlyDays={};
  lines.forEach(l=>{
    l=l.trim();
    if(!l) return;
    if(l.match(/^\d{4}-\d{2}-\d{2}$/)){
      currentDate=l;
      data.hourlyDays[currentDate]=Array(24).fill("—");
    } else if(l.match(/^\d{2}:\d{2}:/)){
      const h=parseInt(l.split(":")[0]);
      data.hourlyDays[currentDate][h]=l.split(": ").slice(1).join(": ");
    }
  });

  data.daily=dailyInput.value.split("\n").map(l=>l.trim()).filter(Boolean);

  data.sunDays={};
  sunInput.value.split("\n").forEach(l=>{
    const m=l.match(/^(\d{2}\.\d{2}):\s*(\d{2}:\d{2})\s+(\d{2}:\d{2})$/);
    if(m) data.sunDays[m[1]]=m[2]+"|"+m[3];
  });

  data.updated=Date.now();
  localStorage.setItem("weatherData",JSON.stringify(data));
  closeAdmin();
  render();
}
</script>
</body>
</html>
