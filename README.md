<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Воздушная тревога — Украина</title>
<style>
body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #0a0a0a;
  color: #fff;
}
header {
  background: #111;
  padding: 10px;
  text-align: center;
  font-size: 20px;
}
#time {
  font-size: 14px;
  margin-top: 5px;
  color: #aaa;
}
#map {
  max-width: 1000px;
  margin: 20px auto;
}
svg path {
  fill: #2b2b2b;
  stroke: #555;
  stroke-width: 1;
  cursor: pointer;
  transition: 0.2s;
}
svg path.alarm { fill: #b00000; }
svg path.threat { fill: #b38b00; }
svg path:hover { opacity: 0.85; }

#info {
  text-align: center;
  margin-top: 10px;
  min-height: 24px;
}

.legend {
  text-align: center;
  margin-top: 10px;
}
.legend span {
  margin: 0 10px;
}

#adminBtn {
  position: fixed;
  right: 15px;
  bottom: 15px;
  background: #222;
  color: #fff;
  padding: 10px 14px;
  border: 1px solid #555;
  cursor: pointer;
}

#adminPanel {
  display: none;
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.85);
}
#adminBox {
  width: 320px;
  background: #111;
  margin: 80px auto;
  padding: 15px;
  border: 1px solid #444;
}
select, input, button {
  width: 100%;
  margin-top: 10px;
  padding: 6px;
  background: #222;
  color: #fff;
  border: 1px solid #444;
}
</style>
</head>
<body>

<header>
  🛑 Воздушная тревога — Украина
  <div id="time"></div>
</header>

<div class="legend">
  <span>🔴 Тревога</span>
  <span>🟡 Угроза</span>
  <span>⚫ Нет сигнала</span>
</div>

<div id="map">
<svg viewBox="0 0 900 600">
  <!-- Упрощённые области -->
  <path id="Lviv" d="M120 220 L180 200 L210 250 L170 300 L120 270 Z"/>
  <path id="Volyn" d="M150 160 L210 150 L220 200 L180 220 L140 200 Z"/>
  <path id="Kyiv" d="M360 220 L420 220 L440 260 L400 300 L350 260 Z"/>
  <path id="Chernihiv" d="M360 160 L430 150 L450 200 L410 220 L360 200 Z"/>
  <path id="Kharkiv" d="M560 220 L630 220 L650 260 L610 300 L560 260 Z"/>
  <path id="Dnipro" d="M500 300 L560 300 L580 350 L540 380 L490 350 Z"/>
  <path id="Odessa" d="M360 380 L420 380 L440 430 L400 460 L350 430 Z"/>
  <path id="Zaporizhzhia" d="M520 360 L580 360 L600 410 L560 440 L510 410 Z"/>
</svg>
</div>

<div id="info">Нажми на область</div>

<button id="adminBtn">👑 Админ</button>

<div id="adminPanel">
  <div id="adminBox">
    <h3>Админ-панель</h3>
    <input id="pass" type="password" placeholder="Пароль">
    <select id="region">
      <option value="">Выбери область</option>
      <option value="Lviv">Львовская</option>
      <option value="Volyn">Волынская</option>
      <option value="Kyiv">Киевская</option>
      <option value="Chernihiv">Черниговская</option>
      <option value="Kharkiv">Харьковская</option>
      <option value="Dnipro">Днепропетровская</option>
      <option value="Zaporizhzhia">Запорожская</option>
      <option value="Odessa">Одесская</option>
    </select>
    <select id="status">
      <option value="none">Нет сигнала</option>
      <option value="threat">🟡 Угроза</option>
      <option value="alarm">🔴 Тревога</option>
    </select>
    <button onclick="save()">Сохранить</button>
    <button onclick="closeAdmin()">Закрыть</button>
  </div>
</div>

<script>
const PASSWORD = "3709";
let alarms = JSON.parse(localStorage.getItem("ua_alarms") || "{}");

function updateTime() {
  const now = new Date(Date.now() + 3*60*60*1000);
  time.textContent = "МСК: " + now.toLocaleTimeString();
}
setInterval(updateTime, 1000);
updateTime();

function render() {
  document.querySelectorAll("svg path").forEach(p => {
    p.classList.remove("alarm","threat");
    if (alarms[p.id]) p.classList.add(alarms[p.id].status);
    p.onclick = () => showInfo(p.id);
  });
}
render();

function showInfo(id) {
  if (!alarms[id]) {
    info.textContent = id + ": сигнала нет";
    return;
  }
  const a = alarms[id];
  const min = Math.floor((Date.now() - a.time)/60000);
  const icon = a.status === "alarm" ? "🔴" : "🟡";
  info.textContent = `${icon} ${id}: ${min} мин.`;
}

adminBtn.onclick = () => adminPanel.style.display = "block";
function closeAdmin() { adminPanel.style.display = "none"; }

function save() {
  if (pass.value !== PASSWORD) return alert("Неверный пароль");
  if (!region.value) return;
  if (status.value === "none") {
    delete alarms[region.value];
  } else {
    alarms[region.value] = {
      status: status.value,
      time: Date.now()
    };
  }
  localStorage.setItem("ua_alarms", JSON.stringify(alarms));
  render();
  alert("Сохранено");
}
</script>

</body>
</html>
