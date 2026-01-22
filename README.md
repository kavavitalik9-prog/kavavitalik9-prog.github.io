<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Графік світла — Львівська область</title>
<style>
body{
  margin:0;
  font-family:system-ui;
  background:#0b0d13;
  color:#fff;
  display:flex;
  justify-content:center;
}
.phone{
  max-width:430px;
  width:100%;
  min-height:100vh;
  padding:12px;
}
.header{
  background:#151a26;
  border-radius:18px;
  padding:14px;
  margin-bottom:10px;
  text-align:center;
}
.group{
  background:#151a26;
  border-radius:16px;
  margin-bottom:12px;
  padding:12px;
}
.status{
  font-weight:bold;
  margin-top:6px;
}
.green{color:#00ff00;}
.red{color:#ff3333;}
.small{font-size:12px;opacity:.7;}
</style>
</head>
<body>
<div class="phone">
  <div class="header">
    <b>⚡ Львівська область</b><br>
    <span id="updated" class="small">Останнє оновлення: 08:29 22 січня 2026</span>
  </div>
  <div id="groups"></div>
</div>

<script>
// Графік на четвер
const data = {
  "1.1":[
    {start:"00:00", end:"01:00", light:true},
    {start:"01:00", end:"03:00", light:false},
    {start:"03:00", end:"13:30", light:true},
    {start:"13:30", end:"17:00", light:false},
    {start:"17:00", end:"24:00", light:true}
  ],
  "1.2":[
    {start:"00:00", end:"06:30", light:true},
    {start:"06:30", end:"10:00", light:false},
    {start:"10:00", end:"13:30", light:true},
    {start:"13:30", end:"17:00", light:false},
    {start:"17:00", end:"24:00", light:true}
  ],
  "2.1":[
    {start:"00:00", end:"01:30", light:true},
    {start:"01:30", end:"03:00", light:false},
    {start:"03:00", end:"10:00", light:true},
    {start:"10:00", end:"13:30", light:false},
    {start:"13:30", end:"24:00", light:true}
  ],
  "2.2":[
    {start:"00:00", end:"10:00", light:true},
    {start:"10:00", end:"13:30", light:false},
    {start:"13:30", end:"17:00", light:true},
    {start:"17:00", end:"22:00", light:false},
    {start:"22:00", end:"24:00", light:true}
  ],
  "3.1":[
    {start:"00:00", end:"01:00", light:true},
    {start:"01:00", end:"03:00", light:false},
    {start:"03:00", end:"17:00", light:true},
    {start:"17:00", end:"22:00", light:false},
    {start:"22:00", end:"24:00", light:true}
  ],
  "3.2":[
    {start:"00:00", end:"10:00", light:true},
    {start:"10:00", end:"13:30", light:false},
    {start:"13:30", end:"17:00", light:true},
    {start:"17:00", end:"22:00", light:false},
    {start:"22:00", end:"24:00", light:true}
  ],
  "4.1":[
    {start:"00:00", end:"06:30", light:true},
    {start:"06:30", end:"10:00", light:false},
    {start:"10:00", end:"13:30", light:true},
    {start:"13:30", end:"17:00", light:false},
    {start:"17:00", end:"24:00", light:true}
  ],
  "4.2":[
    {start:"00:00", end:"03:00", light:true},
    {start:"03:00", end:"04:00", light:false},
    {start:"04:00", end:"06:00", light:true},
    {start:"06:00", end:"06:30", light:false},
    {start:"06:30", end:"20:30", light:true},
    {start:"20:30", end:"24:00", light:false}
  ],
  "5.1":[
    {start:"00:00", end:"10:00", light:true},
    {start:"10:00", end:"13:30", light:false},
    {start:"13:30", end:"17:00", light:true},
    {start:"17:00", end:"20:30", light:false},
    {start:"20:30", end:"24:00", light:true}
  ],
  "5.2":[
    {start:"00:00", end:"01:30", light:true},
    {start:"01:30", end:"03:00", light:false},
    {start:"03:00", end:"13:30", light:true},
    {start:"13:30", end:"17:00", light:false},
    {start:"17:00", end:"20:30", light:true},
    {start:"20:30", end:"24:00", light:false}
  ],
  "6.1":[
    {start:"00:00", end:"03:00", light:true},
    {start:"03:00", end:"04:00", light:false},
    {start:"04:00", end:"06:00", light:true},
    {start:"06:00", end:"06:30", light:false},
    {start:"06:30", end:"13:30", light:true},
    {start:"13:30", end:"17:00", light:false},
    {start:"17:00", end:"24:00", light:true}
  ],
  "6.2":[
    {start:"00:00", end:"10:00", light:true},
    {start:"10:00", end:"13:30", light:false},
    {start:"13:30", end:"17:00", light:true},
    {start:"17:00", end:"20:30", light:false},
    {start:"20:30", end:"24:00", light:true}
  ]
};

// Відображення груп
const container=document.getElementById("groups");

function timeToMinutes(str){
  const [h,m]=str.split(":").map(Number);
  return h*60+m;
}

function render(){
  container.innerHTML="";
  const now=new Date();
  const nowMinutes=now.getHours()*60+now.getMinutes();
  for(const group in data){
    const div=document.createElement("div");
    div.className="group";
    let statusText="", statusClass="";
    const periods=data[group];
    for(const p of periods){
      const startM=timeToMinutes(p.start);
      const endM=timeToMinutes(p.end);
      if(nowMinutes>=startM && nowMinutes<endM){
        statusText=p.light?"🟢 ЗАРАЗ Є СВІТЛО":"⚫ ЗАРАЗ НЕМАЄ СВІТЛА";
        statusClass=p.light?"green":"red";
      }
    }
    // Таймер до зміни
    let nextChange=null;
    for(const p of periods){
      const startM=timeToMinutes(p.start);
      const endM=timeToMinutes(p.end);
      if(nowMinutes<endM){
        nextChange=endM;
        break;
      }
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

</script>
</body>
</html>
