<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<title>Мій прогноз погоди</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
body{
  margin:0;
  font-family:system-ui;
  background:linear-gradient(180deg,#1e293b,#0f172a);
  color:#fff;
}
.container{max-width:900px;margin:auto;padding:15px}
h1{margin:5px 0}
.glass{
  background:rgba(255,255,255,.08);
  border-radius:14px;
  padding:15px;
  margin:10px 0;
}
.row{display:flex;gap:10px;overflow-x:auto}
.card{
  min-width:90px;
  background:rgba(255,255,255,.1);
  padding:10px;
  border-radius:12px;
  text-align:center;
}
small{opacity:.7}
.time{font-size:32px}
.now{font-size:40px}
.sun{display:flex;justify-content:space-around;text-align:center}
</style>
</head>
<body>

<div class="container">
  <h1>🌤 Мій прогноз погоди</h1>
  <small id="updated">Оновлено —</small>

  <!-- ЗАРАЗ -->
  <div class="glass">
    <div class="time" id="clock">--:--</div>
    <div class="now" id="now">--°</div>
  </div>

  <!-- СХІД / ЗАХІД -->
  <div class="glass sun">
    <div>
      <div style="font-size:22px">🌅</div>
      <div id="sunrise">—</div>
      <small>Схід</small>
    </div>
    <div>
      <div style="font-size:22px">🌇</div>
      <div id="sunset">—</div>
      <small>Захід</small>
    </div>
  </div>

  <!-- ПОГОДИННО -->
  <div class="glass">
    <h3>⏰ Погодинно (24 год)</h3>
    <div class="row" id="hourly"></div>
  </div>

  <!-- 7 ДНІВ -->
  <div class="glass">
    <h3>📅 7 днів</h3>
    <div class="row" id="daily"></div>
  </div>
</div>

<script>
// ================== ДАНІ (РЕДАГУЄШ ТУТ) ==================

// Погодинна по днях
let hourlyByDay={
"2026-01-23":{
"00":"-2° 🌙","01":"-2° 🌙","02":"-3° 🌙","03":"-3° 🌙","04":"-4° 🌙",
"05":"-4° 🌙","06":"-3° 🌥","07":"-2° 🌥","08":"0° ☀️","09":"2° ☀️",
"10":"4° ☀️","11":"5° ☀️","12":"6° ☀️","13":"6° ☀️","14":"5° ☀️",
"15":"4° ☀️","16":"3° 🌥","17":"2° 🌥","18":"1° 🌙","19":"0° 🌙",
"20":"-1° 🌙","21":"-1° 🌙","22":"-2° 🌙","23":"-2° 🌙"
}
};

// 7 днів
let dailyText=`
2026-01-23 пт 6°/-2° ☀️
2026-01-24 сб 5°/-3° 🌤
2026-01-25 нд 4°/-4° ☁️
2026-01-26 пн 3°/-5° ❄️
2026-01-27 вт 2°/-6° ❄️
2026-01-28 ср 1°/-7° 🌨
2026-01-29 чт 0°/-8° 🌨
`;

// Схід / захід
let sunText=`
2026-01-23 07:48 16:32
2026-01-24 07:47 16:33
2026-01-25 07:46 16:35
`;

// =========================================================

let lastUpdate=new Date();

function clock(){
  const n=new Date();
  clockEl.textContent=n.toLocaleTimeString("uk-UA",{hour:"2-digit",minute:"2-digit"});
}
clock();
setInterval(clock,1000);

function renderHourly(){
  hourly.innerHTML="";
  const now=new Date();
  let shown=0;

  for(let d=0;shown<24;d++){
    const day=new Date(now);
    day.setDate(now.getDate()+d);
    const ds=day.toISOString().slice(0,10);
    if(!hourlyByDay[ds]) continue;

    for(let h=(d===0?now.getHours():0);h<24 && shown<24;h++){
      const key=String(h).padStart(2,"0");
      const val=hourlyByDay[ds][key];
      if(!val) continue;

      const c=document.createElement("div");
      c.className="card";
      c.innerHTML=`<b>${key}:00</b><br>${val}`;
      hourly.appendChild(c);
      shown++;
    }
  }
}

function renderDaily(){
  daily.innerHTML="";
  const lines=dailyText.trim().split("\n");
  const today=new Date();

  let count=0;
  for(const l of lines){
    if(count>=7) break;
    const [d,wd,temp,icon]=l.split(" ");
    if(new Date(d)<today) continue;

    const c=document.createElement("div");
    c.className="card";
    c.innerHTML=`<b>${wd}</b><br>${temp}<br>${icon}`;
    daily.appendChild(c);
    count++;
  }
}

function renderSun(){
  const t=new Date().toISOString().slice(0,10);
  sunrise.textContent="—";
  sunset.textContent="—";
  sunText.trim().split("\n").forEach(l=>{
    const [d,r,s]=l.split(" ");
    if(d===t){sunrise.textContent=r; sunset.textContent=s;}
  });
}

function renderNow(){
  const n=new Date();
  const ds=n.toISOString().slice(0,10);
  const h=String(n.getHours()).padStart(2,"0");
  nowEl.textContent=hourlyByDay[ds]?.[h] || "--";
}

function updateInfo(){
  const diff=Math.floor((Date.now()-lastUpdate)/60000);
  updated.textContent=diff<1?"Оновлено щойно":
    diff<60?`Оновлено ${diff} хв тому`:
    `Оновлено ${Math.floor(diff/60)} год тому`;
}

function renderAll(){
  renderHourly();
  renderDaily();
  renderSun();
  renderNow();
  updateInfo();
}

renderAll();
setInterval(renderAll,60000);
</script>

</body>
</html>
