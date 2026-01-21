<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Графік світла — Львівська область</title>

<style>
body{
  margin:0;
  background:#0e0e0e;
  color:#fff;
  font-family:system-ui,sans-serif;
}
.container{
  max-width:520px;
  margin:auto;
  padding:14px;
}
.header{
  border:2px solid #ffc400;
  border-radius:14px;
  padding:12px;
}
.header h1{
  margin:0;
  font-size:20px;
}
.header .meta{
  margin-top:6px;
  font-size:14px;
  opacity:.9;
  display:flex;
  justify-content:space-between;
}
select,button{
  width:100%;
  padding:12px;
  margin-top:10px;
  border-radius:10px;
  border:none;
  background:#1f1f1f;
  color:#fff;
  font-size:15px;
}
button{cursor:pointer;}
.card{
  background:#1a1a1a;
  border-radius:14px;
  padding:14px;
  margin-top:12px;
}
.big{
  font-size:22px;
  font-weight:700;
  text-align:center;
}
.schedule-line{
  margin:6px 0;
  font-size:15px;
}
.timer{
  margin-top:8px;
  text-align:center;
  opacity:.85;
}
.footer{
  margin-top:14px;
  text-align:center;
  font-size:13px;
  opacity:.6;
}
</style>
</head>

<body>
<div class="container">

<div class="header">
  <h1>⚡ Львівська область</h1>
  <div class="meta">
    <div>👁 <span id="views"></span></div>
    <div id="lastUpdate"></div>
  </div>
</div>

<select id="group"></select>
<button onclick="pinGroup()">📌 Закріпити мою групу</button>

<div class="card" id="statusCard"></div>

<div class="footer">Автооновлення</div>

</div>

<script>
// ===== НАЛАШТУВАННЯ =====
const updatedAt = new Date("2026-01-21T19:50:00");

// приклад графіка (МОЖЕШ МІНЯТИ)
const schedule = {
  "1.1": [
    ["00:00","12:00","on"],
    ["12:00","14:00","off"],
    ["14:00","24:00","on"]
  ],
  "1.2": [
    ["00:00","06:00","off"],
    ["06:00","18:00","on"],
    ["18:00","24:00","off"]
  ]
};

// ===== ІНІЦ =====
const groupSel = document.getElementById("group");
Object.keys(schedule).forEach(g=>{
  groupSel.innerHTML += `<option value="${g}">Група ${g}</option>`;
});
groupSel.value = localStorage.getItem("group") || "1.1";

// ===== ФУНКЦІЇ =====
function toMin(t){
  let[h,m]=t.split(":");
  return +h*60 + +m;
}
function nowMin(){
  let d=new Date();
  return d.getHours()*60 + d.getMinutes();
}

function render(){
  let g = groupSel.value;
  let now = nowMin();
  let html = `<div class="big">Група ${g}</div>`;

  let current,next;

  schedule[g].forEach(s=>{
    let from=toMin(s[0]), to=toMin(s[1]);
    let isOn=s[2]==="on";
    html += `
      <div class="schedule-line">
        ${isOn?"🟢":"⚫"} ${s[0]}–${s[1]} — ${isOn?"є світло":"нема світла"}
      </div>
    `;
    if(now>=from && now<to){
      current={...s,from,to};
    }
    if(now<from && !next){
      next={...s,from,to};
    }
  });

  if(current){
    let left = current.to - now;
    let h=Math.floor(left/60), m=left%60;
    html += `<div class="timer">⏳ До ${current[2]==="on"?"вимкнення":"увімкнення"}: ${h} год ${m} хв</div>`;
  }

  document.getElementById("statusCard").innerHTML = html;
}

function pinGroup(){
  localStorage.setItem("group",groupSel.value);
}

// ===== ОНОВЛЕННЯ =====
function updateMeta(){
  let diff = Math.floor((Date.now()-updatedAt)/60000);
  let t="щойно";
  if(diff>=1) t=diff+" хв тому";
  if(diff>=60){
    let h=Math.floor(diff/60), m=diff%60;
    t=h+" год "+m+" хв тому";
  }
  lastUpdate.textContent = "Оновлено: "+t;
  views.textContent = Math.floor(1000 + Math.random()*600000);
}

render();
updateMeta();
groupSel.onchange=render;
setInterval(()=>{render();updateMeta();},60000);
</script>
</body>
</html>
