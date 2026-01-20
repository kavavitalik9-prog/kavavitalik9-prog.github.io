<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>XP TV</title>
<style>
body{
  margin:0;
  background:#000;
  color:#fff;
  font-family:Arial, sans-serif;
}
.container{
  max-width:900px;
  margin:auto;
  padding:15px;
}
.player-wrap{
  position:relative;
  width:100%;
  aspect-ratio:16/9;
  background:#111;
  margin-bottom:10px;
}
iframe{
  width:100%;
  height:100%;
  border:none;
}
.status{
  margin:10px 0;
  font-size:18px;
}
.progress{
  width:100%;
}
table{
  width:100%;
  border-collapse:collapse;
  background:#000;
}
td{
  border:1px solid #333;
  padding:8px;
  color:#fff;
}
.onair{
  position:fixed;
  bottom:10px;
  right:10px;
  background:#000;
  color:#fff;
  padding:8px 14px;
  border-radius:8px;
  font-weight:bold;
}
.small{
  opacity:.7;
  margin-top:8px;
}
</style>
</head>
<body>

<div class="container">

  <div class="player-wrap">
    <iframe id="player" allow="autoplay"></iframe>
  </div>

  <div class="status" id="nowText">Сейчас идёт:</div>

  <input type="range" class="progress" id="progress" min="0" max="100" value="0" disabled>

  <h3>Ближайшие программы</h3>
  <table>
    <tbody id="nextList"></tbody>
  </table>

  <div class="small">время МСК</div>

</div>

<div class="onair" id="onair">⚫ ВНЕ ЭФИРА</div>

<script>
// ===== РАСПИСАНИЕ =====
const schedule = [
  {s:"2026-01-20T00:59", e:"2026-01-20T14:00", title:null},
  {s:"2026-01-20T14:00", e:"2026-01-20T17:30", title:"Фиксики - 1 сезон", video:"dQw4w9WgXcQ"},
  {s:"2026-01-20T17:30", e:"2026-01-20T18:30", title:null},
  {s:"2026-01-20T18:30", e:"2026-01-21T00:30", title:"тест эфир", video:"dQw4w9WgXcQ"},
  {s:"2026-01-21T00:30", e:"2026-01-21T00:59", title:null}
].map(p=>({
  ...p,
  s:new Date(p.s),
  e:new Date(p.e)
}));

const player = document.getElementById("player");
const nowText = document.getElementById("nowText");
const progress = document.getElementById("progress");
const nextList = document.getElementById("nextList");
const onair = document.getElementById("onair");

let currentId = null;

function update(){
  const now = new Date();
  const current = schedule.find(p=>now>=p.s && now<p.e);

  nextList.innerHTML = "";

  if(current){
    const total = current.e - current.s;
    const passed = now - current.s;
    progress.value = Math.min(100, (passed/total)*100);

    nowText.textContent = "Сейчас идёт: " + (current.title ?? "null");

    if(current.title && current.video){
      onair.textContent = "🔴 В ЭФИРЕ";
      if(currentId !== current.video){
        const start = Math.floor(passed/1000);
        player.src =
          "https://www.youtube.com/embed/" + current.video +
          "?autoplay=1&controls=0&disablekb=1&start=" + start;
        currentId = current.video;
      }
    }else{
      onair.textContent = "⚫ ВНЕ ЭФИРА";
      player.src = "";
      currentId = null;
    }

    const upcoming = schedule
      .filter(p=>p.s > current.s)
      .slice(0,3);

    upcoming.forEach(p=>{
      const tr = document.createElement("tr");
      tr.innerHTML =
        "<td>"+p.s.toLocaleTimeString("ru-RU",{hour:"2-digit",minute:"2-digit"})+
        " — "+p.e.toLocaleTimeString("ru-RU",{hour:"2-digit",minute:"2-digit"})+
        "</td><td>"+(p.title ?? "null")+"</td>";
      nextList.appendChild(tr);
    });
  }
}

setInterval(update,1000);
update();
</script>

</body>
</html>
