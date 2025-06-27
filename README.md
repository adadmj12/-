<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>完全版クリックゲーム</title>
  <style>
    body { font-family: sans-serif; text-align: center; padding: 20px; }
    button { font-size: 16px; margin: 5px; padding: 10px 20px; }
  </style>
</head>
<body>
  <h1>クリックゲーム：アセンション壊し搭載</h1>

  <p>お金: <span id="money">0</span>円</p>
  <p>クリック収入: <span id="income">1</span>円</p>
  <button onclick="clickMoney()">クリック！</button>

  <hr />
  <h2>アップグレード</h2>
  <p>レベル: <span id="upgrade-level">0</span> / 次のコスト: <span id="upgrade-cost">10</span>円</p>
  <button onclick="buyUpgrade(1)">+1</button>
  <button onclick="buyUpgrade(10)">+10</button>
  <button onclick="buyUpgrade('max')">最大</button>

  <hr />
  <h2>プレステージ</h2>
  <p>ボーナス: <span id="prestige-bonus">0</span>% / 回数: <span id="prestige-count">0</span></p>
  <button onclick="tryPrestige()">プレステージ（1万必要）</button>

  <hr />
  <h2>アセンション</h2>
  <p>回数: <span id="ascension-count">0</span> <span id="ascension-mode"></span></p>
  <p>アセンションボーナス: <span id="ascension-bonus">0</span>%</p>
  <button onclick="tryAscension()">アセンション</button>

  <hr />
  <h2>セーブ / ロード</h2>
  <button onclick="saveGame()">保存</button>
  <button onclick="loadGame()">読み込み</button>
  <button onclick="resetGame()">リセット</button>

  <script>
    // 基本変数
    let money = 0;
    let income = 1;
    let upgradeLevel = 0;

    let prestigeBonus = 0; // 1%ごと
    let prestigeCount = 0;

    let ascensionCount = 0;
    let ascensionBonus = 0;
    let ascensionBreakMode = false;

    const baseUpgradeCost = 10;
    const upgradeCostMultiplier = 1.5; // コスト倍率を1.5倍に変更

    const prestigeCost = 10_000;
    const baseAscensionCost = 1_000_000_000_000;

    function getUpgradeCost(level) {
      return Math.floor(baseUpgradeCost * Math.pow(upgradeCostMultiplier, level));
    }

    function getTotalCostToBuy(n) {
      let total = 0;
      for (let i = 0; i < n; i++) {
        total += getUpgradeCost(upgradeLevel + i);
      }
      return total;
    }

    function getMaxAffordableUpgrade() {
      let count = 0;
      while (money >= getTotalCostToBuy(count + 1)) count++;
      return count;
    }

    function clickMoney() {
      money += Math.floor(income * (1 + prestigeBonus / 100 + ascensionBonus));
      updateDisplay();
    }

    function buyUpgrade(amount) {
      if (amount === "max") amount = getMaxAffordableUpgrade();
      const cost = getTotalCostToBuy(amount);
      if (money >= cost && amount > 0) {
        money -= cost;
        income *= Math.pow(1.5, amount); // 収入を1.5倍ずつ掛ける
        upgradeLevel += amount;
        updateDisplay();
      } else {
        alert("お金が足りません");
      }
    }

    function tryPrestige() {
      if (money >= prestigeCost) {
        money = 0;
        prestigeBonus += 1;
        prestigeCount += 1;
        if (prestigeCount >= 100) {
          prestigeBonus = 0;
          prestigeCount = 0;
        }
        updateDisplay();
        alert("プレステージ成功！");
      } else {
        alert("1万必要です");
      }
    }

    function tryAscension() {
      if (!ascensionBreakMode && money < baseAscensionCost) {
        alert("アセンションには1兆円必要です");
        return;
      }

      let bonusGain = 0;
      if (ascensionBreakMode) {
        const multiplier = money / baseAscensionCost;
        bonusGain = multiplier * 0.05;
        ascensionBonus += bonusGain;
        alert(`突破アセンション成功！+${(bonusGain * 100).toFixed(2)}%`);
      } else {
        ascensionBonus += 0.05;
        alert("通常アセンション成功！+5%");
      }

      ascensionCount++;
      if (ascensionCount >= 100 && !ascensionBreakMode) {
        ascensionBreakMode = true;
        alert("🔥アセンション壊しモード突入！好きなときにアセンション可能！");
      }

      // リセット処理
      money = 0;
      upgradeLevel = 0;
      income = 1;
      prestigeBonus = 0;
      prestigeCount = 0;

      updateDisplay();
    }

    function updateDisplay() {
      document.getElementById("money").textContent = Math.floor(money);
      document.getElementById("income").textContent = Math.floor(income);
      document.getElementById("upgrade-level").textContent = upgradeLevel;
      document.getElementById("upgrade-cost").textContent = getUpgradeCost(upgradeLevel);
      document.getElementById("prestige-bonus").textContent = prestigeBonus;
      document.getElementById("prestige-count").textContent = prestigeCount;
      document.getElementById("ascension-count").textContent = ascensionCount;
      document.getElementById("ascension-bonus").textContent = (ascensionBonus * 100).toFixed(2);
      document.getElementById("ascension-mode").textContent = ascensionBreakMode ? "（突破モード）" : "";
    }

    function saveGame() {
      const data = {
        money,
        income,
        upgradeLevel,
        prestigeBonus,
        prestigeCount,
        ascensionCount,
        ascensionBonus,
        ascensionBreakMode,
      };
      localStorage.setItem("clickGameSave", JSON.stringify(data));
      alert("保存完了");
    }

    function loadGame() {
      const data = JSON.parse(localStorage.getItem("clickGameSave"));
      if (data) {
        money = data.money;
        income = data.income;
        upgradeLevel = data.upgradeLevel;
        prestigeBonus = data.prestigeBonus;
        prestigeCount = data.prestigeCount;
        ascensionCount = data.ascensionCount;
        ascensionBonus = data.ascensionBonus;
        ascensionBreakMode = data.ascensionBreakMode;
        updateDisplay();
        alert("読み込み完了");
      } else {
        alert("保存データがありません");
      }
    }

    function resetGame() {
      if (confirm("すべてのデータを初期化しますか？")) {
        money = 0;
        income = 1;
        upgradeLevel = 0;
        prestigeBonus = 0;
        prestigeCount = 0;
        ascensionCount = 0;
        ascensionBonus = 0;
        ascensionBreakMode = false;
        updateDisplay();
        alert("リセットしました");
      }
    }

    updateDisplay(); // 初期表示
  </script>
</body>
</html>
