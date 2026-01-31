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
  width:360px;
  background:#020617;
  border:1px solid #334155;
  border-radius:18px;
  padding:16px;
}
.title{
  text-align:center;
  font-size:22px;
  font-weight:700;
  letter-spacing:2px;
}
.status{
  text-align:center;
  font-size:12px;
  color:#22c55e;
  margin-bottom:10px;
}
button{
  width:70px;
  height:70px;
  border-radius:50%;
  border:none;
  font-size:24px;
  background:#2563eb;
  color:white;
}
.controls{
  display:flex;
  justify-content:center;
  gap:12px;
  margin:12px 0;
}
.volume input{ width:100%; }
.admin{
  text-align:center;
  font-size:12px;
  opacity:.4;
  margin-top:10px;
  cursor:pointer;
}
.admin-panel{
  margin-top:12px;
  border-top:1px solid #334155;
  padding-top:12px;
  font-size:13px;
}
.hidden{ display:none; }
.list{
  font-size:12px;
  margin-top:6px;
  opacity:.8;
}
.mic-on{ background:#dc2626!important; }
</style>
</head>

<body>
<div class="radio">

<div class="title">📻 YCB-89</div>
<div class="status">● ЭФИР АКТИВЕН</div>

<div class="controls">
  <button id="play">▶</button>
  <button id="mic">🎙</button>
</div>

<div class="volume">
🔊 <input type="range" min="0" max="100" value="60">
</div>

<div class="admin" id="openAdmin">admin</div>

<div class="admin-panel hidden" id="adminPanel">
<b>Админ панель</b><br><br>

<input type="file" id="addSound" accept="audio/*" multiple><br>
<div class="list" id="soundList"></div><br>

<label>
<input type="checkbox" id="randomMode"> случайный сигнал (УВБ)
</label>
</div>

<audio id="audio" loop></audio>

</div>

<script>
const audio = document.getElementById("audio");
const playBtn = document.getElementById("play");
const micBtn = document.getElementById("mic");
const vol = document.querySelector("input[type=range]");
const adminPanel = document.getElementById("adminPanel");
const listDiv = document.getElementById("soundList");
const randomMode = document.getElementById("randomMode");

let playing = false;
let sounds = JSON.parse(localStorage.getItem("sounds")||"[]");
let micStream = null;

// восстановление
if(sounds[0]) audio.src = sounds[0];

// play
playBtn.onclick = ()=>{
  playing = !playing;
  playBtn.textContent = playing ? "⏸" : "▶";
  playing ? audio.play() : audio.pause();
};

vol.oninput = ()=> audio.volume = vol.value/100;

// админка
openAdmin.onclick = ()=>{
  const p = prompt("Пароль:");
  if(p==="3709") adminPanel.classList.toggle("hidden");
};

// добавить звуки
addSound.onchange = e=>{
  for(const f of e.target.files){
    const url = URL.createObjectURL(f);
    sounds.push(url);
  }
  localStorage.setItem("sounds", JSON.stringify(sounds));
  renderList();
};

function renderList(){
  listDiv.innerHTML = sounds.map((s,i)=>`▶ сигнал ${i+1}`).join("<br>");
}
renderList();

// случайный УВБ
setInterval(()=>{
  if(!randomMode.checked || !playing || sounds.length<2) return;
  audio.src = sounds[Math.floor(Math.random()*sounds.length)];
  audio.play();
}, 60000 + Math.random()*60000);

// микрофон
micBtn.onclick = async ()=>{
  if(micStream){
    micStream.getTracks().forEach(t=>t.stop());
    micStream=null;
    micBtn.classList.remove("mic-on");
    audio.muted=false;
    return;
  }
  micStream = await navigator.mediaDevices.getUserMedia({audio:true});
  audio.srcObject = micStream;
  audio.muted=false;
  audio.play();
  micBtn.classList.add("mic-on");
};
</script>
</body>
</html>
