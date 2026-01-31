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
h1{text-align:center;margin:4px 0;}
.time{text-align:center;font-size:12px;opacity:.7;}
button,input{
  width:100%;
  margin-top:6px;
  background:#020617;
  color:#e5e7eb;
  border:1px solid #334155;
  border-radius:8px;
  padding:8px;
}
.admin-btn{
  text-align:center;
  opacity:.4;
  font-size:12px;
  margin-top:10px;
}
.admin{margin-top:10px;border-top:1px solid #334155;padding-top:10px;}
.hidden{display:none;}
small{opacity:.6;}
</style>
</head>

<body>
<div class="radio">

<h1>📻 YCB-89</h1>
<div class="time" id="clock"></div>

<div id="status">● ОЖИДАНИЕ</div>

<div class="admin-btn" id="openAdmin">admin</div>

<div class="admin hidden" id="admin">
<b>АДМИН</b><br>

<label>Аудиофайл</label>
<input type="file" id="file" accept="audio/*">

<label>Время старта (МСК)</label>
<input type="time" id="startTime">

<button id="save">💾 СОХРАНИТЬ</button>

<hr>

<label>Текст сообщения</label>
<input id="msgText">

<label>Время (МСК)</label>
<input type="time" id="msgTime">

<button id="saveMsg">⏰ СОХРАНИТЬ</button>

<small>
⚠ Работает пока сайт открыт
</small>
</div>

<audio id="player"></audio>

</div>

<script>
const clock=document.getElementById("clock");
const status=document.getElementById("status");
const player=document.getElementById("player");

function mskNow(){
  return new Date(Date.now()+3*3600000);
}
setInterval(()=>{
  const d=mskNow();
  clock.textContent="МСК "+d.toISOString().substr(11,8);
},1000);

/* ===== ADMIN ===== */
openAdmin.onclick=()=>{
  if(prompt("Пароль")==="3709")
    admin.classList.toggle("hidden");
};

/* ===== STORAGE ===== */
let schedule=JSON.parse(localStorage.getItem("audioSchedule")||"null");
let msgs=JSON.parse(localStorage.getItem("msgs")||"[]");

save.onclick=()=>{
  if(!file.files[0]) return;
  const reader=new FileReader();
  reader.onload=()=>{
    schedule={
      time:startTime.value,
      data:reader.result,
      played:false
    };
    localStorage.setItem("audioSchedule",JSON.stringify(schedule));
    alert("Сохранено");
  };
  reader.readAsDataURL(file.files[0]);
};

saveMsg.onclick=()=>{
  msgs.push({t:msgTime.value,txt:msgText.value,done:false});
  localStorage.setItem("msgs",JSON.stringify(msgs));
};

/* ===== LOOP ===== */
setInterval(()=>{
  const now=mskNow().toISOString().substr(11,5);

  if(schedule && !schedule.played && schedule.time<=now){
    player.src=schedule.data;
    player.play();
    status.textContent="🔊 ЭФИР";
    schedule.played=true;
    localStorage.setItem("audioSchedule",JSON.stringify(schedule));
  }

  msgs.forEach(m=>{
    if(!m.done && m.t<=now){
      const u=new SpeechSynthesisUtterance(m.txt);
      u.lang="ru-RU";
      speechSynthesis.speak(u);
      m.done=true;
      localStorage.setItem("msgs",JSON.stringify(msgs));
    }
  });
},1000);
</script>
</body>
</html>
