<!DOCTYPE html>
<html>
<head>
<title>Tebak Angka</title>
<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap" rel="stylesheet">
<style>
body {
  font-family: 'Poppins', sans-serif;
  background-color: #f8f9fa;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  margin: 0;
  overflow: hidden;
}

.container {
  background-color: #ffffff;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  padding: 40px;
  width: 350px;
  max-width: 90%;
  text-align: center;
}

h1 {
  font-size: 2.5em;
  margin-bottom: 20px;
  color: #343a40;
  font-weight: 600;
}

label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #343a40;
}

select,
input[type="number"] {
  width: 100%;
  padding: 12px;
  margin-bottom: 15px;
  border: 1px solid #ced4da;
  border-radius: 5px;
  box-sizing: border-box;
  font-size: 1em;
  font-weight: 400;
}

button {
  background-image: linear-gradient(to right, #007bff, #0062cc);
  color: white;
  padding: 15px 25px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  margin: 10px 0;
  font-size: 1.1em;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s ease;
}

button:hover {
  transform: translateY(-2px);
}

#hasil, #timer, #highScore {
  font-weight: 600;
  margin-top: 15px;
  color: #343a40;
}

.winner {
  background-color: #d4edda;
  transition: background-color 1s ease;
}

/* Media Queries untuk Responsiveness */
@media (min-width: 768px) {
  .container {
    width: 450px;
  }
}
</style>
</head>
<body>

<div class="container">
  <h1>Tebak Angka</h1>

  <label for="mode">Pilih Mode Permainan:</label>
  <select id="mode">
    <option value="kesempatan">Mode Kesempatan</option>
    <option value="waktu">Mode Waktu</option>
  </select>

  <label for="level">Pilih Tingkat Kesulitan:</label>
  <select id="level">
    <option value="100">Mudah (1-100)</option>
    <option value="500">Sedang (1-500)</option>
    <option value="1000">Sulit (1-1000)</option>
  </select>

  <p id="timer"></p>
  <p id="highScore"></p>

  <p>Saya telah memilih angka antara 1 dan 100. Coba tebak!</p>
  <input type="number" id="tebakan">
  <button onclick="cekTebakan()">Tebak</button>
  <button onclick="resetGame()">Mulai Ulang</button>
  <p id="hasil"></p>
</div>

<script>
let angkaRahasia;
let kesempatan;
let kesempatanAwal;
let tebakanSebelumnya = null;
let waktuSisa;
let timerInterval;
let highScore = localStorage.getItem('highScore') ? parseInt(localStorage.getItem('highScore')) : 0;
let playerName = localStorage.getItem('playerName') || '';
let score;


function updateHighScore() {
    if (score > highScore && score > 0) {
        highScore = score;
        playerName = prompt('Selamat! Anda mendapatkan skor tertinggi baru! Masukkan nama Anda:');
        localStorage.setItem('highScore', highScore);
        localStorage.setItem('playerName', playerName);
    }
}

function displayHighScore() {
    let highScoreDisplay = document.getElementById('highScore');
    if(highScoreDisplay){
        highScoreDisplay.textContent = `Skor Tertinggi: ${highScore > 0 ? highScore : 'Belum ada skor'} oleh ${playerName}`;
    }
}

function resetGame() {
  let level = parseInt(document.getElementById("level").value);
  let mode = document.getElementById("mode").value;
  angkaRahasia = Math.floor(Math.random() * level) + 1;
  kesempatan = mode === 'kesempatan' ? Math.ceil(Math.log2(level)) + 2 : 7;
  kesempatanAwal = kesempatan;
  waktuSisa = mode === 'waktu' ? 30 : null;
  tebakanSebelumnya = null;
  document.getElementById("tebakan").value = "";
  document.getElementById("hasil").textContent = "";
  document.getElementById("tebakan").disabled = false;
  document.getElementById("timer").textContent = mode === 'waktu' ? "Waktu Tersisa: " + waktuSisa + " detik" : "";
  clearInterval(timerInterval);
  document.body.classList.remove("winner");
  score = 0;
  displayHighScore();

  if (mode === 'waktu') {
    timerInterval = setInterval(updateTimer, 1000);
  }
}

function updateTimer() {
  if (waktuSisa === null) return;

  waktuSisa--;
  document.getElementById("timer").textContent = "Waktu Tersisa: " + waktuSisa + " detik";
  if (waktuSisa <= 0) {
    clearInterval(timerInterval);
    document.getElementById("hasil").textContent = "Waktu habis! Angka rahasianya adalah " + angkaRahasia;
    document.getElementById("tebakan").disabled = true;
  }
}

function cekTebakan() {
  let mode = document.getElementById("mode").value;
  if (mode === 'waktu' && waktuSisa <= 0) return;

  let tebakan = parseInt(document.getElementById("tebakan").value);
  let hasil = document.getElementById("hasil");

  if (isNaN(tebakan) || tebakan < 1 || tebakan > parseInt(document.getElementById("level").value)) {
    hasil.textContent = "Masukkan angka yang valid sesuai level!";
    return;
  }

  kesempatan--;

  if (tebakan === angkaRahasia) {
    clearInterval(timerInterval);
    hasil.textContent = "Selamat! Anda menebak dengan benar";
    if (mode === 'waktu') {
        score = waktuSisa;
        if (score < 0) score = 0;
        hasil.textContent += " dalam " + score + " detik!";
    } else {
        score = kesempatanAwal - kesempatan;
        hasil.textContent += " dalam " + score + " kali percobaan!";
    }
    document.getElementById("tebakan").disabled = true;
    document.body.classList.add("winner");
    setTimeout(() => {
      document.body.classList.remove("winner");
    }, 1000);
    updateHighScore();
    displayHighScore();
  } else if (kesempatan === 0 && mode === 'kesempatan') {
    hasil.textContent = "Maaf, kesempatan Anda telah habis. Angka rahasianya adalah " + angkaRahasia;
    document.getElementById("tebakan").disabled = true;
  } else {
    let jarak = Math.abs(tebakan - angkaRahasia);
    let petunjuk = "";
    if (tebakanSebelumnya !== null) {
      let jarakSebelumnya = Math.abs(tebakanSebelumnya - angkaRahasia);
      if (jarak < jarakSebelumnya) {
        petunjuk = "Angka rahasia lebih dekat!";
      } else {
        petunjuk = "Angka rahasia lebih jauh!";
      }
    }

    if (tebakan < angkaRahasia) {
      hasil.textContent = "Terlalu rendah! " + petunjuk + (mode === 'kesempatan' ? " Anda memiliki " + kesempatan + " kesempatan tersisa." : "");
    } else {
      hasil.textContent = "Terlalu tinggi! " + petunjuk + (mode === 'kesempatan' ? " Anda memiliki " + kesempatan + " kesempatan tersisa." : "");
    }
    tebakanSebelumnya = tebakan;
  }
}

resetGame();
</script>

</body>
</html>
