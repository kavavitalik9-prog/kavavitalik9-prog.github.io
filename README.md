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
 padding:14px 50px 14px 16px;
 border-bottom:1px solid #222;
 font-weight:600;
 position:relative;
}
#adminBtn{
 position:absolute;top:12px;right:12px;
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

.status{font-size:14px}
.green{color:var(--green)}
.red{color:var(--red)}
.blink{animation:blink 1.6s infinite}
@keyframes blink{50%{opacity:.5}}

.timeline{
 display:flex;
 gap:2px;
 margin-top:8px;
}
.hour{
 flex:1;
 height:14px;
 background:#333;
 border-radius:4px;
}
.off{background:var(--red)}
.now{outline:2px solid #fff}

/* ADMIN */
#adminPanel{
 position:fixed;top:0;right:-100%;
 width:320px;height:100%;
 background:#14161c;
 border-left:1px solid #222;
 padding:14px;
 transition:.3s;
 z-index:10;
}
#adminPanel.open{right:0}
select,input,textarea,button{
 width:100%;margin-top:8px;
 padding:8px;border-radius:8px;border:none
}
button{background:#2b6cff;color:#fff}
small{opacity:.6}
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
<select id="groupSel"></select>
<select id="daySel">
<option>Пн</option><option>Вт</option><option>Ср</option>
<option>Чт</option><option>Пт</option><option>Сб</option><option>Нд</option>
</select>
<small>Формат: 08-10, 18-20</small>
<textarea id="hours" rows="4"></textarea>
<button onclick="save()">Зберегти</button>
</div>
</div>

<script>
const PASSWORD="3709";
const days=["Пн","Вт","Ср","Чт","Пт","Сб","Нд"];
const groups={};

for(let g=1;g<=6;g++){
 for(let s=1;s<=2;s++){
   groups[`${g}.${s}`]={};
   days.forEach(d=>groups[`${g}.${s}`][d]=[]);
 }
}

const groupSel=document.getElementById("groupSel");
Object.keys(groups).forEach(g=>{
 groupSel.innerHTML+=`<option>${g}</option>`;
});

function nowInfo(){
 const n=new Date();
 return {h:n.getHours(),m:n.getMinutes(),d:days[n.getDay()-1]};
}

function render(){
 const wrap=document.getElementById("groups");
 wrap.innerHTML="";
 const now=nowInfo();

 Object.keys(groups).forEach(g=>{
  const offs=groups[g][now.d]||[];
  let hasLight=true;
  offs.forEach(r=>{
   const [a,b]=r.split("-");
   if(now.h>=+a && now.h<+b) hasLight=false;
  });

  let timeline="";
  for(let h=0;h<24;h++){
   let cls="hour";
   offs.forEach(r=>{
    const [a,b]=r.split("-");
    if(h>=+a && h<+b) cls+=" off";
   });
   if(h===now.h) cls+=" now";
   timeline+=`<div class="${cls}"></div>`;
  }

  wrap.innerHTML+=`
  <div class="group">
   <h3>Група ${g}</h3>
   <div class="status ${hasLight?'green blink':'red'}">
    ${hasLight?'🟢 Є світло':'🔴 Нема світла'}
   </div>
   <div class="timeline">${timeline}</div>
  </div>`;
 });
}
render();
setInterval(render,60000);

/* ADMIN */
adminBtn.onclick=()=>adminPanel.classList.toggle("open");
function login(){
 if(pass.value===PASSWORD)
  adminContent.style.display="block";
 else alert("Невірний пароль");
}
function save(){
 const g=groupSel.value;
 const d=daySel.value;
 groups[g][d]=hours.value.split(",").map(x=>x.trim());
 alert("Збережено");
 render();
}
</script>
</body>
</html>
