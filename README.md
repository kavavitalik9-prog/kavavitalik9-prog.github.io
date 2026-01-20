<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>XP tv — эфир</title>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
body{margin:0;font-family:Arial,sans-serif;background:#000;color:#fff;text-align:center;}
#playerWrap{width:100%;max-width:960px;margin:20px auto;position:relative;aspect-ratio:16/9;background:#000;border:1px solid #222;}
#playerWrap iframe{width:100%;height:100%;border:0;}
#noLive{position:absolute;inset:0;display:flex;align-items:center;justify-content:center;font-size:22px;color:#aaa;background:#000;}
#fullScheduleBtn{position:fixed;top:10px;right:10px;background:#222;color:#fff;border:none;padding:10px 12px;cursor:pointer;z-index:1000;}
#modal{display:none;position:fixed;inset:0;background:rgba(0,0,0,0.9);color:#fff;overflow:auto;z-index:1001;padding:20px;}
#modal table{width:100%;border-collapse:collapse;}
#modal th,#modal td{border:1px solid #333;padding:8px;color:#fff;background:#000;}
#modal th{background:#111;}
#modalClose{position:absolute;top:10px;right:20px;font-size:24px;cursor:pointer;}
#status{font-size:20px;margin:10px 0;}
#progressWrap{width:90%;max-width:960px;margin:10px auto;}
#progressTime{font-size:14px;margin-bottom:5px;}
progress{width:100%;height:16px;}
#schedule{width:90%;max-width:960px;margin:20px auto;border-collapse:collapse;background:#000 !important;}
#schedule th,#schedule td{border:1px solid #333;padding:12px;color:#fff !important;background:#000 !important;}
#schedule th{background:#111 !important;}
#timeLabel{color:#aaa;margin-bottom:30px;font-size:14px;}
#viewers{margin:20px 0;font-size:18px;color:#0f0;}
</style>
</head>
<body>

<button id="fullScheduleBtn">📅 Полное расписание</button>

<div id="modal">
  <span id="modalClose">✖</span>
  <h2>Полное расписание</h2>
  <table id="modalTable">
    <thead>
      <tr>
        <th>Дата</th>
        <th>Время МСК</th>
        <th>Ваше время</th>
        <th>Передача</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>
</div>

<div id="playerWrap">
  <iframe id="player" src="" allow="autoplay; encrypted-media" allowfullscreen></iframe>
  <div id="noLive">⏳ Подождите немного, расписание ещё формируется</div>
</div>

<div id="status">⏳ Подождите немного, расписание ещё формируется</div>

<div id="progressWrap">
  <div id="progressTime"></div>
  <progress id="progress" value="0" max="100"></progress>
</div>

<table id="schedule">
<thead>
<tr><th>Следующие передачи</th><th>Время МСК</th></tr>
</thead>
<tbody id="scheduleBody"></tbody>
</table>
<div id="timeLabel">Время МСК</div>

<div id="viewers">Зрителей сейчас: 1</div>

<script>
// ===== РАСПИСАНИЕ =====
// Видео-заглушки YouTube вставляй свои ссылки
const schedule = [
{start:"2026-01-20T01:00", end:"2026-01-20T14:00", title:null, video:""},
{start:"2026-01-20T14:00", end:"2026-01-20T17:30", title:"Фиксики - 1 сезон", video:"dQw4w9WgXcQ"},
{start:"2026-01-20T17:30", end:"2026-01-21T00:59", title:"технические роботы", video:"dQw4w9WgXcQ"}
];

// ===== ФУНКЦИИ =====
function parseMSK(dateStr){
  const [y,m,dT]=dateStr.split("-");
  const [d,hm]=dT.split("T");
  const [h,min]=hm.split(":");
  return new Date(Date.UTC(+y,+m-1,+d,+h-3,+min));
}
function nowMSK(){return new Date(new Date().toLocaleString("en-US",{timeZone:"Europe/Moscow"}));}

let currentVideo=null;

// ===== ОБНОВЛЕНИЕ ЭФИРА =====
function update(){
  const now = nowMSK();
  let current=null;
  let upcoming=[];

  schedule.forEach(p=>{
    const s=parseMSK(p.start);
    const e=parseMSK(p.end);
    if(now>=s && now<e && p.title) current={...p,s,e};
    if(e>now && p.title) upcoming.push({...p,s,e});
  });

  const player=document.getElementById("player");
  const noLive=document.getElementById("noLive");
  const scheduleBody=document.getElementById("scheduleBody");
  scheduleBody.innerHTML="";

  // Главная таблица: следующие 4 передачи
  const main4 = upcoming.slice(0,4);
  main4.forEach(p=>{
    const s=parseMSK(p.start), e=parseMSK(p.end);
    const tr=document.createElement("tr");
    tr.innerHTML="<td>"+p.title+"</td><td>"+s.toLocaleTimeString('ru-RU',{hour:'2-digit',minute:'2-digit'})+" — "+e.toLocaleTimeString('ru-RU',{hour:'2-digit',minute:'2-digit'})+"</td>";
    scheduleBody.appendChild(tr);
  });

  if(current){
    document.getElementById("status").textContent="🔴 Сейчас в эфире: "+current.title;
    noLive.style.display="none";

    if(currentVideo!==current.video){
      player.src="https://www.youtube.com/embed/"+current.video+"?autoplay=1&mute=1&controls=0&disablekb=1&modestbranding=1&start="+Math.floor((now-current.s)/1000);
      currentVideo=current.video;
    }

    const percent=((now-current.s)/(current.e-current.s))*100;
    document.getElementById("progress").value=percent;
    document.getElementById("progressTime").textContent=
      current.s.toLocaleTimeString("ru-RU",{hour:"2-digit",minute:"2-digit"})+" — "+
      current.e.toLocaleTimeString("ru-RU",{hour:"2-digit",minute:"2-digit"});

  } else {
    document.getElementById("status").textContent="⏳ Подождите немного, расписание ещё формируется";
    noLive.style.display="flex";
    document.getElementById("progress").value=0;
    document.getElementById("progressTime").textContent="";
  }

  // Обновление модального окна
  const modalBody=document.querySelector("#modal tbody");
  modalBody.innerHTML="";
  schedule.forEach(p=>{
    const s=parseMSK(p.start), e=parseMSK(p.end);
    const userTimeStart=new Date(s.getTime() + (new Date().getTimezoneOffset()+180)*60000);
    const userTimeEnd=new Date(e.getTime() + (new Date().getTimezoneOffset()+180)*60000);
    const tr=document.createElement("tr");
    tr.innerHTML="<td>"+s.toLocaleDateString()+"</td><td>"+s.toLocaleTimeString('ru-RU',{hour:'2-digit',minute:'2-digit'})+" — "+e.toLocaleTimeString('ru-RU',{hour:'2-digit',minute:'2-digit'})+"</td><td>"+userTimeStart.toLocaleTimeString('ru-RU',{hour:'2-digit',minute:'2-digit'})+" — "+userTimeEnd.toLocaleTimeString('ru-RU',{hour:'2-digit',minute:'2-digit'})+"</td><td>"+(p.title?p.title:"null")+"</td>";
    modalBody.appendChild(tr);
  });
}

setInterval(update,1000);
update();

// ===== МОДАЛЬНОЕ ОКНО =====
document.getElementById("fullScheduleBtn").onclick=function(){
  document.getElementById("modal").style.display="block";
}
document.getElementById("modalClose").onclick=function(){
  document.getElementById("modal").style.display="none";
}

// ===== СЧЁТЧИК ЗРИТЕЛЕЙ =====
let viewers=Math.floor(Math.random()*5)+1;
setInterval(()=>{
  viewers+=Math.random()>0.5?1:-1;
  if(viewers<1) viewers=1;
  document.getElementById("viewers").textContent="Зрителей сейчас: "+viewers;
},4000);

</script>
</body>
</html>
