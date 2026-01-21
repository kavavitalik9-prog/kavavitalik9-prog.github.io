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
  justify-content:space-between;
  align-items:center;
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
  margin:8px 0;
  font-size:16px;
}
.card{
  background:#151515;
  border-radius:12px;
  padding:15px;
  margin-top:10px;
}
.row{
  display:flex;
  justify-content:space-between;
  border-bottom:1px solid #222;
  padding:6px 0;
  font-size:15px;
}
.on{color:#4cff4c;}
.off{color:#ff4c4c;}
.now{
  background:#222;
  border-radius:6px;
  padding:4px 6px;
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
    <h2>⚡ Львівська область</h2>
    <div class="viewers">👁 <span id="viewers">...</span></div>
  </div>

  <select id="day">
    <option value="forming">Понеділок</option>
    <option value="forming">Вівторок</option>
    <option value="wednesday">Середа</option>
    <option value="forming">Четвер</option>
    <option value="forming">Пʼятниця</option>
    <option value="forming">Субота</option>
    <option value="forming">Неділя</option>
  </select>

  <select id="group">
    <option>1.1</option><option>1.2</option>
    <option>2.1</option><option>2.2</option>
    <option>3.1</option><option>3.2</option>
    <option>4.1</option><option>4.2</option>
    <option>5.1</option><option>5.2</option>
    <option>6.1</option><option>6.2</option>
  </select>

  <div id="content" class="card"></div>

</div>

<footer>Демонстраційний графік • Середа</footer>

<script>
// ===== ФЕЙК ОНЛАЙН =====
let viewers=Math.floor(Math.random()*(700000-975)+975);
const v=document.getElementById("viewers");
function upd(){
  viewers+=Math.floor(Math.random()*5000-2500);
  viewers=Math.max(975,Math.min(700000,viewers));
  v.textContent=viewers.toLocaleString("uk-UA");
}
upd(); setInterval(upd,3000);

// ===== ГРАФІК СЕРЕДИ =====
const schedule={
"1.1":[["00:00","18:00","on"],["18:00","20:00","off"],["20:00","23:59","on"]],
"1.2":[["00:00","01:30","off"],["01:30","23:59","on"]],
"2.1":[["00:00","20:00","on"],["20:00","23:59","off"]],
"2.2":[["00:00","23:59","on"]],
"3.1":[["00:00","20:00","on"],["20:00","23:59","off"]],
"3.2":[["00:00","23:59","on"]],
"4.1":[["00:00","20:00","on"],["20:00","22:00","off"],["22:00","23:59","on"]],
"4.2":[["00:00","18:00","on"],["18:00","20:00","off"],["20:00","23:59","on"]],
"5.1":[["00:00","18:00","on"],["18:00","20:00","off"],["20:00","23:59","on"]],
"5.2":[["00:00","23:59","on"]],
"6.1":[["00:00","01:30","off"],["01:30","23:59","on"]],
"6.2":[["00:00","23:59","on"]]
};

function minutes(t){let[a,b]=t.split(":");return +a*60+ +b;}

function render(){
 const day=document.getElementById("day").value;
 const group=document.getElementById("group").value;
 const box=document.getElementById("content");
 box.innerHTML="";

 if(day!=="wednesday"){
   box.innerHTML="⏳ <b>Графік ще формується</b>";
   return;
 }

 const now=new Date();
 const cur=now.getHours()*60+now.getMinutes();

 schedule[group].forEach(i=>{
   const s=minutes(i[0]), e=minutes(i[1]);
   const isNow=cur>=s && cur<=e;
   box.innerHTML+=`
     <div class="row ${isNow?"now":""}">
       <span>${i[0]}–${i[1]}</span>
       <span class="${i[2]}">
         ${i[2]==="on"?"⚡ світло є":"⛔ світла нема"}
       </span>
     </div>`;
 });
}

day.onchange=render;
group.onchange=render;
render();
</script>
</body>
</html>
