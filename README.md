<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>График отключений — Львовская область</title>

<style>
*{box-sizing:border-box}
body{
  margin:0;
  background:#020617;
  font-family:system-ui,sans-serif;
  color:#fff;
}
.app{
  max-width:430px;
  margin:auto;
  padding:10px;
}
h1{
  font-size:18px;
  margin:0 0 10px;
}

/* ===== ТАБЛИЦА ===== */
.wrap{
  overflow:auto;
}
.table{
  display:grid;
  grid-template-columns:40px repeat(11, 1fr) 40px;
  min-width:600px;
}
.cell, .head, .side{
  font-size:10px;
  text-align:center;
  padding:4px 2px;
}
.head, .side{
  background:#020617;
  opacity:.7;
}
.hour{
  background:#020617;
  font-size:10px;
}
.on{
  background:#22c55e;
  height:16px;
  border-radius:3px;
}
.off{
  background:#ef4444;
  height:16px;
  border-radius:3px;
}
.footer{
  font-size:11px;
  opacity:.6;
  margin-top:8px;
}
</style>
</head>

<body>
<div class="app">

<h1>⚡ Львовская область</h1>

<div class="wrap">
  <div class="table" id="table"></div>
</div>

<div class="footer">
🟢 есть свет · 🔴 нет света
</div>

</div>

<script>
const groups=["1.1","1.2","2.1","3.1","3.2","4.1","4.2","5.1","5.2","6.1","6.2"];

const data={
  "1.1":[[19,22]],
  "1.2":[],
  "2.1":[[19,21]],
  "3.1":[[21,24]],
  "3.2":[],
  "4.1":[],
  "4.2":[],
  "5.1":[[0,3]],
  "5.2":[[19,21]],
  "6.1":[],
  "6.2":[[21,24]]
};

function build(){
  table.innerHTML="";

  // верхняя строка
  table.innerHTML+=`<div></div>`;
  groups.forEach(g=>table.innerHTML+=`<div class="head">${g}</div>`);
  table.innerHTML+=`<div></div>`;

  for(let h=0;h<24;h++){
    // слева час
    table.innerHTML+=`<div class="side">${h}:00</div>`;

    groups.forEach(g=>{
      let off=false;
      data[g].forEach(p=>{
        if(h>=p[0] && h<p[1]) off=true;
      });
      table.innerHTML+=`<div class="cell"><div class="${off?"off":"on"}"></div></div>`;
    });

    // справа час
    table.innerHTML+=`<div class="side">${h}:00</div>`;
  }

  // нижняя строка
  table.innerHTML+=`<div></div>`;
  groups.forEach(g=>table.innerHTML+=`<div class="head">${g}</div>`);
  table.innerHTML+=`<div></div>`;
}

build();
</script>
</body>
</html>
