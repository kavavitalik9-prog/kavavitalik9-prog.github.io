<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>YCB-89 Радио УВЬ-76</title>
<style>
body{background:#020617;color:#e5e7eb;font-family:system-ui;display:flex;justify-content:center;padding:20px;}
.radio{width:500px;border:1px solid #334155;border-radius:16px;padding:14px;}
h1{text-align:center;margin:4px 0;}
.time{text-align:center;font-size:12px;opacity:.7;}
.status{text-align:center;margin:8px 0;font-weight:600;}
.admin-btn{text-align:center;opacity:.4;font-size:12px;margin-top:10px;cursor:pointer;}
.admin{margin-top:10px;border-top:1px solid #334155;padding-top:10px;}
.hidden{display:none;}
button,input{width:100%;margin-top:6px;background:#020617;color:#e5e7eb;border:1px solid #334155;border-radius:8px;padding:8px;}
label{font-size:12px;margin-top:6px;display:block;}
canvas{width:100%;height:200px;margin-top:10px;background:#020617;border:1px solid #334155;border-radius:8px;display:block;}
.legend{display:flex;justify-content:space-around;margin-top:4px;font-size:12px;}
.legend div{display:flex;align-items:center;}
.legend span{width:12px;height:12px;margin-right:4px;display:inline-block;}
</style>
</head>
<body>
<div class="radio">
<h1>📻 YCB-89 УВЬ-76</h1>
<div class="time" id="clock"></div>
<div class="status" id="status">● НЕТ СИГНАЛА</div>

<canvas id="spectrum"></canvas>
<div class="legend">
  <div><span style="background:#22c55e"></span>Шум</div>
  <div><span style="background:#3b82f6"></span>Аудио</div>
  <div><span style="background:#facc15"></span>Сообщение</div>
</div>

<div class="admin-btn" id="openAdmin">admin</div>

<div class="admin hidden" id="admin">
<button id="toggle">▶ ВКЛЮЧИТЬ ЭФИР</button>
<button id="pauseNoise">⏸ Пауза шума</button>
<button id="save">💾 СОХРАНИТЬ СОСТОЯНИЕ</button>

<hr>
<label>Громкость шума (0-1000%)</label>
<input type="number" id="noiseVolumeInput" min="0" max="1000" value="150">
<input type="range" id="noiseVolumeRange" min="0" max="1000" value="150">

<hr>
<label>Громкость аудио (0-1000%)</label>
<input type="number" id="audioVolumeInput" min="0" max="1000" value="1000">
<input type="range" id="audioVolumeRange" min="0" max="1000" value="1000">
<input type="file" id="audioFile" accept="audio/*">
<button id="playAudio">▶ ПУСТИТЬ ЗВУК</button>

<hr>
<label>Громкость сообщений (0-1000%)</label>
<input type="number" id="msgVolumeInput" min="0" max="1000" value="1000">
<input type="range" id="msgVolumeRange" min="0" max="1000" value="1000">
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
  noiseVolume:150,
  audioVolume:1000,
  msgVolume:1000
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
noiseGain.gain.value = air.noiseVolume/1000;
noise.connect(noiseGain).connect(ctx.destination);
noise.start();

/* ===== Аудио ===== */
const player=document.getElementById("player");
const playerGain = ctx.createGain();
playerGain.gain.value = air.audioVolume/1000;
const audioSource = ctx.createMediaElementSource(player);
audioSource.connect(playerGain).connect(ctx.destination);

/* ===== Анализатор ===== */
const analyser = ctx.createAnalyser();
noiseGain.connect(analyser);
playerGain.connect(analyser);
analyser.fftSize = 256;
const canvas=document.getElementById("spectrum");
const ctx2=canvas.getContext("2d");

/* ===== УВЬ-76 визуализатор ===== */
const barsCount = 64;
function drawSpectrum(){
  ctx2.clearRect(0,0,canvas.width,canvas.height);
  const barWidth = canvas.width/barsCount;

  for(let i=0;i<barsCount;i++){
    const nh = (air.on && !air.noisePaused) ? air.noiseVolume/1000*canvas.height : 0;
    const ah = (air.on && air.currentAudio && !player.paused) ? air.audioVolume/1000*canvas.height : 0;
    const mh = (air.on && speechSynthesis.speaking) ? air.msgVolume/1000*canvas.height : 0;

    // шум
    if(nh>0.5){
      ctx2.fillStyle="#22c55e";
      ctx2.fillRect(i*barWidth,canvas.height-nh,barWidth-2,nh);
    }

    // аудио
    if(ah>0.5){
      ctx2.fillStyle="#3b82f6";
      ctx2.fillRect(i*barWidth,canvas.height-ah,barWidth-2,ah);
    }

    // сообщение
    if(mh>0.5){
      ctx2.fillStyle="#facc15";
      ctx2.fillRect(i*barWidth,canvas.height-mh,barWidth-2,mh);
    }
  }

  // шкала слева 0-100%
  ctx2.fillStyle="#e5e7eb";
  ctx2.font="10px system-ui";
  for(let p=0;p<=100;p+=10){
    const y = canvas.height - (p/100)*canvas.height;
    ctx2.fillText(p+"%",2,y+4);
    ctx2.beginPath();
    ctx2.moveTo(20,y);
    ctx2.lineTo(canvas.width,y);
    ctx2.strokeStyle="#334155";
    ctx2.stroke();
  }

  requestAnimationFrame(drawSpectrum);
}
drawSpectrum();

/* ===== Эфир ===== */
const statusEl=document.getElementById("status");
const toggle=document.getElementById("toggle");
const pauseNoiseBtn=document.getElementById("pauseNoise");

function updateState(){
  if(air.on){
    noiseGain.gain.value = air.noisePaused ? 0 : air.noiseVolume/1000;
    playerGain.gain.value = air.audioVolume/1000;
    statusEl.textContent="● В ЭФИРЕ";
    toggle.textContent="⏸ ВЫКЛЮЧИТЬ ЭФИР";
  }else{
    noiseGain.gain.value = 0;
    playerGain.gain.value = 0;
    statusEl.textContent="● НЕТ СИГНАЛА";
    toggle.textContent="▶ ВКЛЮЧИТЬ ЭФИР";
  }
  if(air.currentAudio){ player.src = URL.createObjectURL(air.currentAudio); player.play(); }
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
pauseNoiseBtn.onclick = ()=>{
  air.noisePaused = !air.noisePaused;
  updateState();
  pauseNoiseBtn.textContent = air.noisePaused ? "▶ Включить шум" : "⏸ Пауза шума";
};
document.getElementById("save").onclick = ()=>{
  localStorage.setItem("airState", JSON.stringify(air));
  alert("Состояние сохранено");
};

/* ===== Настройка громкости ===== */
function syncVolume(input, range, key){
  input.oninput = ()=>{ air[key] = parseInt(input.value); range.value = input.value; updateState(); }
  range.oninput = ()=>{ air[key] = parseInt(range.value); input.value = range.value; updateState(); }
}
syncVolume(document.getElementById("noiseVolumeInput"),document.getElementById("noiseVolumeRange"),'noiseVolume');
syncVolume(document.getElementById("audioVolumeInput"),document.getElementById("audioVolumeRange"),'audioVolume');
syncVolume(document.getElementById("msgVolumeInput"),document.getElementById("msgVolumeRange"),'msgVolume');

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
  msgUtter.volume = air.msgVolume/1000;
  speechSynthesis.speak(msgUtter);
  air.messages.push({text: text, time: Date.now()});
};

/* ===== Загрузка состояния при старте ===== */
const saved = localStorage.getItem("airState");
if(saved){
  air = JSON.parse(saved);
  updateState();
  document.getElementById("noiseVolumeInput").value = air.noiseVolume;
  document.getElementById("noiseVolumeRange").value = air.noiseVolume;
  document.getElementById("audioVolumeInput").value = air.audioVolume;
  document.getElementById("audioVolumeRange").value = air.audioVolume;
  document.getElementById("msgVolumeInput").value = air.msgVolume;
  document.getElementById("msgVolumeRange").value = air.msgVolume;
  pauseNoiseBtn.textContent = air.noisePaused ? "▶ Включить шум" : "⏸ Пауза шума";
}
</script>
</body>
</html>
