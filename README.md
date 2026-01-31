<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>YCB-89 Локальное Радио</title>
<style>
body{background:#020617;color:#e5e7eb;font-family:system-ui;display:flex;justify-content:center;padding:20px;}
.radio{width:360px;border:1px solid #334155;border-radius:16px;padding:14px;}
h1{text-align:center;margin:4px 0;}
.time{text-align:center;font-size:12px;opacity:.7;}
.status{text-align:center;margin:8px 0;font-weight:600;}
.meter{height:40px;background:#020617;border:1px solid #334155;border-radius:6px;overflow:hidden;position:relative;}
.meter>div{height:100%;width:0%;background:#22c55e;transition:.05s;}
.meter span{position:absolute;right:4px;top:0;color:#e5e7eb;font-size:12px;line-height:40px;}
.admin-btn{text-align:center;opacity:.4;font-size:12px;margin-top:10px;cursor:pointer;}
.admin{margin-top:10px;border-top:1px solid #334155;padding-top:10px;}
.hidden{display:none;}
button,input{width:100%;margin-top:6px;background:#020617;color:#e5e7eb;border:1px solid #334155;border-radius:8px;padding:8px;}
label{font-size:12px;margin-top:6px;display:block;}
canvas{width:100%;height:100px;margin-top:10px;background:#020617;border:1px solid #334155;border-radius:8px;}
.section{margin-top:10px;}
</style>
</head>
<body>
<div class="radio">
<h1>📻 YCB-89</h1>
<div class="time" id="clock"></div>
<div class="status" id="status">● НЕТ СИГНАЛА</div>

<div class="meter">
  <div id="level"></div>
  <span id="percent">0%</span>
</div>

<canvas id="spectrum"></canvas>

<div class="admin-btn" id="openAdmin">admin</div>

<div class="admin hidden" id="admin">
<button id="toggle">▶ ВКЛЮЧИТЬ ЭФИР</button>
<button id="pauseNoise">⏸ Пауза шума</button>
<button id="save">💾 СОХРАНИТЬ СОСТОЯНИЕ</button>

<hr>
<label>Громкость (шум + аудио + сообщения):</label>
<input type="range" id="masterVolume" min="0" max="100" value="100">

<hr>
<b>Аудио из файла</b>
<input type="file" id="audioFile" accept="audio/*">
<button id="playAudio">▶ ПУСТИТЬ ЗВУК</button>

<hr>
<b>Сообщение</b>
<input id="msgText" placeholder="Текст сообщения">
<button id="say">📢 ПРОИЗНЕСТИ</button>
</div>

<audio id="player" loop></audio>
</div>

<script>
/* ===== Время МСК ===== */
const clock=document.getElementById("clock");
setInterval(()=>{
  const d=new Date(Date.now()+3*3600000);
  clock.textContent="МСК "+d.toISOString().substr(11,8);
},1000);

/* ===== Состояние эфира ===== */
let air = {
  on: false,
  currentAudio: null,
  messages: [],
  noisePaused: false,
  volume: 100
};

/* ===== AudioContext ===== */
const ctx = new (window.AudioContext||window.webkitAudioContext)();

/* ===== Шум ===== */
const noiseBuf = ctx.createBuffer(1, ctx.sampleRate*2, ctx.sampleRate);
const data = noiseBuf.getChannelData(0);
for(let i=0;i<data.length;i++) data[i]=Math.random()*2-1;
const noise = ctx.createBufferSource();
noise.buffer=noiseBuf;
noise.loop=true;
const noiseGain=ctx.createGain();
noiseGain.gain.value = air.volume/100;
noise.connect(noiseGain).connect(ctx.destination);
noise.start();

/* ===== Аудио ===== */
const player=document.getElementById("player");
const playerGain = ctx.createGain();
playerGain.gain.value = air.volume/100;
const audioSource = ctx.createMediaElementSource(player);
audioSource.connect(playerGain).connect(ctx.destination);

/* ===== Анализатор ===== */
const analyser = ctx.createAnalyser();
noiseGain.connect(analyser);
playerGain.connect(analyser);
analyser.fftSize = 256;
const canvas=document.getElementById("spectrum");
const ctx2=canvas.getContext("2d");

/* ===== Индикатор ===== */
const levelDiv=document.getElementById("level");
const percentSpan=document.getElementById("percent");

function updateLevels(){
  const a=new Uint8Array(analyser.frequencyBinCount);
  analyser.getByteFrequencyData(a);
  const avg = a.reduce((s,x)=>s+x,0)/a.length;
  const percent = Math.min(1000, Math.floor(avg*10)); // до 1000%
  levelDiv.style.width = Math.min(100, avg/2)+"%";
  percentSpan.textContent = percent+"%";

  ctx2.clearRect(0,0,canvas.width,canvas.height);
  const barWidth=canvas.width/a.length;
  for(let i=0;i<a.length;i++){
    const barHeight=a[i];
    ctx2.fillStyle="#22c55e";
    ctx2.fillRect(i*barWidth,canvas.height-barHeight,barWidth,barHeight);
  }
}
setInterval(updateLevels,50);

/* ===== Эфир ===== */
const statusEl=document.getElementById("status");
const toggle=document.getElementById("toggle");
const pauseNoiseBtn=document.getElementById("pauseNoise");

function updateState(){
  if(air.on){
    noiseGain.gain.value = air.noisePaused ? 0 : air.volume/100;
    playerGain.gain.value = air.volume/100;
    statusEl.textContent="● В ЭФИРЕ";
    toggle.textContent="⏸ ВЫКЛЮЧИТЬ ЭФИР";
  }else{
    noiseGain.gain.value = 0;
    playerGain.gain.value = 0;
    statusEl.textContent="● НЕТ СИГНАЛА";
    toggle.textContent="▶ ВКЛЮЧИТЬ ЭФИР";
  }

  if(air.currentAudio){
    player.src = URL.createObjectURL(air.currentAudio);
    player.play();
  }
}

/* ===== Admin ===== */
const admin=document.getElementById("admin");
document.getElementById("openAdmin").onclick = ()=>{
  const pass = prompt("Введите пароль:");
  if(pass==="3709") admin.classList.toggle("hidden");
  else alert("Неверный пароль!");
};

/* ===== Кнопки ===== */
toggle.onclick = async ()=>{
  await ctx.resume();
  air.on = !air.on;
  updateState();
};

/* ===== Пауза шума ===== */
pauseNoiseBtn.onclick = ()=>{
  air.noisePaused = !air.noisePaused;
  updateState();
  pauseNoiseBtn.textContent = air.noisePaused ? "▶ Включить шум" : "⏸ Пауза шума";
};

/* ===== Сохранение состояния ===== */
document.getElementById("save").onclick = ()=>{
  localStorage.setItem("airState", JSON.stringify(air));
  alert("Состояние сохранено");
};

/* ===== Ползунок громкости ===== */
document.getElementById("masterVolume").oninput = e=>{
  air.volume = e.target.value;
  noiseGain.gain.value = air.on && !air.noisePaused ? air.volume/100 : 0;
  playerGain.gain.value = air.volume/100;
};

/* ===== Аудио ===== */
document.getElementById("playAudio").onclick = ()=>{
  if(!air.on) return alert("Эфир выключен");
  const file = document.getElementById("audioFile").files[0];
  if(!file) return;
  air.currentAudio = file;
  player.src = URL.createObjectURL(file);
  player.play();
  updateState();
};

/* ===== TTS Сообщение ===== */
document.getElementById("say").onclick = ()=>{
  if(!air.on) return alert("Эфир выключен");
  const text = document.getElementById("msgText").value;
  if(!text) return;
  const msgUtter = new SpeechSynthesisUtterance(text);
  msgUtter.volume = air.volume/100;
  speechSynthesis.speak(msgUtter);
  air.messages.push({text: text, time: Date.now()});
};

/* ===== Загрузка состояния при старте ===== */
const saved = localStorage.getItem("airState");
if(saved){
  air = JSON.parse(saved);
  updateState();
  document.getElementById("masterVolume").value = air.volume;
  pauseNoiseBtn.textContent = air.noisePaused ? "▶ Включить шум" : "⏸ Пауза шума";
}
</script>
</body>
</html>
