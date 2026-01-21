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
  max-width:950px;
  margin:auto;
  padding:15px;
}
.header{
  display:flex;
  justify-content:space-between;
  align-items:center;
  flex-wrap:wrap;
  gap:10px;
}
.viewers{
  border:1px solid #ffd000;
  padding:6px 12px;
  border-radius:10px;
  background:#111;
  font-size:14px;
}
select,button{
  width:100%;
  padding:10px;
  border-radius:8px;
  border:none;
  margin:6px 0;
  font-size:16px;
}
button{
  background:#222;
  color:#fff;
}
.card{
  background:#151515;
  border-radius:12px;
  padding:15px;
  margin-top:10px;
}
.groupCard{
  border:1px solid #222;
  border-radius:10px;
  padding:10px;
  margin-bottom:10px;
}
.row{
  display:flex;
  justify-content:space-between;
  border-bottom:1px solid #222;
  padding:5px 0;
  font-size:14px;
}
.row:last-child{border:none;}
.on{color:#4cff4c;}
.off{color:#ff4c4c;}
.now{
  background:#222;
  border-radius:6px;
  padding:4px 6px;
}
.timer{
  margin:8px 0;
  font-size:15px;
}
.pin{
  color:#ffd000;
  font-size:14px;
}
footer{
  text-align:center;
  opacity:.5;
  margin:20px 0;
  font-size:13px;
}
</style>
</head>

<body>
<div class="container">

<div class="header">
  <h2>⚡ Львівська область</h2>
  <div class="viewers">👁 <span id="viewers"></span></div>
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

<select id="group"></select>

<button onclick="pinGroup()">📌 Закріпити мою групу</button>
<button onclick="toggleAll()">📊 Показати всі групи</button>

<div id="content" class="card"></div>

</div>

<footer>Оновлюється автоматично • Демонстрація</footer>

<script>
// ====== ФЕЙК ОНЛАЙН ======
let viewers=Math.floor(Math.random()*(700000-975)+975);
const v=document.getElementById("viewers");
function updView(){
  viewers+=Math.floor(Math.random()*4000-2000);
  viewers=Math.max(975,Math.min(700000,viewers));
  v.textContent=viewers.toLocaleString("uk-UA");
}
updView(); setInterval(updView,3000);

// ====== ГРУПИ ======
const groups=["1.1","1.2","2.1","2.2","3.1","3.2","4.1","4.2","5.1","5.2","6.1","6.2"];
const groupSelect=document.getElementById("group");
groups.forEach(g=>{
  let o=document.createElement("option");
  o.textContent=g; groupSelect.appendChild(o);
});
if(localStorage.group) groupSelect.value=localStorage.group;

// ====== ГРАФІК СЕРЕДИ ======
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

function min(t){let[a,b]=t.split(":");return +a*60+ +b;}
let showAll=false;

function toggleAll(){
  showAll=!showAll;
  render();
}

function pinGroup(){
  localStorage.group=groupSelect.value;
  alert("Групу закріплено ✅");
}

function timerText(next){
  let d=Math.max(0,next*60);
  let h=Math.floor(d/3600);
  let m=Math.floor((d%3600)/60);
  let s=d%60;
  return `${h}г ${m}хв ${s}с`;
}

function render(){
  const box=document.getElementById("content");
  box.innerHTML="";
  if(day.value!=="wednesday"){
    box.innerHTML="⏳ <b>Графік ще формується</b>";
    return;
  }

  const now=new Date();
  const cur=now.getHours()*60+now.getMinutes()+now.getSeconds()/60;

  const renderGroup=(g)=>{
    let html=`<div class="groupCard"><b>Група ${g}</b>`;
    if(localStorage.group===g) html+=` <span class="pin">📌</span>`;
    let nextChange=null;

    schedule[g].forEach(i=>{
      const s=min(i[0]), e=min(i[1]);
      const isNow=cur>=s && cur<=e;
      if(isNow) nextChange=e-cur;
      html+=`
      <div class="row ${isNow?"now":""}">
        <span>${i[0]}–${i[1]}</span>
        <span class="${i[2]}">${i[2]==="on"?"⚡ є":"⛔ нема"}</span>
      </div>`;
    });

    if(nextChange!==null){
      html+=`<div class="timer">⏱ До зміни: ${timerText(nextChange)}</div>`;
    }

    html+="</div>";
    box.innerHTML+=html;
  };

  if(showAll){
    groups.forEach(renderGroup);
  }else{
    renderGroup(groupSelect.value);
  }
}

day.onchange=render;
groupSelect.onchange=render;
setInterval(render,1000);
render();
</script>
</body>
</html>
