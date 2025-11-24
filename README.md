<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Voucher System</title>
<link href="https://fonts.googleapis.com/css2?family=League+Spartan:wght@300;400;600&display=swap" rel="stylesheet">
<style>
body {
  font-family: 'League Spartan', sans-serif;
  background: #ffffff;
  color: #000000;
  text-align: center;
  padding: 20px;
  position: relative;
}

.inputBox {
  margin: 12px 0;
  width: 100%;
  max-width: 280px;
  padding: 12px 14px;
  font-size: 18px;
  border: 2px solid #ddd;
  border-radius: 10px;
  outline: none;
  transition: 0.2s;
}

.inputBox:focus {
  border-color: #ff1e1e;
}

.redBtn {
  background: #ff1e1e;
  color: #ffffff;
  font-size: 20px;
  padding: 14px 20px;
  margin-top: 10px;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  font-weight: 600;
  transition: 0.2s;
}

.redBtn:hover {
  background: #dd0000;
}

#voucher {
  margin-top: 25px;
  font-size: 24px;
  font-weight: 600;
}

#timer {
  position: fixed;
  left: 10px;
  bottom: 10px;
  font-size: 16px;
  color: #000000;
  font-weight: 400;
}
</style>

<!-- Firebase SDK -->
<script src="https://www.gstatic.com/firebasejs/9.17.1/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.17.1/firebase-auth.js"></script>

<script>
// === Глобальні змінні ===
var recaptchaInitialized = false;
var confirmationResult;

// === Firebase ===
var firebaseConfig = {
  apiKey: "AIzaSyDVoV3jg4MZXVxY4bemiBrrrFF_xcpHc3M",
  authDomain: "phoneid-6e2af.firebaseapp.com",
  projectId: "phoneid-6e2af",
  storageBucket: "phoneid-6e2af.firebasestorage.app",
  messagingSenderId: "242388555474",
  appId: "1:242388555474:web:159ac4422d985fd84dff0b",
  measurementId: "G-D5YL1E4BVL"
};
firebase.initializeApp(firebaseConfig);

// === Відправка SMS ===
function sendCode() {
  var number = document.getElementById("phone").value.trim();
  
  if(!/^\d{9}$/.test(number)){
    alert("VveditÍ 9 cyfr pislä kodu +380");
    return;
  }

  var phone = "+380" + number;

  if(!recaptchaInitialized){
    window.recaptchaVerifier = new firebase.auth.RecaptchaVerifier('recaptcha', { 'size': 'normal' });
    window.recaptchaVerifier.render();
    recaptchaInitialized = true;
  }

  firebase.auth().signInWithPhoneNumber(phone, window.recaptchaVerifier)
    .then(function(result){
      confirmationResult = result;
      alert("Kod nadislano na vaš nomer");
    })
    .catch(function(err){
      alert(err.message);
    });
}

// === Підтвердження SMS ===
function verifyCode() {
  var sms = document.getElementById("smsCode").value.trim();
  if(!sms){ alert("VveditÍ kod iz SMS"); return; }

  confirmationResult.confirm(sms)
    .then(async function(result){
      var phone = result.user.phoneNumber;
      var res = await fetch("https://script.google.com/macros/s/YOUR_WEBAPP_ID/exec?phone=" + encodeURIComponent(phone));
      var voucher = await res.text();

      if(voucher === "ERR_ALREADY"){
        document.getElementById("voucher").innerHTML = "Ýoý! Ne mahlüý tam!";
        return;
      }
      if(voucher === "ERR_EMPTY"){
        document.getElementById("voucher").innerHTML = "Vaučery zakinčylysÍ!";
        return;
      }

      showVoucherForFiveMinutes(voucher);
    })
    .catch(function(err){
      alert(err.message);
    });
}

// === Показ ваучера з таймером ===
function showVoucherForFiveMinutes(code){
  var timeLeft = 300;
  var voucherEl = document.getElementById("voucher");
  var timerEl = document.getElementById("timer");

  voucherEl.style.opacity = 1;
  voucherEl.innerHTML = "Vaš vaučer: <b>" + code + "</b>";

  var timer = setInterval(function(){
    timeLeft--;
    timerEl.innerText = "ZalyšylosÍ: " + timeLeft + " сек";

    if(timeLeft <= 0){
      clearInterval(timer);
      voucherEl.style.transition = "opacity 2s";
      voucherEl.style.opacity = 0;

      setTimeout(function(){
        voucherEl.innerHTML = "<span style='color:red'>Čas vyčerpano.</span>";
        voucherEl.style.opacity = 1;
        timerEl.innerText = "";
      }, 2000);
    }
  }, 1000);
}
</script>
</head>
<body>
<div>
  <h2>Otrymaty vaučer</h2>
  <input id="phone" class="inputBox" placeholder="VveditÍ 9 cyfr pislä +380" />
  <br>
  <button class="redBtn" onclick="sendCode()">Vidpravyty SMS-kod</button>
  <div id="recaptcha" style="margin-top: 15px;"></div>
  <br><br>
  <input id="smsCode" class="inputBox" placeholder="Kod iz SMS" />
  <br>
  <button class="redBtn" onclick="verifyCode()">Pidtverdyty</button>
  <div id="voucher"></div>
  <div id="timer"></div>
</div>
</body>
</html>
