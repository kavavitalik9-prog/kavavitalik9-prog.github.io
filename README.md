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
header{padding:14px 16px;border-bottom:1px solid #222;font-weight:600;display:flex;justify-content:space-between;align-items:center}
main{padding:14px}
.group{background:var(--card);border-radius:14px;padding:12px;margin-bottom:14px}
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
#adminContent{background:#1a1d24;padding:12px;margin-top:20px;border-radius:10px}
input,textarea,button{width:100%;margin-top:6px;padding:8px;border-radius:6px;border:none}
button{background:#2b6cff;color:#fff;cursor:pointer}
small{opacity:.6;margin-top:4px;display:block}
</style>
</head>
<body>

<header>
⚡ Львівська область
<div id="lastUpdate">Останнє оновлення: щойно</div>
</header>

<main id="groups"></main>

<div id="adminContent">
<h3>Редагування графіків</h3>
<small>Вводь години без світла, формат 18:00-22:00, через кому</small>
<textarea id="hours1" rows="2" placeholder="Група 1.1"></textarea>
<textarea id="hours2" rows="2" placeholder="Група 1.2"></textarea>
<textarea id="hours3" rows="2" placeholder="Група 2.1"></textarea>
<textarea id="hours4" rows="2" placeholder="Група 2.2"></textarea>
<textarea id="hours5" rows="2" placeholder="Група 3.1"></textarea>
<textarea id="hours6" rows="2" placeholder="Група 3.2"></textarea>
<textarea id="hours7" rows="2" placeholder="Група 4.1"></textarea>
<textarea id="hours8" rows="2" placeholder="Група 4.2"></textarea>
<textarea id="hours9" rows="2" placeholder="Група 5.1"></textarea>
<textarea id="hours10" rows="2" placeholder="Група 5.2"></textarea>
<textarea id="hours11" rows="2" placeholder="Група 6.1"></textarea>
<textarea id="hours12" rows="2" placeholder="Група 6.2"></textarea>
<select id="daySel">
  <option>Пн</option><option>Вт</option><option>Ср</option>
  <option>Чт</option><option>Пт</option><option>Сб</option><option>Нд</option>
</select>
<button onclick="save()">Зберегти графіки</button>
</div>

<script>
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
 for(let i=0;i<periods.length;i++){
   if(curr<periods[i][0]) return periods[i][0]-curr;
   if(curr>=periods[i][0] && curr<periods[i][1]) return periods[i][1]-curr;
 }
 return 24*60-curr;
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

// Збереження графіків
function save(){
 const day=daySel.value;
 for(let i=1;i<=12;i++){
   const val=document.getElementById(`hours${i}`).value.trim();
   if(val) groups[`${Math.ceil(i/2)}.${i%2===0?2:1}`][day]=val.split(",").map(s=>s.trim());
 }
 localStorage.setItem("lastUpdate",Date.now());
 updateLast();
 render();
 alert("Графіки оновлено!");
}

render(); updateLast();
setInterval(()=>{render(); updateLast();},60000);
</script>

</body>
</html>
