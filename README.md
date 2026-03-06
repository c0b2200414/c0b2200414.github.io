# c0b2200414.github.io[smoking_tracker_webapp.html](https://github.com/user-attachments/files/25794707/smoking_tracker_webapp.html)
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>たばこ記録ツール</title>
  <style>
    :root {
      --bg: #f6f7fb;
      --card: #ffffff;
      --text: #1f2937;
      --muted: #6b7280;
      --accent: #2563eb;
      --danger: #dc2626;
      --border: #e5e7eb;
    }

    * { box-sizing: border-box; }

    body {
      margin: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
      background: var(--bg);
      color: var(--text);
      padding: 16px;
    }

    .container {
      max-width: 720px;
      margin: 0 auto;
      display: grid;
      gap: 16px;
    }

    .card {
      background: var(--card);
      border: 1px solid var(--border);
      border-radius: 16px;
      padding: 16px;
      box-shadow: 0 4px 16px rgba(0,0,0,0.05);
    }

    h1, h2 {
      margin: 0 0 12px;
    }

    .muted {
      color: var(--muted);
      font-size: 14px;
    }

    .main-button {
      width: 100%;
      border: none;
      border-radius: 16px;
      padding: 18px;
      font-size: 20px;
      font-weight: 700;
      background: var(--accent);
      color: white;
      cursor: pointer;
    }

    .main-button:active {
      transform: scale(0.98);
    }

    .stats {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 12px;
    }

    .stat-box {
      background: #f9fafb;
      border: 1px solid var(--border);
      border-radius: 14px;
      padding: 12px;
      text-align: center;
    }

    .stat-label {
      font-size: 13px;
      color: var(--muted);
      margin-bottom: 6px;
    }

    .stat-value {
      font-size: 26px;
      font-weight: 800;
    }

    .limit-row {
      display: flex;
      gap: 8px;
      align-items: center;
      flex-wrap: wrap;
    }

    input[type="number"] {
      width: 120px;
      padding: 10px;
      font-size: 16px;
      border: 1px solid var(--border);
      border-radius: 10px;
    }

    .sub-button {
      border: none;
      border-radius: 10px;
      padding: 10px 14px;
      background: #111827;
      color: white;
      cursor: pointer;
    }

    .warning {
      margin-top: 12px;
      padding: 12px;
      border-radius: 12px;
      background: #fef2f2;
      color: var(--danger);
      border: 1px solid #fecaca;
      font-weight: 700;
      display: none;
    }

    .bar-wrap {
      margin-top: 12px;
    }

    .bar {
      height: 18px;
      width: 100%;
      background: #e5e7eb;
      border-radius: 999px;
      overflow: hidden;
    }

    .bar-fill {
      height: 100%;
      width: 0%;
      background: var(--accent);
      transition: width 0.25s ease;
    }

    ul {
      list-style: none;
      padding: 0;
      margin: 12px 0 0;
      max-height: 280px;
      overflow: auto;
    }

    li {
      border-bottom: 1px solid var(--border);
      padding: 10px 0;
      display: flex;
      justify-content: space-between;
      gap: 10px;
      font-size: 14px;
    }

    .danger-text {
      color: var(--danger);
      font-weight: 700;
    }

    .small-button {
      margin-top: 12px;
      border: none;
      border-radius: 10px;
      padding: 10px 14px;
      background: var(--danger);
      color: white;
      cursor: pointer;
    }

    @media (max-width: 600px) {
      .stats {
        grid-template-columns: 1fr;
      }

      li {
        flex-direction: column;
      }
    }
  </style>
</head>
<body>
  <div class="container">
    <div class="card">
      <h1>たばこ記録ツール</h1>
      <p class="muted">ボタンを押すたびに、吸った日時を記録します。</p>
      <button class="main-button" id="recordBtn">1本吸った</button>
    </div>

    <div class="card">
      <h2>今日の状況</h2>
      <div class="stats">
        <div class="stat-box">
          <div class="stat-label">今日の本数</div>
          <div class="stat-value" id="todayCount">0</div>
        </div>
        <div class="stat-box">
          <div class="stat-label">設定した上限</div>
          <div class="stat-value" id="limitValue">10</div>
        </div>
        <div class="stat-box">
          <div class="stat-label">残り本数</div>
          <div class="stat-value" id="remainingCount">10</div>
        </div>
      </div>

      <div class="bar-wrap">
        <div class="muted">上限に対する使用率</div>
        <div class="bar"><div class="bar-fill" id="barFill"></div></div>
      </div>

      <div class="limit-row" style="margin-top: 14px;">
        <label for="dailyLimit">1日の上限</label>
        <input type="number" id="dailyLimit" min="1" value="10" />
        <button class="sub-button" id="saveLimitBtn">保存</button>
      </div>

      <div class="warning" id="warningBox">今日の上限を超えています。</div>
    </div>

    <div class="card">
      <h2>今日の記録一覧</h2>
      <p class="muted">最新20件まで表示</p>
      <ul id="logList"></ul>
      <button class="small-button" id="clearTodayBtn">今日の記録を削除</button>
    </div>
  </div>

  <script>
    const STORAGE_KEY = "smoking_logs_v1";
    const LIMIT_KEY = "smoking_daily_limit_v1";

    const recordBtn = document.getElementById("recordBtn");
    const todayCountEl = document.getElementById("todayCount");
    const limitValueEl = document.getElementById("limitValue");
    const remainingCountEl = document.getElementById("remainingCount");
    const dailyLimitInput = document.getElementById("dailyLimit");
    const saveLimitBtn = document.getElementById("saveLimitBtn");
    const logList = document.getElementById("logList");
    const clearTodayBtn = document.getElementById("clearTodayBtn");
    const barFill = document.getElementById("barFill");
    const warningBox = document.getElementById("warningBox");

    function getLogs() {
      return JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]");
    }

    function saveLogs(logs) {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(logs));
    }

    function getLimit() {
      return Number(localStorage.getItem(LIMIT_KEY) || 10);
    }

    function saveLimit(limit) {
      localStorage.setItem(LIMIT_KEY, String(limit));
    }

    function isToday(isoString) {
      const d = new Date(isoString);
      const now = new Date();
      return (
        d.getFullYear() === now.getFullYear() &&
        d.getMonth() === now.getMonth() &&
        d.getDate() === now.getDate()
      );
    }

    function formatDateTime(isoString) {
      return new Date(isoString).toLocaleString("ja-JP", {
        year: "numeric",
        month: "2-digit",
        day: "2-digit",
        hour: "2-digit",
        minute: "2-digit",
        second: "2-digit"
      });
    }

    function render() {
      const logs = getLogs();
      const todayLogs = logs.filter(log => isToday(log));
      const limit = getLimit();
      const todayCount = todayLogs.length;
      const remaining = Math.max(limit - todayCount, 0);
      const ratio = Math.min((todayCount / limit) * 100, 100);

      todayCountEl.textContent = todayCount;
      limitValueEl.textContent = limit;
      remainingCountEl.textContent = remaining;
      dailyLimitInput.value = limit;
      barFill.style.width = ratio + "%";

      warningBox.style.display = todayCount > limit ? "block" : "none";

      logList.innerHTML = "";
      const recentLogs = todayLogs.slice().reverse().slice(0, 20);

      if (recentLogs.length === 0) {
        const li = document.createElement("li");
        li.textContent = "まだ記録はありません。";
        logList.appendChild(li);
      } else {
        recentLogs.forEach((log, index) => {
          const li = document.createElement("li");
          li.innerHTML = `<span>${recentLogs.length - index}本目</span><span>${formatDateTime(log)}</span>`;
          logList.appendChild(li);
        });
      }
    }

    recordBtn.addEventListener("click", () => {
      const logs = getLogs();
      logs.push(new Date().toISOString());
      saveLogs(logs);
      render();
    });

    saveLimitBtn.addEventListener("click", () => {
      const value = Number(dailyLimitInput.value);
      if (!value || value < 1) {
        alert("1以上の数字を入れてください。");
        return;
      }
      saveLimit(value);
      render();
    });

    clearTodayBtn.addEventListener("click", () => {
      const ok = confirm("今日の記録だけ削除しますか？");
      if (!ok) return;
      const logs = getLogs().filter(log => !isToday(log));
      saveLogs(logs);
      render();
    });

    render();
  </script>
</body>
</html>
