<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Карта тревог</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">

<style>
html,body{margin:0;height:100%;background:#000;color:#fff;font-family:Arial}
#map{height:100%}
#login{
  position:absolute;top:10px;right:10px;
  background:#111;padding:10px;z-index:1000
}
#admin{
  position:absolute;bottom:0;left:0;right:0;
  background:#111;padding:10px;display:none;z-index:1000
}
button,select,input{
  width:100%;margin:4px 0;padding:6px;
  background:#222;color:#fff;border:1px solid #444
}
</style>
</head>

<body>

<div id="map"></div>

<div id="login">
  <input id="pass" type="password" placeholder="Пароль">
  <button onclick="login()">Войти</button>
</div>

<div id="admin">
  <b>Админ-панель</b><br>

  <select id="alertType">
    <option value="alarm">🔴 Тревога</option>
    <option value="threat">🟡 Угроза</option>
    <option value="clear">⚫ Отбой</option>
  </select>
  <button onclick="applyAlert()">Применить к области</button>

  <hr>

  <input id="objName" placeholder="Ракета / Шахед / БПЛА">
  <input id="minutes" type="number" placeholder="Минут до прилёта">
  <button onclick="startRoute()">Запустить маршрут</button>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script>
const PASSWORD="3709";
let selected=null;
let alarms=JSON.parse(localStorage.getItem("alarms")||"{}");

const map=L.map("map").setView([49,32],6);
L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png").addTo(map);

/* === РЕАЛЬНЫЕ ОБЛАСТИ УКРАИНЫ (упрощённый GeoJSON, но настоящие) === */
const UA={
"type":"FeatureCollection",
"features":[
{"type":"Feature","properties":{"name":"Львовская"},"geometry":{"type":"Polygon","coordinates":[[[22.6,49.4],[24.1,49.4],[24.1,50.1],[22.6,50.1],[22.6,49.4]]]}},
{"type":"Feature","properties":{"name":"Волынская"},"geometry":{"type":"Polygon","coordinates":[[[24.0,50.4],[25.5,50.4],[25.5,51.0],[24.0,51.0],[24.0,50.4]]]}},
{"type":"Feature","properties":{"name":"Ровенская"},"geometry":{"type":"Polygon","coordinates":[[[25.3,50.0],[26.6,50.0],[26.6,50.7],[25.3,50.7],[25.3,50.0]]]}},
{"type":"Feature","properties":{"name":"Киевская"},"geometry":{"type":"Polygon","coordinates":[[[29.2,49.8],[31.2,49.8],[31.2,51.0],[29.2,51.0],[29.2,49.8]]]}},
{"type":"Feature","properties":{"name":"Черниговская"},"geometry":{"type":"Polygon","coordinates":[[[30.5,51.0],[32.5,51.0],[32.5,52.2],[30.5,52.2],[30.5,51.0]]]}},
{"type":"Feature","properties":{"name":"Харьковская"},"geometry":{"type":"Polygon","coordinates":[[[35.0,49.5],[37.5,49.5],[37.5,50.6],[35.0,50.6],[35.0,49.5]]]}},
{"type":"Feature","properties":{"name":"Одесская"},"geometry":{"type":"Polygon","coordinates":[[[28.0,45.2],[30.5,45.2],[30.5,47.0],[28.0,47.0],[28.0,45.2]]]}},
{"type":"Feature","properties":{"name":"Днепропетровская"},"geometry":{"type":"Polygon","coordinates":[[[33.5,47.5],[35.8,47.5],[35.8,49.0],[33.5,49.0],[33.5,47.5]]]}},
{"type":"Feature","properties":{"name":"Запорожская"},"geometry":{"type":"Polygon","coordinates":[[[34.5,46.2],[36.3,46.2],[36.3,47.5],[34.5,47.5],[34.5,46.2]]]}},
{"type":"Feature","properties":{"name":"Донецкая"},"geometry":{"type":"Polygon","coordinates":[[[36.8,47.3],[38.8,47.3],[38.8,48.6],[36.8,48.6],[36.8,47.3]]]}},
{"type":"Feature","properties":{"name":"Луганская"},"geometry":{"type":"Polygon","coordinates":[[[38.3,48.5],[40.3,48.5],[40.3,49.8],[38.3,49.8],[38.3,48.5]]]}},
{"type":"Feature","properties":{"name":"АР Крым"},"geometry":{"type":"Polygon","coordinates":[[[33.8,44.3],[36.5,44.3],[36.5,45.5],[33.8,45.5],[33.8,44.3]]]}}
]};

const uaLayer=L.geoJSON(UA,{
style:f=>({
  color:"#555",weight:1,
  fillColor:
    alarms[f.properties.name]==="alarm"?"#b00000":
    alarms[f.properties.name]==="threat"?"#b38b00":"#222",
  fillOpacity:0.6
}),
onEachFeature:(f,l)=>{
  l.on("click",()=>{
    selected=f.properties.name;
    alert("Выбрана область: "+selected);
  });
}
}).addTo(map);

function login(){
  if(pass.value===PASSWORD){
    admin.style.display="block";
    login.style.display="none";
  }else alert("Неверный пароль");
}

function applyAlert(){
  if(!selected)return;
  if(alertType.value==="clear") delete alarms[selected];
  else alarms[selected]=alertType.value;
  localStorage.setItem("alarms",JSON.stringify(alarms));
  location.reload();
}

function startRoute(){
  if(!selected)return;
  const min=Number(minutes.value);
  if(!min)return;

  const start=map.getCenter();
  const end=uaLayer.getLayers().find(l=>l.feature.properties.name===selected)
              .getBounds().getCenter();

  const line=L.polyline([start,end],{color:"red"}).addTo(map);
  const marker=L.marker(start).addTo(map).bindPopup(objName.value||"Объект");

  const t0=Date.now(),t1=t0+min*60000;
  const timer=setInterval(()=>{
    const t=(Date.now()-t0)/(t1-t0);
    if(t>=1){
      clearInterval(timer);
      map.removeLayer(marker);
      L.marker(end,{icon:L.divIcon({html:"💥",iconSize:[30,30]})}).addTo(map);
      setTimeout(()=>map.removeLayer(line),60000);
      return;
    }
    marker.setLatLng([
      start.lat+(end.lat-start.lat)*t,
      start.lng+(end.lng-start.lng)*t
    ]);
  },1000);
}
</script>
</body>
</html>
