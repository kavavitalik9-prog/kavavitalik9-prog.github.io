<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Мої карти</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>

<style>
body{
  margin:0;
  background:#0f172a;
  display:flex;
  justify-content:center;
  font-family:system-ui;
}
.phone{
  width:390px;
  height:100dvh;
  background:#020617;
  border-radius:26px;
  overflow:hidden;
  display:flex;
  flex-direction:column;
}
header{
  padding:12px;
  background:#020617;
  color:white;
  font-weight:700;
  display:flex;
  justify-content:space-between;
  align-items:center;
}
#map{
  flex:1;
}
.admin-btn{
  cursor:pointer;
  font-size:20px;
}
.admin-panel{
  position:absolute;
  bottom:0;
  left:0;
  right:0;
  background:#020617;
  color:white;
  padding:12px;
  display:none;
}
.admin-panel input, .admin-panel button{
  width:100%;
  margin-top:8px;
  padding:8px;
  font-size:14px;
}
.note{
  font-size:12px;
  opacity:.7;
}
</style>
</head>

<body>

<div class="phone">
  <header>
    🗺 Мої карти
    <span class="admin-btn" id="adminBtn">🔒</span>
  </header>

  <div id="map"></div>

  <div class="admin-panel" id="adminPanel">
    <input type="password" id="pass" placeholder="Пароль">
    <input type="file" id="imgInput" accept="image/*">
    <button onclick="enableAdd()">Додати знімок</button>
    <button onclick="closeAdmin()">Закрити</button>
    <div class="note">Натисни на карту, щоб поставити знімок</div>
  </div>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script>
const map = L.map('map').setView([49.8,24.0], 7);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{
  maxZoom:19
}).addTo(map);

let admin=false;
let addMode=false;
let currentImage=null;
let overlays=[];

const saved = JSON.parse(localStorage.getItem("overlays")||"[]");

saved.forEach(o=>{
  const img = L.imageOverlay(o.src, o.bounds).addTo(map);
  overlays.push(img);
});

document.getElementById("adminBtn").onclick=()=>{
  document.getElementById("adminPanel").style.display="block";
};

function closeAdmin(){
  document.getElementById("adminPanel").style.display="none";
  admin=false;
  addMode=false;
}

document.getElementById("pass").onchange=e=>{
  if(e.target.value==="3709"){
    admin=true;
    alert("Адмін доступ увімкнено");
  }else{
    alert("Невірний пароль");
  }
};

document.getElementById("imgInput").onchange=e=>{
  const file=e.target.files[0];
  const reader=new FileReader();
  reader.onload=ev=>currentImage=ev.target.result;
  reader.readAsDataURL(file);
};

function enableAdd(){
  if(!admin || !currentImage){
    alert("Немає доступу або картинки");
    return;
  }
  addMode=true;
}

map.on("click",e=>{
  if(!addMode) return;

  const size=0.05;
  const bounds=[
    [e.latlng.lat-size,e.latlng.lng-size],
    [e.latlng.lat+size,e.latlng.lng+size]
  ];

  const overlay=L.imageOverlay(currentImage,bounds).addTo(map);
  overlays.push(overlay);

  saved.push({src:currentImage,bounds});
  localStorage.setItem("overlays",JSON.stringify(saved));

  addMode=false;
  alert("Знімок додано");
});
</script>

</body>
</html>
