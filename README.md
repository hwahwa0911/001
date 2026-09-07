<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>班級潔牙登記、服務輪值與未潔牙月統計系統</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    html {
      scroll-behavior: smooth;
    }
    ::-webkit-scrollbar { width: 8px; height: 8px; }
    ::-webkit-scrollbar-track { background: #f1f5f9; }
    ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }
    ::-webkit-scrollbar-thumb:hover { background: #94a3b8; }

    .tooth-card {
      transition: all 0.12s ease-in-out;
      user-select: none;
    }
    .tooth-card:active {
      transform: scale(0.92);
    }
    @keyframes pulse-banner {
      0%, 100% { transform: scale(1); }
      50% { transform: scale(1.03); }
    }
    .animate-pulse-banner {
      animation: pulse-banner 1.5s infinite;
    }
    
    .sticky-col {
      position: sticky;
      left: 0;
      z-index: 10;
    }
  </style>
</head>
<body class="bg-slate-100 min-h-screen text-slate-800 font-sans">

  <!-- ========================================================= -->
  <!-- 第一層：學生看板展示區 (標準一頁式滿屏、大字輪值看板) -->
  <!-- ========================================================= -->
  <main class="min-h-screen p-2.5 flex flex-col justify-between max-w-[1920px] mx-auto box-border">
    
    <!-- 1. 頂部狀態列 -->
    <header class="bg-white rounded-xl shadow-xs border border-slate-200 px-3 py-1.5 flex items-center justify-between shrink-0">
      <div class="flex items-center gap-3">
        <span class="text-xl">🪥</span>
        <div>
          <h1 class="text-base md:text-lg font-black text-slate-800 tracking-tight leading-none">
            班級潔牙登記與服務輪值系統
          </h1>
          <span id="date-display" class="text-[11px] font-bold text-slate-500">📅 讀取中...</span>
        </div>

        <!-- 星期預覽快速測試按鈕群 -->
        <div class="hidden sm:flex items-center gap-1 bg-slate-100 px-2 py-0.5 rounded-lg border border-slate-200">
          <span class="text-[10px] font-black text-slate-500">預覽:</span>
          <button onclick="previewDay(1)" class="px-1.5 py-0.5 text-[11px] font-bold bg-white rounded hover:bg-indigo-50 border border-slate-200">一</button>
          <button onclick="previewDay(2)" class="px-1.5 py-0.5 text-[11px] font-bold bg-white rounded hover:bg-indigo-50 border border-slate-200 text-indigo-700">二</button>
          <button onclick="previewDay(3)" class="px-1.5 py-0.5 text-[11px] font-bold bg-white rounded hover:bg-indigo-50 border border-slate-200">三</button>
          <button onclick="previewDay(4)" class="px-1.5 py-0.5 text-[11px] font-bold bg-white rounded hover:bg-indigo-50 border border-slate-200">四</button>
          <button onclick="previewDay(5)" class="px-1.5 py-0.5 text-[11px] font-bold bg-white rounded hover:bg-indigo-50 border border-slate-200">五</button>
          <button onclick="previewDay(null)" class="px-1 py-0.5 text-[10px] font-bold text-slate-400 underline">今日</button>
        </div>

        <!-- 靠左含氟漱口水醒目提示橫幅 -->
        <div id="fluoride-banner" class="hidden animate-pulse-banner ml-2">
          <div class="bg-rose-600 border-2 border-yellow-300 text-yellow-100 px-3 py-0.5 rounded-full shadow-md flex items-center gap-1.5">
            <span class="text-sm">🧪</span>
            <span class="text-xs md:text-sm font-black tracking-wider drop-shadow-sm">今日要用含氟漱口水</span>
          </div>
        </div>
      </div>

      <!-- 右側：統計與功能按鈕 -->
      <div class="flex items-center gap-2.5">
        <div class="flex items-center gap-2 text-xs font-black">
          <span class="text-blue-700 bg-blue-50 px-2.5 py-1 rounded-md border border-blue-200">
            已潔牙：<span id="done-count" class="text-sm font-black">0</span>
          </span>
          <span class="text-rose-600 bg-rose-50 px-2.5 py-1 rounded-md border border-rose-200">
            未潔牙：<span id="undone-count" class="text-sm font-black">0</span>
          </span>
        </div>

        <label class="flex items-center gap-1.5 bg-indigo-50 hover:bg-indigo-100 border border-indigo-200 px-2.5 py-1 rounded-lg cursor-pointer text-xs font-bold text-indigo-900 transition">
          <input type="checkbox" id="fluoride-toggle" class="w-4 h-4 accent-indigo-600 rounded">
          <span>含氟漱口水</span>
        </label>

        <a href="#stats-and-settings" class="bg-slate-700 hover:bg-slate-800 text-white text-xs font-bold px-3 py-1.5 rounded-lg shadow-xs transition flex items-center gap-1">
          <span>⚙️ 往下拉後台與月統計表</span>
          <span>↓</span>
        </a>
      </div>
    </header>

    <!-- 2. 全班潔牙點名板 -->
    <section class="bg-white rounded-xl shadow-xs border border-blue-200 px-3 py-1.5 shrink-0 my-1">
      <div class="flex items-center justify-between pb-1 mb-1 border-b border-slate-100 text-[11px] font-black text-slate-500">
        <span class="flex items-center gap-1">🦷 全班潔牙點名板（點擊座號即時自動存檔，完成呈藍底白牙）</span>
        <div class="flex items-center gap-3">
          <span class="text-orange-600 font-bold">■ 橘底：打菜</span>
          <span class="text-emerald-600 font-bold">■ 綠底：抬回</span>
          <span class="text-rose-600 font-bold">■ 紅底：值日</span>
          <span class="text-amber-700 font-bold">👑：輪值組長 (參與輪值並協助監督)</span>
        </div>
      </div>
      <div id="teeth-grid" class="grid grid-cols-10 gap-1.5"></div>
    </section>

    <!-- 3. 今日服務人員輪值看板 (組長與座號放大、監督幹部標註清楚) -->
    <section class="bg-white rounded-xl shadow-xs border-2 border-indigo-300 p-2.5 flex-1 flex flex-col justify-between shrink-0">
      <div class="flex items-center justify-between pb-1 border-b border-slate-100 shrink-0">
        <div class="flex items-center gap-2">
          <span class="text-base">📋</span>
          <h2 class="text-sm md:text-base font-black text-slate-800 tracking-wide">今日服務人員輪值看板</h2>
          <span id="roster-status-badge" class="bg-indigo-600 text-white text-[11px] font-black px-2.5 py-0.5 rounded-full shadow-2xs">
            載入中...
          </span>
        </div>
        <span class="text-[10px] font-bold text-amber-800 bg-amber-50 px-2 py-0.5 rounded border border-amber-200">
          😷 衛生規範：打菜與抬回人員請確實穿戴圍裙、帽子與口罩
        </span>
      </div>

      <!-- 3 大橫列容器 -->
      <div class="flex flex-col gap-2 flex-1 justify-around my-1">
        
        <!-- 第一列：打菜組 -->
        <div class="bg-amber-50/90 border-3 border-amber-400 rounded-xl px-3 py-2 flex items-center justify-between gap-3 shadow-xs">
          <div class="flex items-center gap-2.5 shrink-0 min-w-[280px]">
            <div class="border-2 border-amber-600 bg-amber-100 rounded-lg px-2.5 py-1 flex items-center gap-1.5 shadow-2xs">
              <span class="text-base md:text-lg font-black text-amber-950">🍱 打菜組</span>
              <span id="serve-group-tag" class="bg-amber-600 text-white px-2 py-0.5 rounded text-xs font-black">A組</span>
            </div>
            <span class="text-xs md:text-sm bg-amber-200 text-amber-950 font-black px-2.5 py-1 rounded-lg border-2 border-amber-400 shadow-2xs">
              👮 班長監督
            </span>
            <span id="serve-leader-badge" class="text-xs md:text-sm bg-white text-amber-950 font-black px-2.5 py-1 rounded-lg border-2 border-amber-500 shadow-2xs">
              👑 組長：--
            </span>
          </div>

          <div id="serve-row-list" class="grid grid-cols-5 gap-2 flex-1 max-w-[840px]"></div>
        </div>

        <!-- 第二列：抬回組 -->
        <div class="bg-emerald-50/90 border-3 border-emerald-400 rounded-xl px-3 py-2 flex items-center justify-between gap-3 shadow-xs">
          <div class="flex items-center gap-2.5 shrink-0 min-w-[280px]">
            <div class="border-2 border-emerald-600 bg-emerald-100 rounded-lg px-2.5 py-1 flex items-center gap-1.5 shadow-2xs">
              <span class="text-base md:text-lg font-black text-emerald-950">🪣 抬回組</span>
              <span id="return-group-tag" class="bg-emerald-600 text-white px-2 py-0.5 rounded text-xs font-black">B組</span>
            </div>
            <span class="text-xs md:text-sm bg-emerald-200 text-emerald-950 font-black px-2.5 py-1 rounded-lg border-2 border-emerald-400 shadow-2xs">
              👮 副班長監督
            </span>
            <span id="return-leader-badge" class="text-xs md:text-sm bg-white text-emerald-950 font-black px-2.5 py-1 rounded-lg border-2 border-emerald-500 shadow-2xs">
              👑 組長：--
            </span>
          </div>

          <div id="return-row-list" class="grid grid-cols-5 gap-2 flex-1 max-w-[840px]"></div>
        </div>

        <!-- 第三列：值日生組 -->
        <div class="bg-rose-50/90 border-3 border-rose-400 rounded-xl px-3 py-2 flex items-center justify-between gap-3 shadow-xs">
          <div class="flex items-center gap-2.5 shrink-0 min-w-[280px]">
            <div class="border-2 border-rose-600 bg-rose-100 rounded-lg px-2.5 py-1 flex items-center gap-1.5 shadow-2xs">
              <span class="text-base md:text-lg font-black text-rose-950">🧹 值日生組</span>
              <span id="duty-group-tag" class="bg-rose-500 text-white px-2 py-0.5 rounded text-xs font-black">C組</span>
            </div>
            <span class="text-xs md:text-sm bg-rose-200 text-rose-950 font-black px-2.5 py-1 rounded-lg border-2 border-rose-400 shadow-2xs">
              👮 風紀監督
            </span>
            <span id="duty-leader-badge" class="text-xs md:text-sm bg-white text-rose-950 font-black px-2.5 py-1 rounded-lg border-2 border-rose-500 shadow-2xs">
              👑 組長：--
            </span>
          </div>

          <div id="duty-row-all-members" class="flex items-center gap-2.5 flex-1 justify-end overflow-x-auto"></div>
        </div>

      </div>
    </section>
  </main>


  <!-- ========================================================= -->
  <!-- 第二層：教師分組管理後台 + 橫向日期矩陣未潔牙表格 -->
  <!-- ========================================================= -->
  <section id="stats-and-settings" class="mt-16 pt-8 pb-20 px-4 md:px-6 bg-slate-200 border-t-8 border-indigo-600 space-y-8">
    
    <!-- 教師設定控制台 -->
    <div class="max-w-7xl mx-auto bg-white rounded-2xl shadow-md border border-slate-300 p-5 space-y-5">
      <div class="flex flex-wrap items-center justify-between pb-3 border-b-2 border-slate-100 gap-3">
        <div class="flex items-center gap-2">
          <span class="text-2xl">⚙️</span>
          <div>
            <h2 class="text-lg md:text-xl font-black text-slate-800">
              教師管理後台：組別與輪值工作指派（依點選順序入組）
            </h2>
            <p class="text-xs font-semibold text-slate-500">
              先選組別再點座號，將嚴格按照您點擊的順序排列；第一位點選者即為【👑 組長】，每日輪值工作亦按此順序輪轉！
            </p>
          </div>
        </div>

        <!-- 控制按鈕群：已改為「重新分組」（歸零清空） -->
        <div class="flex items-center gap-2">
          <button id="btn-reset-groups" class="bg-rose-600 hover:bg-rose-700 text-white text-xs font-bold px-3 py-2 rounded-xl shadow-xs transition flex items-center gap-1">
            🗑️ 重新分組
          </button>
          <button id="btn-save-settings" class="bg-indigo-600 hover:bg-indigo-700 text-white text-xs font-bold px-4 py-2 rounded-xl shadow-sm transition">
            💾 儲存分組與輪值設定
          </button>
          <a href="#" class="bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-bold px-3 py-2 rounded-xl transition border border-slate-300">
            ↑ 回到學生看板頂端
          </a>
        </div>
      </div>

      <!-- 人數與三組職責指派選單 -->
      <div class="grid grid-cols-1 md:grid-cols-4 gap-3 bg-slate-50 p-3 rounded-xl border border-slate-200">
        <div>
          <label class="block text-xs font-black text-slate-600 mb-1">班級總人數 (25~30人)：</label>
          <select id="setting-student-count" class="w-full bg-white border border-slate-300 rounded-lg p-1.5 text-xs font-black text-indigo-700">
            <option value="25">25 人</option>
            <option value="26">26 人</option>
            <option value="27" selected>27 人</option>
            <option value="28">28 人</option>
            <option value="29">29 人</option>
            <option value="30">30 人</option>
          </select>
        </div>

        <div>
          <label class="block text-xs font-black text-amber-900 mb-1">🍱 本月【打菜組】：</label>
          <select id="setting-serve-group" class="w-full bg-amber-50 border border-amber-300 rounded-lg p-1.5 text-xs font-black text-amber-950">
            <option value="A" selected>A 組 (打菜)</option>
            <option value="B">B 組 (打菜)</option>
            <option value="C">C 組 (打菜)</option>
            <option value="D">D 組 (打菜)</option>
            <option value="E">E 組 (打菜)</option>
          </select>
        </div>

        <div>
          <label class="block text-xs font-black text-emerald-900 mb-1">🪣 本月【抬回組】：</label>
          <select id="setting-return-group" class="w-full bg-emerald-50 border border-emerald-300 rounded-lg p-1.5 text-xs font-black text-emerald-950">
            <option value="A">A 組 (抬回)</option>
            <option value="B" selected>B 組 (抬回)</option>
            <option value="C">C 組 (抬回)</option>
            <option value="D">D 組 (抬回)</option>
            <option value="E">E 組 (抬回)</option>
          </select>
        </div>

        <div>
          <label class="block text-xs font-black text-rose-900 mb-1">🧹 本月【值日生組】(紅色)：</label>
          <select id="setting-duty-group" class="w-full bg-rose-50 border border-rose-300 rounded-lg p-1.5 text-xs font-black text-rose-950">
            <option value="A">A 組 (值日)</option>
            <option value="B">B 組 (值日)</option>
            <option value="C" selected>C 組 (值日)</option>
            <option value="D">D 組 (值日)</option>
            <option value="E">E 組 (值日)</option>
          </select>
        </div>
      </div>

      <!-- 分組挑選區 (按順序入組、第一位為組長) -->
      <div class="bg-indigo-50/60 border border-indigo-200 rounded-2xl p-4 space-y-4">
        <div>
          <div class="text-xs font-black text-indigo-950 mb-2">
            👉 步驟 1：選擇要設定的組別標籤：
          </div>
          <div id="group-tabs" class="grid grid-cols-5 gap-2"></div>
        </div>

        <div class="bg-white border border-indigo-200 rounded-xl p-3 shadow-2xs">
          <div class="flex items-center justify-between mb-2">
            <span class="text-xs font-black text-slate-700">
              👉 步驟 2：依期望順序點選座號 ➔ 加入【<span id="current-active-group-label" class="text-indigo-600 font-black">A</span>組】：
            </span>
            <span class="text-[10px] text-slate-400">點選的第一位會自動成為組長，若點擊已在該組的座號則退出該組</span>
          </div>
          <div id="seat-number-pool" class="grid grid-cols-6 sm:grid-cols-10 gap-2"></div>
        </div>

        <div class="bg-white border border-slate-200 rounded-xl p-3">
          <div class="flex items-center justify-between mb-2 pb-1 border-b border-slate-100">
            <span class="text-xs font-black text-slate-800">
              📋 【<span id="current-active-group-title" class="text-indigo-600">A</span>組】現有名冊順序（按加入先後排列）：
              <span id="current-active-group-count" class="text-[11px] text-slate-500 font-bold ml-1">共 0 人</span>
            </span>
            <span class="text-[10px] font-bold text-rose-600">★ 第 1 位為【組長】，輪值時優先從此順序開始輪動</span>
          </div>
          <div id="current-group-members-list" class="flex flex-wrap gap-2 min-h-[38px] items-center"></div>
        </div>
      </div>
    </div>

    <!-- 未潔牙統計表總覽 -->
    <div class="max-w-7xl mx-auto space-y-6">
      <div class="bg-white rounded-2xl p-4 border border-slate-300 shadow-sm flex flex-wrap items-center justify-between gap-3">
        <div>
          <h2 class="text-lg md:text-xl font-black text-slate-800 flex items-center gap-2">
            <span>📊</span> 全班每日未潔牙紀錄統計表（表格呈現）
          </h2>
          <p class="text-xs font-bold text-slate-500 mt-0.5">
            第一列為月份與操作、第二列為 1~31 日、左欄為座號。若當天未潔牙標註為 <span class="text-rose-600 font-black">✕</span>。
          </p>
        </div>

        <div class="flex items-center gap-2 bg-slate-50 px-3 py-1.5 rounded-xl border border-slate-200">
          <span class="text-xs font-bold text-slate-600">已封存月份：</span>
          <select id="select-archived-months" class="bg-white border border-slate-300 rounded text-xs font-bold px-2 py-1 text-slate-700">
            <option value="">(無封存月份)</option>
          </select>
          <button id="btn-unarchive-selected" class="bg-slate-700 hover:bg-slate-800 text-white text-xs font-bold px-2.5 py-1 rounded transition">
            🔓 解除封存／重新顯示
          </button>
        </div>
      </div>

      <div id="monthly-tables-container" class="space-y-6"></div>
    </div>

  </section>


  <!-- ========================================================= -->
  <!-- 第三層：JavaScript 核心資料模型與自訂順序輪轉邏輯 -->
  <!-- ========================================================= -->
  <script>
    let totalStudents = 27;
    let currentActiveGroup = 'A';
    let selectedServeGroup = 'A';
    let selectedReturnGroup = 'B';
    let selectedDutyGroup = 'C';
    let simulatedDay = null;

    const groupKeys = ['A', 'B', 'C', 'D', 'E'];
    const serveSingleJobs = ['湯', '飯', '菜', '菜', '菜'];
    const returnSingleJobs = ['湯', '湯', '飯', '菜', '菜'];

    let classGroups = { 'A': [], 'B': [], 'C': [], 'D': [], 'E': [] };
    let todayTeethStatus = {};
    let archivedMonths = [];

    const WHITE_TOOTH_SVG = `
      <svg class="w-4 h-4 text-white drop-shadow-xs" viewBox="0 0 24 24" fill="currentColor">
        <path d="M12 2C8.5 2 6 4.2 6 7.5C6 9.2 6.6 11 7.2 12.8C7.8 14.8 8.5 17.2 8.5 20C8.5 21.1 9.4 22 10.5 22C11.3 22 12 21.4 12.2 20.6L12.7 18C12.8 17.4 13.6 17.4 13.7 18L14.2 20.6C14.4 21.4 15.1 22 15.9 22C17 22 17.9 21.1 17.9 20C17.9 17.2 18.6 14.8 19.2 12.8C19.8 11 20.4 9.2 20.4 7.5C20.4 4.2 17.9 2 12.4 2H12Z"/>
      </svg>
    `;

    const GRAY_TOOTH_SVG = `
      <svg class="w-4 h-4 text-slate-300" viewBox="0 0 24 24" fill="currentColor">
        <path d="M12 2C8.5 2 6 4.2 6 7.5C6 9.2 6.6 11 7.2 12.8C7.8 14.8 8.5 17.2 8.5 20C8.5 21.1 9.4 22 10.5 22C11.3 22 12 21.4 12.2 20.6L12.7 18C12.8 17.4 13.6 17.4 13.7 18L14.2 20.6C14.4 21.4 15.1 22 15.9 22C17 22 17.9 21.1 17.9 20C17.9 17.2 18.6 14.8 19.2 12.8C19.8 11 20.4 9.2 20.4 7.5C20.4 4.2 17.9 2 12.4 2H12Z"/>
      </svg>
    `;

    function getTodayString() {
      const now = new Date();
      const y = now.getFullYear();
      const m = String(now.getMonth() + 1).padStart(2, '0');
      const d = String(now.getDate()).padStart(2, '0');
      return `${y}-${m}-${d}`;
    }

    function initSystem() {
      const todayStr = getTodayString();
      const now = new Date();
      const weekStrs = ['星期日', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六'];
      document.getElementById('date-display').textContent = `📅 ${todayStr} ${weekStrs[now.getDay()]}`;

      const savedCount = localStorage.getItem('class_total_students');
      if (savedCount) {
        totalStudents = parseInt(savedCount);
        document.getElementById('setting-student-count').value = totalStudents;
      }

      const savedGroups = localStorage.getItem('class_groups_data');
      if (savedGroups) {
        try { classGroups = JSON.parse(savedGroups); } catch(e) { clearAllGroupsToZero(); }
      } else {
        clearAllGroupsToZero();
      }

      const sServe = localStorage.getItem('duty_serve_group');
      const sReturn = localStorage.getItem('duty_return_group');
      const sDuty = localStorage.getItem('duty_duty_group');
      if (sServe) selectedServeGroup = sServe;
      if (sReturn) selectedReturnGroup = sReturn;
      if (sDuty) selectedDutyGroup = sDuty;

      document.getElementById('setting-serve-group').value = selectedServeGroup;
      document.getElementById('setting-return-group').value = selectedReturnGroup;
      document.getElementById('setting-duty-group').value = selectedDutyGroup;

      const savedArchived = localStorage.getItem('brush_archived_months');
      if (savedArchived) {
        try { archivedMonths = JSON.parse(savedArchived); } catch(e) { archivedMonths = []; }
      }

      checkMidnightReset();

      const savedFluoride = localStorage.getItem('brush_record_fluoride');
      const isFluoride = (savedFluoride !== null) ? JSON.parse(savedFluoride) : false;
      document.getElementById('fluoride-toggle').checked = isFluoride;
      toggleFluorideBanner(isFluoride);

      renderStudentBoard();
      renderTeacherModule();
      renderAllMonthlyTables();

      scheduleMidnightChecker();
    }

    window.previewDay = function(dayNum) {
      simulatedDay = dayNum;
      renderStudentBoard();
    };

    function checkMidnightReset() {
      const todayStr = getTodayString();
      const lastRecordedDate = localStorage.getItem('brush_active_date');
      const savedStatus = localStorage.getItem('brush_active_status');

      if (lastRecordedDate === todayStr && savedStatus) {
        todayTeethStatus = JSON.parse(savedStatus);
      } else {
        todayTeethStatus = {};
        for (let i = 1; i <= totalStudents; i++) {
          todayTeethStatus[i] = false;
        }
        localStorage.setItem('brush_active_date', todayStr);
        localStorage.setItem('brush_active_status', JSON.stringify(todayTeethStatus));
      }
    }

    function scheduleMidnightChecker() {
      setInterval(() => {
        const currentToday = getTodayString();
        const savedDate = localStorage.getItem('brush_active_date');
        if (savedDate && savedDate !== currentToday) {
          checkMidnightReset();
          renderStudentBoard();
          renderAllMonthlyTables();
        }
      }, 30000);
    }

    // 將所有組員清空歸零（全部退回未分配）
    function clearAllGroupsToZero() {
      classGroups = { 'A': [], 'B': [], 'C': [], 'D': [], 'E': [] };
    }

    function findStudentGroup(memberStr) {
      for (let key of groupKeys) {
        if (classGroups[key].includes(memberStr)) return key;
      }
      return null;
    }

    function getTodayDutyMap(offset) {
      const map = {};
      const serveMembers = classGroups[selectedServeGroup] || [];
      const returnMembers = classGroups[selectedReturnGroup] || [];
      const dutyMembers = classGroups[selectedDutyGroup] || [];

      const serveLeader = serveMembers[0];
      const returnLeader = returnMembers[0];
      const dutyLeader = dutyMembers[0];

      serveSingleJobs.forEach((job, idx) => {
        if (serveMembers.length > 0) {
          const mem = serveMembers[(offset + idx) % serveMembers.length];
          const num = parseInt(mem);
          const isLeader = (mem === serveLeader);
          if (!isNaN(num)) {
            map[num] = {
              type: 'serve',
              isLeader: isLeader,
              text: isLeader ? `👑組長•打菜:${job}` : `打菜:${job}`
            };
          }
        }
      });

      returnSingleJobs.forEach((job, idx) => {
        if (returnMembers.length > 0) {
          const mem = returnMembers[(offset + idx) % returnMembers.length];
          const num = parseInt(mem);
          const isLeader = (mem === returnLeader);
          if (!isNaN(num)) {
            map[num] = {
              type: 'return',
              isLeader: isLeader,
              text: isLeader ? `👑組長•抬回:${job}` : `抬回:${job}`
            };
          }
        }
      });

      if (dutyMembers.length > 0) {
        const mem = dutyMembers[offset % dutyMembers.length];
        const num = parseInt(mem);
        const isLeader = (mem === dutyLeader);
        if (!isNaN(num)) {
          map[num] = {
            type: 'duty',
            isLeader: isLeader,
            text: isLeader ? `👑組長•值日生` : `輪值:值日生`
          };
        }
      }

      return map;
    }

    function getDutyBadgeStyle(dutyInfo, isDone) {
      const type = dutyInfo.type;
      const isLeader = dutyInfo.isLeader;

      if (isDone) {
        if (type === 'serve') return 'bg-amber-800 text-amber-100 border border-amber-400';
        if (type === 'return') return 'bg-emerald-800 text-emerald-100 border border-emerald-400';
        if (type === 'duty') return 'bg-rose-800 text-rose-100 border border-rose-400';
      } else {
        if (type === 'serve') return isLeader ? 'bg-amber-200 text-amber-950 font-black border-2 border-amber-500' : 'bg-orange-100 text-orange-900 border border-orange-300';
        if (type === 'return') return isLeader ? 'bg-emerald-200 text-emerald-950 font-black border-2 border-emerald-500' : 'bg-emerald-100 text-emerald-900 border border-emerald-300';
        if (type === 'duty') return isLeader ? 'bg-rose-200 text-rose-950 font-black border-2 border-rose-500' : 'bg-rose-100 text-rose-900 border border-rose-300';
      }
      return 'bg-slate-100 text-slate-700';
    }

    function renderStudentBoard() {
      const now = new Date();
      const day = (simulatedDay !== null) ? simulatedDay : now.getDay();
      const offset = (day >= 1 && day <= 5) ? (day - 1) : 0;
      const weekStrs = ['星期日', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六'];
      const dayName = (day >= 1 && day <= 5) ? weekStrs[day] : '星期一(示範)';

      document.getElementById('roster-status-badge').textContent = `${dayName}（第 ${offset + 1} 位起輪）`;
      document.getElementById('serve-group-tag').textContent = `${selectedServeGroup}組`;
      document.getElementById('return-group-tag').textContent = `${selectedReturnGroup}組`;
      document.getElementById('duty-group-tag').textContent = `${selectedDutyGroup}組`;

      const curServeMems = classGroups[selectedServeGroup] || [];
      const curReturnMems = classGroups[selectedReturnGroup] || [];
      const curDutyMems = classGroups[selectedDutyGroup] || [];

      const serveLeader = curServeMems[0] || '無';
      const returnLeader = curReturnMems[0] || '無';
      const dutyLeader = curDutyMems[0] || '無';

      document.getElementById('serve-leader-badge').textContent = `👑 組長：${serveLeader}`;
      document.getElementById('return-leader-badge').textContent = `👑 組長：${returnLeader}`;
      document.getElementById('duty-leader-badge').textContent = `👑 組長：${dutyLeader}`;

      const dutyMap = getTodayDutyMap(offset);

      const grid = document.getElementById('teeth-grid');
      grid.innerHTML = '';
      let doneCount = 0;

      for (let num = 1; num <= totalStudents; num++) {
        const isDone = !!todayTeethStatus[num];
        if (isDone) doneCount++;
        const dutyInfo = dutyMap[num];

        const card = document.createElement('div');
        card.className = `tooth-card cursor-pointer flex flex-col items-center justify-between py-1 px-0.5 rounded-lg border transition-all ${
          isDone 
            ? 'bg-blue-600 border-blue-700 text-white shadow-xs' 
            : 'bg-slate-50 border-slate-200 text-slate-700 hover:bg-slate-100'
        }`;

        card.innerHTML = `
          <div class="flex items-center gap-0.5">
            ${isDone ? WHITE_TOOTH_SVG : GRAY_TOOTH_SVG}
            <span class="text-xs font-black ${isDone ? 'text-white' : 'text-slate-800'}">${num}</span>
          </div>
          <div class="w-full text-center mt-0.5">
            ${dutyInfo ? `
              <span class="inline-block text-[9px] font-black leading-tight px-1 rounded truncate max-w-full ${getDutyBadgeStyle(dutyInfo, isDone)}">
                ${dutyInfo.text}
              </span>
            ` : `
              <span class="text-[9px] leading-tight font-semibold ${isDone ? 'text-blue-100' : 'text-slate-400'}">
                ${isDone ? '已潔牙' : '未登記'}
              </span>
            `}
          </div>
        `;

        card.onclick = () => {
          todayTeethStatus[num] = !todayTeethStatus[num];
          saveTodayTeethStateDirectly();
          renderStudentBoard();
          renderAllMonthlyTables();
        };

        grid.appendChild(card);
      }

      document.getElementById('done-count').textContent = doneCount;
      document.getElementById('undone-count').textContent = totalStudents - doneCount;

      const serveRowContainer = document.getElementById('serve-row-list');
      serveRowContainer.innerHTML = serveSingleJobs.map((singleJob, idx) => {
        const mem = curServeMems.length > 0 ? curServeMems[(offset + idx) % curServeMems.length] : '--';
        return `
          <div class="flex items-center justify-between bg-white border-2 border-amber-300 rounded-lg px-2.5 py-1 shadow-2xs">
            <span class="text-base md:text-xl font-black text-slate-900 truncate tracking-tight">${mem}</span>
            <span class="text-xl md:text-2xl font-black text-amber-900 bg-amber-100 border border-amber-400 rounded px-2 leading-none py-0.5">${singleJob}</span>
          </div>
        `;
      }).join('');

      const returnRowContainer = document.getElementById('return-row-list');
      returnRowContainer.innerHTML = returnSingleJobs.map((singleJob, idx) => {
        const mem = curReturnMems.length > 0 ? curReturnMems[(offset + idx) % curReturnMems.length] : '--';
        return `
          <div class="flex items-center justify-between bg-white border-2 border-emerald-300 rounded-lg px-2.5 py-1 shadow-2xs">
            <span class="text-base md:text-xl font-black text-slate-900 truncate tracking-tight">${mem}</span>
            <span class="text-xl md:text-2xl font-black text-emerald-900 bg-emerald-100 border border-emerald-400 rounded px-2 leading-none py-0.5">${singleJob}</span>
          </div>
        `;
      }).join('');

      const dutyRowContainer = document.getElementById('duty-row-all-members');
      const todayDutyIndex = offset % (curDutyMems.length || 1);

      dutyRowContainer.innerHTML = curDutyMems.map((mem, idx) => {
        const isTodayDuty = (idx === todayDutyIndex);
        if (isTodayDuty) {
          return `
            <div class="flex items-center gap-2 bg-rose-600 text-white border-3 border-rose-700 rounded-xl px-3 py-1.5 shadow-md ring-2 ring-rose-300">
              <span class="text-base md:text-xl font-black tracking-tight">${mem}</span>
              <span class="text-xs md:text-sm bg-yellow-300 text-rose-950 font-black px-2 py-0.5 rounded shadow-2xs">★今日值日</span>
            </div>
          `;
        } else {
          return `
            <div class="flex items-center gap-1.5 bg-white border-2 border-rose-200 text-slate-700 rounded-lg px-2.5 py-1 shadow-2xs">
              <span class="text-sm md:text-base font-black">${mem}</span>
              <span class="text-[10px] text-slate-400 font-bold">待命</span>
            </div>
          `;
        }
      }).join('');
    }

    function saveTodayTeethStateDirectly() {
      const todayStr = getTodayString();
      const currentMonth = todayStr.substring(0, 7);

      if (archivedMonths.includes(currentMonth)) return;

      localStorage.setItem('brush_active_date', todayStr);
      localStorage.setItem('brush_active_status', JSON.stringify(todayTeethStatus));

      let historyRecords = {};
      const savedHistory = localStorage.getItem('brush_history_records');
      if (savedHistory) {
        try { historyRecords = JSON.parse(savedHistory); } catch(e) { historyRecords = {}; }
      }
      historyRecords[todayStr] = todayTeethStatus;
      localStorage.setItem('brush_history_records', JSON.stringify(historyRecords));
    }

    function getDaysInMonth(year, month) {
      return new Date(year, month, 0).getDate();
    }

    function renderAllMonthlyTables() {
      const container = document.getElementById('monthly-tables-container');
      container.innerHTML = '';

      let historyRecords = {};
      const savedHistory = localStorage.getItem('brush_history_records');
      if (savedHistory) {
        try { historyRecords = JSON.parse(savedHistory); } catch(e) { historyRecords = {}; }
      }
      const thisMonth = getTodayString().substring(0, 7);
      if (!archivedMonths.includes(thisMonth)) {
        historyRecords[getTodayString()] = todayTeethStatus;
      }

      const monthsSet = new Set();
      monthsSet.add(thisMonth);
      Object.keys(historyRecords).forEach(dateStr => {
        if (dateStr.length >= 7) monthsSet.add(dateStr.substring(0, 7));
      });
      const sortedMonths = Array.from(monthsSet).sort();

      updateArchivedSelect();

      const visibleMonths = sortedMonths.filter(m => !archivedMonths.includes(m));

      if (visibleMonths.length === 0) {
        container.innerHTML = `
          <div class="bg-white rounded-2xl p-8 text-center text-slate-500 font-bold border border-slate-300">
            📭 目前所有月份資料皆已封存隱藏。若要調閱歷史表格，請從上方「已封存月份」選單選擇並解除封存。
          </div>
        `;
        return;
      }

      visibleMonths.forEach(mStr => {
        const [yearStr, monthStr] = mStr.split('-');
        const year = parseInt(yearStr);
        const month = parseInt(monthStr);
        const daysCount = getDaysInMonth(year, month);

        const tableCard = document.createElement('div');
        tableCard.className = 'bg-white rounded-2xl shadow-md border-2 border-slate-300 overflow-hidden';

        let totalMonthMisses = 0;
        for (let d = 1; d <= daysCount; d++) {
          const dateKey = `${mStr}-${String(d).padStart(2, '0')}`;
          if (historyRecords[dateKey]) {
            for (let s = 1; s <= totalStudents; s++) {
              if (!historyRecords[dateKey][s]) totalMonthMisses++;
            }
          }
        }

        tableCard.innerHTML = `
          <div class="bg-indigo-900 text-white px-4 py-3 flex flex-wrap items-center justify-between gap-3 border-b-2 border-indigo-950">
            <div class="flex items-center gap-3">
              <span class="text-xl">📅</span>
              <h3 class="text-base md:text-lg font-black tracking-wide">
                【${year} 年 ${month} 月】學生未潔牙紀錄統計表
              </h3>
              <span class="bg-rose-500 text-white text-xs font-black px-2.5 py-0.5 rounded-full shadow-xs">
                本月未潔牙人次累計：${totalMonthMisses} 次
              </span>
            </div>

            <div class="flex items-center gap-2">
              <button onclick="exportMonthCSV('${mStr}')" class="bg-white/10 hover:bg-white/20 text-white text-xs font-bold px-3 py-1.5 rounded-lg transition border border-white/20">
                📄 匯出此月報表 (CSV)
              </button>
              <button onclick="archiveMonth('${mStr}')" class="bg-rose-600 hover:bg-rose-700 text-white text-xs font-bold px-3 py-1.5 rounded-lg shadow-xs transition border border-rose-400">
                🔒 封存本月資料（表格隱藏不再變動）
              </button>
            </div>
          </div>

          <div class="overflow-x-auto p-3">
            <table class="w-full text-center border-collapse text-xs">
              <thead>
                <tr class="bg-slate-100 text-slate-700 border-b-2 border-slate-300">
                  <th class="sticky-col bg-slate-200 px-3 py-2 text-slate-800 font-black min-w-[70px] border-r border-slate-300">座號</th>
                  ${Array.from({ length: daysCount }, (_, i) => `
                    <th class="px-2 py-1.5 font-bold min-w-[28px] border-r border-slate-200">${i + 1}</th>
                  `).join('')}
                  <th class="px-3 py-1.5 bg-rose-50 text-rose-900 font-black min-w-[65px]">未潔牙總計</th>
                </tr>
              </thead>
              <tbody class="divide-y divide-slate-100">
                ${generateTableRows(mStr, daysCount, historyRecords)}
              </tbody>
            </table>
          </div>
        `;

        container.appendChild(tableCard);
      });
    }

    function generateTableRows(monthStr, daysCount, historyRecords) {
      let html = '';
      const todayStr = getTodayString();

      for (let s = 1; s <= totalStudents; s++) {
        let studentMissTotal = 0;
        let cellsHtml = '';

        for (let d = 1; d <= daysCount; d++) {
          const dateKey = `${monthStr}-${String(d).padStart(2, '0')}`;
          const isFuture = (dateKey > todayStr);
          const dayData = historyRecords[dateKey];

          if (isFuture) {
            cellsHtml += `<td class="border-r border-slate-100 text-slate-200 text-[10px]">-</td>`;
          } else if (dayData !== undefined) {
            const isBrushed = !!dayData[s];
            if (!isBrushed) {
              studentMissTotal++;
              cellsHtml += `<td class="border-r border-slate-100 bg-rose-50/70 text-rose-600 font-black text-sm">✕</td>`;
            } else {
              cellsHtml += `<td class="border-r border-slate-100 text-slate-300 text-[10px]">✓</td>`;
            }
          } else {
            cellsHtml += `<td class="border-r border-slate-100 text-slate-200 text-[10px]">-</td>`;
          }
        }

        const alertBg = studentMissTotal >= 5 ? 'bg-rose-100 font-black text-rose-700' : (studentMissTotal > 0 ? 'text-rose-600 font-bold' : 'text-emerald-700');

        html += `
          <tr class="hover:bg-slate-50 transition-colors">
            <td class="sticky-col bg-slate-100 px-3 py-1.5 font-black text-slate-800 border-r border-slate-300">${s} 號</td>
            ${cellsHtml}
            <td class="px-2 py-1.5 ${alertBg} text-xs">${studentMissTotal} 次</td>
          </tr>
        `;
      }
      return html;
    }

    window.archiveMonth = function(monthStr) {
      const [y, m] = monthStr.split('-');
      if (confirm(`確定要封存【${y}年${parseInt(m)}月】的潔牙紀錄嗎？\n\n封存後：\n1. 該月紀錄將永久凍結，不再隨點名更動。\n2. 該月份表格將自畫面中隱藏消失。\n(日後需要可從上方「已封存月份」選單解除封存重新顯示)`)) {
        if (!archivedMonths.includes(monthStr)) {
          archivedMonths.push(monthStr);
          localStorage.setItem('brush_archived_months', JSON.stringify(archivedMonths));
        }
        renderAllMonthlyTables();
        alert(`✅ 【${y}年${parseInt(m)}月】已成功封存並隱藏！`);
      }
    };

    function updateArchivedSelect() {
      const select = document.getElementById('select-archived-months');
      if (!select) return;

      if (archivedMonths.length === 0) {
        select.innerHTML = '<option value="">(目前無封存月份)</option>';
        return;
      }

      select.innerHTML = archivedMonths.map(mStr => {
        const [y, m] = mStr.split('-');
        return `<option value="${mStr}">${y} 年 ${parseInt(m)} 月 (已封存)</option>`;
      }).join('');
    }

    document.getElementById('btn-unarchive-selected').onclick = () => {
      const select = document.getElementById('select-archived-months');
      const targetMonth = select.value;
      if (!targetMonth) {
        alert('請先選擇要解除封存的月份！');
        return;
      }

      archivedMonths = archivedMonths.filter(m => m !== targetMonth);
      localStorage.setItem('brush_archived_months', JSON.stringify(archivedMonths));
      renderAllMonthlyTables();
      alert(`✅ 已將【${targetMonth}】解除封存，表格已重新呈現在下方！`);
    };

    window.exportMonthCSV = function(monthStr) {
      let historyRecords = {};
      const savedHistory = localStorage.getItem('brush_history_records');
      if (savedHistory) {
        try { historyRecords = JSON.parse(savedHistory); } catch(e) { historyRecords = {}; }
      }
      if (monthStr === getTodayString().substring(0, 7) && !archivedMonths.includes(monthStr)) {
        historyRecords[getTodayString()] = todayTeethStatus;
      }

      const [yearStr, monthStrNum] = monthStr.split('-');
      const year = parseInt(yearStr);
      const month = parseInt(monthStrNum);
      const daysCount = getDaysInMonth(year, month);

      let csvContent = '\uFEFF';
      csvContent += `"${year}年${month}月 班級潔牙登記統計表"\n`;

      const headers = ['座號'];
      for (let d = 1; d <= daysCount; d++) headers.push(`${d}日`);
      headers.push('未潔牙總計(次)');
      csvContent += headers.map(h => `"${h}"`).join(',') + '\n';

      for (let s = 1; s <= totalStudents; s++) {
        const row = [`${s}號`];
        let missCount = 0;

        for (let d = 1; d <= daysCount; d++) {
          const dateKey = `${monthStr}-${String(d).padStart(2, '0')}`;
          const dayData = historyRecords[dateKey];
          if (dayData !== undefined) {
            if (!dayData[s]) {
              missCount++;
              row.push('X');
            } else {
              row.push('');
            }
          } else {
            row.push('-');
          }
        }
        row.push(missCount);
        csvContent += row.map(v => `"${v}"`).join(',') + '\n';
      }

      const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = `潔牙統計表_${monthStr}.csv`;
      a.click();
    };

    // --- 教師分組管理模組 ---
    function renderTeacherModule() {
      document.getElementById('current-active-group-label').textContent = currentActiveGroup;
      document.getElementById('current-active-group-title').textContent = currentActiveGroup;

      const tabsContainer = document.getElementById('group-tabs');
      tabsContainer.innerHTML = groupKeys.map(key => {
        const isActive = (key === currentActiveGroup);
        const count = classGroups[key].length;
        
        let roleName = '';
        if (key === selectedServeGroup) roleName = '🍱打菜';
        if (key === selectedReturnGroup) roleName = '🪣抬回';
        if (key === selectedDutyGroup) roleName = '🧹值日';

        return `
          <button onclick="switchActiveGroup('${key}')" 
                  class="p-2.5 rounded-xl border-2 transition-all flex flex-col items-center justify-center ${
                    isActive 
                      ? 'bg-indigo-600 border-indigo-700 text-white shadow-md scale-102 ring-2 ring-indigo-300' 
                      : 'bg-white border-slate-200 text-slate-700 hover:bg-indigo-50'
                  }">
            <span class="text-base md:text-lg font-black">${key} 組</span>
            <span class="text-xs font-bold mt-0.5 ${isActive ? 'text-indigo-100' : 'text-slate-500'}">
              (${count}人) ${roleName}
            </span>
          </button>
        `;
      }).join('');

      const poolContainer = document.getElementById('seat-number-pool');
      poolContainer.innerHTML = '';

      for (let num = 1; num <= totalStudents; num++) {
        const memberStr = `${num}號`;
        const belongingGroup = findStudentGroup(memberStr);
        const isCurrent = (belongingGroup === currentActiveGroup);

        const btn = document.createElement('button');
        btn.onclick = () => handleSeatClick(memberStr);

        let styleClass = '';
        let badgeText = '';

        if (isCurrent) {
          styleClass = 'bg-indigo-600 border-indigo-700 text-white shadow-xs';
          badgeText = `✓ ${currentActiveGroup}組`;
        } else if (belongingGroup) {
          styleClass = 'bg-slate-100 border-slate-300 text-slate-700 hover:bg-indigo-50';
          badgeText = `${belongingGroup}組`;
        } else {
          styleClass = 'bg-white border-dashed border-2 border-rose-300 text-rose-600 hover:bg-rose-50';
          badgeText = '未分配';
        }

        btn.className = `p-1.5 rounded-lg border flex flex-col items-center justify-between transition text-center ${styleClass}`;
        btn.innerHTML = `
          <span class="text-sm font-black">${num}</span>
          <span class="text-[10px] font-bold px-1 rounded ${isCurrent ? 'bg-indigo-800 text-indigo-100' : 'bg-slate-200 text-slate-600'}">
            ${badgeText}
          </span>
        `;
        poolContainer.appendChild(btn);
      }

      const currentList = classGroups[currentActiveGroup] || [];
      document.getElementById('current-active-group-count').textContent = `共 ${currentList.length} 人`;
      const curListContainer = document.getElementById('current-group-members-list');
      curListContainer.innerHTML = '';

      if (currentList.length === 0) {
        curListContainer.innerHTML = '<span class="text-xs text-slate-400">目前尚無組員，請依順序點擊上方座號加入（第一位為組長）。</span>';
      } else {
        currentList.forEach((mem, index) => {
          const isLeader = (index === 0);
          const badge = document.createElement('div');
          badge.className = `flex items-center gap-1.5 px-3 py-1.5 rounded-lg border-2 shadow-2xs ${
            isLeader 
              ? 'bg-amber-100 border-amber-500 text-amber-950 font-black' 
              : 'bg-slate-50 border-slate-300 text-slate-800 font-bold'
          }`;
          badge.innerHTML = `
            ${isLeader ? '<span class="text-xs bg-amber-500 text-white px-1 rounded mr-0.5">👑組長</span>' : `<span class="text-xs text-slate-400 mr-0.5">#${index+1}</span>`}
            <span class="text-sm font-black">${mem}</span>
            <button onclick="removeMember('${mem}')" title="退出該組" class="text-slate-400 hover:text-rose-600 font-black ml-1.5 text-xs">
              ✕
            </button>
          `;
          curListContainer.appendChild(badge);
        });
      }
    }

    window.switchActiveGroup = function(groupKey) {
      currentActiveGroup = groupKey;
      renderTeacherModule();
    };

    window.handleSeatClick = function(memberStr) {
      const belonging = findStudentGroup(memberStr);

      if (belonging === currentActiveGroup) {
        classGroups[currentActiveGroup] = classGroups[currentActiveGroup].filter(m => m !== memberStr);
      } else {
        if (belonging) {
          classGroups[belonging] = classGroups[belonging].filter(m => m !== memberStr);
        }
        classGroups[currentActiveGroup].push(memberStr);
      }

      renderTeacherModule();
      renderStudentBoard();
    };

    window.removeMember = function(memberStr) {
      classGroups[currentActiveGroup] = classGroups[currentActiveGroup].filter(m => m !== memberStr);
      renderTeacherModule();
      renderStudentBoard();
    };

    document.getElementById('setting-student-count').onchange = (e) => {
      totalStudents = parseInt(e.target.value);
      clearAllGroupsToZero();
      renderTeacherModule();
      renderStudentBoard();
      renderAllMonthlyTables();
    };

    document.getElementById('setting-serve-group').onchange = (e) => {
      selectedServeGroup = e.target.value;
      renderTeacherModule();
      renderStudentBoard();
    };

    document.getElementById('setting-return-group').onchange = (e) => {
      selectedReturnGroup = e.target.value;
      renderTeacherModule();
      renderStudentBoard();
    };

    document.getElementById('setting-duty-group').onchange = (e) => {
      selectedDutyGroup = e.target.value;
      renderTeacherModule();
      renderStudentBoard();
    };

    // 重新分組按鈕：按下後所有組員沒有分組、全部歸零！
    document.getElementById('btn-reset-groups').onclick = () => {
      if (confirm('確定要「重新分組」嗎？\n\n按下後，A～E 五組現有的所有組員將全數清空歸零（退回未分配狀態），讓您重新自訂組員與順序！')) {
        clearAllGroupsToZero();
        renderTeacherModule();
        renderStudentBoard();
        alert('✅ 所有組別已歸零清空！請點擊上方組別標籤，依序挑選座號入組。設定完成後請記得點選「儲存分組與輪值設定」。');
      }
    };

    document.getElementById('btn-save-settings').onclick = () => {
      localStorage.setItem('class_total_students', totalStudents);
      localStorage.setItem('class_groups_data', JSON.stringify(classGroups));
      localStorage.setItem('duty_serve_group', selectedServeGroup);
      localStorage.setItem('duty_return_group', selectedReturnGroup);
      localStorage.setItem('duty_duty_group', selectedDutyGroup);
      alert('✅ 分組名冊順序與每月輪值工作設定已成功存檔！');
    };

    function toggleFluorideBanner(show) {
      const banner = document.getElementById('fluoride-banner');
      if (show) banner.classList.remove('hidden');
      else banner.classList.add('hidden');
    }

    document.getElementById('fluoride-toggle').onchange = (e) => {
      const isChecked = e.target.checked;
      toggleFluorideBanner(isChecked);
      localStorage.setItem('brush_record_fluoride', JSON.stringify(isChecked));
    };

    initSystem();
  </script>
</body>
</html>
