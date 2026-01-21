<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>⚡ Графіки відключення світла — Львівська область</title>

<style>
body{
  margin:0;
  font-family:system-ui,Arial;
  background:#0f0f0f;
  color:#fff;
}
.container{
  max-width:900px;
  margin:auto;
  padding:15px;
}
.header{
  display:flex;
  flex-wrap:wrap;
  align-items:center;
  justify-content:center;
  gap:10px;
}
.viewers{
  border:1px solid #ffd000;
  padding:6px 12px;
  border-radius:10px;
  background:#111;
  font-size:14px;
}
select{
  width:100%;
  padding:10px;
  border-radius:8px;
  border:none;
  margin:10px 0;
  font-size:16px;
}
.card{
  background:#151515;
  border-radius:12px;
  padding:15px;
  margin-top:10px;
}
.status-on{color:#4cff4c;}
.status-off{color:#ff4c4c;}
.timer{
  font-size:18px;
  margin-top:8px;
}
.fake-ip{
  font-size:13px;
  opacity:.7;
  margin-top:5px;
}
footer{
  text-align:center;
  margin:20px 0;
  opacity:.5;
  font-size:13px;
}
</style>
</head>

<body>
<div class="container">

  <div class="header">
    <h1>⚡ Львівська область</h1>
    <div class="viewers">
      👁 <span id="viewers">...</span>
    </div>
  </div>

  <select id="daySelect">
    <option value="forming">Понеділок</option>
    <option value="forming">Вівторок</option>
    <option value="wednesday">Середа</option>
    <option value="forming">Четвер</option>
    <option value="forming">Пʼятниця</option>
    <option value="forming">Субота</option>
    <option value="forming">Неділя</option>
  </select>

  <select id="groupSelect">
    <option>1.1</option><option>1.2</option>
    <option>2.1</option><option>2.2</option>
    <option>3.1</option><option>3.2</option>
    <option>4.1</option><option>4.2</option>
    <option>5.1</option><option>5.2</option>
    <option>6.1</option><option>6.2</option>
  </select>

  <div id="content" class="card"></div>

</div>

<footer>Дані можуть змінюватись • Демонстраційний проєкт</footer>

<script>
// ===== ФЕЙК ОНЛАЙН =====
let viewers = Math.floor(Math.random()*(700000-975)+975);
const viewersEl = document.getElementById("viewers");

function updateViewers(){
  viewers += Math.floor(Math.random()*6000-3000);
  if(viewers<975) viewers=975;
  if(viewers>700000) viewers=700000;
  viewersEl.textContent = viewers.toLocaleString("uk-UA");
}
updateViewers();
setInterval(updateViewers,3000);

// ===== ФЕЙК IP =====
function fakeIP(){
  return `${Math.floor(Math.random()*255)}.${Math.floor(Math.random()*255)}.*.*`;
}

// ===== ГРАФІК СЕРЕДИ =====
const schedule = {
 "1.1":[["00:00","18:00",1],["18:00","20:00",0],["20:00","23:59",1]],
 "1.2":[["00:00","01:30",0],["01:30","23:59",1]],
 "2.1":[["00:00","20:00",1],["20:00","23:59",0]],
 "2.2":[["00:00","23:59",1]],
 "3.1":[["00:00","20:00",1],["20:00","23:59",0]],
 "3.2":[["00:00","23:59",1]],
 "4.1":[["00:00","20:00",1],["20:00","22:00",0],["22:00","23:59",1]],
 "4.2":[["00:00","18:00",1],["18:00","20:00",0],["20:00","23:59",1]],
 "5.1":[["00:00","18:00",1],["18:00","20:00",0],["20:00","23:59",1]],
 "5.2":[["00:00","23:59",1]],
 "6.1":[["00:00","01:30",0],["01:30","23:59",1]],
 "6.2":[["00:00","23:59",1]]
};

const content=document.getElementById("content");

function show(){
 const day=daySelect.value;
 const group=groupSelect.value;

 if(day!=="wednesday"){
   content.innerHTML="⏳ <b>Графік ще формується</b>";
   return;
 }

 const now=new Date();
 const cur=now.getHours()*60+now.getMinutes();
 let current=null,next=null;

 for(let i of schedule[group]){
  const s=i[0].split(":"), e=i[1].split(":");
  const sm=+s[0]*60+ +s[1], em=+e[0]*60+ +e[1];
  if(cur>=sm && cur<=em) current=i;
  if(cur<sm && !next) next=i;
 }

 if(!current){
   content.innerHTML="⏳ Немає даних";
   return;
 }

 const status=current[2]?
   "<span class='status-on'>⚡ Світло є</span>":
   "<span class='status-off'>⛔ Світла нема</span>";

 content.innerHTML=`
   <h3>Група ${group}</h3>
   ${status}
   <div class="fake-ip">👤 Глядач: ${fakeIP()}</div>
 `;
}

daySelect.onchange=show;
groupSelect.onchange=show;
show();
</script>

</body>
</html>
