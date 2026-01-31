<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>YCB-89</title>
<style>
body{background:#020617;color:#e5e7eb;font-family:system-ui;display:flex;justify-content:center;padding:20px;}
.radio{width:360px;border:1px solid #334155;border-radius:16px;padding:14px;}
h1{text-align:center;margin:4px 0;}
.time{text-align:center;font-size:12px;opacity:.7;}
.status{text-align:center;margin:8px 0;font-weight:600;}
.meter{height:10px;background:#020617;border:1px solid #334155;border-radius:6px;overflow:hidden;}
.meter>div{height:100%;width:0%;background:#22c55e;transition:.1s;}
.admin-btn{text-align:center;opacity:.4;font-size:12px;margin-top:10px;cursor:pointer;}
.admin{margin-top:10px;border-top:1px solid #334155;padding-top:10px;}
.hidden{display:none;}
button,input{width:100%;margin-top:6px;background:#020617;color:#e5e7eb;border:1px solid #334155;border-radius:8px;padding:8px;}
</style>
</head>
<body>
<div class="radio">
<h1>📻 YCB-89</h1>
<div class="time" id="clock"></div>
<div class="status" id="status">● НЕТ СИГНАЛА</div>
<div class="meter"><div id="level"></div></div>
<div class="admin-btn" id="openAdmin">admin</div>

<div class="admin hidden" id="admin">
<button id="toggle">▶ ВКЛЮЧИТЬ ЭФИР</button>
<button id="save">💾 СОХРАНИТЬ СОСТОЯНИЕ</button>

<hr>
<b>🔊 Звук</b>
<input type="file" id="audioFile" accept="audio/*">
<button id="playAudio">▶ ПУСТИТЬ ЗВУК</button>

<hr>
<b>🗣 Сообщение</b>
<input id="msgText" placeholder="Текст сообщения">
<button id="say">📢 ПРОИЗНЕСТИ</button>
</div>

<audio id="player" loop></audio>
</div>

<script>
const clock=document.getElementById("clock");
setInterval(()=>clock.textContent="МСК "+new Date(Date.now()+3*3600000).toISOString().substr(11,8),1000);

/* WebSocket */
const ws = new WebSocket(`ws://${location.host}`);
let air={on:false,currentAudio:null,messages:[]};
ws.onmessage = e=>{
  const msg=JSON.parse(e.data);
  if(msg.type==="state"){
    air = msg.air;
    updateState();
  }
};

/* AudioContext шум */
const ctx = new (window.AudioContext||window.webkitAudioContext)();
const noiseBuf = ctx.createBuffer(1, ctx.sampleRate*2, ctx.sampleRate);
const data = noiseBuf.getChannelData(0);
for(let i=0;i<data.length;i++) data[i]=Math.random()*2-1;
const noise = ctx.createBufferSource();
noise.buffer = noiseBuf;
noise.loop = true;
const gain = ctx.createGain();
gain.gain.value = 0;
noise.connect(gain).connect(ctx.destination);
noise.start();

/* Уровень */
const analyser = ctx.createAnalyser();
gain.connect(analyser);
setInterval(()=>{
  const a=new Uint8Array(analyser.frequencyBinCount);
  analyser.getByteFrequencyData(a);
  const v=a.reduce((s,x)=>s+x,0)/a.length;
  document.getElementById("level").style.width = Math.min(100,v/2)+"%";
},100);

const statusEl = document.getElementById("status");
const toggle = document.getElementById("toggle");

/* Эфир */
function updateState(){
  if(air.on){
    gain.gain.value = 0.15;
    statusEl.textContent="● В ЭФИРЕ";
    toggle.textContent="⏸ ВЫКЛЮЧИТЬ ЭФИР";
  }else{
    gain.gain.value = 0;
    statusEl.textContent="● НЕТ СИГНАЛА";
    toggle.textContent="▶ ВКЛЮЧИТЬ ЭФИР";
  }
}

/* ADMIN */
const admin=document.getElementById("admin");
document.getElementById("openAdmin").onclick=()=>{
  if(prompt("Пароль")==="3709") admin.classList.toggle("hidden");
};

toggle.onclick=async()=>{
  await ctx.resume();
  ws.send(JSON.stringify({type:"setAir",on:!air.on}));
};

document.getElementById("save").onclick=()=>{
  ws.send(JSON.stringify({type:"setAir",on:air.on}));
  alert("Состояние сохранено для всех");
};

/* Плей аудио */
const player=document.getElementById("player");
document.getElementById("playAudio").onclick=()=>{
  if(!air.on) return alert("Эфир выключен");
  const file=document.getElementById("audioFile").files[0];
  if(!file) return;
  const fd=new FormData();
  fd.append("audio",file);
  fetch("/upload",{method:"POST",body:fd})
    .then(r=>r.json())
    .then(j=>{
      player.src="/uploads/"+j.file;
      player.play();
      ws.send(JSON.stringify({type:"setAudio",file:j.file}));
    });
};

/* TTS */
document.getElementById("say").onclick=()=>{
  if(!air.on) return alert("Эфир выключен");
  const text=document.getElementById("msgText").value;
  if(!text) return;
  speechSynthesis.speak(new SpeechSynthesisUtterance(text));
  ws.send(JSON.stringify({type:"addMessage",text:text}));
};
</script>
</body>
</html>
