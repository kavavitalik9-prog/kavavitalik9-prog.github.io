<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Карта воздушной тревоги</title>
<style>
body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #0b0b0b;
  color: #fff;
}
header {
  padding: 10px;
  text-align: center;
  background: #111;
  font-size: 20px;
  font-weight: bold;
}
#map {
  max-width: 900px;
  margin: 20px auto;
}
svg path {
  fill: #2a2a2a;
  stroke: #555;
  stroke-width: 1;
  cursor: pointer;
  transition: 0.2s;
}
svg path.alarm {
  fill: #b30000;
}
svg path.threat {
  fill: #b38f00;
}
svg path:hover {
  opacity: 0.8;
}
#regionInfo {
  text-align: center;
  margin-top: 10px;
  font-size: 16px;
}
#adminBtn {
  position: fixed;
  right: 15px;
  bottom: 15px;
  padding: 10px 14px;
  background: #222;
  border: 1px solid #555;
  color: #fff;
  cursor: pointer;
}
#adminPanel {
  display: none;
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.8);
}
#adminBox {
  background: #111;
  width: 300px;
  margin: 80px auto;
  padding: 15px;
  border: 1px solid #444;
}
select, button, input {
  width: 100%;
  margin-top: 10px;
  padding: 6px;
  background: #222;
  color: #fff;
  border: 1px solid #444;
}
.legend {
  text-align: center;
  margin-top: 10px;
}
</style>
</head>
<body>

<header>🛑 Карта воздушной тревоги</header>

<div class="legend">
  🔴 Воздушная тревога &nbsp;&nbsp; 🟡 Угроза воздушной тревоги
</div>

<div id="map">
<svg viewBox="0 0 600 400">
  <path id="Kyiv" d="M280 120 L320 120 L330 160 L300 180 L260 160 Z"/>
  <path id="Lviv" d="M120 140 L170 140 L180 180 L150 200 L110 180 Z"/>
  <path id="Odessa" d="M260 260 L310 260 L330 300 L290 320 L250 300 Z"/>
  <path id="Kharkiv" d="M400 140 L450 140 L470 180 L430 200 L390 180 Z"/>
  <path id="Dnipro" d="M340 200 L390 200 L410 240 L370 260 L330 240 Z"/>
</svg>
</div>

<div id="regionInfo">Нажми на область</div>

<button id="adminBtn">👑 Админ</button>

<div id="adminPanel">
  <div id="adminBox">
    <h3>Админ-панель</h3>
    <input type="password" id="pass" placeholder="Пароль">
    <select id="regionSelect">
      <option value="">Выбери область</option>
      <option value="Kyiv">Киевская</option>
      <option value="Lviv">Львовская</option>
      <option value="Odessa">Одесская</option>
      <option value="Kharkiv">Харьковская</option>
      <option value="Dnipro">Днепропетровская</option>
    </select>
    <select id="statusSelect">
      <option value="none">Нет сигнала</option>
      <option value="threat">🟡 Угроза</option>
      <option value="alarm">🔴 Тревога</option>
    </select>
    <button onclick="saveStatus()">Сохранить</button>
    <button onclick="closeAdmin()">Закрыть</button>
  </div>
</div>

<script>
const PASSWORD = "3709";
let data = JSON.parse(localStorage.getItem("alarms") || "{}");

function updateMap() {
  document.querySelectorAll("svg path").forEach(p => {
    p.classList.remove("alarm","threat");
    const info = data[p.id];
    if (!info) return;
    p.classList.add(info.status);
  });
}
updateMap();

document.querySelectorAll("svg path").forEach(p => {
  p.onclick = () => showInfo(p.id);
});

function showInfo(id) {
  const el = data[id];
  if (!el) {
    regionInfo.textContent = id + ": сигнала нет";
    return;
  }
  const minutes = Math.floor((Date.now() - el.time)/60000);
  const icon = el.status === "alarm" ? "🔴" : "🟡";
  regionInfo.textContent = `${icon} ${id}: ${minutes} мин.`;
}

adminBtn.onclick = () => adminPanel.style.display = "block";

function closeAdmin() {
  adminPanel.style.display = "none";
}

function saveStatus() {
  if (pass.value !== PASSWORD) {
    alert("Неверный пароль");
    return;
  }
  const r = regionSelect.value;
  const s = statusSelect.value;
  if (!r) return;

  if (s === "none") {
    delete data[r];
  } else {
    data[r] = { status: s, time: Date.now() };
  }
  localStorage.setItem("alarms", JSON.stringify(data));
  updateMap();
  alert("Сохранено");
}

setInterval(updateMap, 60000);
</script>

</body>
</html>
