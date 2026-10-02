<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Kuis Online TJKT</title>
  <style>
    * { box-sizing: border-box; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
    body { background-color: #f4f6f9; display: flex; justify-content: center; align-items: center; min-height: 100vh; margin: 0; padding: 20px; }
    .card { background: white; padding: 30px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); width: 100%; max-width: 600px; }
    h2, h3 { text-align: center; color: #333; margin-top: 0; }
    .hidden { display: none; }
    
    /* Form Login */
    .form-group { margin-bottom: 15px; }
    label { display: block; margin-bottom: 5px; font-weight: bold; color: #555; }
    input[type="text"], input[type="email"] { width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 6px; }
    
    /* General Button */
    button { width: 100%; padding: 12px; background: #007bff; color: white; border: none; border-radius: 6px; font-size: 16px; font-weight: bold; cursor: pointer; transition: 0.2s; }
    button:hover { background: #0056b3; }
    
    /* Header Kuis */
    .quiz-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; border-bottom: 2px solid #eee; padding-bottom: 10px; }
    .timer { font-size: 18px; font-weight: bold; color: #d9534f; background: #fdf7f7; padding: 5px 12px; border-radius: 20px; border: 1px solid #f5c6cb; }
    
    /* Opsi Jawaban */
    .option-btn { display: block; width: 100%; text-align: left; background: #f8f9fa; color: #333; border: 1px solid #ddd; margin-bottom: 10px; padding: 12px; border-radius: 6px; }
    .option-btn:hover { background: #e2e6ea; }
    
    /* Leaderboard */
    table { width: 100%; border-collapse: collapse; margin-top: 15px; }
    th, td { border: 1px solid #ddd; padding: 10px; text-align: center; }
    th { background-color: #007bff; color: white; }
    tr:nth-child(even) { background-color: #f9f9f9; }
    
    .alert { background: #f8d7da; color: #721c24; padding: 10px; border-radius: 6px; margin-bottom: 15px; display: none; text-align: center; }
  </style>
</head>
<body>

  <div class="card">
    <!-- Halaman Register/Login Sederhana -->
    <div id="login-screen">
      <h2>Kuis Mata Pelajaran TJKT</h2>
      <p style="text-align: center; color: #666; font-size: 14px;">Waktu: 1 Menit (60 Detik) | Kesempatan: 1x Per Akun</p>
      <div id="login-error" class="alert"></div>
      <form id="login-form">
        <div class="form-group">
          <label for="username">Nama Lengkap</label>
          <input type="text" id="username" required placeholder="Masukkan Nama Anda">
        </div>
        <div class="form-group">
          <label for="email">Email / ID Akun</label>
          <input type="email" id="email" required placeholder="nama@sekolah.sch.id">
        </div>
        <button type="submit">Mulai Mengerjakan</button>
      </form>
    </div>

    <!-- Halaman Soal Kuis -->
    <div id="quiz-screen" class="hidden">
      <div class="quiz-header">
        <span>Soal <span id="current-question">1</span> / <span id="total-questions">0</span></span>
        <div class="timer">Sisa Waktu: <span id="time-left">60</span>s</div>
      </div>
      <h3 id="question-text">Pertanyaan akan muncul di sini...</h3>
      <div id="options-container"></div>
    </div>

    <!-- Halaman Hasil & Peringkat -->
    <div id="result-screen" class="hidden">
      <h2>Hasil Kuis Anda</h2>
      <p style="text-align: center; font-size: 18px;">Skor Akhir: <b id="final-score">0</b></p>
      
      <h3>Papan Peringkat (Leaderboard)</h3>
      <table>
        <thead>
          <tr>
            <th>Posisi</th>
            <th>Nama</th>
            <th>Skor</th>
          </tr>
        </thead>
        <tbody id="leaderboard-body"></tbody>
      </table>
    </div>
  </div>

  <script>
    // Database Soal TJKT
    const questions = [
      {
        question: "Perangkat jaringan yang berfungsi menghubungkan dua jaringan dengan protokol berbeda adalah...",
        options: ["Switch", "Router", "Hub", "Access Point"],
        answer: 1
      },
      {
        question: "Urutan standar warna kabel UTP untuk kategori T568B pin ke-1 adalah...",
        options: ["Putih-Hijau", "Putih-Cokelat", "Putih-Oranye", "Oranye"],
        answer: 2
      },
      {
        question: "Layer OSI yang bertanggung jawab untuk routing paket data adalah...",
        options: ["Data Link Layer", "Network Layer", "Transport Layer", "Physical Layer"],
        answer: 1
      },
      {
        question: "Berapa panjang bit dari IP address versi 4 (IPv4)?",
        options: ["32 bit", "64 bit", "128 bit", "16 bit"],
        answer: 0
      },
      {
        question: "Perintah CLI pada Windows yang digunakan untuk mengecek konektivitas jaringan adalah...",
        options: ["ipconfig", "tracert", "ping", "netstat"],
        answer: 2
      }
    ];

    // Elemen DOM
    const loginScreen = document.getElementById('login-screen');
    const quizScreen = document.getElementById('quiz-screen');
    const resultScreen = document.getElementById('result-screen');
    const loginForm = document.getElementById('login-form');
    const loginError = document.getElementById('login-error');
    
    const questionText = document.getElementById('question-text');
    const optionsContainer = document.getElementById('options-container');
    const currentQuestionEl = document.getElementById('current-question');
    const totalQuestionsEl = document.getElementById('total-questions');
    const timeLeftEl = document.getElementById('time-left');
    const finalScoreEl = document.getElementById('final-score');
    const leaderboardBody = document.getElementById('leaderboard-body');

    // Variabel Status Kuis
    let currentQuestionIndex = 0;
    let score = 0;
    let timer;
    let timeLeft = 60; // 1 Menit
    let currentUser = null;

    totalQuestionsEl.textContent = questions.length;

    // Login & Validasi 1x Pengerjaan
    loginForm.addEventListener('submit', function(e) {
      e.preventDefault();
      const username = document.getElementById('username').value.trim();
      const email = document.getElementById('email').value.trim().toLowerCase();

      // Cek apakah akun email ini sudah pernah mengerjakan
      const completedUsers = JSON.parse(localStorage.getItem('completed_quiz_users')) || [];
      if (completedUsers.includes(email)) {
        loginError.textContent = "Akun ini sudah pernah mengerjakan kuis dan tidak bisa lagi!";
        loginError.style.display = 'block';
        return;
      }

      currentUser = { username, email };
      loginScreen.classList.add('hidden');
      quizScreen.classList.remove('hidden');

      startQuiz();
    });

    function startQuiz() {
      showQuestion();
      timer = setInterval(updateTimer, 1000);
    }

    function updateTimer() {
      timeLeft--;
      timeLeftEl.textContent = timeLeft;

      if (timeLeft <= 0) {
        clearInterval(timer);
        finishQuiz();
      }
    }

    function showQuestion() {
      const q = questions[currentQuestionIndex];
      questionText.textContent = q.question;
      currentQuestionEl.textContent = currentQuestionIndex + 1;
      optionsContainer.innerHTML = '';

      q.options.forEach((opt, index) => {
        const btn = document.createElement('button');
        btn.className = 'option-btn';
        btn.textContent = opt;
        btn.onclick = () => selectAnswer(index);
        optionsContainer.appendChild(btn);
      });
    }

    function selectAnswer(selectedIndex) {
      if (selectedIndex === questions[currentQuestionIndex].answer) {
        score += Math.round(100 / questions.length); // Menghitung proporsi nilai total 100
      }

      currentQuestionIndex++;
      if (currentQuestionIndex < questions.length) {
        showQuestion();
      } else {
        clearInterval(timer);
        finishQuiz();
      }
    }

    function finishQuiz() {
      quizScreen.classList.add('hidden');
      resultScreen.classList.remove('hidden');

      // 1. Simpan email bahwa akun sudah pernah mengerjakan
      const completedUsers = JSON.parse(localStorage.getItem('completed_quiz_users')) || [];
      completedUsers.push(currentUser.email);
      localStorage.setItem('completed_quiz_users', JSON.stringify(completedUsers));

      // 2. Simpan Skor ke Leaderboard
      const leaderboard = JSON.parse(localStorage.getItem('tjkt_leaderboard')) || [];
      leaderboard.push({ username: currentUser.username, score: score });
      
      // Urutkan Peringkat (Dari Skor Terbesar ke Terkecil)
      leaderboard.sort((a, b) => b.score - a.score);
      localStorage.setItem('tjkt_leaderboard', JSON.stringify(leaderboard));

      // 3. Tampilkan Skor dan Leaderboard
      finalScoreEl.textContent = score;
      renderLeaderboard(leaderboard);
    }

    function renderLeaderboard(leaderboard) {
      leaderboardBody.innerHTML = '';
      leaderboard.forEach((data, index) => {
        const row = document.createElement('tr');
        row.innerHTML = `
          <td><b>#${index + 1}</b></td>
          <td>${data.username}</td>
          <td>${data.score}</td>
        `;
        leaderboardBody.appendChild(row);
      });
    }
  </script>
</body>
</html>
