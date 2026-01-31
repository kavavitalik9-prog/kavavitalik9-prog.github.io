<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>YCB-89</title>

<style>
body{
  margin:0;
  background:#020617;
  color:#e5e7eb;
  font-family:system-ui,sans-serif;
  display:flex;
  justify-content:center;
  align-items:center;
  height:100vh;
}
.radio{
  width:340px;
  border:1px solid #334155;
  border-radius:16px;
  padding:16px;
  text-align:center;
}
.title{
  font-size:22px;
  font-weight:700;
  letter-spacing:1px;
}
.status{
  font-size:12px;
  color:#22c55e;
  margin-bottom:10px;
}
.track{
  font-size:14px;
  margin:12px 0;
  opacity:.8;
}
button{
  width:64px;
  height:64px;
  border-radius:50%;
  border:none;
  font-size:22px;
  background:#2563eb;
  color:white;
  cursor:pointer;
}
.volume{
  margin-top:14px;
}
input[type=range]{ width:100%; }
.admin{
  font-size:11px;
  opacity:.35;
  margin-top:10px;
  cursor:pointer;
}
.hidden{ display:none; }
.admin-panel{
  margin-top:10px;
  border-top:1px solid #334155;
  padding-top:10px;
  font-size:13px;
}
label{ cursor:pointer; }
</style>
</head>

<body>
<div class="radio">

<div class="title">📻 YCB-89</div>
<div class="status">● ЭФИР АКТИВЕН 24/7</div>

<div class="track" id="trackName">Шум эфира</div>

<button id="play">▶</button>

<div class="volume">
🔊 <input type="range" min="0" max="100" value="60">
</div>

<div class="admin" id="openAdmin">admin</div>

<div class="admin-panel hidden" id="adminPanel">
  <input type="file" id="file" accept="audio/*"><br><br>
  <label>
    <input type="checkbox" id="uvMode"> режим УВБ-76
  </label>
</div>

<audio id="audio" loop></audio>

</div>

<script>
const audio = document.getElementById("audio");
const playBtn = document.getElementById("play");
const vol = document.querySelector("input[type=range]");
const trackName = document.getElementById("trackName");
const adminPanel = document.getElementById("adminPanel");
const uvMode = document.getElementById("uvMode");

let playing = false;

// восстановление звука
const savedAudio = localStorage.getItem("radioAudio");
if(savedAudio){
  audio.src = savedAudio;
}

// play / pause
playBtn.onclick = ()=>{
  playing = !playing;
  playBtn.textContent = playing ? "⏸" : "▶";
  playing ? audio.play() : audio.pause();
};

// громкость
vol.oninput = ()=> audio.volume = vol.value/100;

// без перемотки
audio.controls = false;

// админка
openAdmin.onclick = ()=>{
  const pass = prompt("Пароль:");
  if(pass === "3709"){
    adminPanel.classList.toggle("hidden");
  }
};

// загрузка нового аудио
file.onchange = e=>{
  const f = e.target.files[0];
  if(!f) return;
  const url = URL.createObjectURL(f);
  audio.src = url;
  localStorage.setItem("radioAudio", url);
  trackName.textContent = f.name;
};

// УВБ-режим — случайные «провалы»
setInterval(()=>{
  if(!uvMode.checked || !playing) return;
  const oldVol = audio.volume;
  audio.volume = 0.15;
  setTimeout(()=>audio.volume = oldVol, 1200);
}, Math.random()*60000 + 30000);
</script>

</body>
</html>
