<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>YCB-89</title>

<style>
body{
  background:#020617;
  color:#e5e7eb;
  font-family:system-ui;
  display:flex;
  justify-content:center;
  padding:20px;
}
.radio{
  width:360px;
  border:1px solid #334155;
  border-radius:16px;
  padding:14px;
}
h1{text-align:center;margin:4px 0;}
.time{text-align:center;font-size:12px;opacity:.7;}
.status{text-align:center;margin:8px 0;font-weight:600;}

.meter{
  height:10px;
  background:#020617;
  border:1px solid #334155;
  border-radius:6px;
  overflow:hidden;
}
.meter>div{
  height:100%;
  width:0%;
  background:#22c55e;
  transition:.1s;
}

.admin-btn{
  text-align:center;
  opacity:.4;
  font-size:12px;
  margin-top:10px;
  cursor:pointer;
}
.admin{
  margin-top:10px;
  border-top:1px solid #334155;
  padding-top:10px;
}
.hidden{display:none;}

button,input{
  width:100%;
  margin-top:6px;
  background:#020617;
  color:#e5e7eb;
  border:1px solid #334155;
  border-radius:8px;
  padding:8px;
}
small{opacity:.6}
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

<hr>

<b>🔊 Звук</b>
<input type="file" id="audioFile" accept="audio/*">
<button id="playAudio">▶ ПУСТИТЬ ЗВУК</button>

<hr>

<b>🗣 Сообщение</b>
<input id="msgText" placeholder="Текст">
<button id="say">📢 ПРОИЗНЕСТИ</button>

<small>
Шум всегда работает, если эфир включён
</small>
</div>

</div>

<script>
/* ===== ВРЕМЯ МСК ===== */
function msk(){
  return new Date(Date.now()+3*3600000)
    .toISOString().substr(11,8);
}
setInterval(()=>clock.textContent="МСК "+msk(),1000);

/* ===== AUDIO CONTEXT ===== */
const ctx=new AudioContext();

/* --- ШУМ --- */
const noiseBuf=ctx.createBuffer(1,ctx.sampleRate*2,ctx.sampleRate);
const data=noiseBuf.getChannelData(0);
for(let i=0;i<data.length;i++)data[i]=Math.random()*2-1;

const noise=ctx.createBufferSource();
noise.buffer=noiseBuf;
noise.loop=true;

const noiseGain=ctx.createGain();
noiseGain.gain.value=0;

const analyser=ctx.createAnalyser();
analyser.fftSize=256;

noise.connect(noiseGain).connect(analyser).connect(ctx.destination);

/* --- УРОВЕНЬ --- */
setInterval(()=>{
  const a=new Uint8Array(analyser.frequencyBinCount);
  analyser.getByteFrequencyData(a);
  const v=a.reduce((s,x)=>s+x,0)/a.length;
  level.style.width=Math.min(100,v/2)+"%";
},100);

/* ===== СОСТОЯНИЕ ===== */
let on=localStorage.getItem("on")==="1";

/* ===== ВОССТАНОВЛЕНИЕ ===== */
(async()=>{
  await ctx.resume();
  noise.start();
  if(on) startAir(); else stopAir();
})();

/* ===== ФУНКЦИИ ===== */
function startAir(){
  noiseGain.gain.value=0.3;
  status.textContent="● В ЭФИРЕ";
  toggle.textContent="⏸ ВЫКЛЮЧИТЬ ЭФИР";
  on=true;
  localStorage.setItem("on","1");
}
function stopAir(){
  noiseGain.gain.value=0;
  status.textContent="● НЕТ СИГНАЛА";
  toggle.textContent="▶ ВКЛЮЧИТЬ ЭФИР";
  on=false;
  localStorage.setItem("on","0");
}

/* ===== ADMIN ===== */
openAdmin.onclick=()=>{
  if(prompt("Пароль")==="3709")
    admin.classList.toggle("hidden");
};

toggle.onclick=async()=>{
  await ctx.resume();
  on ? stopAir() : startAir();
};

/* ===== ЗВУК ===== */
playAudio.onclick=async()=>{
  if(!on) return alert("Эфир выключен");
  if(!audioFile.files[0]) return;

  const reader=new FileReader();
  reader.onload=()=>{
    const a=new Audio(reader.result);
    a.play();
  };
  reader.readAsDataURL(audioFile.files[0]);
};

/* ===== TTS ===== */
say.onclick=()=>{
  if(!on) return alert("Эфир выключен");
  const u=new SpeechSynthesisUtterance(msgText.value);
  u.lang="ru-RU";
  speechSynthesis.speak(u);
};
</script>
</body>
</html>
