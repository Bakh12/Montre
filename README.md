<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Clock with Alarm &amp; Themes</title>

<style>
/* ======= Reset / Layout ======= */
*{box-sizing:border-box;margin:0;padding:0}
body{
  display:flex;flex-direction:column;align-items:center;justify-content:flex-start;
  min-height:100vh;background:#222;color:#fff;font-family:Arial,Helvetica,sans-serif;
  padding:20px;transition:background .5s;
}
h1{margin-bottom:10px;font-size:1.4rem}

/* ======= Clock ======= */
.clock{
  position:relative;width:300px;height:300px;margin:20px auto;
  border:12px solid #555;border-radius:50%;
  background:linear-gradient(135deg,#333,#222);
  transition:background .5s,border-color .5s;
}
.hand{
  position:absolute;top:50%;left:50%;transform-origin:left;
  width:50%;height:6px;border-radius:3px;background:#fff;transform:rotate(0deg);
  transition:background .3s;
}
.hand.hour  {height:8px;width:35%}
.hand.minute{height:6px;width:45%}
.hand.second{height:4px;width:48%}

/* ======= Theme-dependent colours ======= */
body.theme-default{background:#222}
body.theme-night  {background:#000033}
body.theme-alarm  {background:#660000}

body.theme-default .hand.hour  {background:#f39c12}
body.theme-default .hand.minute{background:#3498db}
body.theme-default .hand.second{background:#e74c3c}

body.theme-night .hand.hour  {background:#f1c40f}
body.theme-night .hand.minute{background:#2980b9}
body.theme-night .hand.second{background:#c0392b}

body.theme-alarm .hand.hour  {background:#ffeb3b}
body.theme-alarm .hand.minute{background:#ff9800}
body.theme-alarm .hand.second{background:#ff0000}

/* ======= Buttons & Controls ======= */
#themeButtons,#alarmControls{display:flex;gap:10px;margin-top:20px;flex-wrap:wrap}
button{
  padding:8px 14px;border:none;border-radius:4px;font-size:14px;cursor:pointer;
  background:#555;color:#fff;transition:background .3s;
}
button:hover:not(:disabled){background:#777}
button:disabled{opacity:.4;cursor:not-allowed}

input[type="time"]{
  padding:8px 6px;border:none;border-radius:4px;font-size:14px;
  background:#fff;color:#000
}
.notice{
  margin-top:15px;font-size:1rem;min-height:1.2em;color:#ffeb3b;text-align:center
}
</style>
</head>

<body class="theme-default">
  <h1>ساعة تناظرية مع منبّه</h1>

  <!-- Clock Dial -->
  <div class="clock">
    <div class="hand hour"   id="hourHand"></div>
    <div class="hand minute" id="minuteHand"></div>
    <div class="hand second" id="secondHand"></div>
  </div>

  <!-- Alarm controls -->
  <div id="alarmControls">
    <input type="time" id="alarmTime">
    <button id="setAlarmBtn">تفعيل المنبّه</button>
    <button id="clearAlarmBtn" disabled>إلغاء المنبّه</button>
  </div>

  <!-- Theme buttons -->
  <div id="themeButtons">
    <button data-theme="default">المظهر الافتراضي</button>
    <button data-theme="night">مظهر ليلي</button>
  </div>

  <!-- Status / notices -->
  <div class="notice" id="notice"></div>

<script>
/* ========= Clock Logic ========= */
const hourHand   = document.getElementById('hourHand');
const minuteHand = document.getElementById('minuteHand');
const secondHand = document.getElementById('secondHand');
let alarmTime    = null;
let alarmTimeout = null;

/* تحديث عقارب الساعة كل ثانية */
function updateClock(){
  const now = new Date();
  const seconds = now.getSeconds();
  const minutes = now.getMinutes();
  const hours   = now.getHours()%12 + minutes/60;

  secondHand.style.transform = `rotate(${seconds*6}deg)`;
  minuteHand.style.transform = `rotate(${minutes*6}deg)`;
  hourHand.style.transform   = `rotate(${hours*30}deg)`;

  checkAlarm(now);
}
setInterval(updateClock,1000);
updateClock(); // أول تشغيل

/* ========= Alarm Logic ========= */
const alarmInput   = document.getElementById('alarmTime');
const setAlarmBtn  = document.getElementById('setAlarmBtn');
const clearAlarmBtn= document.getElementById('clearAlarmBtn');
const notice       = document.getElementById('notice');

setAlarmBtn.addEventListener('click',()=>{
  if(!alarmInput.value){return alert('اختر وقت المنبّه أولاً!');}
  alarmTime = alarmInput.value; // HH:MM
  notice.textContent = `منبّه مضبوط على ${alarmTime}`;
  setAlarmBtn.disabled = true;
  clearAlarmBtn.disabled = false;
});

clearAlarmBtn.addEventListener('click',clearAlarm);

function clearAlarm(){
  alarmTime = null;
  notice.textContent = 'تم تعطيل المنبّه';
  document.body.classList.remove('theme-alarm');
  setAlarmBtn.disabled = false;
  clearAlarmBtn.disabled = true;
}

function checkAlarm(now){
  if(!alarmTime) return;

  const [alarmH,alarmM] = alarmTime.split(':').map(Number);
  if(now.getHours()===alarmH && now.getMinutes()===alarmM && now.getSeconds()===0){
    triggerAlarm();
  }
}

function triggerAlarm(){
  document.body.classList.add('theme-alarm');
  notice.textContent = '⏰⏰⏰ وقت المنبّه! ⏰⏰⏰';

  /* صفّارة بسيطة باستخدام Web Audio API */
  const ctx = new (window.AudioContext||window.webkitAudioContext)();
  const oscillator = ctx.createOscillator();
  oscillator.type = 'square';
  oscillator.frequency.value = 900;
  oscillator.connect(ctx.destination);
  oscillator.start();
  setTimeout(()=>{oscillator.stop();ctx.close();},2000);

  // ألغِ المنبّه بعد التشغيل
  setTimeout(clearAlarm,2000);
}

/* ========= Theme Switching ========= */
document.getElementById('themeButtons').addEventListener('click',e=>{
  if(!e.target.dataset.theme) return;
  const theme = e.target.dataset.theme;
  document.body.classList.remove('theme-default','theme-night');
  document.body.classList.add(`theme-${theme}`);
});
</script>
</body>
</html>

كود سورس لساعة مع بعض الخصائص المهمة التالبعة لها
