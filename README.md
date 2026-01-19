<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>XP TV — Прямой эфир</title>
<style>
body { margin:0; background:#070707; color:#fff; font-family:Arial,sans-serif; text-align:center; }
.logo { font-size:36px; font-weight:bold; padding:15px; background:linear-gradient(90deg,#00ffcc,#00aaff); -webkit-background-clip:text; -webkit-text-fill-color:transparent; }
.live { color:red; font-size:16px; margin-left:10px; }
#player { width:92%; max-width:800px; height:360px; margin:20px auto; background:#000; border-radius:14px; display:flex; align-items:center; justify-content:center; }
#offline { color:#777; font-size:22px; }
#schedule { width:92%; max-width:800px; margin:20px auto; border-collapse:collapse; background:#000; color:#fff; font-size:18px; border-radius:12px; overflow:hidden; }
#schedule th, #schedule td { padding:12px; text-align:center; border:1px solid #111; color:#fff; background:#000; }
#schedule tr.current td { background-color:#2222aa; font-weight:bold; }
#viewerCounter { font-size:18px; margin:10px 0; color:#0f0; }
footer { opacity:0.6; margin:20px 0; }
</style>
</head>
<body>

<div class="logo">XP TV <span id="liveText"></span></div>

<div id="player"><div id="offline">⏸ Эфир не идёт</div></div>

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
  ],
  "2026-01-20":[
    { start:"09:00", end:"10:00", title:"XP Morning Special", videoId:"tAGnKpE4NCI" },
    { start:"13:00", end:"14:00", title:"XP News Extra", videoId:"DWcJFNfaw9c" },
    { start:"17:00", end:"18:00", title:"XP Show 2", videoId:"dQw4w9WgXcQ" },
    { start:"20:00", end:"21:00", title:"XP Night Live", videoId:"hHW1oY26kxQ" }
  ]
};

// ===== Выбираем расписание по дате =====
const todayStr = new Date().toISOString().split("T")[0]; // формат YYYY-MM-DD
const todaysSchedule = scheduleByDate[todayStr] || [];

// ===== Построение таблицы =====
const table = document.getElementById("schedule");
todaysSchedule.forEach((item)=>{
  const row = table.insertRow();
  const cell1 = row.insertCell();
  const cell2 = row.insertCell();
  cell1.textContent = item.start+" – "+item.end;
  cell2.textContent = item.title;
});

// ===== Автоэфир =====
function toMinutes(t){ const [h,m]=t.split(":").map(Number); return h*60+m; }
function checkLive(){
  const now = new Date(); const current=now.getHours()*60+now.getMinutes();
  let show = null;
  todaysSchedule.forEach(s=>{ if(current>=toMinutes(s.start)&&current<toMinutes(s.end)){show=s;} });
  const player = document.getElementById("player");
  const liveText = document.getElementById("liveText");
  if(show){ player.innerHTML=`<iframe width="100%" height="360" src="https://www.youtube.com/embed/${show.videoId}?autoplay=1" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>`; liveText.innerHTML="🔴 LIVE"; }
  else{ player.innerHTML=`<div id="offline">⏸ Эфир не идёт</div>`; liveText.innerHTML=""; }
}

// ===== Подсветка текущей передачи =====
function highlightCurrent(){
  const now = new Date(); const current=now.getHours()*60+now.getMinutes();
  for(let i=1;i<table.rows.length;i++){ table.rows[i].classList.remove("current"); }
  todaysSchedule.forEach((s,index)=>{ if(current>=toMinutes(s.start)&&current<toMinutes(s.end)){ table.rows[index+1].classList.add("current"); } });
}

// ===== Счётчик зрителей =====
const viewersRef = db.ref("viewers");
const myViewer = viewersRef.push(true);
window.addEventListener("beforeunload",()=>{myViewer.remove();});
viewersRef.on("value",snap=>{document.getElementById("viewerCounter").innerText="Зрителей сейчас: "+snap.numChildren();});

// ===== Запуск =====
checkLive(); highlightCurrent();
setInterval(checkLive,30000);
setInterval(highlightCurrent,30000);
</script>

</body>
</html>
