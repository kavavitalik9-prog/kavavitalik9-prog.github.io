<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Air Alert Map</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">

<style>
html,body{margin:0;height:100%;font-family:Arial}
#map{height:100%}

#adminBtn{
  position:absolute;top:10px;right:10px;
  z-index:1000;padding:8px 12px;
}

#adminPanel{
  position:absolute;bottom:0;left:0;right:0;
  background:#111;color:#fff;
  padding:10px;display:none;
  z-index:1000;
}

select,button,input{
  margin:5px;
}
</style>
</head>

<body>

<button id="adminBtn">Админ</button>

<div id="map"></div>

<div id="adminPanel">
  <b>Админ панель</b><br>
  Статус:
  <select id="statusSelect">
    <option value="clear">Отбой</option>
    <option value="threat">🟡 Угроза</option>
    <option value="alert">🔴 Тревога</option>
  </select>

  <button onclick="saveState()">Сохранить</button>
  <br><br>

  <b>Маршрут (кликни 2 точки на карте)</b>
  <button onclick="startRoute()">Начать</button>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script>
/* ===== ВРЕМЯ МСК ===== */
function mskTime(){
  const d=new Date(Date.now()+3*3600*1000);
  return d.toISOString().slice(11,19);
}

/* ===== КАРТА ===== */
const map=L.map('map').setView([49,31],6);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{
  attribution:'© OpenStreetMap'
}).addTo(map);

/* ===== ОБЛАСТИ (центры) ===== */
const regions={
  "Киев":[50.45,30.52],
  "Львов":[49.84,24.03],
  "Одесса":[46.48,30.73],
  "Харьков":[49.99,36.23],
  "Днепр":[48.45,34.98],
  "Запорожье":[47.84,35.14],
  "Херсон":[46.64,32.61],
  "Николаев":[46.97,31.99],
  "Полтава":[49.59,34.55],
  "Чернигов":[51.49,31.29],
  "Сумы":[50.91,34.80]
};

let state=JSON.parse(localStorage.getItem("alertState")||"{}");
let markers={};

for(let r in regions){
  const m=L.circleMarker(regions[r],{
    radius:14,
    color:'#555',
    fillColor:'#ccc',
    fillOpacity:0.8
  }).addTo(map)
   .bindPopup(r);

  m.on('click',()=>{
    if(admin){
      state[r]={status:statusSelect.value,time:mskTime()};
      updateMap();
    }
  });

  markers[r]=m;
}

/* ===== ОБНОВЛЕНИЕ ЦВЕТОВ ===== */
function updateMap(){
  for(let r in markers){
    let s=state[r]?.status;
    let color='#ccc';
    if(s==='alert') color='red';
    if(s==='threat') color='yellow';
    markers[r].setStyle({fillColor:color});
  }
  localStorage.setItem("alertState",JSON.stringify(state));
}
updateMap();

/* ===== АДМИН ===== */
let admin=false;
adminBtn.onclick=()=>{
  const p=prompt("Пароль:");
  if(p==="3709"){
    admin=true;
    adminPanel.style.display='block';
  }
};

/* ===== СОХРАНЕНИЕ ===== */
function saveState(){
  alert("Сохранено ("+mskTime()+" МСК)");
}

/* ===== МАРШРУТЫ ===== */
let routeMode=false;
let routePts=[];
let routeLine=null;

function startRoute(){
  routeMode=true;
  routePts=[];
  alert("Кликни 2 точки на карте");
}

map.on('click',e=>{
  if(!admin||!routeMode)return;
  routePts.push(e.latlng);
  if(routePts.length===2){
    if(routeLine) map.removeLayer(routeLine);
    routeLine=L.polyline(routePts,{color:'orange'}).addTo(map);
    routeMode=false;
  }
});
</script>

</body>
</html>
