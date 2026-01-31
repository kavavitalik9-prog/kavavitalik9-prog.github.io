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
  padding-top:20px;
}
.radio{
  width:360px;
  border:1px solid #334155;
  border-radius:16px;
  padding:14px;
}
h1{text-align:center;margin:6px 0;}
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
}
.admin{
  margin-top:10px;
  border-top:1px solid #334155;
  padding-top:10px;
}
.hidden{display:none;}
button{
  width:100%;
  margin-top:6px;
  background:#020617;
  color:#e5e7eb;
  border:1px solid #334155;
  border-radius:8px;
  padding:8px;
}
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
<button id="mic">🎙 МИКРОФОН</button>
</div>

</div>

<script>
/* ===== ВРЕМЯ МСК ===== */
function msk(){
  return new Date(Date.now()+3*3600000)
    .toISOString().substr(11,8);
}
setInterval(()=>clock.textContent="МСК "+msk(),1000);

/* ===== AUDIO ===== */
const ctx=new AudioContext();
const noise=ctx.createBufferSource();
const buffer=ctx.createBuffer(1,ctx.sampleRate*2,ctx.sampleRate);
const data=buffer.getChannelData(0);
for(let i=0;i<data.length;i++)data[i]=Math.random()*2-1;
noise.buffer=buffer;
noise.loop=true;

const gain=ctx.createGain();
const analyser=ctx.createAnalyser();
analyser.fftSize=256;

noise.connect(gain).connect(analyser).connect(ctx.destination);

let on=false;
let micStream=null;

/* ===== УРОВЕНЬ ===== */
setInterval(()=>{
  const a=new Uint8Array(analyser.frequencyBinCount);
  analyser.getByteFrequencyData(a);
  const v=a.reduce((s,x)=>s+x,0)/a.length;
  level.style.width=Math.min(100,v/2)+"%";
},100);

/* ===== ADMIN ===== */
openAdmin.onclick=()=>{
  if(prompt("Пароль")==="3709")
    admin.classList.toggle("hidden");
};

toggle.onclick=async()=>{
  await ctx.resume();
  if(!on){
    noise.start();
    gain.gain.value=0.3;
    status.textContent="● В ЭФИРЕ";
    toggle.textContent="⏸ ПАУЗА";
    on=true;
  }else{
    gain.gain.value=0;
    status.textContent="● В ЭФИРЕ (ПАУЗА)";
    toggle.textContent="▶ ВКЛЮЧИТЬ ЭФИР";
    on=false;
  }
};

mic.onclick=async()=>{
  if(!micStream){
    micStream=await navigator.mediaDevices.getUserMedia({audio:true});
    const micSrc=ctx.createMediaStreamSource(micStream);
    micSrc.connect(gain);
    mic.textContent="⛔ ВЫКЛ МИКРОФОН";
  }else{
    micStream.getTracks().forEach(t=>t.stop());
    micStream=null;
    mic.textContent="🎙 МИКРОФОН";
  }
};
</script>
</body>
</html>
