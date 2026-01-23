<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<title>⚡ Львівська область — графіки світла</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
:root{
  --bg:#0f1115;
  --card:#1a1d24;
  --green:#2ecc71;
  --red:#ff4d4d;
}
*{box-sizing:border-box;font-family:system-ui}
body{margin:0;background:var(--bg);color:#fff}

header{
  padding:14px 16px;
  border-bottom:1px solid #222;
  font-weight:600;
  display:flex;
  justify-content:space-between;
  align-items:center;
  position:relative;
}

#adminBtn{
  cursor:pointer;font-size:22px;opacity:.7
}
#adminBtn:hover{opacity:1}

main{padding:14px}

.group{
  background:var(--card);
  border-radius:14px;
  padding:12px;
  margin-bottom:14px;
}
.group h3{margin:0 0 6px}
.status{font-size:14px;display:flex;align-items:center;gap:6px}
.green{color:var(--green)}
.red{color:var(--red)}
.blink{animation:blink 1.6s infinite}
@keyframes blink{50%{opacity:.5}}

.timeline{display:flex;gap:2px;margin-top:6px}
.hour{flex:1;height:14px;background:#333;border-radius:4px}
.off{background:var(--red)}
.now{outline:2px solid #fff}

/* ADMIN */
#adminPanel{
  position:fixed;top:0;right:-100%;
  width:320px;height:100%;
  background:#14161c;
  border-left:1px solid #222;
  padding:16px;
  transition:.3s;
  z-index:10;
}
#adminPanel.open{right:0}
select,input,textarea,button{
 width:100%;margin-top:8px;
 padding:8px;border-radius:8px;border:none
}
button{background:#2b6cff;color:#fff;cursor:pointer}
small{opacity:.6;margin-top:4px;display:block}
</style>
</head>
<body>

<header>
⚡ Львівська область
<div id="adminBtn">🔒</div>
</header>

<main id="groups"></main>

<div id="adminPanel">
<h3>Адмін панель</h3>
<input id="pass" type="password" placeholder="Пароль">
<button onclick="login()">Увійти</button>

<div id="adminContent" style="display:none">
<small>Введи години без світла через кому, формат 18:00-22:00</small>
<textarea id="hours" rows="6" placeholder="18:00-22:00, 01:00-03:00"></textarea>
<select id="daySel">
  <option>Пн</option><option>Вт</option><option>Ср</option>
  <option>Чт</option><option>Пт</option><option>Сб</option><option>Нд</option>
</select>
<label><input type="checkbox" id="allGroups" checked> Оновити для всіх груп</label>
<button onclick="save()">Зберегти</button>
<small id="lastUpdate">Останнє оновлення: щойно</small>
</div>
</div>

<script>
const PASSWORD="3709";
const days=["Пн","Вт","Ср","Чт","Пт","Сб","Нд"];
const groups={};
for(let g=1;g<=6;g++){for(let s=1;s<=2;s++){
  groups[`${g}.${s}`]={}; days.forEach(d=>groups[`${g}.${s}`][d]=[]);
}}

function nowInfo(){
 const n=new Date();
 return {h:n.getHours(),m:n.getMinutes(),d:days[n.getDay()-1]};
}

function minutesUntilChange(offs){
 const now=nowInfo();
 let curr=now.h*60+now.m;
 let periods=offs.map(r=>{
   const [a,b]=r.split(":").map(Number);
   return [a*60,b*60];
 });
 periods.sort((a,b)=>a[0]-b[0]);
 // знайти наступну зміну
 for(let i=0;i<periods.length;i++){
   if(curr<periods[i][0]) return periods[i][0]-curr;
   if(curr>=periods[i][0] && curr<periods[i][1]) return periods[i][1]-curr;
 }
 return 24*60-curr; // до кінця дня
}

function render(){
 const wrap=document.getElementById("groups");
 wrap.innerHTML="";
 const now=nowInfo();
 Object.keys(groups).forEach(g=>{
  const offs=groups[g][now.d];
  let hasLight=true;
  offs.forEach(r=>{
    const [a,b]=r.split(":").map(Number);
    const start=a*60,end=b*60,curr=now.h*60+now.m;
    if(curr>=start && curr<end) hasLight=false;
  });

  let timeline="";
  for(let h=0;h<24;h++){
   let cls="hour";
   offs.forEach(r=>{
     const [a,b]=r.split(":").map(Number);
     if(h>=a && h<b) cls+=" off";
   });
   if(h===now.h) cls+=" now";
   timeline+=`<div class="${cls}"></div>`;
  }

  const minsChange=minutesUntilChange(offs);
  let timerText="";
  if(minsChange<60) timerText=`До зміни: ${minsChange} хв`;
  else timerText=`До зміни: ${Math.floor(minsChange/60)} год ${minsChange%60} хв`;

  wrap.innerHTML+=`
  <div class="group">
   <h3>Група ${g}</h3>
   <div class="status ${hasLight?'green blink':'red'}">
     ${hasLight?'🟢 Є світло':'🔴 Нема світла'} — ${timerText}
   </div>
   <div class="timeline">${timeline}</div>
  </div>`;
 });
}

// Оновлення часу
function updateLast(){
 const el=document.getElementById("lastUpdate");
 const t=localStorage.getItem("lastUpdate");
 if(!t){el.textContent="Останнє оновлення: щойно"; return;}
 const diff=Math.floor((Date.now()-t)/60000);
 if(diff<1) el.textContent="Останнє оновлення: щойно";
 else if(diff<60) el.textContent=`Останнє оновлення: ${diff} хв тому`;
 else if(diff<1440) el.textContent=`Останнє оновлення: ${Math.floor(diff/60)} год ${diff%60} хв тому`;
 else el.textContent=`Останнє оновлення: ${Math.floor(diff/1440)} дн ${Math.floor((diff%1440)/60)} год`;
}

// ADMIN
const adminBtn=document.getElementById("adminBtn");
const adminPanel=document.getElementById("adminPanel");
const adminContent=document.getElementById("adminContent");
const pass=document.getElementById("pass");
const hoursInput=document.getElementById("hours");
const daySel=document.getElementById("daySel");
const allGroups=document.getElementById("allGroups");

adminBtn.onclick=()=>adminPanel.classList.toggle("open");

function login(){
 if(pass.value===PASSWORD){adminContent.style.display="block";}
 else alert("Невірний пароль");
}

function save(){
 const text=hoursInput.value.trim();
 const day=daySel.value;
 let periods=text.split(",").map(s=>s.trim());
 if(allGroups.checked){
   Object.keys(groups).forEach(g=>groups[g][day]=periods);
 }
 localStorage.setItem("lastUpdate",Date.now());
 updateLast();
 render();
 alert("Збережено!");
}

render(); updateLast();
setInterval(()=>{render(); updateLast();},60000);
</script>

</body>
</html>
