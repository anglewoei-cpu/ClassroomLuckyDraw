<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>課堂抽籤 - Classroom Lucky Draw</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Canvas Confetti -->
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;600;700;900&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Noto Sans TC', sans-serif;
            background-color: #EBF0EC;
            background-image: 
                radial-gradient(circle at 10% 10%, rgba(253, 224, 71, 0.35) 0%, transparent 25%),
                radial-gradient(circle at 95% 20%, rgba(147, 197, 253, 0.4) 0%, transparent 30%),
                radial-gradient(circle at 90% 85%, rgba(252, 165, 165, 0.4) 0%, transparent 30%);
            background-attachment: fixed;
            min-height: 100vh;
        }

        .custom-card {
            background: rgba(255, 255, 255, 0.85);
            backdrop-filter: blur(8px);
            border: 1px solid rgba(255, 255, 255, 0.6);
            box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.05), 0 8px 10px -6px rgba(0, 0, 0, 0.03);
        }

        /* Custom Scrollbar */
        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-track {
            background: rgba(0,0,0,0.03);
            border-radius: 8px;
        }
        ::-webkit-scrollbar-thumb {
            background: rgba(0,0,0,0.15);
            border-radius: 8px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: rgba(0,0,0,0.25);
        }

        .slot-rolling {
            animation: pulse-light 0.15s infinite alternate;
        }

        @keyframes pulse-light {
            0% { opacity: 0.7; transform: translateY(-1px); }
            100% { opacity: 1; transform: translateY(1px); }
        }
    </style>
</head>
<body class="text-gray-700 pb-12 antialiased">

    <!-- Toast Notification -->
    <div id="toast" class="fixed top-5 right-5 z-50 transform translate-y-[-100px] opacity-0 transition-all duration-300 bg-gray-800 text-white px-5 py-3 rounded-xl shadow-xl flex items-center gap-3">
        <i id="toast-icon" class="fas fa-info-circle text-amber-400 text-lg"></i>
        <span id="toast-msg" class="text-sm font-medium">通知訊息</span>
    </div>

    <!-- Main Container -->
    <div class="max-w-6xl mx-mx-auto px-4 sm:px-6 lg:px-8 pt-8">
        
        <!-- Header -->
        <header class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-8 gap-4">
            <div>
                <div class="flex items-center gap-3">
                    <div class="text-3xl sm:text-4xl">🎉</div>
                    <h1 class="text-2xl sm:text-3xl font-black text-gray-800 tracking-tight">課堂抽籤</h1>
                </div>
                <p class="text-xs sm:text-sm text-gray-500 mt-1.5 font-normal tracking-wide">
                    本地端使用，不需上網，名單保留在自己的電腦
                </p>
            </div>
            
            <div class="bg-gradient-to-r from-indigo-500 to-purple-600 text-white px-5 py-2.5 rounded-full shadow-md hover:shadow-lg transition-all flex items-center gap-2 font-medium text-sm">
                <span>✨ Classroom Lucky Draw</span>
            </div>
        </header>

        <!-- Main Grid -->
        <div class="grid grid-cols-1 lg:grid-cols-12 gap-8 items-start">

            <!-- LEFT COLUMN: Roster Setup & Config -->
            <div class="lg:col-span-5 flex flex-col gap-6">

                <!-- Section 1: Load Roster Card -->
                <div class="custom-card rounded-3xl p-6 transition-all">
                    <h2 class="text-base font-bold text-gray-800 mb-4 flex items-center gap-2">
                        <span class="text-amber-500">📁</span> 1. 載入班級名單
                    </h2>

                    <!-- Upload Box -->
                    <div class="border-2 border-dashed border-sky-300 bg-sky-50/50 hover:bg-sky-50 transition-colors rounded-2xl p-5 text-center cursor-pointer relative group" id="drop-zone">
                        <input type="file" id="file-input" multiple accept=".csv,.txt" class="absolute inset-0 w-full h-full opacity-0 cursor-pointer z-10">
                        <p class="text-xs font-semibold text-gray-600 mb-1">選擇「班級名單」資料夾或檔案</p>
                        <p class="text-[11px] text-gray-400 mb-3">每個科目發放 1 個 CSV / TXT 檔，檔名會自動成為科目名稱。</p>
                        <button type="button" class="bg-sky-600 hover:bg-sky-700 text-white font-medium text-xs px-5 py-2.5 rounded-xl shadow-sm transition-all pointer-events-none group-hover:scale-[1.02]">
                            連結班級名單資料夾 / 檔案
                        </button>
                    </div>

                    <!-- Single Temporary Input Button -->
                    <button id="btn-manual-input" onclick="toggleManualModal(true)" class="w-full mt-3 bg-gray-100/80 hover:bg-gray-200/80 text-gray-600 font-medium text-xs py-2.5 rounded-xl transition-all flex items-center justify-center gap-1.5 border border-gray-200">
                        <span>+</span> 臨時載入單一名單
                    </button>

                    <!-- Dropdown Select -->
                    <div class="mt-5">
                        <label class="block text-xs font-semibold text-gray-500 mb-2">科目 / 班級</label>
                        <select id="class-select" onchange="handleClassChange()" class="w-full bg-white border border-gray-200 rounded-xl px-3.5 py-2.5 text-xs text-gray-700 font-medium shadow-sm focus:outline-none focus:ring-2 focus:ring-sky-400 transition-all">
                            <option value="">-- 請先載入或選擇名單 --</option>
                        </select>
                    </div>

                    <!-- Loaded Info Box -->
                    <div id="loaded-info-box" class="mt-4 bg-amber-50/80 border border-amber-200/60 rounded-xl p-3.5 text-xs text-amber-900 leading-relaxed hidden">
                        <div class="font-medium mb-0.5">已載入 <span id="loaded-count-num">0</span> 個科目：</div>
                        <ul id="loaded-subjects-list" class="list-disc list-inside space-y-0.5 text-amber-800 text-[11px] opacity-90">
                            <!-- JS injected -->
                        </ul>
                    </div>
                </div>

                <!-- Section 2: Duplicate Rules Card -->
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
                        <button type="button" id="rule-repeat-btn" onclick="setRepeatRule(true)" class="p-3.5 rounded-2xl border-2 text-left transition-all relative overflow-hidden bg-white border-gray-200 hover:border-gray-300">
                            <div class="flex items-center gap-1.5 font-bold text-xs text-gray-800 mb-1">
                                <span>🍀</span> 可以重複
                            </div>
                            <p class="text-[10px] text-gray-500 leading-tight">可再次抽中</p>
                        </button>
                    </div>
                </div>

                <!-- Statistics Summary Cards -->
                <div class="grid grid-cols-3 gap-3">
                    <div class="custom-card rounded-2xl p-3.5 text-center">
                        <div id="stat-total" class="text-xl sm:text-2xl font-black text-gray-800">0</div>
                        <div class="text-[11px] text-gray-400 font-medium mt-0.5">名單人數</div>
                    </div>
                    <div class="custom-card rounded-2xl p-3.5 text-center">
                        <div id="stat-remaining" class="text-xl sm:text-2xl font-black text-amber-500">0</div>
                        <div class="text-[11px] text-gray-400 font-medium mt-0.5">不重複池剩餘</div>
                    </div>
                    <div class="custom-card rounded-2xl p-3.5 text-center">
                        <div id="stat-drawn" class="text-xl sm:text-2xl font-black text-gray-800">0</div>
                        <div class="text-[11px] text-gray-400 font-medium mt-0.5">已抽次數</div>
                    </div>
                </div>

            </div>

            <!-- RIGHT COLUMN: Draw Display & Controls & History -->
            <div class="lg:col-span-7 flex flex-col gap-6">

                <!-- Main Draw Canvas / Display Area -->
                <div class="custom-card rounded-3xl p-6 sm:p-8 flex flex-col justify-between min-h-[380px]">
                    <div>
                        <div class="flex justify-between items-center mb-6">
                            <span class="text-xs font-bold tracking-widest text-gray-400 uppercase">Today's Lucky Pick</span>
                            <button onclick="resetPool()" title="重置抽籤池" class="text-xs text-gray-400 hover:text-rose-500 transition-colors flex items-center gap-1">
                                <i class="fas fa-undo-alt"></i> 重置剩餘池
                            </button>
                        </div>

                        <!-- Result Display Box -->
                        <div id="display-area" class="py-6 flex flex-col justify-center items-start min-h-[180px]">
                            <div class="w-full text-center text-gray-300 py-10 font-medium text-lg">
                                點擊下方「開始抽籤」進行抽籤
                            </div>
                        </div>
                    </div>

                    <!-- Action Control Bar -->
                    <div class="pt-6 border-t border-gray-100 flex flex-col sm:flex-row items-center gap-3">
                        <button id="draw-btn" onclick="startDraw()" class="w-full sm:flex-1 bg-gradient-to-r from-orange-500 to-amber-500 hover:from-orange-600 hover:to-amber-600 text-white font-bold text-base py-3.5 px-6 rounded-2xl shadow-lg shadow-orange-500/20 active:scale-[0.99] transition-all flex items-center justify-center gap-2">
                            <span>🎲</span> 3. 開始抽籤
                        </button>

                        <div class="flex items-center gap-2 bg-gray-100/80 rounded-2xl p-1.5 w-full sm:w-auto justify-between sm:justify-start">
                            <span class="text-xs text-gray-500 font-medium pl-3">一次抽出位數:</span>
                            <input type="number" id="draw-count-input" min="1" max="60" value="2" class="w-16 bg-white border border-gray-200 rounded-xl px-2 py-1.5 text-center text-sm font-bold text-gray-800 focus:outline-none focus:ring-2 focus:ring-amber-400">
                        </div>
                    </div>
                    <div class="text-[11px] text-gray-400 text-center sm:text-right mt-2">
                        右側數字可設定一次抽出幾位 ( 1 - 60 )
                    </div>
                </div>

                <!-- History Section -->
                <div class="custom-card rounded-3xl p-6">
                    <div class="flex justify-between items-center mb-4">
                        <h3 class="text-sm font-bold text-gray-800 flex items-center gap-2">
                            <span>📜</span> 抽籤紀錄
                        </h3>
                        <div class="flex items-center gap-2">
                            <button onclick="clearHistory()" class="text-xs text-gray-400 hover:text-rose-500 transition-colors px-2 py-1 rounded-lg hover:bg-gray-100">
                                清除紀錄
                            </button>
                            <button onclick="copyHistory()" class="bg-gray-100 hover:bg-gray-200 text-gray-700 text-xs font-semibold px-3 py-1.5 rounded-xl transition-all flex items-center gap-1.5">
                                <i class="far fa-copy"></i> 複製紀錄
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

    <!-- Manual Input Modal -->
    <div id="manual-modal" class="fixed inset-0 bg-black/40 backdrop-blur-sm z-50 flex items-center justify-center p-4 opacity-0 pointer-events-none transition-all duration-300">
        <div class="bg-white rounded-3xl p-6 max-w-md w-full shadow-2xl transform scale-95 transition-all duration-300" id="modal-card">
            <h3 class="text-base font-bold text-gray-800 mb-2">臨時載入單一名單</h3>
            <p class="text-xs text-gray-500 mb-4">請輸入科目/班級名稱，並貼上學員名單（每行一位姓名）。</p>
            
            <div class="space-y-3">
                <div>
                    <label class="block text-xs font-semibold text-gray-600 mb-1">科目/班級名稱</label>
                    <input type="text" id="manual-subject-name" placeholder="例如：日語會話入門" class="w-full bg-gray-50 border border-gray-200 rounded-xl px-3 py-2 text-xs focus:outline-none focus:ring-2 focus:ring-sky-400">
                </div>
                <div>
                    <label class="block text-xs font-semibold text-gray-600 mb-1">名單 (每行名字隔開)</label>
                    <textarea id="manual-names-text" rows="6" placeholder="林毅&#10;張鐘&#10;陳小明&#10;李美華" class="w-full bg-gray-50 border border-gray-200 rounded-xl p-3 text-xs focus:outline-none focus:ring-2 focus:ring-sky-400 font-mono leading-relaxed"></textarea>
                </div>
            </div>

            <div class="flex justify-end gap-2 mt-5">
                <button onclick="toggleManualModal(false)" class="px-4 py-2 rounded-xl text-xs font-medium text-gray-500 hover:bg-gray-100 transition-colors">取消</button>
                <button onclick="submitManualList()" class="px-4 py-2 rounded-xl text-xs font-medium bg-sky-600 hover:bg-sky-700 text-white shadow-md transition-all">確認載入</button>
            </div>
        </div>
    </div>

    <script>
        // --- State Management ---
        let rosters = {}; // { subjectName: [ "林毅", "張鐘", ... ] }
        let currentSubject = "";
        let remainingPool = [];
        let allowRepeat = false;
        let drawHistory = [];
        let isDrawing = false;

        // --- DOM Elements ---
        const classSelect = document.getElementById('class-select');
        const fileInput = document.getElementById('file-input');
        const dropZone = document.getElementById('drop-zone');
        const displayArea = document.getElementById('display-area');
        const historyList = document.getElementById('history-list');
        const loadedInfoBox = document.getElementById('loaded-info-box');
        const loadedCountNum = document.getElementById('loaded-count-num');
        const loadedSubjectsList = document.getElementById('loaded-subjects-list');
        
        // Stats
        const statTotal = document.getElementById('stat-total');
        const statRemaining = document.getElementById('stat-remaining');
        const statDrawn = document.getElementById('stat-drawn');

        window.addEventListener('DOMContentLoaded', () => {
            loadFromLocalStorage();
            setupDragAndDrop();
            
            // Set default example if empty
            if (Object.keys(rosters).length === 0) {
                rosters["CSV115-1學期-週六中日口譯入門-20260926"] = [
                    "林毅", "張鐘", "陳志強", "林雅婷", "黃怡君", "張家豪", "許智偉",
                    "蔡佩珊", "吳建宏", "鄭雅文", "郭家瑋", "曾怡婷", "洪健銘", "邱思婷"
                ];
                saveToLocalStorage();
                updateClassDropdown();
            }
        });

        // Toast Helper
        function showToast(message, iconClass = "fas fa-info-circle text-amber-400") {
            const toast = document.getElementById('toast');
            const toastMsg = document.getElementById('toast-msg');
            const toastIcon = document.getElementById('toast-icon');

            toastMsg.innerText = message;
            toastIcon.className = `${iconClass} text-lg`;

            toast.classList.remove('translate-y-[-100px]', 'opacity-0');
            toast.classList.add('translate-y-0', 'opacity-100');

            setTimeout(() => {
                toast.classList.remove('translate-y-0', 'opacity-100');
                toast.classList.add('translate-y-[-100px]', 'opacity-0');
            }, 3000);
        }

        // --- File Handling & Drag Drop ---
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

            fileArray.forEach(file => {
                const reader = new FileReader();
                reader.onload = function(e) {
                    const content = e.target.result;
                    const subjectName = file.name.replace(/\.[^/.]+$/, ""); // Strip extension
                    const names = parseFileContent(content);

                    if (names.length > 0) {
                        rosters[subjectName] = names;
                        loadedCount++;
                    }

                    if (loadedCount === fileArray.length) {
                        saveToLocalStorage();
                        updateClassDropdown(subjectName);
                        showToast(`已成功匯入 ${loadedCount} 個班級名單！`, "fas fa-check-circle text-emerald-400");
                    }
                };
                reader.readAsText(file, 'UTF-8');
            });
        }

        function parseFileContent(content) {
            // Split by line breaks, trim spaces, drop empty lines
            return content
                .split(/\r?\n/)
                .map(line => line.trim())
                .filter(line => line.length > 0 && !line.startsWith("#"));
        }

        // --- Manual Modal Input ---
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
            updateClassDropdown(nameInput);#	學號	姓名	性別	備註
1	3311442001	陳宥彤	女	 
2	3311442003	涂巧妤	女	 
3	3311442005	劉郁欣	女	 
4	3311442006	賴冠宏	男	 
5	3311442007	張寶鐿	女	 
6	3311442008	許乃文	女	 
7	3311442009	吳承霖	男	 
8	3311442010	胡睿麒	男	 
9	3311442011	邱怡恩	女	 
10	3311442012	侯婷珺	女	 
11	3311442013	石永濬	男	 
12	3311442015	邱筱菁	女	 
13	3311442018	夏珍萍	女	 
14	3311442019	陳依欣	女	 
15	3311442020	賴姿婷	女	 
16	3311442021	王俊貽	女	 
17	3311442022	邱月華	女	 
18	3311442023	沈惠卿	女	 
19	3311442026	林駿毅	男	 
20	3311442027	黃慈吟	女	 
21	3311442028	許沛璇	女	 
22	3311442029	陳慈玟	女	 
23	3311442035	傅婕茹	女	 
24	3311442036	林詩穎	女	 
25	3311442037	張雅紅	女	 
26	3311442042	許瑋玲	女	 
27	3311442043	蔡明峻	男	 
28	3311442045	羅杏姿	女	 
29	3311342020	李典隆	男	延
30	3311342021	唐宥慈	女	延
31	3311342027	黃仲營	男	延
32	3311342028	周辰威	男	延
33	3311342041	陳松伶	女	 
34	3311342043	林義雄	男	延
35	3311342046	湯凱勛	男	延
36	3311342047	湯竣頡	男	延
37	3311242056	廖思晴	女	 
38	3311242058	蔡宇揚	男	 
39	3311242060	謝素貞	女	延
40	3311142058	黃暉雲	男	延
41	3311142059	林懷哲	男	延
42	3311042025	李品璇	女	延
43	3310542048	黃俊吉	男	延
            toggleManualModal(false);

            // Clear inputs
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
                option.textContent = `${key} (${rosters[key].length} 人)`;
                classSelect.appendChild(option);
            });

            // Select priority
            if (selectTargetKey && rosters[selectTargetKey]) {
                classSelect.value = selectTargetKey;
            } else if (!currentSubject || !rosters[currentSubject]) {
                classSelect.value = keys[0];
            } else {
                classSelect.value = currentSubject;
            }

            currentSubject = classSelect.value;

            // Render Info Box
            loadedCountNum.innerText = keys.length;
            loadedSubjectsList.innerHTML = keys.map(k => `• ${k} (${rosters[k].length} 人)`).join('<br>');
            loadedInfoBox.classList.remove('hidden');

            resetPool();
        }

        function handleClassChange() {
            currentSubject = classSelect.value;
            resetPool();
        }

        function setRepeatRule(allow) {
            allowRepeat = allow;
            const noRepeatBtn = document.getElementById('rule-norepeat-btn');
            const repeatBtn = document.getElementById('rule-repeat-btn');

            if (!allowRepeat) {
                noRepeatBtn.className = "p-3.5 rounded-2xl border-2 text-left transition-all relative overflow-hidden bg-white border-amber-400 shadow-sm";
                repeatBtn.className = "p-3.5 rounded-2xl border-2 text-left transition-all relative overflow-hidden bg-white border-gray-200 hover:border-gray-300 opacity-80";
            } else {
                repeatBtn.className = "p-3.5 rounded-2xl border-2 text-left transition-all relative overflow-hidden bg-white border-amber-400 shadow-sm";
                noRepeatBtn.className = "p-3.5 rounded-2xl border-2 text-left transition-all relative overflow-hidden bg-white border-gray-200 hover:border-gray-300 opacity-80";
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
            displayArea.innerHTML = `<div class="w-full text-center text-gray-300 py-10 font-medium text-lg">點擊下方「開始抽籤」進行抽籤</div>`;
            updateStats();
        }

        function updateStats() {
            const total = (currentSubject && rosters[currentSubject]) ? rosters[currentSubject].length : 0;
            statTotal.innerText = total;
            statRemaining.innerText = remainingPool.length;
            statDrawn.innerText = drawHistory.filter(h => h.subject === currentSubject).length;
        }

        // Mask Name formatting (e.g., 林毅 -> 林○毅, 張學友 -> 張○友)
        function formatMaskedName(name) {
            if (!name) return '';
            if (name.length <= 1) return name;
            if (name.length === 2) {
                return name[0] + '○' + name[1];
            }
            // For names longer than 2 chars, replace inner chars with ○
            return name[0] + '○' + name.slice(2);
        }

        function startDraw() {
            if (isDrawing) return;
            if (!currentSubject || !rosters[currentSubject] || rosters[currentSubject].length === 0) {
                showToast("請先選擇或載入名單！", "fas fa-exclamation-circle text-rose-400");
                return;
            }

            const countInput = document.getElementById('draw-count-input');
            let drawCount = parseInt(countInput.value) || 1;
            drawCount = Math.max(1, Math.min(60, drawCount)); // Clamp 1 to 60

            if (!allowRepeat && remainingPool.length === 0) {
                showToast("不重複池名單已全數抽完！請重置池子或切換允許重複。", "fas fa-info-circle text-amber-400");
                return;
            }

            // Adjust drawCount if remainingPool is smaller than requested under no-repeat mode
            if (!allowRepeat && drawCount > remainingPool.length) {
                drawCount = remainingPool.length;
                showToast(`剩餘人數不足，本次自動調整抽出 ${drawCount} 位`, "fas fa-info-circle text-sky-400");
            }

            isDrawing = true;
            const drawBtn = document.getElementById('draw-btn');
            drawBtn.disabled = true;
            drawBtn.classList.add('opacity-75', 'cursor-not-allowed');

            // Sound or visual preparation
            let rolls = 0;
            const maxRolls = 18; // total animation frames
            const currentPool = allowRepeat ? rosters[currentSubject] : remainingPool;

            const rollInterval = setInterval(() => {
                rolls++;
                // Pick random dummy candidates
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
            }, 80);
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

            // Render Final Winners
            renderWinnersDisplay(winners, false);

            // Record History
            const timestamp = new Date().toLocaleTimeString('zh-TW', { hour12: true, hour: '2-digit', minute: '2-digit', second: '2-digit' });
            drawHistory.unshift({
                id: Date.now(),
                subject: currentSubject,
                winners: winners,
                timestamp: timestamp
            });

            // Trigger Confetti Celebration Effect
            confetti({
                particleCount: 50,
                spread: 60,
                origin: { y: 0.7 }
            });

            // Update UI State
            updateStats();
            renderHistory();
            saveToLocalStorage();

            isDrawing = false;
            const drawBtn = document.getElementById('draw-btn');
            drawBtn.disabled = false;
            drawBtn.classList.remove('opacity-75', 'cursor-not-allowed');
        }

        function renderWinnersDisplay(winners, isRolling = false) {
            let html = `<div class="w-full space-y-3 ${isRolling ? 'slot-rolling' : ''}">`;
            
            winners.forEach((name, index) => {
                const masked = formatMaskedName(name);
                html += `
                    <div class="flex items-center gap-4 text-2xl sm:text-3xl font-black text-gray-800 tracking-wide">
                        <span class="text-gray-400 font-bold text-lg sm:text-xl w-6">${index + 1}.</span>
                        <span class="${isRolling ? 'text-gray-500' : 'text-gray-900'}">${masked}</span>
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
                const formattedWinners = item.winners.map(w => formatMaskedName(w)).join('、');
                return `
                    <div class="bg-gray-50/80 hover:bg-gray-100/80 border border-gray-100 rounded-2xl p-3 text-xs flex flex-col sm:flex-row justify-between items-start sm:items-center gap-2 transition-all">
                        <div class="font-bold text-gray-800 tracking-wide">
                            1. ${formattedWinners}
                        </div>
                        <div class="text-[11px] text-gray-400 flex items-center gap-1.5 self-end sm:self-auto">
                            <span class="bg-amber-100 text-amber-800 text-[10px] px-2 py-0.5 rounded-md font-medium">【可重複】</span>
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

            const textLines = drawHistory.map((item, idx) => {
                const winnersStr = item.winners.map(w => formatMaskedName(w)).join('、');
                return `[${item.timestamp}] ${item.subject} - 抽中: ${winnersStr}`;
            });

            const fullText = textLines.join('\n');

            // ExecCommand Copy fallback for webview iFrames
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

        // --- LocalStorage Logic ---
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
                    updateClassDropdown();
                    renderHistory();
                } catch (e) {
                    console.error("Failed to parse LocalStorage data", e);
                }
            }
        }
    </script>
</body>
</html>
