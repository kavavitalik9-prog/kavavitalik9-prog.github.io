<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Мої карти</title>
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
<style>
body{margin:0;background:#020617;font-family:system-ui;display:flex;justify-content:center}
.app{width:390px;height:100dvh;background:#020617;color:#fff;display:flex;flex-direction:column}
.top{padding:12px;display:flex;justify-content:space-between;align-items:center;font-weight:700}
#map{flex:1;position:relative}
.admin-btn{font-size:22px;cursor:pointer}
.admin{width:100%;background:#020617;border-top:1px solid #334155;padding:12px;display:none}
.admin input, .admin button{width:100%;margin-top:8px;padding:8px;font-size:14px}
.list{margin-top:10px;max-height:150px;overflow:auto}
.item{display:flex;justify-content:space-between;border:1px solid #334155;padding:6px;margin-top:6px;font-size:12px}
</style>
</head>
<body>
<div class="app">
  <div class="top">
    🗺 Мої карти
    <span class="admin-btn" id="adminBtn">🔒</span>
  </div>
  <div id="map"></div>

  <div class="admin" id="adminPanel">
    <input type="password" id="pass" placeholder="Пароль">
    <input type="file" id="img" accept="image/*">
    <label>Прозорість: <input type="range" id="opacity" min="0.1" max="1" step="0.05" value="0.7"></label>
    <button id="addBtn">Додати знімок</button>
    <div class="list" id="list"></div>
    <button onclick="closeAdmin()">Закрити</button>
  </div>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://cdn.jsdelivr.net/npm/leaflet-distortableimage@0.7.0/dist/leaflet.distortableimage.min.js"></script>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/leaflet-distortableimage@0.7.0/dist/leaflet.distortableimage.min.css"/>

<script>
const map = L.map('map').setView([49.8, 24.0], 7);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{maxZoom:19}).addTo(map);

let admin = false, imgData = null;
let images = JSON.parse(localStorage.getItem("maps")||"[]");
let overlays = [];

function redraw(){
  overlays.forEach(o=>map.removeLayer(o));
  overlays=[];
  images.forEach((d)=>{
    const overlay = L.distortableImageOverlay(d.src, {
      corners: d.corners,
      opacity: d.opacity,
      selected: false
    }).addTo(map);
    overlay.on('edit',()=>saveImages());
    overlays.push(overlay);
  });
  renderList();
}

function renderList(){
  const list = document.getElementById("list");
  list.innerHTML = "";
  images.forEach((d,i)=>{
    const div = document.createElement("div");
    div.className = "item";
    div.innerHTML = `Знімок ${i+1} <button onclick="del(${i})">❌</button>`;
    list.appendChild(div);
  });
}

redraw();

document.getElementById("adminBtn").onclick = ()=>document.getElementById("adminPanel").style.display="block";
function closeAdmin(){document.getElementById("adminPanel").style.display="none";}

document.getElementById("pass").onchange = e=>{
  admin = e.target.value==="3709";
  alert(admin?"Адмін доступ OK":"Невірний пароль");
};

document.getElementById("img").onchange = e=>{
  const f = e.target.files[0];
  if(!f) return;
  const r = new FileReader();
  r.onload = x => imgData = x.target.result;
  r.readAsDataURL(f);
};

document.getElementById("addBtn").onclick = ()=>{
  if(!admin || !imgData){ alert("Пароль або картинка відсутні"); return; }
  const opacity = parseFloat(document.getElementById("opacity").value);
  const overlay = L.distortableImageOverlay(imgData, {
    corners: [
      map.getCenter(),
      [map.getCenter().lat, map.getCenter().lng+0.05],
      [map.getCenter().lat+0.05, map.getCenter().lng+0.05],
      [map.getCenter().lat+0.05, map.getCenter().lng]
    ],
    opacity: opacity
  }).addTo(map);
  overlay.on('edit',()=>saveImages());
  overlays.push(overlay);
  images.push({src: imgData, corners: overlay.getCorners(), opacity});
  saveImages();
  imgData = null;
};

function saveImages(){
  const arr = overlays.map(o=>({src: o._url, corners: o.getCorners(), opacity: o.options.opacity}));
  localStorage.setItem("maps", JSON.stringify(arr));
}

function del(i){
  images.splice(i,1);
  overlays[i].remove();
  overlays.splice(i,1);
  saveImages();
  renderList();
}
</script>
</body>
</html>
