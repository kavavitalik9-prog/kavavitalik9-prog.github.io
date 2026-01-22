<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Карти з супутником</title>
<style>
body, html { margin:0; padding:0; height:100%; font-family:system-ui; }
#map { height:100vh; width:100%; }

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

<div id="map"></div>
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
    <input id="title" placeholder="Назва знімку">
    <input id="lat" placeholder="Широта">
    <input id="lng" placeholder="Довгота">
    <input type="file" id="file">
    <button onclick="addOverlay()">Додати на карту</button>
    <button class="close" onclick="closeAll()">Закрити</button>
  </div>
</div>

<script>
let PASS="3709";
let map;
let overlays=[]; // наші додані знімки

function initMap(){
  map = new google.maps.Map(document.getElementById("map"), {
    center: { lat: 49.8397, lng: 24.0297 }, // Львів
    zoom: 12,
    mapTypeId: 'satellite'
  });

  // Завантаження збережених overlay з localStorage
  let saved = JSON.parse(localStorage.getItem("overlays"))||[];
  saved.forEach(o=>{
    createOverlay(o);
  });
}

function createOverlay(o){
  const imageBounds = new google.maps.LatLngBounds(
    { lat: o.lat-0.01, lng: o.lng-0.01 },
    { lat: o.lat+0.01, lng: o.lng+0.01 }
  );
  const overlay = new google.maps.GroundOverlay(o.img, imageBounds);
  overlay.setMap(map);
  overlay.title=o.title;
  overlay.time=o.time;
  overlays.push(overlay);
}

/* ADMIN */
function openLogin(){login.style.display="flex"}
function closeAll(){login.style.display="none";admin.style.display="none"}

function check(){
  if(pass.value!==PASS){alert("Невірний пароль");return;}
  login.style.display="none";
  admin.style.display="flex";
}

function addOverlay(){
  let f=file.files[0];
  if(!f){alert("Обери файл"); return;}
  let t=parseFloat(lat.value);
  let g=parseFloat(lng.value);
  if(isNaN(t)||isNaN(g)){alert("Введи координати"); return;}

  let r=new FileReader();
  r.onload=()=>{
    let o={
      title:title.value||"Без назви",
      lat:t,
      lng:g,
      img:r.result,
      time:Date.now()
    };
    createOverlay(o);
    let saved = JSON.parse(localStorage.getItem("overlays"))||[];
    saved.unshift(o);
    localStorage.setItem("overlays",JSON.stringify(saved));
    closeAll();
  }
  r.readAsDataURL(f);
}
</script>

<script src="https://maps.googleapis.com/maps/api/js?key=ВАШ_API_KEY&callback=initMap" async defer></script>
</body>
</html>
