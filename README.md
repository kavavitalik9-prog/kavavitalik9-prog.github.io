<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Мої карти</title>
<style>
body{
  margin:0;
  background:#0b0d13;
  font-family:system-ui;
  color:#fff;
  display:flex;
  justify-content:center;
}
.phone{
  max-width:430px;
  width:100%;
  min-height:100vh;
  padding:0;
  position:relative;
}
#map{
  width:100%;
  height:100vh;
  background:url('https://i.imgur.com/5rB4vTU.jpg') no-repeat center center;
  background-size:cover;
  position:relative;
}

/* Додані знімки */
.snapshot{
  position:absolute;
  width:60px;
  height:60px;
  border:2px solid #2b61ff;
  border-radius:8px;
  overflow:hidden;
  cursor:pointer;
}

/* Кнопка адміна збоку */
.adminBtn{
  position:fixed;
  right:0;
  top:50%;
  transform:translateY(-50%);
  background:#222;
  color:#fff;
  border:none;
  border-radius:14px 0 0 14px;
  padding:12px;
  font-size:18px;
  z-index:10;
  cursor:pointer;
}

/* Модальні вікна */
.modal{
  position:fixed;
  inset:0;
  background:#000a;
  display:none;
  justify-content:center;
  align-items:center;
  z-index:20;
}
.modalBox{
  background:#151a26;
  padding:16px;
  border-radius:16px;
  width:90%;
  max-width:380px;
}
input,button{
  width:100%;
  margin-top:10px;
  padding:10px;
  border-radius:10px;
  border:none;
  background:#222;
  color:#fff;
}
button{
  background:#2b61ff;
}
button.close{
  background:#333;
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
    <h3>Адмін доступ</h3>
    <input id="pass" placeholder="Пароль">
    <button onclick="check()">Увійти</button>
    <button class="close" onclick="closeAll()">Скасувати</button>
  </div>
</div>

<!-- ADMIN -->
<div class="modal" id="admin">
  <div class="modalBox">
    <h3>Додати знімок</h3>
    <p>Натисни на карту, де хочеш додати знімок</p>
    <input id="file" type="file">
    <button onclick="finishAdd()">Додати</button>
    <button class="close" onclick="closeAll()">Закрити</button>
  </div>
</div>

<script>
const PASS="3709";
const map=document.getElementById("map");
let adminMode=false;
let addFile=null;
let snapshots=JSON.parse(localStorage.getItem("snapshots"))||[];

// Відображаємо всі знімки зі збереження
function renderSnapshots(){
  document.querySelectorAll('.snapshot').forEach(el=>el.remove());
  snapshots.forEach(s=>{
    const imgEl=document.createElement('img');
    imgEl.src=s.img;
    imgEl.className='snapshot';
    imgEl.style.left=s.x+'px';
    imgEl.style.top=s.y+'px';
    map.appendChild(imgEl);
  });
}
renderSnapshots();

// Кнопка адміна
function openLogin(){document.getElementById('login').style.display='flex'}
function closeAll(){document.getElementById('login').style.display='none';document.getElementById('admin').style.display='none'; adminMode=false; addFile=null;}

function check(){
  if(document.getElementById('pass').value!==PASS){alert("Невірний пароль"); return;}
  document.getElementById('login').style.display='none';
  document.getElementById('admin').style.display='flex';
  alert("Тепер натисни на карту, де хочеш додати знімок");
}

// Вибір файлу
document.getElementById('file').addEventListener('change',e=>{
  addFile=e.target.files[0];
  if(addFile){
    adminMode=true;
  }
});

// Натискання на карту
map.addEventListener('click', e=>{
  if(!adminMode || !addFile) return;
  const rect=map.getBoundingClientRect();
  const x=e.clientX-rect.left-30; // центр знімка
  const y=e.clientY-rect.top-30;
  const reader=new FileReader();
  reader.onload=()=>{
    snapshots.push({img:reader.result, x:x, y:y, time:Date.now()});
    localStorage.setItem("snapshots", JSON.stringify(snapshots));
    renderSnapshots();
    adminMode=false;
    addFile=null;
    closeAll();
  }
  reader.readAsDataURL(addFile);
});

// Кнопка додати після вибору файлу
function finishAdd(){
  if(!addFile){alert("Оберіть файл"); return;}
  alert("Тепер натисніть на карту, де хочете розмістити знімок");
}
</script>

</body>
</html>
