<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Мої карти</title>
<style>
body {
  margin: 0;
  font-family: system-ui;
  background: #000;
  color: #fff;
  display: flex;
  justify-content: center;
}

.phone {
  width: 100%;
  max-width: 430px;
  height: 100vh;
  position: relative;
  overflow: hidden;
}

#map {
  width: 100%;
  height: 100%;
  background-image: url('https://i.imgur.com/5rB4vTU.jpg');
  background-size: cover;
  background-position: center;
  position: relative;
}

/* Кнопка адміна */
.adminBtn {
  position: fixed;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
  background: #222;
  color: #fff;
  border: none;
  border-radius: 14px 0 0 14px;
  padding: 12px;
  font-size: 18px;
  cursor: pointer;
  z-index: 100;
}

/* Додані знімки */
.snapshot {
  position: absolute;
  width: 60px;
  height: 60px;
  border: 2px solid #2b61ff;
  border-radius: 8px;
  background-size: cover;
  background-position: center;
}

/* Модалки */
.modal {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.7);
  display: none;
  justify-content: center;
  align-items: center;
  z-index: 200;
}

.modalBox {
  background: #151a26;
  padding: 16px;
  border-radius: 14px;
  width: 90%;
  max-width: 380px;
}

input, button {
  width: 100%;
  margin-top: 10px;
  padding: 10px;
  border: none;
  border-radius: 10px;
  background: #222;
  color: #fff;
}

button {
  background: #2b61ff;
  cursor: pointer;
}

button.close {
  background: #333;
}
</style>
</head>

<body>
<div class="phone">
  <div id="map"></div>
</div>

<button class="adminBtn" onclick="openLogin()">⚙</button>

<!-- LOGIN -->
<div class="modal" id="login">
  <div class="modalBox">
    <h3>Введи пароль</h3>
    <input id="pass" placeholder="Пароль">
    <button onclick="check()">Увійти</button>
    <button class="close" onclick="closeAll()">Скасувати</button>
  </div>
</div>

<!-- ADMIN -->
<div class="modal" id="admin">
  <div class="modalBox">
    <h3>Додати знімок</h3>
    <input type="file" id="file">
    <button onclick="enablePlace()">ОК</button>
    <button class="close" onclick="closeAll()">Закрити</button>
  </div>
</div>

<script>
// логіка знімків
let snapshots = JSON.parse(localStorage.getItem("snapshots")) || [];

const mapEl = document.getElementById("map");
function renderSnapshots() {
  document.querySelectorAll(".snapshot").forEach(el => el.remove());
  snapshots.forEach(s => {
    const div = document.createElement("div");
    div.className = "snapshot";
    div.style.left = `${s.x}px`;
    div.style.top = `${s.y}px`;
    div.style.backgroundImage = `url(${s.img})`;
    mapEl.appendChild(div);
  });
}
renderSnapshots();

let adminMode = false;
let selectedFile = null;

// відкриття/закриття модалок
function openLogin(){login.style.display="flex"}
function closeAll(){login.style.display="none"; admin.style.display="none"; adminMode=false; selectedFile=null;}
function check(){
  if(pass.value !== "3709"){ alert("Невірний пароль"); return; }
  login.style.display="none";
  admin.style.display="flex";
}

// вибір файлу
file.addEventListener("change", () => {
  selectedFile = file.files[0];
});

// ставимо на карту
function enablePlace() {
  if(!selectedFile){ alert("Оберіть файл"); return; }
  adminMode = true;
  admin.style.display = "none";
  alert("Тисни на карту, де хочеш поставити знімок.");
}

mapEl.addEventListener("click", e => {
  if(!adminMode) return;
  const rect = mapEl.getBoundingClientRect();
  const x = e.clientX - rect.left - 30;
  const y = e.clientY - rect.top - 30;
  const reader = new FileReader();
  reader.onload = () => {
    snapshots.push({
      img: reader.result,
      x: x,
      y: y,
      time: Date.now()
    });
    localStorage.setItem("snapshots", JSON.stringify(snapshots));
    renderSnapshots();
  };
  reader.readAsDataURL(selectedFile);
  adminMode = false;
  selectedFile = null;
}
);
</script>

</body>
</html>
