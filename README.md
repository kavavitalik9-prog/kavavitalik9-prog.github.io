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
}
.time{
  text-align:center;
  font-size:12px;
  opacity:.7;
  margin-bottom:10px;
}
.controls{
  display:flex;
  justify-content:center;
  gap:12px;
  margin:12px 0;
}
button{
  border:none;
  border-radius:12px;
  padding:10px;
  background:#2563eb;
  color:white;
}
.admin-btn{
  text-align:center;
  font-size:12px;
  opacity:.4;
  cursor:pointer;
}
.admin{
  margin-top:12px;
  border-top:1px solid #334155;
  padding-top:12px;
  font-size:13px;
}
.hidden{ display:none; }

.meter{
  width:100%;
  height:10px;
  background:#020617;
  border:1px solid #334155;
  border-radius:6px;
  overflow:hidden;
}
.meter div{
  height:100%;
  width:0%;
  background:#22c55e;
}

textarea,input{
  width:100%;
  background:#020617;
  border:1px solid #334155;
  color:#e5e7eb;
  border-radius:8px;
  padding:6px;
}
label{ font-size:12px; opacity:.7; }
</style>
</head>

<body>
<div class="radio">

<div class="title">📻 YCB-89</div>
<div class="status">● ЭФИР АКТИВЕН</div>
<div class="time" id="mskTime"></div>

<div class="controls">
  <button id="start">▶ ПУСК</button>
</div>

<div class="admin-btn" id="openAdmin">admin</div>

<div class="admin hidden" id="admin">
<b>АДМИН ПАНЕЛЬ</b><br><br>

<button id="micBtn">🎙 МИКРОФОН</button><br><br>

<label>Сила сигнала</label>
<div class="meter"><div id="vu"></div></div><br>

<label>ИИ-сообщение</label>
<textarea id="ttsText" rows="2">Говорит YCB восемь девять</textarea>
<button id="sayNow">▶ ДИКТОВАТЬ</button><br><br>

<label>Сообщение по времени (МСК)</label>
<input id="ttsTime" placeholder="HH:MM">
<button id="addMsg">⏰ ДОБАВИТЬ</button>

</div>

</div>

<script>
/* ====== ВРЕМЯ МСК ====== */
function updateTime(){
  const d=new Date(Date.now()+3*3600000);
  mskTime.textContent=
    "МСК "+d.toISOString().substring(11,19);
}
setInterval(updateTime,1000); updateTime();

/* ====== AUDIO ====== */
const ctx=new AudioContext();
const master=ctx.createGain();
master.connect(ctx.destination);

/* ШУМ */
const noise=ctx.createBufferSource();
const buf=ctx.createBuffer(1,ctx.sampleRate*2,ctx.sampleRate);
const data=buf.getChannelData(0);
for(let i=0;i<data.length;i++) data[i]=Math.random()*2-1;
noise.buffer=buf;
noise.loop=true;
const noiseGain=ctx.createGain();
noiseGain.gain.value=0.15;
noise.connect(noiseGain).connect(master);

/* МИКРОФОН */
let micStream, micGain, analyser;
const vu=document.getElementById("vu");

/* VU */
function drawVU(){
  if(!analyser) return;
  const a=new Uint8Array(analyser.frequencyBinCount);
  analyser.getByteFrequencyData(a);
  const v=a.reduce((s,x)=>s+x,0)/a.length;
  vu.style.width=Math.min(100,v/2)+"%";
  requestAnimationFrame(drawVU);
}

/* КНОПКИ */
start.onclick=()=>{
  ctx.resume();
  noise.start();
  start.disabled=true;
};

openAdmin.onclick=()=>{
  if(prompt("Пароль:")==="3709")
    admin.classList.toggle("hidden");
};

micBtn.onclick=async()=>{
  if(micStream){
    micStream.getTracks().forEach(t=>t.stop());
    micStream=null;
    micBtn.textContent="🎙 МИКРОФОН";
    return;
  }
  micStream=await navigator.mediaDevices.getUserMedia({audio:true});
  const src=ctx.createMediaStreamSource(micStream);
  micGain=ctx.createGain();
  analyser=ctx.createAnalyser();
  src.connect(analyser);
  analyser.connect(micGain).connect(master);
  micBtn.textContent="⛔ ВЫКЛ";
  drawVU();
};

/* ====== ИИ ГОЛОС ====== */
sayNow.onclick=()=>speak(ttsText.value);

function speak(text){
  const u=new SpeechSynthesisUtterance(text);
  u.lang="ru-RU";
  u.rate=0.9;
  speechSynthesis.speak(u);
}

/* ====== СООБЩЕНИЯ ПО ВРЕМЕНИ ====== */
const msgs=[];
addMsg.onclick=()=>{
  msgs.push({t:ttsTime.value,txt:ttsText.value});
};

setInterval(()=>{
  const d=new Date(Date.now()+3*3600000);
  const now=d.toISOString().substring(11,16);
  msgs.forEach(m=>{
    if(m.t===now){
      speak(m.txt);
      m.t=null;
    }
  });
},1000);
</script>
</body>
</html>
