<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>課堂抽籤 - Classroom Lucky Draw</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Canvas Confetti -->
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;600;700;900&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Noto Sans TC', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            background-color: #E6EFEA;
            background-image: 
                radial-gradient(circle at 8% 12%, rgba(254, 240, 138, 0.45) 0%, transparent 28%),
                radial-gradient(circle at 92% 18%, rgba(186, 230, 253, 0.5) 0%, transparent 32%),
                radial-gradient(circle at 88% 88%, rgba(254, 202, 202, 0.45) 0%, transparent 35%);
            background-attachment: fixed;
            min-height: 100vh;
        }

        .custom-card {
            background: rgba(255, 255, 255, 0.88);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.7);
            box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.04), 0 8px 10px -6px rgba(0, 0, 0, 0.02);
        }

        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-track {
            background: rgba(0,0,0,0.03);
            border-radius: 8px;
        }
        ::-webkit-scrollbar-thumb {
            background: rgba(0,0,0,0.18);
            border-radius: 8px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: rgba(0,0,0,0.28);
        }

        .slot-rolling {
            animation: pulse-rolling 0.12s infinite alternate ease-in-out;
        }

        @keyframes pulse-rolling {
            0% { opacity: 0.65; transform: translateY(-1.5px); }
            100% { opacity: 1; transform: translateY(1.5px); }
        }
    </style>
</head>
<body class="text-gray-700 pb-12 antialiased selection:bg-amber-200 selection:text-amber-900">

    <!-- Toast Notification -->
    <div id="toast" class="fixed top-5 right-5 z-50 transform translate-y-[-100px] opacity-0 transition-all duration-300 bg-gray-900/90 backdrop-blur text-white px-5 py-3 rounded-2xl shadow-2xl flex items-center gap-3 border border-gray-700/50">
        <i id="toast-icon" class="fas fa-info-circle text-amber-400 text-lg"></i>
        <span id="toast-msg" class="text-xs sm:text-sm font-medium">通知訊息</span>
    </div>

    <div class="max-w-6xl mx-auto px-4 sm:px-6 lg:px-8 pt-8">
        
        <!-- Header Section -->
        <header class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-7 gap-4">
            <div>
                <div class="flex items-center gap-3">
                    <span class="text-3xl sm:text-4xl">🎉</span>
                    <h1 class="text-2xl sm:text-3xl font-black text-gray-800 tracking-tight">課堂抽籤</h1>
                </div>
                <p class="text-xs sm:text-sm text-gray-500 mt-1.5 font-normal tracking-wide">
                    本地端使用，不需上網，名單保留在自己的電腦
                </p>
            </div>
            
            <div class="bg-gradient-to-r from-indigo-600 via-purple-600 to-pink-500 text-white px-5 py-2.5 rounded-full shadow-md hover:shadow-lg transition-all flex items-center gap-2 font-semibold text-xs sm:text-sm tracking-wide">
                <span>✨ Classroom Lucky Draw</span>
            </div>
        </header>

        <div class="grid grid-cols-1 lg:grid-cols-12 gap-7 items-start">

            <!-- LEFT COLUMN: Roster Setup & Config -->
            <div class="lg:col-span-5 flex flex-col gap-6">

                <div class="custom-card rounded-3xl p-6 transition-all">
                    <h2 class="text-base font-bold text-gray-800 mb-4 flex items-center gap-2">
                        <span class="text-amber-500">📁</span> 1. 載入班級名單
                    </h2>

                    <!-- Upload Box -->
                    <div class="border-2 border-dashed border-sky-300 bg-sky-50/40 hover:bg-sky-50/80 transition-colors rounded-2xl p-5 text-center relative group" id="drop-zone">
                        <input type="file" id="file-input" multiple accept=".csv,.txt" class="absolute inset-0 w-full h-full opacity-0 cursor-pointer z-10">
                        <p class="text-xs font-bold text-gray-700 mb-1">選擇「班級名單」資料夾或檔案</p>
                        <p class="text-[11px] text-gray-400 mb-3 leading-snug">每個科目發放 1 個 CSV / TXT 檔，檔名會自動成為科目名稱。</p>
                        <button type="button" class="bg-sky-500 hover:bg-sky-600 active:bg-sky-700 text-white font-semibold text-xs px-5 py-2.5 rounded-xl shadow-sm transition-all pointer-events-none group-hover:scale-[1.02]">
                            連結班級名單資料夾
                        </button>
                    </div>

                    <!-- Single Temporary Input Button -->
                    <button id="btn-manual-input" onclick="toggleManualModal(true)" class="w-full mt-3 bg-gray-100/90 hover:bg-gray-200/90 active:bg-gray-300/80 text-gray-700 font-medium text-xs py-2.5 rounded-xl transition-all flex items-center justify-center gap-1.5 border border-gray-200/80">
                        <span class="font-bold text-gray-500">+</span> 臨時載入單一名單
                    </button>

                    <!-- Dropdown Select -->
                    <div class="mt-5">
                        <label for="class-select" class="block text-xs font-bold text-gray-600 mb-2">科目 / 班級</label>
                        <select id="class-select" onchange="handleClassChange()" class="w-full bg-white border border-gray-200/90 rounded-xl px-3.5 py-2.5 text-xs text-gray-800 font-medium shadow-sm focus:outline-none focus:ring-2 focus:ring-sky-400 transition-all cursor-pointer">
                            <option value="">-- 請先載入或選擇名單 --</option>
                        </select>
                    </div>

                    <!-- Loaded Info Box -->
                    <div id="loaded-info-box" class="mt-4 bg-amber-50/90 border border-amber-200/70 rounded-xl p-3.5 text-xs text-amber-950 leading-relaxed hidden">
                        <div class="font-bold mb-1">已載入 <span id="loaded-count-num">0</span> 個科目：</div>
                        <ul id="loaded-subjects-list" class="space-y-1 text-amber-900 text-[11px]">
                            <!-- Injected by JavaScript -->
                        </ul>
                    </div>
                </div>

                <div class="custom-card rounded-3xl p-6">
                    <h2 class="text-base font-bold text-gray-800 mb-4 flex items-center gap-2">
                        <span class="text-amber-500">🎲</span> 2. 同一次抽籤是否允許重複？
                    </h2>

                    <div class="grid grid-cols-2 gap-3">
                        <!-- Option 1: No Repeat -->
                        <button type="button" id="rule-norepeat-btn" onclick="setRepeatRule(false)" class="p-3.5 rounded-2xl border-2 text-left transition-all relative overflow-hidden bg-white border-amber-400 shadow-sm">
                            <div class="flex items-center gap-1.5 font-bold text-xs text-gray-800 mb-1">
                                <span>🎭</span> 不重複
                            </div>
                            <p class="text-[10px] text-gray-500 leading-tight">先前已抽到者不再進入抽籤</p>
                        </button>

                        <!-- Option 2: Allow Repeat -->
                        <button type="button" id="rule-repeat-btn" onclick="setRepeatRule(true)" class="p-3.5 rounded-2xl border-2 text-left transition-all relative overflow-hidden bg-white border-gray-200 hover:border-gray-300 opacity-75">
                            <div class="flex items-center gap-1.5 font-bold text-xs text-gray-800 mb-1">
                                <span>🍀</span> 可以重複
                            </div>
                            <p class="text-[10px] text-gray-500 leading-tight">可再次抽中</p>
                        </button>
                    </div>
                </div>

                <div class="grid grid-cols-3 gap-3">
                    <div class="custom-card rounded-2xl p-3.5 text-center">
                        <div id="stat-total" class="text-xl sm:text-2xl font-black text-gray-800">0</div>
                        <div class="text-[11px] text-gray-400 font-semibold mt-0.5">名單人數</div>
                    </div>
                    <div class="custom-card rounded-2xl p-3.5 text-center">
                        <div id="stat-remaining" class="text-xl sm:text-2xl font-black text-amber-500">0</div>
                        <div class="text-[11px] text-gray-400 font-semibold mt-0.5">不重複池剩餘</div>
                    </div>
                    <div class="custom-card rounded-2xl p-3.5 text-center">
                        <div id="stat-drawn" class="text-xl sm:text-2xl font-black text-gray-800">0</div>
                        <div class="text-[11px] text-gray-400 font-semibold mt-0.5">已抽次數</div>
                    </div>
                </div>

            </div>

            <!-- RIGHT COLUMN: Draw Display & Controls & History -->
            <div class="lg:col-span-7 flex flex-col gap-6">

                <div class="custom-card rounded-3xl p-6 sm:p-8 flex flex-col justify-between min-h-[360px]">
                    <div>
                        <div class="flex justify-between items-center mb-4">
                            <span class="text-xs font-extrabold tracking-widest text-gray-400 uppercase">TODAY'S LUCKY PICK</span>
                            <div class="flex items-center gap-3">
                                <label class="text-[11px] text-gray-400 hover:text-gray-600 flex items-center gap-1.5 cursor-pointer">
                                    <input type="checkbox" id="mask-toggle" checked onchange="toggleMaskNames()" class="rounded text-amber-500 focus:ring-amber-400">
                                    <span>遮蔽部分名字 (○)</span>
                                </label>
                                <button onclick="resetPool()" title="重置抽籤池" class="text-xs text-gray-400 hover:text-rose-500 transition-colors flex items-center gap-1 font-medium">
                                    <i class="fas fa-undo-alt text-[10px]"></i> 重置剩餘池
                                </button>
                            </div>
                        </div>

                        <!-- Result Display Box -->
                        <div id="display-area" class="py-6 flex flex-col justify-center items-start min-h-[170px]">
                            <div class="w-full text-center text-gray-300 py-8 font-semibold text-lg">
                                點擊下方「開始抽籤」進行抽籤
                            </div>
                        </div>
                    </div>

                    <!-- Action Control Bar -->
                    <div>
                        <div class="pt-5 border-t border-gray-100 flex flex-col sm:flex-row items-center gap-3">
                            <button id="draw-btn" onclick="startDraw()" class="w-full sm:flex-1 bg-gradient-to-r from-orange-500 to-amber-500 hover:from-orange-600 hover:to-amber-600 text-white font-black text-base py-3.5 px-6 rounded-2xl shadow-lg shadow-orange-500/20 active:scale-[0.99] transition-all flex items-center justify-center gap-2">
                                <span>🎲</span> 3. 開始抽籤
                            </button>

                            <div class="flex items-center gap-2 bg-gray-100/90 rounded-2xl p-1.5 w-full sm:w-auto justify-between sm:justify-start border border-gray-200/50">
                                <span class="text-xs text-gray-500 font-semibold pl-2">一次抽出位數:</span>
                                <input type="number" id="draw-count-input" min="1" max="60" value="2" class="w-16 bg-white border border-gray-200 rounded-xl px-2 py-1.5 text-center text-sm font-bold text-gray-800 focus:outline-none focus:ring-2 focus:ring-amber-400">
                            </div>
                        </div>
                        <div class="text-[11px] text-gray-400 text-center sm:text-right mt-2 font-medium">
                            右側數字可設定一次抽出幾位 ( 1 - 60 )
                        </div>
                    </div>
                </div>

                <div class="custom-card rounded-3xl p-6">
                    <div class="flex justify-between items-center mb-4">
                        <h3 class="text-sm font-bold text-gray-800 flex items-center gap-2">
                            <span>📜</span> 抽籤紀錄
                        </h3>
                        <div class="flex items-center gap-2">
                            <button onclick="clearHistory()" class="text-xs text-gray-400 hover:text-rose-500 transition-colors px-2 py-1 rounded-lg hover:bg-gray-100">
                                清除紀錄
                            </button>
                            <button onclick="copyHistory()" class="bg-gray-100/90 hover:bg-gray-200/90 text-gray-700 text-xs font-bold px-3 py-1.5 rounded-xl transition-all flex items-center gap-1.5 border border-gray-200/60">
                                <i class="far fa-copy text-xs"></i> 複製紀錄
                            </button>
                        </div>
                    </div>

                    <!-- History List -->
                    <div id="history-list" class="max-h-[220px] overflow-y-auto space-y-2.5 pr-1">
                        <div class="text-center py-6 text-xs text-gray-300">尚無抽籤紀錄</div>
                    </div>
                </div>

            </div>
        </div>
    </div>

    <div id="manual-modal" class="fixed inset-0 bg-black/40 backdrop-blur-sm z-50 flex items-center justify-center p-4 opacity-0 pointer-events-none transition-all duration-300">
        <div class="bg-white rounded-3xl p-6 max-w-md w-full shadow-2xl transform scale-95 transition-all duration-300" id="modal-card">
            <h3 class="text-base font-bold text-gray-800 mb-1">臨時載入單一名單</h3>
            <p class="text-xs text-gray-500 mb-4">請輸入科目/班級名稱，並貼上學員名單（可直接從 Excel 或文字檔複製貼上）。</p>
            
            <div class="space-y-3">
                <div>
                    <label for="manual-subject-name" class="block text-xs font-bold text-gray-600 mb-1">科目/班級名稱</label>
                    <input type="text" id="manual-subject-name" placeholder="例如：日語會話入門" class="w-full bg-gray-50 border border-gray-200 rounded-xl px-3.5 py-2 text-xs focus:outline-none focus:ring-2 focus:ring-sky-400 font-medium">
                </div>
                <div>
                    <label for="manual-names-text" class="block text-xs font-bold text-gray-600 mb-1">名單 (每行姓名隔開，可包含學號)</label>
                    <textarea id="manual-names-text" rows="7" placeholder="陳宥彤&#10;涂巧妤&#10;劉郁欣&#10;賴冠宏&#10;張寶鐿" class="w-full bg-gray-50 border border-gray-200 rounded-xl p-3 text-xs focus:outline-none focus:ring-2 focus:ring-sky-400 font-mono leading-relaxed"></textarea>
                </div>
            </div>

            <div class="flex justify-end gap-2 mt-5">
                <button onclick="toggleManualModal(false)" class="px-4 py-2 rounded-xl text-xs font-semibold text-gray-500 hover:bg-gray-100 transition-colors">取消</button>
                <button onclick="submitManualList()" class="px-4 py-2 rounded-xl text-xs font-bold bg-sky-600 hover:bg-sky-700 text-white shadow-md transition-all">確認載入</button>
            </div>
        </div>
    </div>

    <script>
        // --- Application State ---
        let rosters = {}; 
        let currentSubject = "";
        let remainingPool = [];
        let allowRepeat = false;
        let drawHistory = [];
        let isDrawing = false;
        let maskNames = true;

        // Default roster matching the user's uploaded class list image exactly
        const defaultSubjectName = "CSV115-1學期-週六中日口譯入門-20260926";
        const defaultRosterList = [
            "陳宥彤", "涂巧妤", "劉郁欣", "賴冠宏", "張寶鐿", "許乃文", "吳承霖", "胡睿麒", 
            "邱怡恩", "侯婷珺", "石永濬", "邱筱菁", "夏珍萍", "陳依欣", "賴姿婷", "王俊貽", 
            "邱月華", "沈惠卿", "林駿毅", "黃慈吟", "許沛璇", "陳慈玟", "傅婕茹", "林詩穎", 
            "張雅紅", "許瑋玲", "蔡明峻", "羅杏姿", "李典隆", "唐宥慈", "黃仲營", "周辰威", "陳松伶"
        ];

        // --- DOM Elements ---
        const classSelect = document.getElementById('class-select');
        const fileInput = document.getElementById('file-input');
        const dropZone = document.getElementById('drop-zone');
        const displayArea = document.getElementById('display-area');
        const historyList = document.getElementById('history-list');
        const loadedInfoBox = document.getElementById('loaded-info-box');
        const loadedCountNum = document.getElementById('loaded-count-num');
        const loadedSubjectsList = document.getElementById('loaded-subjects-list');
        
        // Stat Counters
        const statTotal = document.getElementById('stat-total');
        const statRemaining = document.getElementById('stat-remaining');
        const statDrawn = document.getElementById('stat-drawn');

        window.addEventListener('DOMContentLoaded', () => {
            loadFromLocalStorage();
            setupDragAndDrop();
            
            // Seed default data if none exists
            if (Object.keys(rosters).length === 0) {
                rosters[defaultSubjectName] = defaultRosterList;
                saveToLocalStorage();
            }

            updateClassDropdown();
            renderHistory();
        });

        // Toast Helper
        function showToast(message, iconClass = "fas fa-info-circle text-amber-400") {
            const toast = document.getElementById('toast');
            const toastMsg = document.getElementById('toast-msg');
            const toastIcon = document.getElementById('toast-icon');

            toastMsg.innerText = message;
            toastIcon.className = `${iconClass} text-base sm:text-lg`;

            toast.classList.remove('translate-y-[-100px]', 'opacity-0');
            toast.classList.add('translate-y-0', 'opacity-100');

            setTimeout(() => {
                toast.classList.remove('translate-y-0', 'opacity-100');
                toast.classList.add('translate-y-[-100px]', 'opacity-0');
            }, 2800);
        }

        // --- File Upload & Drag-and-Drop ---
        function setupDragAndDrop() {
            fileInput.addEventListener('change', (e) => {
                handleFiles(e.target.files);
            });

            ['dragenter', 'dragover'].forEach(eventName => {
                dropZone.addEventListener(eventName, (e) => {
                    e.preventDefault();
                    dropZone.classList.add('bg-sky-100/70', 'border-sky-500');
                }, false);
            });

            ['dragleave', 'drop'].forEach(eventName => {
                dropZone.addEventListener(eventName, (e) => {
                    e.preventDefault();
                    dropZone.classList.remove('bg-sky-100/70', 'border-sky-500');
                }, false);
            });

            dropZone.addEventListener('drop', (e) => {
                const dt = e.dataTransfer;
                const files = dt.files;
                handleFiles(files);
            });
        }

        function handleFiles(files) {
            if (!files || files.length === 0) return;

            let loadedCount = 0;
            const fileArray = Array.from(files);
            let lastSubjectName = "";

            fileArray.forEach(file => {
                const reader = new FileReader();
                reader.onload = function(e) {
                    const content = e.target.result;
                    const subjectName = file.name.replace(/\.[^/.]+$/, ""); // Strip file extension
                    const names = parseFileContent(content);

                    if (names.length > 0) {
                        rosters[subjectName] = names;
                        lastSubjectName = subjectName;
                        loadedCount++;
                    }

                    if (loadedCount === fileArray.length) {
                        saveToLocalStorage();
                        updateClassDropdown(lastSubjectName);
                        showToast(`已成功匯入 ${loadedCount} 個班級名單！`, "fas fa-check-circle text-emerald-400");
                    }
                };
                reader.readAsText(file, 'UTF-8');
            });
        }

        function parseFileContent(content) {
            // Intelligent parsing: extracts names even from tabular input like student lists
            return content
                .split(/\r?\n/)
                .map(line => line.trim())
                .filter(line => line.length > 0 && !line.startsWith("#"))
                .map(line => {
                    const parts = line.split(/[\t,;]+/).map(p => p.trim()).filter(Boolean);
                    if (parts.length === 0) return "";
                    // Search for 2-4 character Chinese name column if tab-separated
                    for (let p of parts) {
                        if (/^[\u4e00-\u9fa5]{2,4}$/.test(p) && p !== "姓名" && p !== "性別") {
                            return p;
                        }
                    }
                    return parts[parts.length - 1]; // Fallback
                })
                .filter(name => name.length >= 2 && !/^(姓名|性別|學號|備註|序號)$/.test(name));
        }

        // --- Manual Input Modal ---
        function toggleManualModal(show) {
            const modal = document.getElementById('manual-modal');
            const card = document.getElementById('modal-card');
            if (show) {
                modal.classList.remove('opacity-0', 'pointer-events-none');
                card.classList.remove('scale-95');
                card.classList.add('scale-100');
            } else {
                modal.classList.add('opacity-0', 'pointer-events-none');
                card.classList.remove('scale-100');
                card.classList.add('scale-95');
            }
        }

        function submitManualList() {
            const nameInput = document.getElementById('manual-subject-name').value.trim();
            const textInput = document.getElementById('manual-names-text').value;

            if (!nameInput) {
                showToast("請輸入科目/班級名稱", "fas fa-exclamation-triangle text-amber-400");
                return;
            }

            const names = parseFileContent(textInput);
            if (names.length === 0) {
                showToast("請輸入至少一位學員名字", "fas fa-exclamation-triangle text-amber-400");
                return;
            }

            rosters[nameInput] = names;
            saveToLocalStorage();
            updateClassDropdown(nameInput);
            toggleManualModal(false);

            document.getElementById('manual-subject-name').value = '';
            document.getElementById('manual-names-text').value = '';

            showToast(`已成功新增：${nameInput} (${names.length} 人)`, "fas fa-check-circle text-emerald-400");
        }

        function updateClassDropdown(selectTargetKey = null) {
            classSelect.innerHTML = '';

            const keys = Object.keys(rosters);
            if (keys.length === 0) {
                classSelect.innerHTML = '<option value="">-- 請先載入或選擇名單 --</option>';
                loadedInfoBox.classList.add('hidden');
                currentSubject = "";
                resetState();
                return;
            }

            keys.forEach(key => {
                const option = document.createElement('option');
                option.value = key;
                option.textContent = `${key} (${rosters[key].length}人)`;
                classSelect.appendChild(option);
            });

            if (selectTargetKey && rosters[selectTargetKey]) {
                classSelect.value = selectTargetKey;
            } else if (!currentSubject || !rosters[currentSubject]) {
                classSelect.value = keys[0];
            } else {
                classSelect.value = currentSubject;
            }

            currentSubject = classSelect.value;

            // Update Section 1 Info Box
            loadedCountNum.innerText = keys.length;
            loadedSubjectsList.innerHTML = keys.map(k => `<li>• ${k} (${rosters[k].length} 人)</li>`).join('');
            loadedInfoBox.classList.remove('hidden');

            resetPool();
        }

        function handleClassChange() {
            currentSubject = classSelect.value;
            resetPool();
            saveToLocalStorage();
        }

        function setRepeatRule(allow) {
            allowRepeat = allow;
            const noRepeatBtn = document.getElementById('rule-norepeat-btn');
            const repeatBtn = document.getElementById('rule-repeat-btn');

            if (!allowRepeat) {
                noRepeatBtn.className = "p-3.5 rounded-2xl border-2 text-left transition-all relative overflow-hidden bg-white border-amber-400 shadow-sm opacity-100";
                repeatBtn.className = "p-3.5 rounded-2xl border-2 text-left transition-all relative overflow-hidden bg-white border-gray-200 hover:border-gray-300 opacity-75";
            } else {
                repeatBtn.className = "p-3.5 rounded-2xl border-2 text-left transition-all relative overflow-hidden bg-white border-amber-400 shadow-sm opacity-100";
                noRepeatBtn.className = "p-3.5 rounded-2xl border-2 text-left transition-all relative overflow-hidden bg-white border-gray-200 hover:border-gray-300 opacity-75";
            }
        }

        function resetPool() {
            if (!currentSubject || !rosters[currentSubject]) {
                remainingPool = [];
            } else {
                remainingPool = [...rosters[currentSubject]];
            }
            updateStats();
        }

        function resetState() {
            displayArea.innerHTML = `<div class="w-full text-center text-gray-300 py-8 font-semibold text-lg">點擊下方「開始抽籤」進行抽籤</div>`;
            updateStats();
        }

        function updateStats() {
            const total = (currentSubject && rosters[currentSubject]) ? rosters[currentSubject].length : 0;
            statTotal.innerText = total;
            statRemaining.innerText = remainingPool.length;
            statDrawn.innerText = drawHistory.filter(h => h.subject === currentSubject).length;
        }

        function toggleMaskNames() {
            maskNames = document.getElementById('mask-toggle').checked;
            renderHistory();
        }

        // Mask Name Formatting (林毅 -> 林○毅, 張學友 -> 張○友)
        function formatName(name) {
            if (!maskNames || !name) return name;
            if (name.length <= 1) return name;
            if (name.length === 2) return name[0] + '○' + name[1];
            return name[0] + '○' + name.slice(2);
        }

        // --- Core Draw Engine ---
        function startDraw() {
            if (isDrawing) return;
            if (!currentSubject || !rosters[currentSubject] || rosters[currentSubject].length === 0) {
                showToast("請先選擇或載入班級名單！", "fas fa-exclamation-circle text-rose-400");
                return;
            }

            const countInput = document.getElementById('draw-count-input');
            let drawCount = parseInt(countInput.value) || 1;
            drawCount = Math.max(1, Math.min(60, drawCount));

            if (!allowRepeat && remainingPool.length === 0) {
                showToast("不重複池名單已全數抽完！請點選上方重置剩餘池。", "fas fa-info-circle text-amber-400");
                return;
            }

            if (!allowRepeat && drawCount > remainingPool.length) {
                drawCount = remainingPool.length;
                showToast(`剩餘人數不足，本次自動調整抽出 ${drawCount} 位`, "fas fa-info-circle text-sky-400");
            }

            isDrawing = true;
            const drawBtn = document.getElementById('draw-btn');
            drawBtn.disabled = true;
            drawBtn.classList.add('opacity-70', 'cursor-not-allowed');

            let rolls = 0;
            const maxRolls = 18;
            const currentPool = allowRepeat ? rosters[currentSubject] : remainingPool;

            const rollInterval = setInterval(() => {
                rolls++;
                const tempPicked = [];
                for (let i = 0; i < drawCount; i++) {
                    const randomIdx = Math.floor(Math.random() * currentPool.length);
                    tempPicked.push(currentPool[randomIdx]);
                }
                renderWinnersDisplay(tempPicked, true);

                if (rolls >= maxRolls) {
                    clearInterval(rollInterval);
                    finalizeDraw(drawCount);
                }
            }, 70);
        }

        function finalizeDraw(count) {
            const winners = [];
            const sourcePool = allowRepeat ? rosters[currentSubject] : remainingPool;

            for (let i = 0; i < count; i++) {
                if (sourcePool.length === 0) break;
                const randomIdx = Math.floor(Math.random() * (allowRepeat ? sourcePool.length : remainingPool.length));
                
                if (allowRepeat) {
                    winners.push(sourcePool[randomIdx]);
                } else {
                    const picked = remainingPool.splice(randomIdx, 1)[0];
                    winners.push(picked);
                }
            }

            renderWinnersDisplay(winners, false);

            const now = new Date();
            const year = now.getFullYear() - 1911; // Taiwan Year Format like image 115/10/05
            const month = String(now.getMonth() + 1).padStart(2, '0');
            const day = String(now.getDate()).padStart(2, '0');
            const timeStr = now.toLocaleTimeString('zh-TW', { hour12: true, hour: '2-digit', minute: '2-digit', second: '2-digit' });
            
            const fullTimestamp = `${year}/${month}/${day} ${timeStr}`;

            drawHistory.unshift({
                id: Date.now(),
                subject: currentSubject,
                winners: winners,
                timestamp: fullTimestamp,
                isRepeat: allowRepeat
            });

            // Confetti Celebration Effect
            confetti({
                particleCount: 65,
                spread: 70,
                origin: { y: 0.65 }
            });

            updateStats();
            renderHistory();
            saveToLocalStorage();

            isDrawing = false;
            const drawBtn = document.getElementById('draw-btn');
            drawBtn.disabled = false;
            drawBtn.classList.remove('opacity-70', 'cursor-not-allowed');
        }

        function renderWinnersDisplay(winners, isRolling = false) {
            let html = `<div class="w-full space-y-4 ${isRolling ? 'slot-rolling' : ''}">`;
            
            winners.forEach((name, index) => {
                const formatted = formatName(name);
                html += `
                    <div class="flex items-center gap-4 text-3xl sm:text-4xl font-black text-gray-800 tracking-wider">
                        <span class="text-gray-400 font-bold text-xl sm:text-2xl w-8">${index + 1}.</span>
                        <span class="${isRolling ? 'text-gray-400' : 'text-gray-900'}">${formatted}</span>
                    </div>
                `;
            });

            html += `</div>`;
            displayArea.innerHTML = html;
        }

        function renderHistory() {
            if (drawHistory.length === 0) {
                historyList.innerHTML = `<div class="text-center py-6 text-xs text-gray-300">尚無抽籤紀錄</div>`;
                return;
            }

            historyList.innerHTML = drawHistory.map(item => {
                const winnersFormatted = item.winners.map(w => formatName(w)).join('、');
                const repeatBadge = item.isRepeat ? "【可重複】" : "【不重複】";
                return `
                    <div class="bg-gray-50/90 hover:bg-gray-100/90 border border-gray-100 rounded-2xl p-3 text-xs flex flex-col sm:flex-row justify-between items-start sm:items-center gap-2 transition-all">
                        <div class="font-bold text-gray-800 tracking-wide">
                            1. ${winnersFormatted}
                        </div>
                        <div class="text-[11px] text-gray-400 flex items-center gap-1.5 self-end sm:self-auto font-medium">
                            <span class="bg-amber-100/90 text-amber-800 text-[10px] px-2 py-0.5 rounded-md font-semibold">${repeatBadge}</span>
                            <span>✨ 彩球洗牌 ${item.timestamp}</span>
                        </div>
                    </div>
                `;
            }).join('');
        }

        function clearHistory() {
            if (drawHistory.length === 0) return;
            drawHistory = [];
            renderHistory();
            updateStats();
            saveToLocalStorage();
            showToast("已清除抽籤紀錄", "fas fa-trash text-gray-400");
        }

        function copyHistory() {
            if (drawHistory.length === 0) {
                showToast("尚無紀錄可供複製", "fas fa-info-circle text-amber-400");
                return;
            }

            const textLines = drawHistory.map((item) => {
                const winnersStr = item.winners.map(w => formatName(w)).join('、');
                return `[${item.timestamp}] ${item.subject} - 抽中: ${winnersStr}`;
            });

            const fullText = textLines.join('\n');

            const textarea = document.createElement('textarea');
            textarea.value = fullText;
            document.body.appendChild(textarea);
            textarea.select();
            try {
                document.execCommand('copy');
                showToast("已成功複製抽籤紀錄至剪貼簿！", "fas fa-check-circle text-emerald-400");
            } catch (err) {
                showToast("複製失敗，請手動選取", "fas fa-times-circle text-rose-400");
            }
            document.body.removeChild(textarea);
        }

        // --- LocalStorage Persistence ---
        function saveToLocalStorage() {
            const data = {
                rosters: rosters,
                currentSubject: currentSubject,
                drawHistory: drawHistory
            };
            localStorage.setItem('classroom_lucky_draw_data', JSON.stringify(data));
        }

        function loadFromLocalStorage() {
            const saved = localStorage.getItem('classroom_lucky_draw_data');
            if (saved) {
                try {
                    const parsed = JSON.parse(saved);
                    rosters = parsed.rosters || {};
                    currentSubject = parsed.currentSubject || "";
                    drawHistory = parsed.drawHistory || [];
                } catch (e) {
                    console.error("Failed to load LocalStorage data", e);
                }
            }
        }
    </script>
</body>
</html>
