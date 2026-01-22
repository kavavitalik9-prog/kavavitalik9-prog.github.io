<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Графік світла — Львівська область</title>
<style>
body{margin:0;font-family:system-ui;background:#0b0d13;color:#fff;display:flex;justify-content:center;}
.phone{max-width:430px;width:100%;min-height:100vh;padding:12px;}
.header{background:#151a26;border-radius:18px;padding:14px;margin-bottom:10px;text-align:center;}
.group{background:#151a26;border-radius:16px;margin-bottom:12px;padding:12px;}
.status{font-weight:bold;margin-top:6px;}
.green{color:#00ff00;}
.red{color:#ff3333;}
.small{font-size:12px;opacity:.7;}
.adminBtn{position:fixed;right:0;top:50%;transform:translateY(-50%);background:#222;color:#fff;border:none;border-radius:14px 0 0 14px;padding:12px;font-size:18px;cursor:pointer;z-index:100;}
.modal{position:fixed;inset:0;background:rgba(0,0,0,0.7);display:none;justify-content:center;align-items:center;z-index:200;}
.modalBox{background:#151a26;padding:16px;border-radius:14px;width:90%;max-width:380px;}
input,button,textarea{width:100%;margin-top:10px;padding:10px;border:none;border-radius:10px;background:#222;color:#fff;}
button{background:#2b61ff;cursor:pointer;}
button.close{background:#333;}
</style>
</head>
<body>
<div class="phone">
  <div class="header">
    <b>⚡ Львівська область</b><br>
    <span id="updated" class="small">Останнє оновлення: 0 хвилин тому</span>
  </div>
  <div id="groups"></div>
</div>

<button class="adminBtn" onclick="openLogin()">⚙</button>

<!-- LOGIN -->
<div class="modal" id="login">
  <div class="modalBox">
    <h3>Адмін доступ</h3>
    <input id="pass" placeholder="Пароль">
    <button onclick="check()">Увійти</button>
    <button class="close" onclick="closeAll()">Скасувати</button>
  </div>
</div>

<!-- ADMIN PANEL -->
<div class="modal" id="admin">
  <div class="modalBox">
    <h3>Редагування групи</h3>
    <select id="selectGroup"></select>
    <textarea id="inputPeriods" rows="5" placeholder="Введи години без світла, наприклад: 01:00-03:00,13:30-17:00"></textarea>
    <button onclick="saveGroup()">Зберегти</button>
    <button class="close" onclick="closeAll()">Закрити</button>
  </div>
</div>

<script>
// Початкові дані
let data = JSON.parse(localStorage.getItem("data")) || {
  "1.1": [],
  "1.2": [],
  "2.1": [],
  "2.2": [],
  "3.1": [],
  "3.2": [],
  "4.1": [],
  "4.2": [],
  "5.1": [],
  "5.2": [],
  "6.1": [],
  "6.2": []
};
let lastUpdate = parseInt(localStorage.getItem("lastUpdate")) || Date.now();

const container=document.getElementById("groups");
const updated=document.getElementById("updated");

// Функції для часу
function timeToMinutes(str){const [h,m]=str.split(":").map(Number);return h*60+m;}
function formatDiff(ms){
  let diff=Math.floor((Date.now()-ms)/1000); 
  const days=Math.floor(diff/86400); diff%=86400;
  const hours=Math.floor(diff/3600); diff%=3600;
  const minutes=Math.floor(diff/60); 
  let str="";
  if(days>0) str+=days+"д ";
  if(hours>0) str+=hours+"г ";
  str+=minutes+"хв";
  return str;
}

// Відображення груп
function render(){
  container.innerHTML="";
  updated.textContent="Останнє оновлення: "+formatDiff(lastUpdate)+" тому";
  const now=new Date();
  const nowMinutes=now.getHours()*60+now.getMinutes();
  for(const group in data){
    const div=document.createElement("div");
    div.className="group";
    let statusText="", statusClass="";
    let periods=data[group];
    if(periods.length===0){statusText="🟢 ЗАРАЗ Є СВІТЛО"; statusClass="green";}
    for(const p of periods){
      const startM=timeToMinutes(p.start);
      const endM=timeToMinutes(p.end);
      if(nowMinutes>=startM && nowMinutes<endM){
        statusText=p.light?"🟢 ЗАРАЗ Є СВІТЛО":"⚫ ЗАРАЗ НЕМАЄ СВІТЛА";
        statusClass=p.light?"green":"red";
      }
    }
    // Таймер
    let nextChange=null;
    for(const p of periods){
      const startM=timeToMinutes(p.start);
      const endM=timeToMinutes(p.end);
      if(nowMinutes<endM){nextChange=endM;break;}
    }
    let timer="";
    if(nextChange!==null){
      let diff=nextChange-nowMinutes;
      const h=Math.floor(diff/60);
      const m=diff%60;
      timer=`До зміни: ${h>0?h+"г ":""}${m}хв`;
    }
    div.innerHTML=`<b>${group}</b><br><span class="status ${statusClass}">${statusText}</span><br><span class="small">${timer}</span>`;
    container.appendChild(div);
  }
}
render();
setInterval(render,60000);

// --- Адмінка ---
const login=document.getElementById("login");
const admin=document.getElementById("admin");
const pass=document.getElementById("pass");
const selectGroup=document.getElementById("selectGroup");
const inputPeriods=document.getElementById("inputPeriods");

function openLogin(){login.style.display="flex";}
function closeAll(){login.style.display="none"; admin.style.display="none";}

function check(){
  if(pass.value!=="3709"){alert("Невірний пароль");return;}
  login.style.display="none";
  admin.style.display="flex";
  selectGroup.innerHTML="";
  for(const g in data){
    let opt=document.createElement("option");
    opt.value=g; opt.textContent=g;
    selectGroup.appendChild(opt);
  }
  loadGroup();
}
function loadGroup(){
  const g=selectGroup.value;
  let arr=data[g].filter(p=>!p.light).map(p=>p.start+"-"+p.end);
  inputPeriods.value=arr.join(",");
}
selectGroup.addEventListener("change",loadGroup);

function saveGroup(){
  const g=selectGroup.value;
  let raw=inputPeriods.value.split(",").map(s=>s.trim()).filter(s=>s.length>0);
  let periods=[];
  let prev=0;
  for(const s of raw){
    let [start,end]=s.split("-");
    if(timeToMinutes(start)>prev){periods.push({start:minutesToStr(prev),end:start,light:true});}
    periods.push({start:start,end:end,light:false});
    prev=timeToMinutes(end);
  }
  if(prev<1440){periods.push({start:minutesToStr(prev),end:"24:00",light:true});}
  data[g]=periods;
  lastUpdate=Date.now();
  localStorage.setItem("data",JSON.stringify(data));
  localStorage.setItem("lastUpdate",lastUpdate);
  render();
  alert("Збережено!");
}

function minutesToStr(m){
  let h=Math.floor(m/60); let min=m%60;
  return (h<10?"0"+h:h)+":"+(min<10?"0"+min:min);
}
</script>
</body>
</html>
