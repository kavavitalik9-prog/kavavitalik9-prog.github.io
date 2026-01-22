<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Мої карти</title>
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
  max-width:430px;
  width:100%;
  min-height:100vh;
  padding:12px 12px 90px;
}
.header{
  background:#151a26;
  border-radius:18px;
  padding:14px;
  margin-bottom:10px;
}
.card{
  background:#151a26;
  border-radius:18px;
  margin-bottom:12px;
  overflow:hidden;
}
.card img{
  width:100%;
  display:block;
}
.cardInfo{
  padding:10px;
}
.small{
  font-size:13px;
  opacity:.8;
}

/* Кнопка адміна збоку */
.adminBtn{
  position:fixed;
  right:0;
  top:50%;
  transform:translateY(-50%);
  background:#222;
  color:#fff;
  border:none;
  border-radius:14px 0 0 14px;
  padding:12px;
  font-size:18px;
  z-index:10;
  cursor:pointer;
}

/* Модальні вікна */
.modal{
  position:fixed;
  inset:0;
  background:#000a;
  display:none;
  justify-content:center;
  align-items:center;
  z-index:20;
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
button{
  background:#2b61ff;
}
button.close{
  background:#333;
}
</style>
</head>
<body>

<div class="phone">
  <div class="header">
    <b>🗺 Мої карти</b><br>
    <span class="small" id="updated"></span>
  </div>
  <div id="list"></div>
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

<!-- ADMIN -->
<div class="modal" id="admin">
  <div class="modalBox">
    <h3>Додати карту</h3>
    <input id="title" placeholder="Назва карти">
    <input type="file" id="file">
    <button onclick="addMap()">Додати карту</button>
    <button class="close" onclick="closeAll()">Закрити</button>
  </div>
</div>

<script>
const PASS="3709";
let maps=JSON.parse(localStorage.getItem("maps"))||[];
const list=document.getElementById("list");
const updatedEl=document.getElementById("updated");

function formatAgo(time){
  let m=Math.floor((Date.now()-time)/60000);
  if(m<60) return `${m} хв тому`;
  let h=Math.floor(m/60);
  let mm=m%60;
  if(h<24) return `${h} год ${mm} хв тому`;
  let d=Math.floor(h/24);
  return `${d} д ${mm} год тому`;
}

function render(){
  list.innerHTML="";
  if(maps.length===0){
    list.innerHTML="<div class='small'>Карт ще немає</div>";
    updatedEl.textContent="";
    return;
  }
  maps.forEach(m=>{
    list.innerHTML+=`
      <div class="card">
        <img src="${m.img}">
        <div class="cardInfo">
          <b>${m.title}</b><br>
          <span class="small">Останнє оновлення: ${formatAgo(m.time)}</span>
        </div>
      </div>
    `;
  });
  updatedEl.textContent=`Всього карт: ${maps.length}`;
}

render();
setInterval(render,60000);

/* ADMIN FUNCTIONS */
function openLogin(){login.style.display="flex"}
function closeAll(){login.style.display="none";admin.style.display="none"}

function check(){
  if(pass.value!==PASS){alert("Невірний пароль"); return;}
  login.style.display="none";
  admin.style.display="flex";
}

function addMap(){
  let f=file.files[0];
  if(!f){alert("Обери файл"); return;}
  let r=new FileReader();
  r.onload=()=>{
    maps.unshift({
      title:title.value||"Без назви",
      img:r.result,
      time:Date.now()
    });
    localStorage.setItem("maps",JSON.stringify(maps));
    closeAll();
    render();
  }
  r.readAsDataURL(f);
}
</script>
</body>
</html>
