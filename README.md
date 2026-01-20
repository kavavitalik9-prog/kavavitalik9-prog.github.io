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
<tr><th>Время (МСК)</th><th>Передача</th></tr>
</thead>
<tbody id="scheduleBody"></tbody>
</table>
<div id="timeLabel">Время МСК</div>

<div id="viewers">Зрителей сейчас: 1</div>

<script>
// ===== РАСПИСАНИЕ =====
// Видео: подставь свои ссылки YouTube
const schedule = [
  {start:"2026-01-20T01:00", end:"2026-01-20T14:00", title:null, video:""},
  {start:"2026-01-20T14:00", end:"2026-01-20T17:30", title:"Фиксики - 1 сезон", video:"dQw4w9WgXcQ"},
  {start:"2026-01-20T17:30", end:"2026-01-20T22:00", title:"Фиксики - 2 сезон", video:"dQw4w9WgXcQ"},
  {start:"2026-01-20T22:00", end:"2026-01-21T00:40", title:"Фиксики - 3 сезон", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T00:40", end:"2026-01-21T05:40", title:"Фиксики - 4 сезон", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T05:40", end:"2026-01-21T05:50", title:"КОРОЧЕ ГОВОРЯ, ОСТАЛСЯ ОДИН НА ЗЕМЛЕ", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T05:50", end:"2026-01-21T06:00", title:null, video:""},
  {start:"2026-01-21T06:00", end:"2026-01-21T06:30", title:"Мама Читера ИЗДЕВАЛАСЬ надо Мной на этом Сервере в Майнкрафт", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T06:30", end:"2026-01-21T08:20", title:"Я ОТОМСТИЛ ХЕЙТЕРШЕ Моей Девушки! ЭТО КОНЕЦ...", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T08:20", end:"2026-01-21T09:40", title:"Фиксики БОЛЬШОЙ СЕКРЕТ", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T09:40", end:"2026-01-21T10:00", title:null, video:""},
  {start:"2026-01-21T10:00", end:"2026-01-21T10:10", title:"Жену укусили за лицо... (Анимация)", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T10:10", end:"2026-01-21T10:30", title:"25-ЛЕТНИЙ СЫНОЧКА КОРЗИНОЧКА УЧИТСЯ В НАШЕМ КЛАССЕ", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T10:30", end:"2026-01-21T12:30", title:"музыка", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T12:30", end:"2026-01-21T13:00", title:null, video:""},
  {start:"2026-01-21T13:00", end:"2026-01-21T13:20", title:"25-ЛЕТНИЙ СЫНОЧКА КОРЗИНОЧКА УЧИТСЯ В НАШЕМ КЛАССЕ", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T13:20", end:"2026-01-21T13:40", title:"25-ЛЕТНИЙ СЫНОЧКА-КОРЗИНОЧКА В ЖЕНСКОЙ РАЗДЕВАЛКЕ", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T13:40", end:"2026-01-21T14:00", title:"25-ЛЕТНИЙ СЫНОЧКА-КОРЗИНОЧКА В ТЮРЬМЕ", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T14:00", end:"2026-01-21T14:20", title:"25-ЛЕТНИЙ СЫНОЧКА-КОРЗИНОЧКА ПРОТИВ ЯЖЕМАТЕРИ", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T14:20", end:"2026-01-21T14:40", title:"25-ЛЕТНИЙ СЫНОЧКА-КОРЗИНОЧКА ПРАЗДНУЕТ НОВЫЙ ГОД", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T14:40", end:"2026-01-21T15:00", title:"25-ЛЕТНИЙ СЫНОЧКА-КОРЗИНОЧКА И ОЧЕНЬ СТРАННЫЕ ДЕЛА", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T15:00", end:"2026-01-21T15:20", title:"25-ЛЕТНИЙ СЫНОЧКА-КОРЗИНОЧКУ ВЫГНАЛИ ИЗ ШКОЛЫ", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T15:20", end:"2026-01-21T15:40", title:"25-ЛЕТНИЙ СЫНОЧКА-КОРЗИНОЧКА УМ☠️ЕР И ПОПАЛ В АД", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T15:40", end:"2026-01-21T16:00", title:"25-ЛЕТНИЙ СЫНОЧКА-КОРЗИНОЧКА vs НАТУРАЛ МОЛЬБЕРТОВИЧ", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T16:00", end:"2026-01-21T16:20", title:"25-ЛЕТНИЙ СЫНОЧКА-КОРЗИНОЧКА И ИНОПЛАНЕТЯНЕ", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T16:20", end:"2026-01-21T16:40", title:"25-ЛЕТНИЙ СЫНОЧКА-КОРЗИНОЧКА В ПОЕЗДЕ", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T16:40", end:"2026-01-21T17:00", title:"25-ЛЕТНИЙ СЫНОЧКА-КОРЗИНОЧКА В СЕЛЕ", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T17:00", end:"2026-01-21T17:20", title:"25 И 12 ЛЕТНИЕ СЫНОЧКИ-КОРЗИНОЧКИ ПОДРУЖИЛИСЬ", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T17:20", end:"2026-01-21T17:30", title:"КТО ТАКОЙ 25-ЛЕТНИЙ СЫНОЧКА-КОРЗИНОЧКА? 25 фактов об Олеже", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T17:30", end:"2026-01-21T19:00", title:null, video:""},
  {start:"2026-01-21T19:00", end:"2026-01-21T19:05", title:"MetalFamily 1 сезон 1 серия", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T19:05", end:"2026-01-21T19:10", title:"MetalFamily 1 сезон 2 серия", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T19:10", end:"2026-01-21T19:15", title:"MetalFamily 1 сезон 3 серия", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T19:15", end:"2026-01-21T19:25", title:"MetalFamily 1 сезон 4 серия", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T19:25", end:"2026-01-21T19:35", title:"MetalFamily 1 сезон 5 серия", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T19:35", end:"2026-01-21T19:50", title:"MetalFamily 1 сезон 6 серия", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T19:50", end:"2026-01-21T20:00", title:"MetalFamily 1 сезон 7 серия", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T20:00", end:"2026-01-21T20:10", title:"MetalFamily 1 сезон 8 серия", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T20:10", end:"2026-01-21T20:30", title:"MetalFamily 1 сезон 9 серия", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T20:30", end:"2026-01-21T21:00", title:"MetalFamily 1 сезон 10 серия", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T21:00", end:"2026-01-21T21:15", title:"MetalFamily 2 сезон 1 серия", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T21:15", end:"2026-01-21T21:25", title:"MetalFamily 2 сезон 2 серия", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T21:25", end:"2026-01-21T21:55", title:"MetalFamily 2 сезон 3 серия", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T21:55", end:"2026-01-21T22:05", title:"MetalFamily 2 сезон 4 серия", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T22:05", end:"2026-01-21T22:20", title:"MetalFamily 2 сезон 5 серия", video:"dQw4w9WgXcQ"},
  {start:"2026-01-21T22:20", end:"2026-01-21T22:40", title:"MetalFamily 2 сезон 6 серия", video:"dQw4w9WgXcQ"}
];

// ===== Функции =====
function parseMSK(dateStr){
  const [y,m,dT] = dateStr.split("-");
  const [d, hm] = dT.split("T");
  const [h,min] = hm.split(":");
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
    if(now>=s && now<e) current={...p,s,e};
    if(now<e) upcoming.push({...p,s,e});
  });

  const player=document.getElementById("player");
  const noLive=document.getElementById("noLive");

  if(current && current.title){
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
    document.getElementById("status").text
