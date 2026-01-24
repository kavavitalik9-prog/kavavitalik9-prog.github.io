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
#map{flex:1}
.admin-btn{font-size:22px;cursor:pointer}
.admin{position:fixed;bottom:0;left:50%;transform:translateX(-50%);width:390px;background:#020617;border-top:1px solid #334155;padding:12px;display:none}
.admin input,.admin button{width:100%;margin-top:8px;padding:8px;font-size:14px}
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
</div>

<div class="admin" id="adminPanel">
  <input type="password" id="pass" placeholder="Пароль">
  <input type="file" id="img" accept="image/*">
  <button id="addBtn">Додати знімок</button>
  <div class="list" id="list"></div>
  <button onclick="closeAdmin()">Закрити</button>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script>
const map=L.map('map').setView([49.8,24.0],7);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{maxZoom:19}).addTo(map);

let admin=false, addMode=false, imgData=null;
let data=JSON.parse(localStorage.getItem("maps")||"[]");
let layers=[];

function redraw(){
  layers.forEach(l=>map.removeLayer(l));
  layers=[];
  data.forEach((d,i)=>{
    const layer=L.imageOverlay(d.src,d.bounds).addTo(map);
    layers.push(layer);
  });
  renderList();
}

function renderList(){
  const list=document.getElementById("list");
  list.innerHTML="";
  data.forEach((d,i)=>{
    const div=document.createElement("div");
    div.className="item";
    div.innerHTML=`Знімок ${i+1} <button onclick="del(${i})">❌</button>`;
    list.appendChild(div);
  });
}

redraw();

document.getElementById("adminBtn").onclick=()=>document.getElementById("adminPanel").style.display="block";

function closeAdmin(){document.getElementById("adminPanel").style.display="none"; addMode=false;}

document.getElementById("pass").onchange=e=>{
  admin = e.target.value==="3709";
  alert(admin?"Адмін доступ OK":"Невірний пароль");
};

document.getElementById("img").onchange=e=>{
  const f=e.target.files[0];
  if(!f) return;
  const r=new FileReader();
  r.onload=x=>imgData=x.target.result;
  r.readAsDataURL(f);
};

document.getElementById("addBtn").onclick=()=>{
  if(!admin || !imgData){ alert("Пароль або картинка відсутні"); return; }
  addMode=true;
  alert("Клікни на потрібне місце на карті");
};

map.on("click",e=>{
  if(!addMode) return;
  const s=0.05;
  const bounds=[
    [e.latlng.lat-s,e.latlng.lng-s],
    [e.latlng.lat+s,e.latlng.lng+s]
  ];
  data.push({src:imgData,bounds});
  localStorage.setItem("maps",JSON.stringify(data));
  imgData=null;
  addMode=false;
  redraw();
});

function del(i){
  data.splice(i,1);
  localStorage.setItem("maps",JSON.stringify(data));
  redraw();
}
</script>
</body>
</html>
