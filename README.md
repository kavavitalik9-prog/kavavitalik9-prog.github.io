<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>XP TV</title>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
body{
  margin:0;
  background:#000;
  color:#fff;
  font-family:Arial,sans-serif;
  text-align:center;
}
#playerWrap{
  max-width:960px;
  margin:20px auto;
  aspect-ratio:16/9;
  position:relative;
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
  background:#000;
  color:#aaa;
  font-size:22px;
}
#status{
  font-size:20px;
  margin:10px 0;
}
table{
  width:90%;
  max-width:960px;
  margin:20px auto;
  border-collapse:collapse;
}
th,td{
  border:1px solid #333;
  padding:12px;
  background:#000;
  color:#fff;
}
th{
  background:#111;
}
</style>
</head>
<body>

<h1>XP TV</h1>

<div id="playerWrap">
  <iframe id="player" allow="autoplay; encrypted-media"></iframe>
  <div id="noLive">📴 Эфир сейчас не идёт</div>
</div>

<div id="status">📴 Эфир сейчас не идёт</div>

<h2>Расписание (следующие)</h2>
<table>
<thead>
<tr>
  <th>Время (МСК)</th>
  <th>Передача</th>
</tr>
</thead>
<tbody id="nextBody"></tbody>
</table>

<script>
// ================== РАСПИСАНИЕ ==================
const schedule = [
  // 20 января 2026
  { start:"2026-01-20T01:00", end:"2026-01-20T14:00", title:null },
  { start:"2026-01-20T14:00", end:"2026-01-20T17:30", title:"Фиксики - 1 сезон", video:"dQw4w9WgXcQ" },
  { start:"2026-01-20T17:30", end:"2026-01-21T00:59", title:null },

  // 21 января 2026
  { start:"2026-01-21T00:59", end:"2026-01-21T05:00", title:null },
  { start:"2026-01-21T05:00", end:"2026-01-21T09:30", title:"Фиксики - 1 сезон", video:"dQw4w9WgXcQ" },
  { start:"2026-01-21T09:30", end:"2026-01-21T14:00", title:"Фиксики - 2 сезон", video:"dQw4w9WgXcQ" },
  { start:"2026-01-21T14:00", end:"2026-01-21T18:40", title:"Фиксики - 3 сезон", video:"dQw4w9WgXcQ" },
  { start:"2026-01-21T18:40", end:"2026-01-21T23:40", title:"Фиксики - 4 сезон", video:"dQw4w9WgXcQ" },
  { start:"2026-01-21T23:40", end:"2026-01-22T00:59", title:null }
];

// ================== ВРЕМЯ МСК ==================
function parseMSK(t){
  const [d,h]=t.split("T");
  const [y,m,da]=d.split("-");
  const [hh,mm]=h.split(":");
  return new Date(Date.UTC(y,m-1,da,hh-3,mm));
}
function nowMSK(){
  return new Date(new Date().toLocaleString("en-US",{timeZone:"Europe/Moscow"}));
}

let currentVideo=null;

function update(){
  const now=nowMSK();
  let current=null;
  let next=[];

  schedule.forEach(p=>{
    const s=parseMSK(p.start);
    const e=parseMSK(p.end);
    if(now>=s && now<e) current={...p,s,e};
    if(s>now) next.push({...p,s,e});
  });

  const player=document.getElementById("player");
  const noLive=document.getElementById("noLive");
  const status=document.getElementById("status");

  // ===== ТЕКУЩИЙ ЭФИР =====
  if(!current || current.title===null){
    player.src="";
    noLive.style.display="flex";
    status.textContent="📴 Эфир сейчас не идёт";
  } else {
    noLive.style.display="none";
    status.textContent="🔴 Сейчас в эфире: "+current.title;

    if(currentVideo!==current.video){
      const startSec=Math.floor((now-current.s)/1000);
      player.src="https://www.youtube.com/embed/"+current.video+
        "?autoplay=1&mute=1&controls=0&disablekb=1&start="+startSec;
      currentVideo=current.video;
    }
  }

  // ===== РАСПИСАНИЕ (NULL ВИДНО) =====
  const body=document.getElementById("nextBody");
  body.innerHTML="";
  next.forEach(p=>{
    const tr=document.createElement("tr");
    tr.innerHTML=
      "<td>"+p.s.toLocaleTimeString("ru-RU",{hour:"2-digit",minute:"2-digit"})+
      " — "+p.e.toLocaleTimeString("ru-RU",{hour:"2-digit",minute:"2-digit"})+
      "</td><td>"+(p.title===null ? "null" : p.title)+"</td>";
    body.appendChild(tr);
  });
}

setInterval(update,1000);
update();
</script>

</body>
</html>
