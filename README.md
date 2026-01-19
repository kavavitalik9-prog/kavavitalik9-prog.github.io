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

/* НОВАЯ ТАБЛИЦА РАСПИСАНИЯ */
#schedule { width:92%; max-width:800px; margin:20px auto; border-collapse:separate; border-spacing:0; border-radius:12px; overflow:hidden; background:#000; color:#fff; font-size:18px; box-shadow:0 0 10px rgba(0,0,0,0.5); }
#schedule th, #schedule td { padding:12px; text-align:center; }
#schedule th { background:#111; }
#schedule tr:nth-child(even) td { background:#1a1a1a; }
#schedule tr.current td { background:#2222aa; font-weight:bold; }
#schedule th:first-child { border-top-left-radius:12px; }
#schedule th:last-child { border-top-right-radius:12px; }
#schedule tr:last-child td:first-child { border-bottom-left-radius:12px; }
#schedule tr:last-child td:last-child { border-bottom-right-radius:12px; }

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
<tr><td>10:00 – 11:00</td><td>XP Morning</td></tr>
<tr><td>14:00 – 15:00</td><td>XP News</td></tr>
<tr><td>18:00 – 19:00</td><td>XP Show</td></tr>
<tr><td>21:00 – 22:00</td><td>XP Night</td></tr>
</table>

<div id="viewerCounter">Зрителей сейчас: 0</div>

<footer>© XP TV — выдуманный телеканал</footer>

<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
<script>
// ===== ВАШ КОНФИГ Firebase =====
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

// ===== Автоэфир =====
const scheduleData = [
  { start:"10:00", end:"11:00", videoId:"5qap5aO4i9A" },
  { start:"14:00", end:"15:00", videoId:"DWcJFNfaw9c" },
  { start:"18:00", end:"19:00", videoId:"dQw4w9WgXcQ" },
  { start:"21:00", end:"22:00", videoId:"hHW1oY26kxQ" }
];

function toMinutes(t){ const [h,m]=t.split(":").map(Number); return h*60+m; }

function checkLive(){
  const now = new Date(); const current = now.getHours()*60+now.getMinutes();
  let show = null;
  for(let s of scheduleData){ if(current>=toMinutes(s.start)&&current<toMinutes(s.end)){show=s; break;} }
  const player = document.getElementById("player"); const liveText = document.getElementById("liveText");
  if(show){ player.innerHTML=`<iframe width="100%" height="360" src="https://www.youtube.com/embed/${show.videoId}?autoplay=1" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>`; liveText.innerHTML="🔴 LIVE"; }
  else{ player.innerHTML=`<div id="offline">⏸ Эфир не идёт</div>`; liveText.innerHTML=""; }
}

// ===== Таблица =====
const scheduleTimes=[ {start:"10:00",end:"11:00",row:1},{start:"14:00",end:"15:00",row:2},{start:"18:00",end:"19:00",row:3},{start:"21:00",end:"22:00",row:4} ];
function highlightCurrent(){ const now=new Date(); const current=now.getHours()*60+now.getMinutes(); const table=document.getElementById("schedule"); for(let i=1;i<table.rows.length;i++){table.rows[i].classList.remove("current");} for(let s of scheduleTimes){if(current>=toMinutes(s.start)&&current<toMinutes(s.end)){table.rows[s.row].classList.add("current");break;} } }

// ===== Счётчик зрителей =====
const viewersRef = db.ref("viewers");
const myViewer = viewersRef.push(true);
window.addEventListener("beforeunload",()=>{myViewer.remove();});
viewersRef.on("value",snap=>{document.getElementById("viewerCounter").innerText="Зрителей сейчас: "+snap.numChildren();});

// ===== Запуск =====
checkLive(); highlightCurrent();
setInterval(checkLive,30000); setInterval(highlightCurrent,30000);
</script>

</body>
</html>
