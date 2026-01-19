<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>XP TV — Прямой эфир</title>
<style>
body { margin:0; background:#070707; color:#fff; font-family:Arial,sans-serif; text-align:center; }
.logo { font-size:36px; font-weight:bold; padding:15px; background:linear-gradient(90deg,#00ffcc,#00aaff); -webkit-background-clip:text; -webkit-text-fill-color:transparent; }
.live { color:red; font-size:16px; margin-left:10px; }
#player { width:92%; max-width:800px; height:360px; margin:20px auto; background:#000; border-radius:14px; display:flex; align-items:center; justify-content:center; flex-direction:column; }
#offline { color:#777; font-size:22px; }
#schedule { width:92%; max-width:800px; margin:20px auto; border-collapse:collapse; background:#000; color:#fff; font-size:18px; border-radius:12px; overflow:hidden; }
#schedule th, #schedule td { padding:12px; text-align:center; border:1px solid #111; color:#fff; background:#000; }
#schedule tr.current td { background-color:#2222aa; font-weight:bold; }
#viewerCounter { font-size:18px; margin:10px 0; color:#0f0; }

#progressContainer { width:92%; max-width:800px; background:#111; border-radius:6px; margin:10px auto; height:14px; position:relative; }
#progressBar { height:100%; background:#00aaff; width:0%; border-radius:6px; transition: width 0.3s linear; }
#progressTimes { display:flex; justify-content:space-between; font-size:14px; margin-top:4px; width:92%; max-width:800px; margin-left:auto; margin-right:auto; color:#ccc; }

footer { opacity:0.6; margin:20px 0; }
</style>
</head>
<body>

<div class="logo">XP TV <span id="liveText"></span></div>

<div id="player">
  <div id="offline">⏸ Эфир не идёт</div>
  <div id="progressContainer" style="display:none;">
    <div id="progressBar"></div>
  </div>
  <div id="progressTimes">
    <span id="startTime">00:00</span>
    <span id="currentTime">00:00</span>
    <span id="endTime">00:00</span>
  </div>
</div>

<h2>📅 Программа передач</h2>
<table id="schedule">
<tr><th>Время</th><th>Передача</th></tr>
</table>

<div id="viewerCounter">Зрителей сейчас: 0</div>

<footer>© XP TV — выдуманный телеканал</footer>

<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
<script>
// ===== Firebase config =====
const firebaseConfig = {
  apiKey: "ВАШ_API_KEY",
  authDomain: "ВАШ_ПРОЕКТ.firebaseapp.com",
  databaseURL: "https://ВАШ_ПРОЕКТ-default-rtdb.firebaseio.com",
  projectId: "ВАШ_ПРОЕКТ",
  storageBucket: "ВАШ_ПРОЕКТ.appspot.com",
  messagingSenderId: "SENDER_ID",
  appId: "APP_ID"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.database();

// ===== Расписание по дням =====
const scheduleByDate = {
  "2026-01-19":[
    { start:"10:00", end:"11:00", title:"XP Morning", videoId:"5qap5aO4i9A" },
    { start:"14:00", end:"15:00", title:"XP News", videoId:"DWcJFNfaw9c" },
    { start:"18:00", end:"19:00", title:"XP Show", videoId:"dQw4w9WgXcQ" },
    { start:"21:00", end:"22:00", title:"XP Night", videoId:"hHW1oY26kxQ" }
  ]
};

// ===== Выбираем расписание на текущий день =====
const todayStr = new Date().toISOString().split("T")[0];
const todaysSchedule = scheduleByDate[todayStr] || [];

// ===== Функции =====
function toMinutes(t){ const [h,m]=t.split(":").map(Number); return h*60+m; }

// Построение таблицы (текущая + 3 следующих)
function updateScheduleTable(){
  const table = document.getElementById("schedule");
  while(table.rows.length>1) table.deleteRow(1);
  const now = new Date();
  const currentMin = now.getHours()*60 + now.getMinutes();
  let currentIndex = todaysSchedule.findIndex(s=>currentMin>=toMinutes(s.start) && currentMin<toMinutes(s.end));
  if(currentIndex===-1) currentIndex = todaysSchedule.findIndex(s=>currentMin<toMinutes(s.start));
  if(currentIndex===-1) currentIndex = 0;

  for(let i=currentIndex; i<Math.min(todaysSchedule.length,currentIndex+4); i++){
    const item = todaysSchedule[i];
    const row = table.insertRow();
    const cell1 = row.insertCell();
    const cell2 = row.insertCell();
    cell1.textContent = item.start+" – "+item.end;
    cell2.textContent = item.title;
    if(currentMin>=toMinutes(item.start) && currentMin<toMinutes(item.end)) row.classList.add("current");
  }
}

// ===== Автоэфир =====
function checkLive(){
  const now = new Date(); const current=now.getHours()*60+now.getMinutes();
  let show = todaysSchedule.find(s=>current>=toMinutes(s.start) && current<toMinutes(s.end));
  const player = document.getElementById("player");
  const liveText = document.getElementById("liveText");
  const progressContainer = document.getElementById("progressContainer");
  const progressBar = document.getElementById("progressBar");
  const startTimeEl = document.getElementById("startTime");
  const currentTimeEl = document.getElementById("currentTime");
  const endTimeEl = document.getElementById("endTime");

  if(show){
    player.querySelector('#offline')?.remove();
    player.innerHTML = `<iframe width="100%" height="360" src="https://www.youtube.com/embed/${show.videoId}?autoplay=1" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>`;
    liveText.innerHTML = "🔴 LIVE";

    // Показываем прогресс
    progressContainer.style.display = "block";
    startTimeEl.textContent = show.start;
    endTimeEl.textContent = show.end;
  } else {
    player.innerHTML = `<div id="offline">⏸ Эфир не идёт</div>`;
    liveText.innerHTML = "";
    progressContainer.style.display = "none";
  }
}

// ===== Обновление прогресса передачи =====
function updateProgress(){
  const now = new Date(); 
  const currentMin = now.getHours()*60 + now.getMinutes();
  const show = todaysSchedule.find(s=>currentMin>=toMinutes(s.start) && currentMin<toMinutes(s.end));
  if(show){
    const progressBar = document.getElementById("progressBar");
    const currentTimeEl = document.getElementById("currentTime");
    const startMin = toMinutes(show.start);
    const endMin = toMinutes(show.end);
    const progressPercent = ((currentMin - startMin)/(endMin - startMin))*100;
    progressBar.style.width = progressPercent+"%";
    const nowStr = now.getHours().toString().padStart(2,"0")+":"+now.getMinutes().toString().padStart(2,"0");
    currentTimeEl.textContent = nowStr;
  }
}

// ===== Счётчик зрителей =====
const viewersRef = db.ref("viewers");
const myViewer = viewersRef.push(true);
window.addEventListener("beforeunload",()=>{myViewer.remove();});
viewersRef.on("value",snap=>{document.getElementById("viewerCounter").innerText="Зрителей сейчас: "+snap.numChildren();});

// ===== Запуск =====
updateScheduleTable();
checkLive();
updateProgress();
setInterval(updateScheduleTable,30000);
setInterval(checkLive,30000);
setInterval(updateProgress,1000);
</script>
</body>
</html>
