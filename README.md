<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>XP TV — Прямой эфир</title>

<style>
body{
  margin:0;
  background:#000;
  color:#fff;
  font-family:Arial,sans-serif;
  text-align:center;
}

/* ===== ЛОГО ===== */
.logo{
  font-size:36px;
  font-weight:bold;
  padding:10px;
  background:linear-gradient(90deg,#00ffcc,#00aaff);
  -webkit-background-clip:text;
  -webkit-text-fill-color:transparent;
}

/* ===== ВРЕМЯ МСК ===== */
#mskTime{
  font-size:18px;
  color:#aaa;
  margin-bottom:5px;
}

/* ===== ПЛЕЕР ===== */
#player{
  width:92%;
  max-width:900px;
  height:360px;
  margin:15px auto;
  background:#000;
  border-radius:14px;
}

#offline{
  display:flex;
  align-items:center;
  justify-content:center;
  height:100%;
  color:#666;
  font-size:22px;
}

/* ===== ПРОГРЕСС ===== */
#progressContainer{
  width:92%;
  max-width:900px;
  height:14px;
  background:#111;
  border-radius:6px;
  margin:10px auto;
  overflow:hidden;
}

#progressBar{
  height:100%;
  width:0%;
  background:linear-gradient(90deg,#00aaff,#00ffcc);
  transition:width 0.3s linear;
}

#progressTimes{
  width:92%;
  max-width:900px;
  margin:auto;
  display:flex;
  justify-content:space-between;
  font-size:14px;
  color:#ccc;
}

/* ===== ТАБЛИЦА ===== */
table{
  width:92%;
  max-width:900px;
  margin:20px auto;
  border-collapse:collapse;
  background:#000 !important;
}

th, td{
  background:#000 !important;
  color:#fff !important;
  border:1px solid #111;
  padding:12px;
  text-align:center;
}

tr.current td{
  background:#2222aa !important;
  font-weight:bold;
}

/* ===== ЗРИТЕЛИ ===== */
#viewers{
  color:#0f0;
  font-size:18px;
  margin:10px;
}
</style>
</head>

<body>

<div class="logo">XP TV 🔴 LIVE</div>
<div id="mskTime">МСК: --:--:--</div>

<div id="player">
  <div id="offline">⏸ Эфир не идёт</div>
</div>

<div id="progressContainer">
  <div id="progressBar"></div>
</div>

<div id="progressTimes">
  <span id="startTime">--:--</span>
  <span id="nowTime">--:--</span>
  <span id="endTime">--:--</span>
</div>

<h2>📅 Сейчас в эфире и далее</h2>

<table id="schedule">
<tr>
  <th>Время</th>
  <th>Передача</th>
</tr>
</table>

<div id="viewers">Зрителей сейчас: 0</div>

<script>
/* ===== МОСКОВСКОЕ ВРЕМЯ (UTC+3) ===== */
function updateMSK(){
  const now = new Date();
  const utc = now.getTime() + now.getTimezoneOffset()*60000;
  const msk = new Date(utc + 3*3600000);

  const h = msk.getHours().toString().padStart(2,"0");
  const m = msk.getMinutes().toString().padStart(2,"0");
  const s = msk.getSeconds().toString().padStart(2,"0");

  document.getElementById("mskTime").textContent =
    `МСК: ${h}:${m}:${s}`;
}
setInterval(updateMSK,1000);
updateMSK();

/* ===== РАСПИСАНИЕ ===== */
const scheduleByDate = {
  "2026-01-20":[
    {start:"11:00",end:"15:30",title:"Фиксики — 1 сезон",video:"dQw4w9WgXcQ"},
    {start:"15:30",end:"20:00",title:"Фиксики — 2 сезон",video:"dQw4w9WgXcQ"},
    {start:"20:00",end:"24:00",title:"Фиксики — 3 сезон",video:"dQw4w9WgXcQ"}
  ],
  "2026-01-21":[
    {start:"00:00",end:"00:40",title:"Фиксики — 3 сезон",video:"dQw4w9WgXcQ"},
    {start:"00:40",end:"05:40",title:"Фиксики — 4 сезон",video:"dQw4w9WgXcQ"},
    {start:"05:40",end:"23:59",title:null}
  ]
};

function toMin(t){
  const [h,m]=t.split(":").map(Number);
  return h*60+m;
}

function nowMin(){
  const d=new Date();
  return d.getHours()*60+d.getMinutes();
}

const today=new Date().toISOString().split("T")[0];
const shows=scheduleByDate[today]||[];

function update(){
  const table=document.getElementById("schedule");
  while(table.rows.length>1)table.deleteRow(1);

  const now=nowMin();
  let currentIndex=shows.findIndex(s=>now>=toMin(s.start)&&now<toMin(s.end));
  let startIndex=currentIndex>=0?currentIndex:shows.findIndex(s=>now<toMin(s.start));
  if(startIndex<0)return;

  for(let i=startIndex;i<Math.min(startIndex+4,shows.length);i++){
    const s=shows[i];
    const r=table.insertRow();
    r.insertCell().textContent=s.start+" – "+s.end;
    r.insertCell().textContent=s.title??"Эфир выключен";
    if(i===currentIndex)r.classList.add("current");
  }

  const player=document.getElementById("player");
  const bar=document.getElementById("progressBar");

  if(currentIndex>=0 && shows[currentIndex].title){
    const s=shows[currentIndex];
    player.innerHTML=`<iframe width="100%" height="360"
      src="https://www.youtube.com/embed/${s.video}?autoplay=1"
      allow="autoplay" frameborder="0"></iframe>`;

    bar.style.width=((now-toMin(s.start))/(toMin(s.end)-toMin(s.start))*100)+"%";

    document.getElementById("startTime").textContent=s.start;
    document.getElementById("endTime").textContent=s.end;

    const d=new Date();
    document.getElementById("nowTime").textContent=
      d.getHours().toString().padStart(2,"0")+":"+
      d.getMinutes().toString().padStart(2,"0");
  }else{
    player.innerHTML=`<div id="offline">⏸ Эфир не идёт</div>`;
    bar.style.width="0%";
  }
}

update();
setInterval(update,1000);

/* ===== ФЕЙК СЧЁТЧИК ===== */
let viewers=1;
setInterval(()=>{
  viewers+=Math.random()>0.5?1:-1;
  if(viewers<1)viewers=1;
  document.getElementById("viewers").textContent="Зрителей сейчас: "+viewers;
},3000);
</script>

</body>
</html>
