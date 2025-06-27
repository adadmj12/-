<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>クリックマネーゲーム</title>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-database-compat.js"></script>
  <style>
    body { font-family: sans-serif; text-align: center; margin-top: 50px; }
    button { font-size: 20px; padding: 10px 20px; }
    #rankings { margin-top: 30px; }
  </style>
</head>
<body>
  <h1>クリックマネーゲーム</h1>
  <p>お金: <span id="money">0</span>円</p>
  <p>クリック収入: <span id="income">1</span>円</p>
  <button onclick="clickMoney()">クリック！</button>
  <br><br>
  <button onclick="prestige()">プレステージ（1万で+1%）</button>
  <p>ボーナス: <span id="bonus">0%</span> / プレステージ回数: <span id="prestige">0</span></p>
  <br>
  <input type="text" id="username" placeholder="名前を入力">
  <button onclick="submitScore()">ランキングに送信</button>

  <div id="rankings">
    <h2>ランキング</h2>
    <ol id="ranking-list"></ol>
  </div>

  <script>
    // Firebase設定
    const firebaseConfig = {
      // 自分のFirebase設定で置き換えてください
      apiKey: "YOUR_API_KEY",
      authDomain: "YOUR_PROJECT.firebaseapp.com",
      databaseURL: "https://YOUR_PROJECT.firebaseio.com",
      projectId: "YOUR_PROJECT",
      storageBucket: "YOUR_PROJECT.appspot.com",
      messagingSenderId: "xxxxxxx",
      appId: "xxxxxxx"
    };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();

    // 状態
    let money = 0;
    let income = 1;
    let bonus = 0;
    let prestigeCount = 0;

    function updateDisplay() {
      document.getElementById("money").textContent = money;
      document.getElementById("income").textContent = income;
      document.getElementById("bonus").textContent = Math.floor(bonus * 100);
      document.getElementById("prestige").textContent = prestigeCount;
    }

    function clickMoney() {
      money += Math.floor(income * (1 + bonus));
      updateDisplay();
    }

    function prestige() {
      if (money >= 10000) {
        money = 0;
        bonus += 0.01;
        prestigeCount += 1;
        updateDisplay();
        alert("プレステージ成功！収入+1%");
      } else {
        alert("1万円必要です！");
      }
    }

    function submitScore() {
      const name = document.getElementById("username").value.trim();
      if (!name) {
        alert("名前を入力してください");
        return;
      }
      const scoreData = {
        name: name,
        prestige: prestigeCount,
        timestamp: Date.now()
      };
      db.ref("rankings").push(scoreData);
      alert("ランキングに送信しました！");
    }

    function loadRanking() {
      db.ref("rankings").orderByChild("prestige").limitToLast(10).on("value", (snapshot) => {
        const list = document.getElementById("ranking-list");
        list.innerHTML = "";
        let scores = [];
        snapshot.forEach(child => scores.push(child.val()));
        scores.sort((a, b) => b.prestige - a.prestige);
        for (const s of scores) {
          const li = document.createElement("li");
          li.textContent = `${s.name}：${s.prestige}プレステージ`;
          list.appendChild(li);
        }
      });
    }

    loadRanking();
  </script>
</body>
</html>
