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
    <button id="previewBtn">Попередній перегляд</button>
    <button id="saveBtn">Зберегти</button>
    <div class="list" id="list"></div>
    <button onclick="closeAdmin()">Закрити</button>
  </div>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script>
const map = L.map('map').setView([49.8, 24.0], 7);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{maxZoom:19}).addTo(map);

let admin=false, imgData=null, editingIndex=null;
let images=JSON.parse(localStorage.getItem("maps")||"[]");
let tempOverlay=null;

// Відображення знімків
function redraw(){
  map.eachLayer(l=>{if(l.options && l.options.imgOverlay) map.removeLayer(l);});
  images.forEach((d,i)=>{
    const bounds=[[d.lat,d.lng],[d.lat+0.05,d.lng+0.05]];
    const overlay=L.imageOverlay(d.src,bounds,{opacity:d.opacity,imgOverlay:true}).addTo(map);

    // Редагування існуючого знімка
    overlay.on('click',()=>{
      if(!admin) return;
      editingIndex=i;
      imgData=d.src;
      document.getElementById('opacity').value=d.opacity;
      alert("Ви редагуєте цей знімок. Використайте Попередній перегляд і Зберегти");
    });

    // Видалення подвійним кліком
    overlay.on('dblclick',()=>{if(confirm("Видалити цей знімок?")){images.splice(i,1);saveImages();redraw();}});
  });
  renderList();
}

// Список знімків
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

// Адмін кнопка
document.getElementById("adminBtn").onclick=()=>document.getElementById("adminPanel").style.display="block";
function closeAdmin(){document.getElementById("adminPanel").style.display="none"; if(tempOverlay){map.removeLayer(tempOverlay); tempOverlay=null;}}

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

// Попередній перегляд
document.getElementById("previewBtn").onclick=()=>{
  if(!admin || !imgData){ alert("Пароль або картинка відсутні"); return;}
  alert("Клікніть на карту, щоб поставити/редагувати знімок");
  map.once('click', e=>{
    if(tempOverlay) map.removeLayer(tempOverlay);
    const opacity=parseFloat(document.getElementById("opacity").value);
    const bounds=[[e.lat,e.lng],[e.lat+0.05,e.lng+0.05]];
    tempOverlay=L.imageOverlay(imgData,bounds,{opacity:opacity,imgOverlay:true}).addTo(map);

    // Перетягування простим способом
    let startLatLng=null;
    tempOverlay.on('mousedown', function(ev){
      startLatLng=ev.latlng;
      map.on('mousemove', moveTemp);
      map.once('mouseup', stopMoveTemp);
    });
    function moveTemp(ev){
      const dLat=ev.latlng.lat-startLatLng.lat;
      const dLng=ev.latlng.lng-startLatLng.lng;
      const newBounds=[
        [bounds[0][0]+dLat,bounds[0][1]+dLng],
        [bounds[1][0]+dLat,bounds[1][1]+dLng]
      ];
      tempOverlay.setBounds(newBounds);
    }
    function stopMoveTemp(){map.off('mousemove',moveTemp);}
  });
};

// Збереження знімка
document.getElementById("saveBtn").onclick=()=>{
  if(!tempOverlay && editingIndex===null){alert("Спочатку зробіть попередній перегляд"); return;}
  const bounds=tempOverlay ? tempOverlay.getBounds() : [[images[editingIndex].lat,images[editingIndex].lng],[images[editingIndex].lat+0.05,images[editingIndex].lng+0.05]];
  const opacity=parseFloat(document.getElementById("opacity").value);
  if(editingIndex!==null){
    images[editingIndex]={src:imgData,lat:bounds[0][0],lng:bounds[0][1],opacity};
    editingIndex=null;
  } else {
    images.push({src:imgData,lat:bounds[0][0],lng:bounds[0][1],opacity});
    tempOverlay=null;
    imgData=null;
  }
  saveImages();
  redraw();
};
</script>
</body>
</html>
