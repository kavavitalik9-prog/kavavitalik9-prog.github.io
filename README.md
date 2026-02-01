<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Карта тревог</title>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
body{
  margin:0;
  background:#0b0b0b;
  color:#fff;
  font-family:Arial,sans-serif;
}
header{
  padding:10px;
  text-align:center;
  background:#111;
  font-size:20px;
}
#container{
  display:flex;
  height:calc(100vh - 50px);
}
#map{
  flex:1;
  overflow:auto;
}
svg{
  width:100%;
  height:auto;
}
.region{
  fill:#2a2a2a;
  stroke:#555;
  stroke-width:0.7;
  cursor:pointer;
  transition:.15s;
}
.region:hover{opacity:.85}
.alarm{fill:#b00000}
.threat{fill:#b38b00}
.selected{stroke:#fff;stroke-width:2}

#admin{
  width:260px;
  background:#111;
  border-left:1px solid #333;
  padding:10px;
}
#admin h3{margin-top:0}
button,select,input{
  width:100%;
  margin-top:8px;
  padding:6px;
  background:#222;
  color:#fff;
  border:1px solid #444;
}
#info{
  margin-top:10px;
  font-size:14px;
  min-height:40px;
}
.small{font-size:12px;color:#aaa}
</style>
</head>
<body>

<header>
🗺 Карта воздушных тревог
<div id="time" class="small"></div>
</header>

<div id="container">
<div id="map">
<svg viewBox="0 0 1200 700">

<!-- УКРАИНА -->
<g id="UA">
<path id="UA-Lviv" class="region" d="M180 300 L240 270 L290 310 L250 360 L190 340 Z"/>
<path id="UA-Volyn" class="region" d="M210 240 L270 220 L300 260 L240 270 Z"/>
<path id="UA-Rivne" class="region" d="M270 260 L330 240 L360 290 L300 310 Z"/>
<path id="UA-Ternopil" class="region" d="M250 360 L300 310 L350 350 L300 390 Z"/>
<path id="UA-Ivano" class="region" d="M230 390 L300 390 L320 440 L260 460 Z"/>
<path id="UA-Zakarpattia" class="region" d="M180 420 L260 460 L220 500 L160 460 Z"/>
<path id="UA-Chernivtsi" class="region" d="M320 440 L380 430 L390 480 L330 500 Z"/>
<path id="UA-Khmelnytskyi" class="region" d="M350 350 L410 330 L440 380 L380 430 Z"/>
<path id="UA-Vinnytsia" class="region" d="M350 390 L440 380 L460 430 L390 480 Z"/>
<path id="UA-Kyiv" class="region" d="M460 260 L520 250 L550 300 L500 330 Z"/>
<path id="UA-Zhytomyr" class="region" d="M360 290 L460 260 L410 330 Z"/>
<path id="UA-Cherkasy" class="region" d="M460 330 L520 300 L550 360 L490 390 Z"/>
<path id="UA-Kirovohrad" class="region" d="M490 390 L550 360 L580 420 L520 450 Z"/>
<path id="UA-Poltava" class="region" d="M550 300 L620 280 L650 330 L580 360 Z"/>
<path id="UA-Sumy" class="region" d="M520 210 L600 200 L620 280 L550 250 Z"/>
<path id="UA-Chernihiv" class="region" d="M460 200 L540 180 L600 200 L520 210 Z"/>
<path id="UA-Kharkiv" class="region" d="M650 330 L720 310 L750 360 L690 390 Z"/>
<path id="UA-Dnipro" class="region" d="M580 360 L650 330 L690 390 L630 420 Z"/>
<path id="UA-Zaporizhzhia" class="region" d="M630 420 L690 390 L720 440 L660 470 Z"/>
<path id="UA-Donetsk" class="region" d="M720 360 L800 350 L820 420 L750 440 Z"/>
<path id="UA-Luhansk" class="region" d="M800 300 L880 310 L900 370 L820 420 Z"/>
<path id="UA-Mykolaiv" class="region" d="M520 450 L580 420 L610 470 L550 500 Z"/>
<path id="UA-Odessa" class="region" d="M450 480 L550 500 L520 560 L440 540 Z"/>
<path id="UA-Kherson" class="region" d="M580 470 L660 470 L640 540 L560 520 Z"/>
<path id="UA-Crimea" class="region" d="M660 540 L740 560 L720 620 L640 600 Z"/>
</g>

<!-- РОССИЯ (упрощённо) -->
<g id="RU">
<path id="RU-Rostov" class="region" d="M820 420 L900 420 L920 470 L850 490 Z"/>
<path id="RU-Belgorod" class="region" d="M760 300 L820 300 L820 350 L760 350 Z"/>
<path id="RU-Kursk" class="region" d="M720 260 L780 260 L760 300 L700 300 Z"/>
<path id="RU-Bryansk" class="region" d="M660 240 L720 240 L720 280 L660 280 Z"/>
</g>

</svg>
</div>

<div id="admin">
<h3>👑 Админ</h3>
<input id="pass" type="password" placeholder="Пароль">
<select id="mode">
  <option value="alarm">🔴 Тревога</option>
  <option value="threat">🟡 Угроза</option>
  <option value="none">⚫ Отбой</option>
</select>
<button onclick="apply()">Применить к выбранным</button>
<button onclick="clearSel()">Снять выделение</button>
<div id="info"></div>
</div>
</div>

<script>
const PASSWORD="3709";
let data=JSON.parse(localStorage.getItem("alarmMap")||"{}");
let selected=new Set();

function nowMSK(){
  return new Date(Date.now()+3*3600000);
}
setInterval(()=>{
  time.textContent="МСК: "+nowMSK().toLocaleTimeString();
},1000);

document.querySelectorAll(".region").forEach(r=>{
  r.onclick=()=>{
    r.classList.toggle("selected");
    if(selected.has(r.id))selected.delete(r.id);
    else selected.add(r.id);
    showInfo();
  };
});

function apply(){
  if(pass.value!==PASSWORD){alert("Пароль");return;}
  selected.forEach(id=>{
    if(mode.value==="none") delete data[id];
    else data[id]={status:mode.value,start:Date.now()};
  });
  localStorage.setItem("alarmMap",JSON.stringify(data));
  render();
}

function render(){
  document.querySelectorAll(".region").forEach(r=>{
    r.classList.remove("alarm","threat");
    if(data[r.id]) r.classList.add(data[r.id].status);
  });
  showInfo();
}
render();

function showInfo(){
  let t=[];
  selected.forEach(id=>{
    if(data[id]){
      let m=Math.floor((Date.now()-data[id].start)/60000);
      t.push(id+" — "+m+" мин");
    }else t.push(id+" — нет сигнала");
  });
  info.innerHTML=t.join("<br>");
}
function clearSel(){
  selected.clear();
  document.querySelectorAll(".selected").forEach(e=>e.classList.remove("selected"));
  info.innerHTML="";
}
</script>

</body>
</html>
