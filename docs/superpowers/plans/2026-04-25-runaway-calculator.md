# 跑路计算器 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 构建单页面"跑路计算器"应用，根据存款目标和时间参数计算跑路日期并展示倒计时圆环

**Architecture:** 单页应用，通过 JS 控制两个视图（表单页/结果页）的显示切换。所有 HTML/CSS/JS 写在一个文件中，无外部依赖。使用 `setInterval` 驱动倒计时实时更新，`localStorage` 持久化数据。

**Tech Stack:** 原生 HTML5 + CSS3 + JavaScript (ES6)，SVG 绘制圆环进度条

---

### Task 1: 创建 HTML 骨架和 CSS 样式

**Files:**
- Create: `index.html`

- [ ] **Step 1: 编写完整的 HTML 文件**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>跑路计算器</title>
<link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>🏃</text></svg>">
<style>
/* ===== Reset & Base ===== */
* { margin: 0; padding: 0; box-sizing: border-box; }
body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', sans-serif;
  background: linear-gradient(135deg, #fce4ec 0%, #f8bbd0 50%, #f48fb1 100%);
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  color: #333;
}

/* ===== Card Container ===== */
.card {
  background: #fff;
  border-radius: 24px;
  box-shadow: 0 8px 32px rgba(244, 143, 177, 0.3);
  width: 100%;
  max-width: 480px;
  padding: 40px 32px;
  position: relative;
}

/* ===== Page Toggles ===== */
.form-page, .result-page { display: none; }
.form-page.active, .result-page.active { display: block; }

/* ===== Typography ===== */
.page-title {
  font-size: 28px;
  font-weight: 700;
  text-align: center;
  margin-bottom: 8px;
  background: linear-gradient(135deg, #f093fb, #f5576c);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}
.page-subtitle {
  text-align: center;
  color: #999;
  font-size: 14px;
  margin-bottom: 32px;
}

/* ===== Form Styles ===== */
.form-group { margin-bottom: 20px; }
.form-group label {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 14px;
  color: #666;
  margin-bottom: 8px;
  font-weight: 500;
}
.form-group label .emoji { font-size: 18px; }
.form-group input {
  width: 100%;
  padding: 14px 16px;
  border: 2px solid #f0f0f0;
  border-radius: 14px;
  font-size: 16px;
  transition: border-color 0.2s, box-shadow 0.2s;
  background: #fafafa;
}
.form-group input:focus {
  outline: none;
  border-color: #f5576c;
  box-shadow: 0 0 0 3px rgba(245, 87, 108, 0.15);
  background: #fff;
}

/* ===== Button ===== */
.btn-primary {
  width: 100%;
  padding: 16px;
  border: none;
  border-radius: 16px;
  font-size: 18px;
  font-weight: 700;
  color: #fff;
  background: linear-gradient(135deg, #f093fb, #f5576c);
  cursor: pointer;
  transition: transform 0.15s, box-shadow 0.15s;
  margin-top: 10px;
}
.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(245, 87, 108, 0.4);
}
.btn-primary:active { transform: translateY(0); }

/* ===== Error Message ===== */
.error-msg {
  color: #f5576c;
  font-size: 13px;
  margin-top: 4px;
  display: none;
}
.error-msg.show { display: block; }

/* ===== Result Page Styles ===== */
.settings-btn {
  position: absolute;
  top: 20px;
  right: 20px;
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  padding: 8px;
  border-radius: 50%;
  transition: background 0.2s;
}
.settings-btn:hover { background: #f5f5f5; }

.motto-area {
  text-align: center;
  margin-bottom: 30px;
  padding: 16px;
  background: linear-gradient(135deg, #fff0f3, #ffe4ec);
  border-radius: 16px;
  font-size: 16px;
  font-weight: 600;
  color: #f5576c;
}

.result-date-area {
  text-align: center;
  margin-bottom: 36px;
}
.result-label {
  font-size: 14px;
  color: #999;
  margin-bottom: 8px;
}
.result-date {
  font-size: 36px;
  font-weight: 800;
  color: #1a1a2e;
  margin-bottom: 8px;
}
.result-days {
  font-size: 18px;
  color: #666;
}
.result-days span {
  font-weight: 700;
  color: #f5576c;
}

/* ===== Circle Progress ===== */
.circle-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 10px;
}
.circle-container {
  position: relative;
  width: 220px;
  height: 220px;
}
.circle-container svg {
  width: 100%;
  height: 100%;
  transform: rotate(-90deg);
}
.circle-bg {
  fill: none;
  stroke: #f0f0f0;
  stroke-width: 14;
}
.circle-progress {
  fill: none;
  stroke: url(#pinkGrad);
  stroke-width: 14;
  stroke-linecap: round;
  transition: stroke-dashoffset 0.3s ease;
}
.circle-text {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
}
.circle-time {
  font-size: 32px;
  font-weight: 800;
  color: #1a1a2e;
  font-variant-numeric: tabular-nums;
}
.circle-label {
  font-size: 13px;
  color: #999;
  margin-top: 6px;
}

/* ===== Responsive ===== */
@media (max-width: 520px) {
  body { padding: 12px; align-items: flex-start; padding-top: 24px; }
  .card { padding: 28px 20px; border-radius: 20px; }
  .page-title { font-size: 24px; }
  .form-group input { padding: 12px 14px; font-size: 15px; }
  .btn-primary { padding: 14px; font-size: 16px; }
  .result-date { font-size: 28px; }
  .result-days { font-size: 16px; }
  .circle-container { width: 180px; height: 180px; }
  .circle-time { font-size: 26px; }
  .motto-area { font-size: 14px; padding: 12px; }
}
</style>
</head>
<body>

<div class="card">
  <!-- ===== Form Page ===== -->
  <div class="form-page active" id="formPage">
    <h1 class="page-title">跑路计算器</h1>
    <p class="page-subtitle">计算你的财务自由倒计时 🚀</p>

    <form id="runawayForm">
      <div class="form-group">
        <label><span class="emoji">🎯</span> 目标存款</label>
        <input type="number" id="targetAmount" placeholder="例如：500000" min="0" step="1" required>
        <div class="error-msg" id="err-target">请输入有效的目标存款金额</div>
      </div>

      <div class="form-group">
        <label><span class="emoji">🏦</span> 已有存款</label>
        <input type="number" id="currentAmount" placeholder="例如：50000" min="0" step="1" required>
      </div>

      <div class="form-group">
        <label><span class="emoji">📅</span> 每月存款</label>
        <input type="number" id="monthlyAmount" placeholder="例如：10000" min="0" step="1" required>
        <div class="error-msg" id="err-monthly">请输入有效的每月存款金额（需大于0）</div>
      </div>

      <div class="form-group">
        <label><span class="emoji">🕘</span> 上班时间</label>
        <input type="time" id="onworkTime" required>
        <div class="error-msg" id="err-time">上班时间需早于下班时间</div>
      </div>

      <div class="form-group">
        <label><span class="emoji">🕕</span> 下班时间</label>
        <input type="time" id="offworkTime" required>
      </div>

      <button type="submit" class="btn-primary">冲！💨</button>
    </form>
  </div>

  <!-- ===== Result Page ===== -->
  <div class="result-page" id="resultPage">
    <button class="settings-btn" id="settingsBtn" title="修改参数">⚙️</button>

    <div class="motto-area" id="mottoArea"></div>

    <div class="result-date-area">
      <div class="result-label">最终跑路时间</div>
      <div class="result-date" id="resultDate"></div>
      <div class="result-days" id="resultDays"></div>
    </div>

    <div class="circle-wrapper">
      <div class="circle-container">
        <svg viewBox="0 0 220 220">
          <defs>
            <linearGradient id="pinkGrad" x1="0%" y1="0%" x2="100%" y2="100%">
              <stop offset="0%" style="stop-color:#f093fb"/>
              <stop offset="100%" style="stop-color:#f5576c"/>
            </linearGradient>
          </defs>
          <circle class="circle-bg" cx="110" cy="110" r="95"/>
          <circle class="circle-progress" id="circleProgress" cx="110" cy="110" r="95"
                  stroke-dasharray="596.9" stroke-dashoffset="596.9"/>
        </svg>
        <div class="circle-text">
          <div class="circle-time" id="circleTime">--:--:--</div>
          <div class="circle-label" id="circleLabel">下班倒计时</div>
        </div>
      </div>
    </div>
  </div>
</div>

</body>
</html>
```

- [ ] **Step 2: 验证页面结构**

运行：用浏览器打开 `index.html`
预期：看到表单页面，包含5个输入字段和"冲！💨"按钮，粉色渐变背景，白色圆角卡片。
移动端适配：缩小浏览器窗口，卡片应自适应宽度。

---

### Task 2: 实现表单逻辑与 localStorage 持久化

**Files:**
- Modify: `index.html` — 在 `</body>` 标签前添加 `<script>` 块

- [ ] **Step 1: 添加表单逻辑代码**

在 `</body>` 前添加：

```html
<script>
(function() {
  'use strict';

  const STORAGE_KEY = 'runaway_calculator_data';

  // ===== DOM Elements =====
  const formPage = document.getElementById('formPage');
  const resultPage = document.getElementById('resultPage');
  const form = document.getElementById('runawayForm');
  const settingsBtn = document.getElementById('settingsBtn');

  const targetInput = document.getElementById('targetAmount');
  const currentInput = document.getElementById('currentAmount');
  const monthlyInput = document.getElementById('monthlyAmount');
  const onworkInput = document.getElementById('onworkTime');
  const offworkInput = document.getElementById('offworkTime');

  const errTarget = document.getElementById('err-target');
  const errMonthly = document.getElementById('err-monthly');
  const errTime = document.getElementById('err-time');

  // ===== localStorage =====
  function saveData() {
    const data = {
      targetAmount: targetInput.value,
      currentAmount: currentInput.value,
      monthlyAmount: monthlyInput.value,
      onworkTime: onworkInput.value,
      offworkTime: offworkInput.value,
    };
    localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
  }

  function loadData() {
    try {
      const raw = localStorage.getItem(STORAGE_KEY);
      if (!raw) return null;
      return JSON.parse(raw);
    } catch {
      return null;
    }
  }

  function populateForm() {
    const data = loadData();
    if (!data) return;
    targetInput.value = data.targetAmount || '';
    currentInput.value = data.currentAmount || '';
    monthlyInput.value = data.monthlyAmount || '';
    onworkInput.value = data.onworkTime || '';
    offworkInput.value = data.offworkTime || '';
  }

  // ===== View Switching =====
  function showResult() {
    formPage.classList.remove('active');
    resultPage.classList.add('active');
    initResult();
  }

  function showForm() {
    resultPage.classList.remove('active');
    formPage.classList.add('active');
    stopTimer();
  }

  // ===== Form Validation =====
  function validateForm() {
    let valid = true;
    errTarget.classList.remove('show');
    errMonthly.classList.remove('show');
    errTime.classList.remove('show');

    const target = parseFloat(targetInput.value);
    const monthly = parseFloat(monthlyInput.value);
    const onwork = onworkInput.value;
    const offwork = offworkInput.value;

    if (isNaN(target) || target <= 0) {
      errTarget.textContent = '请输入有效的目标存款金额';
      errTarget.classList.add('show');
      valid = false;
    }
    if (isNaN(monthly) || monthly <= 0) {
      errMonthly.classList.add('show');
      valid = false;
    }
    if (onwork && offwork && onwork >= offwork) {
      errTime.classList.add('show');
      valid = false;
    }

    return valid;
  }

  // ===== Form Submit =====
  form.addEventListener('submit', function(e) {
    e.preventDefault();
    if (!validateForm()) return;
    saveData();
    showResult();
  });

  settingsBtn.addEventListener('click', showForm);

  // ===== Initialize =====
  populateForm();
})();
</script>
```

- [ ] **Step 2: 测试表单功能**

1. 打开页面，填写表单，点"冲！💨" — 预期：切换到结果页
2. 点右上角 ⚙️ 按钮 — 预期：返回表单页，数据保留
3. 清空表单重新打开页面 — 预期：自动填充上次数据
4. 测试错误场景：
   - 目标存款填 0 或负数 — 预期：显示错误提示
   - 每月存款填 0 — 预期：显示错误提示
   - 上班时间晚于下班时间 — 预期：显示错误提示

---

### Task 3: 实现计算逻辑与结果展示

**Files:**
- Modify: `index.html` — 在 `<script>` 块中添加计算逻辑

- [ ] **Step 1: 添加计算逻辑和结果渲染**

将以下代码插入到 `populateForm()` 之后、`})();` 之前：

```javascript
  // ===== Calculations =====
  function calcRunawayDate() {
    const target = parseFloat(targetInput.value);
    const current = parseFloat(currentInput.value) || 0;
    const monthly = parseFloat(monthlyInput.value);

    if (current >= target) {
      return { months: 0, days: 0, date: new Date() };
    }

    const remaining = target - current;
    const monthsNeeded = Math.ceil(remaining / monthly);
    const now = new Date();
    const targetDate = new Date(now.getFullYear(), now.getMonth() + monthsNeeded, now.getDate());
    const diffMs = targetDate.getTime() - now.getTime();
    const daysLeft = Math.ceil(diffMs / (1000 * 60 * 60 * 24));

    return { months: monthsNeeded, days: daysLeft, date: targetDate };
  }

  function formatMotto(result) {
    const current = parseFloat(currentInput.value) || 0;
    const target = parseFloat(targetInput.value);
    if (current >= target) return '还上啥班啊，跑路 🎉';

    const now = new Date();
    const offwork = offworkInput.value;
    const [offH, offM] = offwork.split(':').map(Number);
    const offTime = new Date(now.getFullYear(), now.getMonth(), now.getDate(), offH, offM);

    if (now > offTime) return '别卷了，该走了 🏃💨';
    if (result.days <= 30) return '快了快了，胜利在望！🔥';
    if (result.days <= 365) return '坚持住，好日子不远了！💪';
    return '冲！每一笔存款都是自由的基石 🚀';
  }

  // ===== Result Page Init =====
  const mottoArea = document.getElementById('mottoArea');
  const resultDateEl = document.getElementById('resultDate');
  const resultDaysEl = document.getElementById('resultDays');

  function initResult() {
    const result = calcRunawayDate();

    // Motto
    mottoArea.textContent = formatMotto(result);

    // Date display
    const d = result.date;
    const dateStr = d.getFullYear() + '年' + String(d.getMonth() + 1).padStart(2, '0') + '月';
    resultDateEl.textContent = dateStr;

    if (result.days <= 0) {
      resultDaysEl.innerHTML = '<span>已达成！现在就跑！🎉</span>';
    } else {
      resultDaysEl.innerHTML = '还剩 <span>' + result.days + '</span> 天';
    }

    // Start countdown timer
    startCountdown();
  }
```

- [ ] **Step 2: 测试计算逻辑**

用浏览器打开 `index.html`，填写测试数据：
- 目标存款：500000，已有存款：50000，每月存款：10000
- 预期：需要 45 个月，显示约 1350+ 天，显示对应日期的年月
- 目标存款：50000，已有存款：50000
- 预期：显示"还上啥班啊，跑路 🎉"
- 下班时间设为当前时间之前
- 预期：显示"别卷了，该走了 🏃💨"

---

### Task 4: 实现圆环倒计时

**Files:**
- Modify: `index.html` — 在 `<script>` 块中添加倒计时逻辑

- [ ] **Step 1: 添加圆环倒计时代码**

在 `initResult()` 之后、`})();` 之前添加：

```javascript
  // ===== Countdown Timer =====
  const circleProgress = document.getElementById('circleProgress');
  const circleTimeEl = document.getElementById('circleTime');
  const circleLabel = document.getElementById('circleLabel');
  const CIRCLE_CIRCUMFERENCE = 2 * Math.PI * 95; // ≈ 596.9
  let timerId = null;

  function stopTimer() {
    if (timerId) {
      clearInterval(timerId);
      timerId = null;
    }
  }

  function startCountdown() {
    stopTimer();
    updateCountdown();
    timerId = setInterval(updateCountdown, 1000);
  }

  function updateCountdown() {
    const now = new Date();
    const onwork = onworkInput.value;
    const offwork = offworkInput.value;

    const [onH, onM] = onwork.split(':').map(Number);
    const [offH, offM] = offwork.split(':').map(Number);

    const startTime = new Date(now.getFullYear(), now.getMonth(), now.getDate(), onH, onM);
    const endTime = new Date(now.getFullYear(), now.getMonth(), now.getDate(), offH, offM);

    // Check if already past offwork time
    if (now >= endTime) {
      circleTimeEl.textContent = '已下班！';
      circleLabel.textContent = '🎉 冲！';
      circleProgress.style.strokeDashoffset = '0';
      return;
    }

    // Check if before onwork time
    if (now < startTime) {
      // Show countdown to offwork from now (full day remaining)
      const totalWorkMs = endTime - startTime;
      const remainingMs = endTime - now;
      const remainingSec = Math.floor(remainingMs / 1000);
      const hours = Math.floor(remainingSec / 3600);
      const minutes = Math.floor((remainingSec % 3600) / 60);
      const seconds = remainingSec % 60;

      circleTimeEl.textContent =
        String(hours).padStart(2, '0') + ':' +
        String(minutes).padStart(2, '0') + ':' +
        String(seconds).padStart(2, '0');
      circleLabel.textContent = '下班倒计时';
      circleProgress.style.strokeDashoffset = '0'; // Full circle when before work
      return;
    }

    // During work hours
    const totalWorkMs = endTime - startTime;
    const elapsedMs = now - startTime;
    const remainingMs = endTime - now;

    // Progress (0 at start, 1 at end)
    const progress = Math.min(Math.max(elapsedMs / totalWorkMs, 0), 1);
    const offset = CIRCLE_CIRCUMFERENCE * (1 - progress);
    circleProgress.style.strokeDashoffset = String(offset);

    // Time display
    const remainingSec = Math.floor(remainingMs / 1000);
    const hours = Math.floor(remainingSec / 3600);
    const minutes = Math.floor((remainingSec % 3600) / 60);
    const seconds = remainingSec % 60;

    circleTimeEl.textContent =
      String(hours).padStart(2, '0') + ':' +
      String(minutes).padStart(2, '0') + ':' +
      String(seconds).padStart(2, '0');
    circleLabel.textContent = '下班倒计时';
  }
```

- [ ] **Step 2: 测试圆环倒计时**

1. 设置下班时间为当前时间之后（如 2 小时后）
2. 打开结果页 — 预期：圆环从空开始逐步填充，中心显示倒计时 `HH:MM:SS` 每秒递减
3. 设置下班时间为过去时间 — 预期：显示"已下班！🎉"，圆环满格
4. 设置上班时间为未来时间 — 预期：圆环满格，显示到下班的全部时长

---

### Task 5: 最终测试与优化

**Files:**
- Modify: `index.html` — 根据需要微调

- [ ] **Step 1: 完整流程测试**

在浏览器中测试以下场景：

| 测试场景 | 预期结果 |
|---------|---------|
| 首次打开空白页 | 显示表单页，字段为空 |
| 填写数据提交 → 刷新页面 | 表单自动填充上次数据 |
| 已有存款 = 目标存款 | 结果页显示"还上啥班啊，跑路 🎉" |
| 当前时间 > 下班时间 | 圆环显示"已下班！🎉" |
| 剩余天数 < 30 | 显示"快了快了，胜利在望！🔥" |
| 剩余天数 < 365 | 显示"坚持住，好日子不远了！💪" |
| 点 ⚙️ 返回表单 → 修改 → 重新提交 | 结果页更新 |
| 窗口缩放到手机尺寸 | 布局正常，无溢出 |
| 上班时间 >= 下班时间 | 显示错误提示 |
| 每月存款 = 0 | 显示错误提示 |

- [ ] **Step 2: 删除 design-mockup.html（可选）**

如果不需要保留设计稿，可以删除 `design-mockup.html`。
