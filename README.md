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
canvas{width:100%;height:100px;margin-top:10px;background:#020617;border:1px solid #334155;border-radius:8px;}
</style>
</head>
<body>
<div class="radio">
<h1>📻 YCB-89</h1>
<div class="time" id="clock"></div>
<div class="status" id="status">● НЕТ СИГНАЛА</div>
<div class="meter"><div id="level"></div></div>
<canvas id="spectrum"></canvas>

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
/* ===== ВРЕМЯ МСК ===== */
const clock=document.getElementById("clock");
setInterval(()=>{
  const d=new Date(Date.now()+3*3600000);
  clock.textContent="МСК "+d.toISOString().substr(11,8);
},100);

/* ===== WebSocket ===== */
const ws = new WebSocket(`ws://${location.host}`);
let air={on:false,currentAudio:null,messages:[]};
ws.onmessage=e=>{
  const msg=JSON.parse(e.data);
  if(msg.type==="state"){ air=msg.air; updateState(); }
};

/* ===== AudioContext и шум ===== */
const ctx=new (window.AudioContext||window.webkitAudioContext)();
const noiseBuf=ctx.createBuffer(1,ctx.sampleRate*2,ctx.sampleRate);
const data=noiseBuf.getChannelData(0);
for(let i=0;i<data.length;i++) data[i]=Math.random()*2-1;
const noise=ctx.createBufferSource();
noise.buffer=noiseBuf;
noise.loop=true;
const gain=ctx.createGain();
gain.gain.value=0;
noise.connect(gain).connect(ctx.destination);
noise.start();

/* ===== Анализатор ===== */
const analyser=ctx.createAnalyser();
gain.connect(analyser);
analyser.fftSize=256;
const canvas=document.getElementById("spectrum");
const ctx2=canvas.getContext("2d");

/* ===== Индикатор уровня и спектр ===== */
setInterval(()=>{
  const a=new Uint8Array(analyser.frequencyBinCount);
  analyser.getByteFrequencyData(a);
  const v=a.reduce((s,x)=>s+x,0)/a.length;
  document.getElementById("level").style.width=Math.min(100,v/2)+"%";

  ctx2.clearRect(0,0,canvas.width,canvas.height);
  const barWidth=canvas.width/a.length;
  for(let i=0;i<a.length;i++){
    const barHeight=a[i];
    ctx2.fillStyle="#22c55e";
    ctx2.fillRect(i*barWidth,canvas.height-barHeight,barWidth,barHeight);
  }
},100);

/* ===== Эфир ===== */
const statusEl=document.getElementById("status");
const toggle=document.getElementById("toggle");
function updateState(){
  if(air.on){
    gain.gain.value=0.15;
    statusEl.textContent="● В ЭФИРЕ";
    toggle.textContent="⏸ ВЫКЛЮЧИТЬ ЭФИР";
  }else{
    gain.gain.value=0;
    statusEl.textContent="● НЕТ СИГНАЛА";
    toggle.textContent="▶ ВКЛЮЧИТЬ ЭФИР";
  }
  if(air.currentAudio){
    player.src="/uploads/"+air.currentAudio;
    player.play();
  }
}

/* ===== ADMIN ===== */
const admin=document.getElementById("admin");
document.getElementById("openAdmin").onclick=()=>{
  const pass=prompt("Введите пароль:");
  if(pass==="3709") admin.classList.toggle("hidden");
  else alert("Неверный пароль!");
};

/* ===== Кнопки ===== */
toggle.onclick=async()=>{
  await ctx.resume();
  ws.send(JSON.stringify({type:"setAir", on:!air.on}));
};
document.getElementById("save").onclick=()=>ws.send(JSON.stringify({type:"setAir", on:air.on}));

/* ===== Аудио ===== */
const player=document.getElementById("player");
document.getElementById("playAudio").onclick=()=>{
  if(!air.on) return alert("Эфир выключен");
  const file=document.getElementById("audioFile").files[0];
  if(!file) return;
  const fd=new FormData();
  fd.append("audio",file);
  fetch("/upload",{method:"POST",body:fd}).then(r=>r.json()).then(j=>{
    player.src="/uploads/"+j.file;
    player.play();
    ws.send(JSON.stringify({type:"setAudio", file:j.file}));
  });
};

/* ===== TTS Сообщение ===== */
document.getElementById("say").onclick=()=>{
  if(!air.on) return alert("Эфир выключен");
  const text=document.getElementById("msgText").value;
  if(!text) return;
  speechSynthesis.speak(new SpeechSynthesisUtterance(text));
  ws.send(JSON.stringify({type:"addMessage", text:text}));
};
</script>
</body>
</html>
const express = require("express");
const http = require("http");
const WebSocket = require("ws");
const multer = require("multer");
const path = require("path");
const fs = require("fs");

const app = express();
const server = http.createServer(app);
const wss = new WebSocket.Server({ server });

// Папка для аудио
const upload = multer({ dest: "public/uploads/" });
app.use(express.static("public"));

// Состояние эфира
let airState = {
  on: false,
  currentAudio: null,
  messages: []
};

// Рассылка состояния всем
function broadcastState() {
  const data = JSON.stringify({ type: "state", air: airState });
  wss.clients.forEach(client => {
    if (client.readyState === WebSocket.OPEN) client.send(data);
  });
}

// WebSocket соединение
wss.on("connection", ws => {
  ws.send(JSON.stringify({ type: "state", air: airState }));

  ws.on("message", msg => {
    try {
      const data = JSON.parse(msg);
      if (data.type === "setAir") {
        airState.on = data.on;
        broadcastState();
      } else if (data.type === "addMessage") {
        airState.messages.push({ text: data.text, time: Date.now() });
        broadcastState();
      } else if (data.type === "setAudio") {
        airState.currentAudio = data.file;
        broadcastState();
      }
    } catch(e) {}
  });
});

// Загрузка аудио
app.post("/upload", upload.single("audio"), (req,res)=>{
  const fileName = req.file.filename + path.extname(req.file.originalname);
  fs.renameSync(req.file.path, path.join("public/uploads", fileName));
  res.json({file: fileName});
});

server.listen(3000, ()=>console.log("Сервер запущен на http://localhost:3000"));
