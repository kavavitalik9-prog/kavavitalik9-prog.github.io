<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>XP tv — эфир</title>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
body{
  margin:0;
  font-family:Arial, sans-serif;
  background:#000;
  color:#fff;
  text-align:center;
}

/* ЭКРАН ЭФИРА (16:9) */
#playerWrap{
  width:100%;
  max-width:960px;
  margin:20px auto;
  position:relative;
  aspect-ratio:16/9;
  background:#000;
  border:1px solid #222;
}
#playerWrap iframe{
  width:100%;
  height:100%;
  border:0;
}
#noLive{
  position:absolute;
  inset:0;
  display:flex;
  align-items:center;
  justify-content:center;
  font-size:22px;
  color:#aaa;
  background:#000;
}

/* КНОПКА ПОЛНОЕ РАСПИСАНИЕ */
#fullScheduleBtn{
  position:fixed;
  top:10px;
  right:10px;
  background:#222;
  color:#fff;
  border:none;
  padding:10px 12px;
  cursor:pointer;
  z-index:1000;
}

/* МОДАЛЬНОЕ ОКНО */
#modal{
  display:none;
  position:fixed;
  inset:0;
  background:rgba(0,0,0,0.9);
  color:#fff;
  overflow:auto;
  z-index:1001;
  padding:20px;
}
#modal table{
  width:100%;
  border-collapse:collapse;
}
#modal th,#modal td{
  border:1px solid #333;
  padding:8px;
  color:#fff;
  background:#000;
}
#modal th{
  background:#111;
}
#modalClose{
  position:absolute;
  top:10px;
  right:20px;
  font-size:24px;
  cursor:pointer;
}

/* ВРЕМЯ */
#clock{
  color:#0f0;
  margin:10px 0;
  font-size:18px;
}

/* СТАТУС */
#status{
  font-size:20px;
  margin:10px 0;
}

/* ПОЛЗУНОК */
#progressWrap{
  width:90%;
  max-width:960px;
  margin:10px auto;
}
#progressTime{
  font-size:14px;
  margin-bottom:5px;
}
progress{
  width:100%;
  height:16px;
}

/* РАСПИСАНИЕ */
#schedule{
  width:90%;
  max-width:960px;
  margin:20px auto;
  border-collapse:collapse;
  background:#000 !important;
}
#schedule th,#schedule td{
  border:1px solid #333;
  padding:12px;
  color:#fff !important;
  background:#000 !important;
}
#schedule th{
  background:#111 !important;
}

/* ЗРИТЕЛИ */
#viewers{
  margin:20px 0;
  font-size:18px;
  color:#0f0;
}
</style>
</head>
<body>

<!-- КНОПКА ПОЛНОЕ РАСПИСАНИЕ -->
<button id="fullScheduleBtn">📅 Полное расписание</button>

<!-- МОДАЛЬНОЕ ОКНО -->
<div id="modal">
  <span id="modalClose">✖</span>
  <h2>Полное расписание</h2>
  <table id="modalTable">
    <thead>
      <tr><th>Дата</th><th>Время</th><th>Передача</th></tr>
    </thead>
    <tbody></tbody>
  </table>
</div>

<!-- ЭФИР -->
<div id="playerWrap">
  <iframe id="player" src="" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  <div id="noLive">⏸ Эфир не идёт</div>
</div>

<div id="clock"></div>
<div id="status">⏸ Эфир не идёт</div>

<div id="progressWrap">
  <div id="progressTime"></div>
  <progress id="progress" value="0" max="100"></progress>
</div>

<table id="schedule">
<thead>
<tr><th>Время (МСК)</th><th>Передача</th></tr>
</thead>
<tbody id="scheduleBody"></tbody>
</table>

<div id="viewers">Зрителей сейчас: 1</div>

<script>
// ===== НАСТРОЙКИ =====
const schedule = [
  {start:"2026-01-20T11:00", end:"2026-01-20T15:30", title:"Фиксики — 1 сезон", video:"dQw4w9WgXcQ"},
  {start:"2026-01-20T15:30", end:"2026-01-20T20:00", title:"Фиксики — 2 сезон", video:"dQw4w9WgXcQ"},
  {start:"2026-01-20T20:00", end:"2026-01-21T00:40", title:"Фиксики — 3 сезон", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T00:40", end:"2026-01-21T05:40", title:"Фиксики — 4 сезон", video:"dQw4w9WgXcQ"}
];

// ===== ВРЕМЯ МСК =====
function nowMSK(){
  return new Date(new Date().toLocaleString("en-US",{timeZone:"Europe/Moscow"}));
}

// ===== ОБНОВЛЕНИЕ =====
function update(){
  const now = nowMSK();
  document.getElementById("clock").textContent =
    "МСК: "+now.toLocaleTimeString("ru-RU",{hour:"2-digit",minute:"2-digit",second:"2-digit"});

  let current=null;
  let upcoming=[];

  schedule.forEach(p=>{
    const s=new Date(p.start+"+03:00");
    const e=new Date(p.end+"+03:00");
    if(now>=s && now<e) current={...p,s,e};
    if(now<e) upcoming.push({...p,s,e});
  });

  // ЭФИР
  if(current){
    document.getElementById("status").textContent="🔴 Сейчас в эфире: "+current.title;
    document.getElementById("noLive").style.display="none";
    document.getElementById("player").src="https://www.youtube.com/embed/"+current.video+"?autoplay=1&mute=1";

    const percent=((now-current.s)/(current.e-current.s))*100;
    document.getElementById("progress").value=percent;

    document.getElementById("progressTime").textContent=
      current.s.toLocaleTimeString("ru-RU",{hour:"2-digit",minute:"2-digit"})+
      " — "+
      current.e.toLocaleTimeString("ru-RU",{hour:"2-digit",minute:"2-digit"});
  } else {
    document.getElementById("status").textContent="⏸ Эфир не идёт";
    document.getElementById("player").src="";
    document.getElementById("noLive").style.display="flex";
    document.getElementById("progress").value=0;
    document.getElementById("progressTime").textContent="";
  }

  // таблица (текущая + 3 следующих)
  const body=document.getElementById("scheduleBody");
  body.innerHTML="";
  upcoming.slice(0,4).forEach(p=>{
    const tr=document.createElement("tr");
    tr.innerHTML=
      `<td>${p.s.toLocaleTimeString("ru-RU",{hour:"2-digit",minute:"2-digit"})} – ${p.e.toLocaleTimeString("ru-RU",{hour:"2-digit",minute:"2-digit"})}</td>
       <td>${p.title}</td>`;
    body.appendChild(tr);
  });
}

// ===== СЧЁТЧИК ЗРИТЕЛЕЙ =====
let viewers=Math.floor(Math.random()*5)+1;
setInterval(()=>{
  viewers += Math.random()>0.5?1:-1;
  if(viewers<1) viewers=1;
  document.getElementById("viewers").textContent="Зрителей сейчас: "+viewers;
},4000);

// ===== ПОЛНОЕ РАСПИСАНИЕ В МОДАЛЬНОМ ОКНЕ =====
const modal=document.getElementById("modal");
const modalBtn=document.getElementById("fullScheduleBtn");
const modalClose=document.getElementById("modalClose");
const modalBody=document.querySelector("#modal tbody");

modalBtn.onclick=function(){
  modal.style.display="block";
  modalBody.innerHTML="";
  schedule.forEach(p=>{
    const tr=document.createElement("tr");
    tr.innerHTML=`<td>${p.start.split("T")[0]}</td><td>${p.start.split("T")[1]} – ${p.end.split("T")[1]}</td><td>${p.title}</td>`;
    modalBody.appendChild(tr);
  });
};
modalClose.onclick=function(){modal.style.display="none";};
window.onclick=function(e){if(e.target==modal) modal.style.display="none";};

// запуск
setInterval(update,1000);
update();
</script>
</body>
</html>
