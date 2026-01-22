<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Моя карта</title>

<style>
body{
  margin:0;
  background:#0b0d13;
  font-family:system-ui;
  color:#fff;
  display:flex;
  justify-content:center;
}
.phone{
  width:100%;
  max-width:430px;
  min-height:100vh;
  padding:12px;
}
.header{
  background:#151a26;
  border-radius:18px;
  padding:14px;
  margin-bottom:10px;
}
.mapBox{
  background:#151a26;
  border-radius:18px;
  overflow:hidden;
}
.mapBox img{
  width:100%;
  display:block;
}
.info{
  padding:10px;
  font-size:14px;
  opacity:.85;
}
.adminBtn{
  position:fixed;
  right:0;
  top:40%;
  background:#222;
  color:#fff;
  border:none;
  border-radius:14px 0 0 14px;
  padding:12px;
  font-size:18px;
}
.modal{
  position:fixed;
  inset:0;
  background:#000a;
  display:none;
  justify-content:center;
  align-items:center;
}
.modalBox{
  background:#151a26;
  padding:16px;
  border-radius:16px;
  width:90%;
  max-width:380px;
}
input,button{
  width:100%;
  margin-top:10px;
  padding:10px;
  border-radius:10px;
  border:none;
  background:#222;
  color:#fff;
}
.viewers{
  background:#1d2333;
  border-radius:12px;
  padding:6px 10px;
  font-size:13px;
  display:inline-block;
  margin-top:6px;
}
</style>
</head>

<body>
<div class="phone">
  <div class="header">
    <b id="title">🗺 Моя карта</b><br>
    <span id="updated"></span><br>
    <span class="viewers">👁 <span id="viewers"></span></span>
  </div>

  <div class="mapBox">
    <img id="mapImg" src="">
    <div class="info">Супутниковий знімок</div>
  </div>
</div>

<button class="adminBtn" onclick="openAdmin()">⚙</button>

<div class="modal" id="admin">
  <div class="modalBox">
    <h3>Адмін панель</h3>
    <input id="pass" placeholder="Пароль">
    <input id="mapTitle" placeholder="Назва карти">
    <input type="file" id="file">
    <button onclick="save()">Зберегти</button>
    <button onclick="closeAdmin()">Закрити</button>
  </div>
</div>

<script>
const PASS="3709";

let data=JSON.parse(localStorage.getItem("mapData"))||{
  title:"🗺 Моя карта",
  img:"",
  updated:Date.now()
};

const titleEl=document.getElementById("title");
const imgEl=document.getElementById("mapImg");
const updatedEl=document.getElementById("updated");

function formatAgo(ms){
  let m=Math.floor((Date.now()-ms)/60000);
  if(m<60) return `Останнє оновлення: ${m} хв тому`;
  let h=Math.floor(m/60);
  let mm=m%60;
  if(h<24) return `Останнє оновлення: ${h} год ${mm} хв тому`;
  let d=Math.floor(h/24);
  return `Останнє оновлення: ${d} д тому`;
}

function render(){
  titleEl.textContent=data.title;
  if(data.img) imgEl.src=data.img;
  updatedEl.textContent=formatAgo(data.updated);
}
render();
setInterval(render,60000);

/* viewers (fake) */
let viewers=975+Math.floor(Math.random()*2000);
setInterval(()=>{
  viewers+=Math.floor(Math.random()*3);
  document.getElementById("viewers").textContent=viewers;
},2000);

function openAdmin(){admin.style.display="flex"}
function closeAdmin(){admin.style.display="none"}

function save(){
  if(pass.value!==PASS){alert("Невірний пароль");return;}
  data.title=mapTitle.value||data.title;

  let f=file.files[0];
  if(f){
    let r=new FileReader();
    r.onload=()=>{
      data.img=r.result;
      data.updated=Date.now();
      localStorage.setItem("mapData",JSON.stringify(data));
      closeAdmin(); render();
    }
    r.readAsDataURL(f);
  }else{
    data.updated=Date.now();
    localStorage.setItem("mapData",JSON.stringify(data));
    closeAdmin(); render();
  }
}
</script>
</body>
</html>
