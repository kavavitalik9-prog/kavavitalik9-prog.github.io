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
.admin input, .admin button, .admin label{width:100%;margin-top:8px;padding:8px;font-size:14px;color:#fff;background:#111;border:none;border-radius:4px}
.admin input[type=range]{width:100%}
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
    <button id="saveBtn">Зберегти</button>
    <div class="list" id="list"></div>
    <button onclick="closeAdmin()">Закрити</button>
  </div>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script>
const map = L.map('map').setView([49.8, 24.0], 7);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{maxZoom:19}).addTo(map);

let admin=false, imgData=null;
let images=JSON.parse(localStorage.getItem("maps")||[]);
let tempOverlay=null;

// Відображення знімків
function redraw(){
  map.eachLayer(l=>{if(l.options && l.options.imgOverlay) map.removeLayer(l);});
  images.forEach((d,i)=>{
    const bounds=[[d.lat,d.lng],[d.lat+0.05,d.lng+0.05]];
    const overlay=L.imageOverlay(d.src,bounds,{opacity:d.opacity,imgOverlay:true}).addTo(map);

    // Перетягування
    overlay.on('mousedown', function(ev){
      let startLatLng = ev.latlng;
      map.on('mousemove', move);
      map.once('mouseup', stop);
      function move(e){
        const dLat=e.latlng.lat-startLatLng.lat;
        const dLng=e.latlng.lng-startLatLng.lng;
        overlay.setBounds([[d.lat+dLat,d.lng+dLng],[d.lat+0.05+dLat,d.lng+0.05+dLng]]);
      }
      function stop(){map.off('mousemove',move);}
    });

    // Видалення через список
  });
  renderList();
}

function renderList(){
  const list=document.getElementById("list");
  list.innerHTML="";
  images.forEach((d,i)=>{
    const div=document.createElement("div");
    div.className="item";
    div.innerHTML=`Знімок ${i+1} <button onclick="deleteImage(${i})">❌</button>`;
    list.appendChild(div);
  });
}

function deleteImage(i){images.splice(i,1);saveImages();redraw();}
function saveImages(){localStorage.setItem("maps",JSON.stringify(images));}

redraw();

// Адмін панель
document.getElementById("adminBtn").onclick=()=>document.getElementById("adminPanel").style.display="block";
function closeAdmin(){document.getElementById("adminPanel").style.display="none";}

// Пароль
document.getElementById("pass").onchange=e=>{
  admin=e.target.value==="3709";
  alert(admin?"Адмін доступ OK":"Невірний пароль");
};

// Вибір знімка
document.getElementById("img").onchange=e=>{
  const f=e.target.files[0];
  if(!f) return;
  const r=new FileReader();
  r.onload=x=>imgData=x.target.result;
  r.readAsDataURL(f);
};

// Попередній перегляд і додавання
document.getElementById("saveBtn").onclick=()=>{
  if(!admin || !imgData){alert("Пароль або картинка відсутні"); return;}
  alert("Клікніть на карту, щоб поставити знімок");
  map.once('click', e=>{
    const opacity=parseFloat(document.getElementById("opacity").value);
    const lat=e.latlng.lat;
    const lng=e.latlng.lng;
    images.push({src:imgData,lat,lng,opacity});
    saveImages();
    imgData=null;
    redraw();
  });
};
</script>
</body>
</html>
