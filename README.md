<!DOCTYPE html><html lang="ru">
<head>
<meta charset="UTF-8">
<title>XP TV — Прямой эфир</title>
<style>
body {
  margin: 0;
  background: #070707;
  color: #ffffff;
  font-family: Arial, sans-serif;
  text-align: center;
}/* ЛОГОТИП */
.logo {
font-size: 36px;
font-weight: bold;
padding: 15px;
background: linear-gradient(90deg, #00ffcc, #00aaff);
-webkit-background-clip: text;
-webkit-text-fill-color: transparent;
}
.live {
color: red;
font-size: 16px;
margin-left: 10px;
}

/* ПЛЕЕР */
#player {
width: 92%;
max-width: 800px;
height: 360px;
margin: 20px auto;
background: #000;
border-radius: 14px;
display: flex;
align-items: center;
justify-content: center;
}
#offline {
color: #777;
font-size: 22px;
}

/* ТАБЛИЦА РАСПИСАНИЯ /
table {
width: 92%;
max-width: 800px;
margin: 20px auto;
border-collapse: collapse;
background-color: #000; / чёрный фон /
color: #fff; / белый текст /
}
th, td {
border: 1px solid #333;
padding: 8px;
color: #fff; / белый текст ячеек /
}
th {
background-color: #111; / тёмный фон заголовка */
color: #fff;
}

footer {
opacity: 0.6;
margin: 20px 0;
}
</style>

</head>
<body><div class="logo">
XP TV <span id="liveText"></span>
</div><div id="player">
  <div id="offline">⏸ Эфир не идёт</div>
</div><h2>📅 Программа передач</h2><table>
<tr><th>Время</th><th>Передача</th></tr>
<tr><td>10:00 – 11:00</td><td>XP Morning</td></tr>
<tr><td>14:00 – 15:00</td><td>XP News</td></tr>
<tr><td>18:00 – 19:00</td><td>XP Show</td></tr>
<tr><td>20:00 – 22:00</td><td>XP Night</td></tr>
</table><footer>
© XP TV — выдуманный телеканал
</footer><script>
// ===== РАСПИСАНИЕ + YOUTUBE =====
const schedule = [
  { start: "10:00", end: "11:00", videoId: "5qap5aO4i9A" }, // Lo-fi радио
  { start: "14:00", end: "15:00", videoId: "DWcJFNfaw9c" }, // Новости (демо)
  { start: "18:00", end: "19:00", videoId: "dQw4w9WgXcQ" }, // XP Show 🙂
  { start: "20:00", end: "22:00", videoId: "https://www.youtube.com/live/jfKfPfyJRdk?si=zf9IEh7vudH_qOSD" }  // Ночной эфир
];

function toMinutes(t) {
  const [h, m] = t.split(":").map(Number);
  return h * 60 + m;
}

function checkLive() {
  const now = new Date();
  const current = now.getHours() * 60 + now.getMinutes();
  let show = null;

  for (let s of schedule) {
    if (current >= toMinutes(s.start) && current < toMinutes(s.end)) {
      show = s;
      break;
    }
  }

  const player = document.getElementById("player");
  const liveText = document.getElementById("liveText");

  if (show) {
    player.innerHTML = `
      <iframe width="100%" height="360"
      src="https://www.youtube.com/embed/${show.videoId}?autoplay=1"
      frameborder="0"
      allow="autoplay; encrypted-media"
      allowfullscreen></iframe>`;
    liveText.innerHTML = "🔴 LIVE";
  } else {
    player.innerHTML = `<div id="offline">⏸ Эфир не идёт</div>`;
    liveText.innerHTML = "";
  }
}

// Проверка каждые 30 секунд
checkLive();
setInterval(checkLive, 30000);
</script></body>
</html>
