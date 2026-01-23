<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<title>🌦 Погода</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
  margin:0;
  background:linear-gradient(180deg,#1e3c72,#2a5298);
  font-family:system-ui;
  color:#fff;
}
.weather-card{
  max-width:360px;
  margin:30px auto;
  background:rgba(0,0,0,.25);
  border-radius:20px;
  padding:20px;
  text-align:center;
  box-shadow:0 10px 30px rgba(0,0,0,.4);
}
.city{font-size:22px;font-weight:600}
.temp{font-size:48px;font-weight:700;margin:10px 0}
.desc{text-transform:capitalize;opacity:.9}
.details{
  display:flex;
  justify-content:space-around;
  margin-top:16px;
  font-size:14px;
}
.details div{opacity:.85}
small{opacity:.6;display:block;margin-top:12px}
</style>
</head>

<body>

<div class="weather-card">
  <div class="city" id="city">Завантаження...</div>
  <img id="icon" width="100">
  <div class="temp" id="temp">--°C</div>
  <div class="desc" id="desc"></div>

  <div class="details">
    <div>💨 <span id="wind"></span> м/с</div>
    <div>💧 <span id="hum"></span>%</div>
  </div>

  <small id="updated">Оновлення...</small>
</div>

<script>
// ====== НАЛАШТУВАННЯ ======
const CITY = "Lviv"; // <-- змінюй місто (Lviv, Mostyska, Kyiv...)
const API_KEY = "ВСТАВ_СВІЙ_API_КЛЮЧ"; // <-- ОБОВʼЯЗКОВО
const LANG = "uk";
// ==========================

async function loadWeather(){
  const url = `https://api.openweathermap.org/data/2.5/weather?q=${CITY}&units=metric&lang=${LANG}&appid=${API_KEY}`;
  const res = await fetch(url);
  const data = await res.json();

  document.getElementById("city").textContent = data.name;
  document.getElementById("temp").textContent = Math.round(data.main.temp) + "°C";
  document.getElementById("desc").textContent = data.weather[0].description;
  document.getElementById("wind").textContent = data.wind.speed;
  document.getElementById("hum").textContent = data.main.humidity;
  document.getElementById("icon").src =
    `https://openweathermap.org/img/wn/${data.weather[0].icon}@2x.png`;

  const now = new Date();
  document.getElementById("updated").textContent =
    "Оновлено о " + now.toLocaleTimeString("uk-UA",{hour:"2-digit",minute:"2-digit"});
}

loadWeather();
setInterval(loadWeather, 30 * 60 * 1000); // оновлення кожні 30 хв
</script>

</body>
</html>
