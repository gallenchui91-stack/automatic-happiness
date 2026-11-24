<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta name="description" content="緊急手術：高階外科臨床模擬 — 互動式外科急救小遊戲，支援鍵盤與觸控操作，立即試玩！">
    <meta property="og:title" content="緊急手術：高階外科臨床模擬">
    <meta property="og:description" content="化身急救外科醫師，止血、清除積血、摘除彈片並完成顯微縫合，挑戰最高 S 級評級！">
    <meta property="og:type" content="website">
    <title>緊急手術：高階外科臨床模擬</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;700;900&family=Orbitron:wght@600;800;900&display=swap');

        * {
            box-sizing: border-box;
            user-select: none;
            -webkit-user-select: none;
            touch-action: none;
        }

        body {
            font-family: 'Noto Sans TC', sans-serif;
            background-color: #030811;
            color: #e2e8f0;
            overflow: hidden;
            margin: 0;
            padding: 0;
        }

        .font-hud {
            font-family: 'Orbitron', 'Noto Sans TC', sans-serif;
        }

        /* 醫療 HUD 玻璃擬態與光暈 */
        .hud-panel {
            background: rgba(8, 18, 32, 0.9);
            border: 1px solid rgba(34, 211, 238, 0.3);
            box-shadow: 0 4px 20px rgba(0, 0, 0, 0.6), inset 0 0 12px rgba(34, 211, 238, 0.08);
            backdrop-filter: blur(10px);
        }

        .hud-panel-danger {
            border-color: rgba(244, 63, 94, 0.85);
            box-shadow: 0 0 28px rgba(244, 63, 94, 0.45), inset 0 0 15px rgba(244, 63, 94, 0.25);
        }

        /* 超執刀專注模式發光特效 */
        .focus-active-border {
            box-shadow: inset 0 0 60px rgba(56, 189, 248, 0.55), inset 0 0 120px rgba(168, 85, 247, 0.35);
            border: 2px solid rgba(56, 189, 248, 0.85);
        }

        @keyframes pulseVignette {
            0%, 100% { opacity: 0.12; }
            50% { opacity: 0.65; }
        }

        .danger-vignette {
            background: radial-gradient(circle, transparent 52%, rgba(225, 29, 72, 0.85) 100%);
            animation: pulseVignette 0.75s infinite ease-in-out;
            pointer-events: none;
        }

        /* 器械按鈕樣式 */
        .tool-btn {
            transition: all 0.14s cubic-bezier(0.4, 0, 0.2, 1);
            border: 1.5px solid rgba(51, 65, 85, 0.85);
            background: linear-gradient(160deg, #0d1626, #172338);
        }

        .tool-btn:hover {
            transform: translateY(-2px);
            border-color: rgba(34, 211, 238, 0.6);
        }

        .tool-btn.active {
            border-color: #22d3ee;
            background: linear-gradient(160deg, #083344, #155e75);
            box-shadow: 0 0 18px rgba(34, 211, 238, 0.45), inset 0 0 10px rgba(34, 211, 238, 0.25);
            transform: translateY(-3px) scale(1.02);
        }

        /* 迷你儀器圖示動態特效 */
        @keyframes sonarPulse {
            0%, 100% { opacity: 0.25; transform: scale(0.95); }
            50% { opacity: 1; transform: scale(1.06); }
        }
        .tool-btn.active .mini-sonar-wave {
            animation: sonarPulse 1.2s infinite ease-in-out;
            transform-origin: center;
        }
        .mini-instrument-svg {
            filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.65));
            transition: transform 0.18s ease;
        }
        .tool-btn.active .mini-instrument-svg {
            transform: scale(1.12) rotate(-3deg);
            filter: drop-shadow(0 0 8px rgba(34, 211, 238, 0.6));
        }

        /* 超執刀按鈕特效 */
        @keyframes focusReadyGlow {
            0%, 100% { box-shadow: 0 0 15px rgba(56, 189, 248, 0.6), 0 0 30px rgba(168, 85, 247, 0.4); }
            50% { box-shadow: 0 0 28px rgba(56, 189, 248, 0.95), 0 0 45px rgba(168, 85, 247, 0.8); }
        }

        .focus-btn-ready {
            background: linear-gradient(135deg, #0284c7 0%, #7c3aed 100%);
            border-color: #7dd3fc;
            animation: focusReadyGlow 1.4s infinite ease-in-out;
            cursor: pointer;
        }

        .arcade-btn {
            background: linear-gradient(135deg, #06b6d4 0%, #2563eb 100%);
            box-shadow: 0 0 24px rgba(6, 182, 212, 0.55);
            transition: all 0.2s ease;
        }

        .arcade-btn:hover {
            transform: scale(1.03);
            box-shadow: 0 0 35px rgba(6, 182, 212, 0.85);
        }

        .arcade-btn:active {
            transform: scale(0.98);
        }

        .scanlines {
            background: linear-gradient(
                to bottom,
                rgba(255,255,255,0),
                rgba(255,255,255,0) 50%,
                rgba(0,0,0,0.16) 50%,
                rgba(0,0,0,0.16)
            );
            background-size: 100% 4px;
            pointer-events: none;
        }
    </style>
</head>
<body class="h-screen w-screen flex flex-col justify-between relative">

    <!-- 危機紅光與超執刀邊框覆蓋層 -->
    <div id="danger-vignette" class="fixed inset-0 z-30 danger-vignette hidden"></div>
    <div id="focus-vignette" class="fixed inset-0 z-30 focus-active-border pointer-events-none hidden transition-opacity duration-300"></div>

    <!-- 分享狀態輕提示 Toast -->
    <div id="share-toast" class="fixed top-20 left-1/2 -translate-x-1/2 z-50 hidden">
        <div class="bg-emerald-950/95 border border-emerald-400 text-emerald-200 px-4 py-2 rounded-full text-xs sm:text-sm font-bold shadow-[0_0_24px_rgba(52,211,153,0.5)]">
            ✓ 試玩連結已複製！快傳給朋友一起挑戰吧
        </div>
    </div>

    <!-- 頂部高精細臨床生命徵象監測儀 HUD -->
    <header id="top-hud" class="hud-panel z-20 px-2.5 py-1.5 sm:px-5 sm:py-2.5 flex flex-wrap items-center justify-between gap-2 rounded-b-xl mx-1 sm:mx-2.5 mt-1">
        <!-- 生命穩定度與 SpO2 / BP -->
        <div class="flex items-center gap-3 flex-1 min-w-[220px] max-w-md">
            <div class="flex-1">
                <div class="flex justify-between items-end mb-0.5">
                    <span class="text-[11px] sm:text-xs font-bold tracking-wider text-cyan-300 flex items-center gap-1.5">
                        <span id="vital-dot" class="inline-block w-2 h-2 rounded-full bg-emerald-400 animate-ping"></span>
                        生命穩定度 STABILITY
                    </span>
                    <span id="vitals-text" class="font-hud text-sm sm:text-lg font-black text-emerald-400">100%</span>
                </div>
                <div class="w-full h-2.5 sm:h-3 bg-slate-950 rounded-full overflow-hidden border border-slate-700/80 p-0.5">
                    <div id="vitals-bar" class="h-full bg-gradient-to-r from-emerald-500 to-cyan-400 rounded-full transition-all duration-150" style="width: 100%;"></div>
                </div>
                <!-- 臨床生理指標列 -->
                <div class="flex justify-between items-center text-[10px] sm:text-[11px] font-hud mt-1 text-slate-300 gap-1">
                    <span class="flex items-center gap-1" title="電子血壓計">
                        <svg class="w-3 h-3 text-cyan-400 shrink-0" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                            <rect x="4" y="6" width="10" height="12" rx="2"/><path d="M14 10h4a2 2 0 0 1 2 2v2a2 2 0 0 1-2 2h-4"/><path d="M7 10h4M7 14h2"/>
                        </svg>
                        BP <strong id="bp-text" class="text-cyan-300">118/76</strong>
                    </span>
                    <span class="flex items-center gap-1" title="指夾式脈搏血氧儀">
                        <svg class="w-3 h-3 text-emerald-400 shrink-0" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                            <path d="M4 9c0-2 2-3 5-3h8a3 3 0 0 1 3 3v2H4V9z"/><path d="M4 13h16v2a3 3 0 0 1-3 3H9c-3 0-5-1-5-3v-2z"/><circle cx="15" cy="8.5" r="1" fill="currentColor"/>
                        </svg>
                        SpO2 <strong id="spo2-text" class="text-emerald-400">99%</strong>
                    </span>
                    <span class="flex items-center gap-1" title="負壓引流集血瓶">
                        <svg class="w-3 h-3 text-rose-400 shrink-0" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                            <path d="M8 3h8M10 3v3h4V3"/><rect x="6" y="6" width="12" height="15" rx="2"/><path d="M6 14h12" stroke-dasharray="2 1"/>
                        </svg>
                        失血 <strong id="bloodloss-text" class="text-rose-400">0</strong>mL
                    </span>
                </div>
            </div>
        </div>

        <!-- 心率與多導程動態心電圖 ECG -->
        <div class="flex items-center gap-2 bg-slate-950/95 border border-slate-800 rounded-lg px-2.5 py-1">
            <div class="text-right">
                <div class="text-[9px] text-slate-400 font-bold tracking-wider flex items-center justify-end gap-1">
                    <svg class="w-3 h-3 text-emerald-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                        <circle cx="12" cy="12" r="8"/><circle cx="12" cy="12" r="2.5" fill="currentColor"/><path d="M12 2v2M12 20v2M2 12h2M20 12h2"/>
                    </svg>
                    ECG II // BPM
                </div>
                <div id="bpm-text" class="font-hud text-lg sm:text-2xl font-black text-emerald-400 leading-none">76</div>
            </div>
            <canvas id="ecg-canvas" class="w-24 sm:w-36 h-10 sm:h-12 rounded bg-slate-950"></canvas>
        </div>

        <!-- 中央：手術階段進度 -->
        <div class="hidden lg:flex flex-col items-center justify-center px-3 py-1 bg-slate-900/90 border border-cyan-500/30 rounded-lg min-w-[260px]">
            <div class="flex items-center gap-2 mb-0.5">
                <span id="diff-tag" class="px-1.5 py-0.2 bg-emerald-500/20 text-emerald-300 text-[10px] font-hud font-bold rounded">RELAXED</span>
                <span id="wave-indicator" class="text-xs font-black text-white">PHASE 1/3：表淺止血與彈片摘除</span>
            </div>
            <div id="objective-hint" class="text-[11px] text-amber-300 font-medium">
                目標：封閉動脈出血點並沿導引角夾出彈片
            </div>
        </div>

        <!-- 右側：超執刀按鈕、時間、分數、分享與音效 -->
        <div class="flex items-center gap-1.5 sm:gap-2.5">
            <button id="focus-btn" disabled class="relative overflow-hidden px-2 py-1.5 rounded-lg border border-slate-700 bg-slate-900/90 text-slate-400 transition-all flex flex-col items-center min-w-[86px] sm:min-w-[102px]">
                <div id="focus-fill" class="absolute inset-y-0 left-0 bg-gradient-to-r from-cyan-500/40 to-purple-500/40 transition-all duration-200" style="width: 0%;"></div>
                <span class="relative z-10 text-[10px] font-black tracking-wider text-cyan-200 flex items-center gap-1">
                    ✦ 超執刀模式
                </span>
                <span id="focus-status-text" class="relative z-10 font-hud text-[10px] sm:text-[11px] font-bold text-slate-300">0% [SPACE]</span>
            </button>

            <div class="text-center">
                <div class="text-[9px] text-slate-400 font-bold">手術時間</div>
                <div id="timer-text" class="font-hud text-sm sm:text-xl font-black text-amber-400">02:00</div>
            </div>

            <div class="text-right">
                <div class="text-[9px] text-slate-400 font-bold flex items-center justify-end gap-1">
                    SCORE <span id="combo-badge" class="text-cyan-400 font-hud hidden">x1</span>
                </div>
                <div id="score-text" class="font-hud text-sm sm:text-xl font-black text-white">00000</div>
            </div>

            <button id="share-hud-btn" class="p-2 rounded-lg bg-cyan-950/80 hover:bg-cyan-900 border border-cyan-500/50 text-cyan-300 transition cursor-pointer" title="複製試玩連結分享給朋友">
                <svg class="w-4 h-4 sm:w-5 sm:h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8.684 13.342C8.886 12.938 9 12.482 9 12c0-.482-.114-.938-.316-1.342m0 2.684a3 3 0 110-2.684m0 2.684l6.632 3.316m-6.632-6l6.632-3.316m0 0a3 3 0 105.367-2.684 3 3 0 00-5.367 2.684zm0 9.316a3 3 0 105.368 2.684 3 3 0 00-5.368-2.684z"/>
                </svg>
            </button>

            <button id="mute-btn" class="p-2 rounded-lg bg-slate-800/90 hover:bg-slate-700 border border-slate-600 text-cyan-300 transition cursor-pointer" title="切換監測儀音效">
                <svg id="icon-sound-on" class="w-4 h-4 sm:w-5 sm:h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.536 8.464a5 5 0 010 7.072m2.828-9.9a9 9 0 010 12.728M5.586 15H4a1 1 0 01-1-1v-4a1 1 0 011-1h1.586l4.707-4.707C10.923 3.663 12 4.109 12 5v14c0 .891-1.077 1.337-1.707.707L5.586 15z"/>
                </svg>
                <svg id="icon-sound-off" class="w-4 h-4 sm:w-5 sm:h-5 hidden text-rose-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5.586 15H4a1 1 0 01-1-1v-4a1 1 0 011-1h1.586l4.707-4.707C10.923 3.663 12 4.109 12 5v14c0 .891-1.077 1.337-1.707.707L5.586 15zM17 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2"/>
                </svg>
            </button>
        </div>
    </header>

    <!-- 行動裝置與平板階段提示條 -->
    <div class="lg:hidden flex items-center justify-between px-3 py-1 bg-cyan-950/70 border-b border-cyan-500/20 text-[11px] font-bold text-cyan-200 z-10">
        <span id="wave-indicator-mobile">PHASE 1/3：表淺止血與彈片摘除</span>
        <span id="objective-hint-mobile" class="text-amber-300">封閉動脈出血並夾出彈片</span>
    </div>

    <!-- 中央高精細手術視野畫布 -->
    <main id="canvas-container" class="relative flex-1 w-full overflow-hidden flex items-center justify-center">
        <canvas id="game-canvas" class="w-full h-full block cursor-none"></canvas>

        <!-- 電燒器過熱溫度指示器 -->
        <div id="heat-hud" class="pointer-events-none absolute top-3 right-3 z-20 hud-panel px-3 py-1.5 rounded-lg hidden">
            <div class="flex items-center justify-between gap-3 text-[10px] font-bold mb-1">
                <span class="text-amber-400">電燒頭溫度 CAUTERY TEMP</span>
                <span id="heat-percent" class="font-hud text-white">0%</span>
            </div>
            <div class="w-32 h-2 bg-slate-950 rounded-full overflow-hidden border border-slate-700">
                <div id="heat-bar" class="h-full bg-gradient-to-r from-amber-400 via-orange-500 to-rose-600 transition-all duration-75" style="width: 0%;"></div>
            </div>
        </div>

        <!-- 積血視野遮蔽警告 -->
        <div id="blood-pool-warning" class="pointer-events-none absolute bottom-3 left-1/2 -translate-x-1/2 z-20 hidden">
            <div class="bg-rose-950/90 border border-rose-500/80 text-rose-200 px-4 py-1 rounded-full text-xs font-bold flex items-center gap-2 shadow-lg">
                <span class="w-2 h-2 rounded-full bg-rose-500 animate-ping"></span>
                創面積血過多！請使用 [3] 吸引器抽吸積血以確保視野
            </div>
        </div>

        <!-- 突發併發症警報橫幅 -->
        <div id="alert-banner" class="pointer-events-none absolute top-5 left-1/2 -translate-x-1/2 z-20 hidden">
            <div class="bg-rose-950/95 border-2 border-rose-500 text-rose-100 px-5 py-2.5 rounded-xl shadow-[0_0_35px_rgba(244,63,94,0.85)] flex items-center gap-3 animate-bounce">
                <span class="text-2xl">🚨</span>
                <div>
                    <div id="alert-title" class="font-black text-sm sm:text-base tracking-wider text-rose-400">臨床緊急警報！</div>
                    <div id="alert-desc" class="text-xs font-bold text-slate-200">檢測到深層組織異物與繼發性動脈破裂！</div>
                </div>
            </div>
        </div>
    </main>

    <!-- 底部六合一專業外科器械托盤 -->
    <footer class="hud-panel z-20 px-1.5 py-1.5 sm:px-4 sm:py-2.5 rounded-t-2xl mx-1 sm:mx-2.5 mb-1">
        <div class="max-w-5xl mx-auto grid grid-cols-6 gap-1.5 sm:gap-2.5">
            <!-- [1] 超音波掃描儀 -->
            <button data-tool="scanner" class="tool-btn rounded-xl p-1.5 sm:p-2 flex flex-col items-center justify-center relative">
                <span class="absolute top-1 left-1.5 font-hud text-[10px] font-black text-sky-400 bg-sky-950/90 px-1 rounded border border-sky-500/40">1</span>
                <div class="w-9 h-9 sm:w-11 sm:h-11 rounded-xl bg-sky-950/70 border border-sky-400/40 flex items-center justify-center mb-1">
                    <svg class="mini-instrument-svg w-7 h-7 sm:w-8 sm:h-8" viewBox="0 0 36 36" fill="none">
                        <path class="mini-sonar-wave" d="M10 26 C14 30, 22 30, 26 26" stroke="#38bdf8" stroke-width="1.8" stroke-linecap="round"/>
                        <path class="mini-sonar-wave" d="M7 29 C13 34, 23 34, 29 29" stroke="#7dd3fc" stroke-width="1.4" stroke-linecap="round" stroke-dasharray="2 2"/>
                        <path d="M18 3 V7" stroke="#64748b" stroke-width="2.5" stroke-linecap="round"/>
                        <rect x="14" y="7" width="8" height="10" rx="2.5" fill="#e2e8f0" stroke="#334155" stroke-width="1.2"/>
                        <rect x="15" y="9" width="6" height="4" rx="1" fill="#0284c7"/>
                        <path d="M11 17 H25 L27 21 C22 24, 14 24, 9 21 L11 17 Z" fill="#38bdf8" stroke="#0369a1" stroke-width="1.2"/>
                    </svg>
                </div>
                <span class="text-[11px] sm:text-xs font-black text-white">超音波掃描</span>
                <span class="hidden md:block text-[10px] text-slate-400">定位深層隱匿彈片</span>
            </button>

            <!-- [2] 精密外科鑷 -->
            <button data-tool="forceps" class="tool-btn rounded-xl p-1.5 sm:p-2 flex flex-col items-center justify-center relative">
                <span class="absolute top-1 left-1.5 font-hud text-[10px] font-black text-cyan-400 bg-cyan-950/90 px-1 rounded border border-cyan-500/40">2</span>
                <div class="w-9 h-9 sm:w-11 sm:h-11 rounded-xl bg-cyan-950/70 border border-cyan-400/40 flex items-center justify-center mb-1">
                    <svg class="mini-instrument-svg w-7 h-7 sm:w-8 sm:h-8" viewBox="0 0 36 36" fill="none">
                        <path d="M27 7 C29 9, 28 11, 25 13 L11 27" stroke="#cbd5e1" stroke-width="2.4" stroke-linecap="round"/>
                        <path d="M27 7 C25 5, 23 6, 21 9 L7 23" stroke="#94a3b8" stroke-width="2.4" stroke-linecap="round"/>
                        <path d="M7 23 L5 29 L11 27" stroke="#22d3ee" stroke-width="1.6" stroke-linejoin="round"/>
                    </svg>
                </div>
                <span class="text-[11px] sm:text-xs font-black text-white">精密外科鑷</span>
                <span class="hidden md:block text-[10px] text-slate-400">沿角度向量夾出異物</span>
            </button>

            <!-- [3] 負壓吸引器 -->
            <button data-tool="suction" class="tool-btn rounded-xl p-1.5 sm:p-2 flex flex-col items-center justify-center relative">
                <span class="absolute top-1 left-1.5 font-hud text-[10px] font-black text-rose-400 bg-rose-950/90 px-1 rounded border border-rose-500/40">3</span>
                <div class="w-9 h-9 sm:w-11 sm:h-11 rounded-xl bg-rose-950/70 border border-rose-400/40 flex items-center justify-center mb-1">
                    <svg class="mini-instrument-svg w-7 h-7 sm:w-8 sm:h-8" viewBox="0 0 36 36" fill="none">
                        <path d="M26 5 C30 8, 30 14, 25 17" stroke="#cbd5e1" stroke-width="3" stroke-linecap="round"/>
                        <path d="M14 17 L8 25" stroke="#94a3b8" stroke-width="2.6" stroke-linecap="round"/>
                        <circle cx="7" cy="26.5" r="2" fill="#f43f5e"/>
                    </svg>
                </div>
                <span class="text-[11px] sm:text-xs font-black text-white">負壓吸引器</span>
                <span class="hidden md:block text-[10px] text-slate-400">抽吸創面擴大積血</span>
            </button>

            <!-- [4] 電燒止血器 -->
            <button data-tool="cautery" class="tool-btn active rounded-xl p-1.5 sm:p-2 flex flex-col items-center justify-center relative">
                <span class="absolute top-1 left-1.5 font-hud text-[10px] font-black text-amber-400 bg-amber-950/90 px-1 rounded border border-amber-500/40">4</span>
                <div class="w-9 h-9 sm:w-11 sm:h-11 rounded-xl bg-amber-950/70 border border-amber-400/40 flex items-center justify-center mb-1">
                    <svg class="mini-instrument-svg w-7 h-7 sm:w-8 sm:h-8" viewBox="0 0 36 36" fill="none">
                        <path d="M25 6 L29 10 L16 23 L12 19 L25 6 Z" fill="#f59e0b" stroke="#b45309" stroke-width="1.2" stroke-linejoin="round"/>
                        <path d="M13 21 L7 27" stroke="#e2e8f0" stroke-width="2" stroke-linecap="round"/>
                        <circle cx="6.5" cy="27.5" r="2.5" fill="#fbbf24"/>
                    </svg>
                </div>
                <span class="text-[11px] sm:text-xs font-black text-white">電燒止血器</span>
                <span class="hidden md:block text-[10px] text-slate-400">點狀燒灼(注意過熱)</span>
            </button>

            <!-- [5] 廣譜抗生素凝膠 -->
            <button data-tool="antibiotic" class="tool-btn rounded-xl p-1.5 sm:p-2 flex flex-col items-center justify-center relative">
                <span class="absolute top-1 left-1.5 font-hud text-[10px] font-black text-purple-400 bg-purple-950/90 px-1 rounded border border-purple-500/40">5</span>
                <div class="w-9 h-9 sm:w-11 sm:h-11 rounded-xl bg-purple-950/70 border border-purple-400/40 flex items-center justify-center mb-1">
                    <svg class="mini-instrument-svg w-7 h-7 sm:w-8 sm:h-8" viewBox="0 0 36 36" fill="none">
                        <path d="M23 5 L30 12 L18 23 L12 17 L23 5 Z" fill="#c084fc" stroke="#7e22ce" stroke-width="1.2" stroke-linejoin="round"/>
                        <path d="M8 25 C6.5 27.5, 6 29, 7.5 30.2 C9 31.2, 10.5 29.8, 9.5 27.8 L8 25 Z" fill="#a855f7" stroke="#e9d5ff" stroke-width="1"/>
                    </svg>
                </div>
                <span class="text-[11px] sm:text-xs font-black text-white">抗生素凝膠</span>
                <span class="hidden md:block text-[10px] text-slate-400">塗抹中和敗血感染</span>
            </button>

            <!-- [6] 顯微縫合針 -->
            <button data-tool="suture" class="tool-btn rounded-xl p-1.5 sm:p-2 flex flex-col items-center justify-center relative">
                <span class="absolute top-1 left-1.5 font-hud text-[10px] font-black text-emerald-400 bg-emerald-950/90 px-1 rounded border border-emerald-500/40">6</span>
                <div class="w-9 h-9 sm:w-11 sm:h-11 rounded-xl bg-emerald-950/70 border border-emerald-400/40 flex items-center justify-center mb-1">
                    <svg class="mini-instrument-svg w-7 h-7 sm:w-8 sm:h-8" viewBox="0 0 36 36" fill="none">
                        <path d="M9 19 C8 11, 16 6, 23 11" stroke="#f8fafc" stroke-width="2.5" stroke-linecap="round"/>
                        <path d="M23 11 C29 14, 29 24, 21 27" stroke="#38bdf8" stroke-width="1.6" stroke-linecap="round" stroke-dasharray="3 1.5"/>
                    </svg>
                </div>
                <span class="text-[11px] sm:text-xs font-black text-white">顯微縫合針</span>
                <span class="hidden md:block text-[10px] text-slate-400">Z字精準閉合裂傷</span>
            </button>
        </div>
    </footer>

    <!-- 術前簡報覆蓋畫面 (含難度選擇與好友試玩連結分享) -->
    <div id="start-modal" class="fixed inset-0 z-40 bg-slate-950/92 backdrop-blur-md flex items-center justify-center p-3 sm:p-4 overflow-y-auto">
        <div class="hud-panel max-w-2xl w-full rounded-2xl p-5 sm:p-7 relative overflow-hidden my-auto">
            <div class="scanlines absolute inset-0"></div>
            <div class="relative z-10">
                <div class="flex flex-wrap items-center justify-between gap-2 border-b border-slate-700/80 pb-3 mb-4">
                    <div>
                        <span class="px-2.5 py-0.5 rounded bg-rose-500/20 border border-rose-500/50 text-rose-400 text-[11px] font-hud font-black tracking-widest">
                            TRAUMA CODE RED // FRIEND PLAY EDITION
                        </span>
                        <h1 class="text-xl sm:text-3xl font-black text-white mt-1">
                            緊急手術：<span class="text-cyan-400">高階外科臨床模擬</span>
                        </h1>
                    </div>
                    <button id="copy-invite-btn" type="button" class="px-3 py-2 rounded-xl bg-cyan-500/15 hover:bg-cyan-500/25 border border-cyan-400/50 text-cyan-200 text-xs font-bold flex items-center gap-1.5 transition cursor-pointer">
                        <svg class="w-4 h-4 text-cyan-300" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z"/>
                        </svg>
                        複製試玩連結
                    </button>
                </div>

                <!-- 難度模式選擇（適合初次試玩與進階挑戰） -->
                <div class="mb-4">
                    <div class="text-xs font-bold text-slate-300 mb-2">選擇臨床節奏模式：</div>
                    <div class="grid grid-cols-2 gap-2.5">
                        <button id="mode-relaxed-btn" type="button" class="p-3 rounded-xl border-2 border-emerald-400 bg-emerald-950/40 text-left transition cursor-pointer">
                            <div class="text-xs sm:text-sm font-black text-emerald-300">從容試玩模式（推薦新手）</div>
                            <div class="text-[11px] text-slate-300 mt-0.5">失血速度較慢，有充裕時間切換器械與練習操作</div>
                        </button>
                        <button id="mode-standard-btn" type="button" class="p-3 rounded-xl border border-slate-700 bg-slate-900/80 text-left transition cursor-pointer">
                            <div class="text-xs sm:text-sm font-black text-amber-300">標準急救模式（進階挑戰）</div>
                            <div class="text-[11px] text-slate-300 mt-0.5">正常失血與積血擴散節奏，考驗反應與連擊技巧</div>
                        </button>
                    </div>
                </div>

                <!-- 六大器械臨床操作規範 -->
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-2 text-left mb-4">
                    <div class="bg-slate-900/85 border border-slate-800 p-2 rounded-xl text-xs text-slate-300">
                        <strong class="text-sky-300">[1] 超音波掃描</strong>：移動探頭讓深層隱匿彈片顯影。
                    </div>
                    <div class="bg-slate-900/85 border border-slate-800 p-2 rounded-xl text-xs text-slate-300">
                        <strong class="text-cyan-300">[2] 精密外科鑷</strong>：沿綠色箭頭角度平穩拖出彈片。
                    </div>
                    <div class="bg-slate-900/85 border border-slate-800 p-2 rounded-xl text-xs text-slate-300">
                        <strong class="text-rose-300">[3] 負壓吸引器</strong>：按住積血區抽吸，避免遮蔽視野。
                    </div>
                    <div class="bg-slate-900/85 border border-slate-800 p-2 rounded-xl text-xs text-slate-300">
                        <strong class="text-amber-300">[4] 電燒止血器</strong>：按住噴血點止血，留意溫度別過熱。
                    </div>
                    <div class="bg-slate-900/85 border border-slate-800 p-2 rounded-xl text-xs text-slate-300">
                        <strong class="text-purple-300">[5] 抗生素凝膠</strong>：塗抹紫色感染區中和敗血毒素。
                    </div>
                    <div class="bg-slate-900/85 border border-slate-800 p-2 rounded-xl text-xs text-slate-300">
                        <strong class="text-emerald-300">[6] 顯微縫合針</strong>：清場後依序點擊節點完成縫合。
                    </div>
                </div>

                <div class="flex flex-col sm:flex-row items-center justify-between gap-3">
                    <span class="text-xs text-slate-400">支援滑鼠、觸控螢幕與鍵盤 <strong class="text-slate-200">1~6 / SPACE</strong></span>
                    <button id="start-btn" class="arcade-btn w-full sm:w-auto px-8 py-3.5 rounded-xl font-black text-base sm:text-lg text-white tracking-wider cursor-pointer">
                        開始手術 // START OP
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- 術後臨床評估報告 (Post-Op Evaluation Report) -->
    <div id="end-modal" class="fixed inset-0 z-40 bg-slate-950/92 backdrop-blur-md hidden items-center justify-center p-4">
        <div class="hud-panel max-w-lg w-full rounded-2xl p-6 sm:p-7 text-center relative overflow-hidden">
            <div class="scanlines absolute inset-0"></div>
            <div class="relative z-10">
                <div id="end-badge" class="inline-block px-3 py-1 rounded-full bg-emerald-500/20 border border-emerald-500/50 text-emerald-400 text-xs font-hud font-black tracking-widest mb-2">
                    POST-OP CLINICAL REPORT
                </div>
                <h2 id="end-title" class="text-2xl sm:text-3xl font-black text-white mb-1">手術圓滿成功！患者生命徵象穩定</h2>
                <p id="end-subtitle" class="text-xs text-slate-400 mb-4">截圖或複製連結向朋友炫耀你的執刀成績</p>

                <div class="my-3 flex items-center justify-center gap-6 bg-slate-900/80 border border-slate-800 rounded-xl py-3">
                    <div>
                        <span class="text-[11px] text-slate-400 font-bold tracking-widest block">外科評級 RANK</span>
                        <div id="end-rank" class="font-hud text-5xl sm:text-6xl font-black text-cyan-400 drop-shadow-[0_0_20px_rgba(34,211,238,0.85)]">S</div>
                    </div>
                    <div class="h-12 w-px bg-slate-800"></div>
                    <div class="text-left space-y-1">
                        <div class="text-xs text-slate-400">手術精準度：<strong id="end-accuracy" class="font-hud text-emerald-400 text-sm">100%</strong></div>
                        <div class="text-xs text-slate-400">最高連擊數：<strong id="end-combo" class="font-hud text-cyan-300 text-sm">0</strong></div>
                        <div class="text-xs text-slate-400">總計失血量：<strong id="end-bloodloss" class="font-hud text-rose-400 text-sm">120 mL</strong></div>
                    </div>
                </div>

                <div class="bg-slate-900/90 border border-slate-800 rounded-xl p-3.5 mb-4 space-y-2 text-xs sm:text-sm">
                    <div class="flex justify-between">
                        <span class="text-slate-400">臨床手術總分</span>
                        <span id="end-score" class="font-hud font-bold text-white">0</span>
                    </div>
                    <div class="flex justify-between">
                        <span class="text-slate-400">術後生命穩定度</span>
                        <span id="end-vitals" class="font-hud font-bold text-emerald-400">100%</span>
                    </div>
                    <div class="flex justify-between">
                        <span class="text-slate-400">手術耗時</span>
                        <span id="end-time" class="font-hud font-bold text-amber-400">01:15</span>
                    </div>
                    <div class="flex justify-between">
                        <span class="text-slate-400">組織創傷／操作失誤</span>
                        <span id="end-mistakes" class="font-hud font-bold text-rose-400">0</span>
                    </div>
                </div>

                <div class="grid grid-cols-1 sm:grid-cols-2 gap-2.5">
                    <button id="share-result-btn" type="button" class="py-3 rounded-xl font-bold text-sm bg-slate-800 hover:bg-slate-700 border border-cyan-500/50 text-cyan-200 transition cursor-pointer">
                        複製戰績與試玩連結
                    </button>
                    <button id="retry-btn" class="arcade-btn py-3 rounded-xl font-black text-sm sm:text-base text-white tracking-wider cursor-pointer">
                        再挑戰一次
                    </button>
                </div>
            </div>
        </div>
    </div>

    <script>
        /**
         * Web Audio API 醫療音效與合成器引擎
         */
        class MedicalAudioEngine {
            constructor() {
                this.ctx = null;
                this.muted = false;
                this.lastToolSound = 0;
            }

            init() {
                if (!this.ctx) {
                    const AudioCtx = window.AudioContext || window.webkitAudioContext;
                    this.ctx = new AudioCtx();
                }
                if (this.ctx.state === 'suspended') {
                    this.ctx.resume();
                }
            }

            toggleMute() {
                this.muted = !this.muted;
                return this.muted;
            }

            playHeartbeat(isCritical = false, isFocus = false) {
                if (this.muted || !this.ctx) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(isFocus ? 640 : (isCritical ? 980 : 840), now);
                gain.gain.setValueAtTime(0.075, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.08);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.09);
            }

            playScanner() {
                if (this.muted || !this.ctx) return;
                const now = this.ctx.currentTime;
                if (now - this.lastToolSound < 0.14) return;
                this.lastToolSound = now;

                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(1150, now);
                osc.frequency.exponentialRampToValueAtTime(1650, now + 0.11);
                gain.gain.setValueAtTime(0.05, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.12);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.13);
            }

            playSuction() {
                if (this.muted || !this.ctx) return;
                const now = this.ctx.currentTime;
                if (now - this.lastToolSound < 0.09) return;
                this.lastToolSound = now;

                const bufferSize = this.ctx.sampleRate * 0.08;
                const buffer = this.ctx.createBuffer(1, bufferSize, this.ctx.sampleRate);
                const data = buffer.getChannelData(0);
                for (let i = 0; i < bufferSize; i++) data[i] = Math.random() * 2 - 1;

                const noise = this.ctx.createBufferSource();
                noise.buffer = buffer;
                const filter = this.ctx.createBiquadFilter();
                filter.type = 'lowpass';
                filter.frequency.setValueAtTime(650, now);
                filter.frequency.linearRampToValueAtTime(1100, now + 0.07);

                const gain = this.ctx.createGain();
                gain.gain.setValueAtTime(0.09, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.08);

                noise.connect(filter);
                filter.connect(gain);
                gain.connect(this.ctx.destination);
                noise.start(now);
            }

            playCautery(heatRatio = 0) {
                if (this.muted || !this.ctx) return;
                const now = this.ctx.currentTime;
                if (now - this.lastToolSound < 0.07) return;
                this.lastToolSound = now;

                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sawtooth';
                const baseFreq = 260 + heatRatio * 340 + Math.random() * 60;
                osc.frequency.setValueAtTime(baseFreq, now);
                osc.frequency.exponentialRampToValueAtTime(130, now + 0.06);
                gain.gain.setValueAtTime(0.065, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.065);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.07);
            }

            playAntibiotic() {
                if (this.muted || !this.ctx) return;
                const now = this.ctx.currentTime;
                if (now - this.lastToolSound < 0.1) return;
                this.lastToolSound = now;

                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'triangle';
                osc.frequency.setValueAtTime(480 + Math.random() * 140, now);
                osc.frequency.exponentialRampToValueAtTime(880, now + 0.08);
                gain.gain.setValueAtTime(0.06, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.09);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.1);
            }

            playStitch(isPerfect = true) {
                if (this.muted || !this.ctx) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(isPerfect ? 720 : 520, now);
                osc.frequency.exponentialRampToValueAtTime(isPerfect ? 1320 : 860, now + 0.07);
                gain.gain.setValueAtTime(0.1, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.08);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.09);
            }

            playSuccess() {
                if (this.muted || !this.ctx) return;
                const now = this.ctx.currentTime;
                [523.25, 659.25, 783.99, 1046.5].forEach((freq, i) => {
                    const osc = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(freq, now + i * 0.045);
                    gain.gain.setValueAtTime(0.11, now + i * 0.045);
                    gain.gain.exponentialRampToValueAtTime(0.001, now + i * 0.045 + 0.2);
                    osc.connect(gain);
                    gain.connect(this.ctx.destination);
                    osc.start(now + i * 0.045);
                    osc.stop(now + i * 0.045 + 0.22);
                });
            }

            playError() {
                if (this.muted || !this.ctx) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(160, now);
                osc.frequency.linearRampToValueAtTime(85, now + 0.28);
                gain.gain.setValueAtTime(0.18, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.29);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.3);
            }

            playFocusActivate() {
                if (this.muted || !this.ctx) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(320, now);
                osc.frequency.exponentialRampToValueAtTime(1280, now + 0.45);
                gain.gain.setValueAtTime(0.18, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.5);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.52);
            }

            playAlert() {
                if (this.muted || !this.ctx) return;
                const now = this.ctx.currentTime;
                [0, 0.16].forEach(offset => {
                    const osc = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();
                    osc.type = 'square';
                    osc.frequency.setValueAtTime(680, now + offset);
                    osc.frequency.setValueAtTime(490, now + offset + 0.07);
                    gain.gain.setValueAtTime(0.09, now + offset);
                    gain.gain.exponentialRampToValueAtTime(0.001, now + offset + 0.14);
                    osc.connect(gain);
                    gain.connect(this.ctx.destination);
                    osc.start(now + offset);
                    osc.stop(now + offset + 0.15);
                });
            }
        }

        const audio = new MedicalAudioEngine();

        /**
         * 分享連結與剪貼簿工具（相容 iFrame 與一般瀏覽器環境）
         */
        function copyTextToClipboard(text, message = '✓ 試玩連結已複製！快傳給朋友一起挑戰吧') {
            try {
                if (navigator.clipboard && window.isSecureContext) {
                    navigator.clipboard.writeText(text).catch(() => fallbackCopy(text));
                } else {
                    fallbackCopy(text);
                }
            } catch (err) {
                fallbackCopy(text);
            }

            const toast = document.getElementById('share-toast');
            if (toast) {
                toast.firstElementChild.textContent = message;
                toast.classList.remove('hidden');
                clearTimeout(copyTextToClipboard._timer);
                copyTextToClipboard._timer = setTimeout(() => {
                    toast.classList.add('hidden');
                }, 2600);
            }
        }

        function fallbackCopy(text) {
            const input = document.createElement('textarea');
            input.value = text;
            input.style.position = 'fixed';
            input.style.opacity = '0';
            document.body.appendChild(input);
            input.select();
            document.execCommand('copy');
            document.body.removeChild(input);
        }

        /**
         * 主畫布與臨床狀態初始化
         */
        const canvas = document.getElementById('game-canvas');
        const ctx = canvas ? canvas.getContext('2d') : null;
        const ecgCanvas = document.getElementById('ecg-canvas');
        const ecgCtx = ecgCanvas ? ecgCanvas.getContext('2d') : null;

        let width = 0;
        let height = 0;
        let woundCenter = { x: 0, y: 0 };
        let woundRadius = { rx: 240, ry: 165 };

        // 難度模式：relaxed（從容試玩） | standard（標準急救）
        let difficultyMode = 'relaxed';

        // 遊戲核心變數
        let gameState = 'MENU';
        let currentTool = 'cautery';
        let vitals = 100;
        let spo2 = 99;
        let bloodLossMl = 0;
        let score = 0;
        let combo = 0;
        let maxCombo = 0;
        let mistakes = 0;
        let totalActions = 0;
        let cleanActions = 0;
        let healDoseCount = 0;
        const TOTAL_TIME = 120;
        let timeLeft = TOTAL_TIME;
        let currentPhase = 1;
        let lastFrameTime = 0;
        let pulsePhase = 0;
        let screenShake = 0;

        let cauteryHeat = 0;
        let cauteryOverheated = false;

        let focusGauge = 0;
        let focusActive = false;
        let focusTimer = 0;

        const pointer = {
            x: 0,
            y: 0,
            prevX: 0,
            prevY: 0,
            isDown: false,
            grabbedShrapnel: null
        };

        let bleeders = [];
        let bloodPools = [];
        let shrapnels = [];
        let infections = [];
        let lacerations = [];
        let charMarks = [];
        let muscleFibers = [];
        let particles = [];
        let popups = [];

        let ecgPoints = [];
        let ecgTimer = 0;
        let lastRank = 'S';

        function setDifficulty(mode) {
            difficultyMode = mode;
            const relaxedBtn = document.getElementById('mode-relaxed-btn');
            const standardBtn = document.getElementById('mode-standard-btn');
            const diffTag = document.getElementById('diff-tag');

            if (mode === 'relaxed') {
                relaxedBtn.className = 'p-3 rounded-xl border-2 border-emerald-400 bg-emerald-950/40 text-left transition cursor-pointer';
                standardBtn.className = 'p-3 rounded-xl border border-slate-700 bg-slate-900/80 text-left transition cursor-pointer';
                if (diffTag) {
                    diffTag.textContent = 'RELAXED';
                    diffTag.className = 'px-1.5 py-0.2 bg-emerald-500/20 text-emerald-300 text-[10px] font-hud font-bold rounded';
                }
            } else {
                standardBtn.className = 'p-3 rounded-xl border-2 border-amber-400 bg-amber-950/40 text-left transition cursor-pointer';
                relaxedBtn.className = 'p-3 rounded-xl border border-slate-700 bg-slate-900/80 text-left transition cursor-pointer';
                if (diffTag) {
                    diffTag.textContent = 'STANDARD';
                    diffTag.className = 'px-1.5 py-0.2 bg-amber-500/20 text-amber-300 text-[10px] font-hud font-bold rounded';
                }
            }
        }

        document.getElementById('mode-relaxed-btn').addEventListener('click', () => setDifficulty('relaxed'));
        document.getElementById('mode-standard-btn').addEventListener('click', () => setDifficulty('standard'));

        document.getElementById('copy-invite-btn').addEventListener('click', () => {
            const shareUrl = window.location.href;
            copyTextToClipboard(
                `來試玩「緊急手術：高階外科臨床模擬」吧！一起挑戰止血、拔彈片與顯微縫合：${shareUrl}`,
                '✓ 邀請連結已複製！快傳給朋友試玩'
            );
        });

        document.getElementById('share-hud-btn').addEventListener('click', () => {
            copyTextToClipboard(
                `我正在挑戰「緊急手術：高階外科臨床模擬」，你也來試試看：${window.location.href}`,
                '✓ 試玩連結已複製！'
            );
        });

        document.getElementById('share-result-btn').addEventListener('click', () => {
            copyTextToClipboard(
                `我在「緊急手術：高階外科臨床模擬」拿到 ${lastRank} 級評級、${score.toLocaleString()} 分！快來挑戰我：${window.location.href}`,
                '✓ 戰績與試玩連結已複製！'
            );
        });

        function resizeCanvas() {
            if (!canvas || !ctx) return;
            const container = document.getElementById('canvas-container');
            if (!container) return;
            const rect = container.getBoundingClientRect();
            const dpr = window.devicePixelRatio || 1;

            width = Math.max(1, rect.width);
            height = Math.max(1, rect.height);

            canvas.width = width * dpr;
            canvas.height = height * dpr;
            ctx.setTransform(dpr, 0, 0, dpr, 0, 0);

            woundCenter.x = width / 2;
            woundCenter.y = height / 2;
            woundRadius.rx = Math.min(width * 0.43, 360);
            woundRadius.ry = Math.min(height * 0.40, 230);

            generateMuscleFibers();

            if (ecgCanvas && ecgCtx) {
                const ecgRect = ecgCanvas.getBoundingClientRect();
                const ew = Math.max(1, ecgRect.width);
                const eh = Math.max(1, ecgRect.height);
                ecgCanvas.width = ew * dpr;
                ecgCanvas.height = eh * dpr;
                ecgCtx.setTransform(dpr, 0, 0, dpr, 0, 0);
                ecgPoints = new Array(Math.ceil(ew)).fill(eh / 2);
            }
        }

        function generateMuscleFibers() {
            muscleFibers = [];
            const count = 16;
            for (let i = 0; i < count; i++) {
                const ratio = (i / (count - 1)) * 2 - 1;
                muscleFibers.push({
                    yOffset: ratio * woundRadius.ry * 0.85,
                    curve: (Math.random() - 0.5) * 35,
                    alpha: 0.12 + Math.random() * 0.12
                });
            }
        }

        window.addEventListener('resize', resizeCanvas);
        if (document.readyState === 'loading') {
            document.addEventListener('DOMContentLoaded', resizeCanvas);
        } else {
            resizeCanvas();
        }

        const toolButtons = document.querySelectorAll('.tool-btn');
        function selectTool(toolName) {
            currentTool = toolName;
            pointer.grabbedShrapnel = null;
            toolButtons.forEach(btn => {
                btn.classList.toggle('active', btn.dataset.tool === toolName);
            });

            const heatHud = document.getElementById('heat-hud');
            if (toolName === 'cautery' || cauteryHeat > 5) {
                heatHud.classList.remove('hidden');
            } else {
                heatHud.classList.add('hidden');
            }
        }

        toolButtons.forEach(btn => {
            btn.addEventListener('click', () => {
                audio.init();
                selectTool(btn.dataset.tool);
            });
        });

        window.addEventListener('keydown', (e) => {
            if (gameState !== 'PLAYING') return;
            if (e.key === '1') selectTool('scanner');
            if (e.key === '2') selectTool('forceps');
            if (e.key === '3') selectTool('suction');
            if (e.key === '4') selectTool('cautery');
            if (e.key === '5') selectTool('antibiotic');
            if (e.key === '6') selectTool('suture');
            if (e.code === 'Space') {
                e.preventDefault();
                activateFocusMode();
            }
        });

        function activateFocusMode() {
            if (gameState !== 'PLAYING' || focusActive || focusGauge < 100) return;
            focusActive = true;
            focusTimer = 6.0;
            focusGauge = 0;
            audio.playFocusActivate();
            document.getElementById('focus-vignette').classList.remove('hidden');
            spawnPopup(woundCenter.x, woundCenter.y - 40, '✦ 超執刀啟動！時間膨脹 ✦', '#38bdf8');
        }

        document.getElementById('focus-btn').addEventListener('click', () => {
            audio.init();
            activateFocusMode();
        });

        document.getElementById('mute-btn').addEventListener('click', () => {
            audio.init();
            const isMuted = audio.toggleMute();
            document.getElementById('icon-sound-on').classList.toggle('hidden', isMuted);
            document.getElementById('icon-sound-off').classList.toggle('hidden', !isMuted);
        });

        function randomPointInWound(scale = 0.65) {
            const angle = Math.random() * Math.PI * 2;
            const r = Math.sqrt(Math.random()) * scale;
            return {
                x: woundCenter.x + Math.cos(angle) * woundRadius.rx * r,
                y: woundCenter.y + Math.sin(angle) * woundRadius.ry * r
            };
        }

        function spawnPopup(x, y, text, color = '#22d3ee') {
            popups.push({
                x,
                y,
                text,
                color,
                life: 1.15,
                vy: -48
            });
        }

        function spawnParticles(x, y, color, count = 10, speed = 130) {
            for (let i = 0; i < count; i++) {
                const angle = Math.random() * Math.PI * 2;
                const spd = (0.25 + Math.random() * 0.75) * speed;
                particles.push({
                    x,
                    y,
                    vx: Math.cos(angle) * spd,
                    vy: Math.sin(angle) * spd,
                    color,
                    size: 2 + Math.random() * 3,
                    life: 0.35 + Math.random() * 0.35
                });
            }
        }

        function showAlertBanner(title, desc) {
            const banner = document.getElementById('alert-banner');
            document.getElementById('alert-title').textContent = title;
            document.getElementById('alert-desc').textContent = desc;
            banner.classList.remove('hidden');
            audio.playAlert();
            setTimeout(() => banner.classList.add('hidden'), 3600);
        }

        function createShrapnel(isDeep = false) {
            const pt = randomPointInWound(0.55);
            const extractAngle = Math.atan2(pt.y - woundCenter.y, pt.x - woundCenter.x) + (Math.random() - 0.5) * 0.6;
            return {
                x: pt.x,
                y: pt.y,
                angle: Math.random() * Math.PI * 2,
                extractAngle,
                size: isDeep ? 22 : 25,
                isDeep,
                scanProgress: isDeep ? 0 : 1
            };
        }

        function setupPhase(phase) {
            currentPhase = phase;
            const phaseMeta = {
                1: {
                    title: 'PHASE 1/3：表淺止血與彈片摘除',
                    hint: '目標：電燒封閉動脈出血點，並沿綠色箭頭向量拔除彈片'
                },
                2: {
                    title: 'PHASE 2/3：深層超音波定位與血壓驟升',
                    hint: '目標：使用 [1] 超音波掃描深層組織，清除敗血感染與積血'
                },
                3: {
                    title: 'PHASE 3/3：敗血性休克控制與顯微縫合',
                    hint: '目標：排空所有積血與感染後，以 [6] 顯微縫合針閉合雙重裂傷'
                }
            };

            document.getElementById('wave-indicator').textContent = phaseMeta[phase].title;
            document.getElementById('objective-hint').textContent = phaseMeta[phase].hint;
            document.getElementById('wave-indicator-mobile').textContent = phaseMeta[phase].title;
            document.getElementById('objective-hint-mobile').textContent = phaseMeta[phase].hint;

            if (phase === 1) {
                for (let i = 0; i < 2; i++) {
                    const pt = randomPointInWound(0.62);
                    bleeders.push({ x: pt.x, y: pt.y, progress: 0, radius: 15, poolTimer: 0 });
                }
                const pPt = randomPointInWound(0.45);
                bloodPools.push({ x: pPt.x, y: pPt.y, radius: 34 });

                for (let i = 0; i < 2; i++) {
                    shrapnels.push(createShrapnel(false));
                }
            } else if (phase === 2) {
                showAlertBanner('臨床緊急警報 // PHASE 2', '檢測到深層隱匿彈片與敗血感染！請先使用 [1] 超音波掃描');
                screenShake = 14;

                for (let i = 0; i < 2; i++) {
                    shrapnels.push(createShrapnel(true));
                }
                for (let i = 0; i < 2; i++) {
                    const pt = randomPointInWound(0.65);
                    bleeders.push({ x: pt.x, y: pt.y, progress: 0, radius: 16, poolTimer: 0 });
                }
                for (let i = 0; i < 2; i++) {
                    const pt = randomPointInWound(0.58);
                    infections.push({ x: pt.x, y: pt.y, progress: 0, radius: 30 });
                }
            } else if (phase === 3) {
                showAlertBanner('最終縫合階段 // PHASE 3', '動脈再度破裂！清除積血與感染後立即縫合深層裂傷');
                screenShake = 16;

                for (let i = 0; i < 2; i++) {
                    const pt = randomPointInWound(0.6);
                    bleeders.push({ x: pt.x, y: pt.y, progress: 0, radius: 16, poolTimer: 0 });
                }
                const infPt = randomPointInWound(0.5);
                infections.push({ x: infPt.x, y: infPt.y, progress: 0, radius: 28 });

                const offsetY = woundRadius.ry * 0.34;
                [-offsetY, offsetY].forEach(dy => {
                    const len = woundRadius.rx * 0.8;
                    const nodes = [];
                    const count = 5;
                    for (let i = 0; i < count; i++) {
                        const t = (i + 0.5) / count;
                        const nx = (woundCenter.x - len / 2) + len * t;
                        const ny = woundCenter.y + dy + (i % 2 === 0 ? -14 : 14);
                        nodes.push({ x: nx, y: ny, stitched: false });
                    }
                    lacerations.push({
                        x1: woundCenter.x - len / 2,
                        y1: woundCenter.y + dy,
                        x2: woundCenter.x + len / 2,
                        y2: woundCenter.y + dy,
                        nodes,
                        currentNode: 0
                    });
                });
            }
        }

        function startGame() {
            audio.init();
            resizeCanvas();

            vitals = 100;
            spo2 = 99;
            bloodLossMl = 0;
            score = 0;
            combo = 0;
            maxCombo = 0;
            mistakes = 0;
            totalActions = 0;
            cleanActions = 0;
            healDoseCount = 0;
            timeLeft = TOTAL_TIME;

            cauteryHeat = 0;
            cauteryOverheated = false;
            focusGauge = 0;
            focusActive = false;
            focusTimer = 0;

            bleeders = [];
            bloodPools = [];
            shrapnels = [];
            infections = [];
            lacerations = [];
            charMarks = [];
            particles = [];
            popups = [];
            pointer.grabbedShrapnel = null;

            document.getElementById('focus-vignette').classList.add('hidden');
            document.getElementById('start-modal').classList.add('hidden');
            document.getElementById('end-modal').classList.add('hidden');
            document.getElementById('end-modal').classList.remove('flex');

            selectTool('cautery');
            setupPhase(1);
            gameState = 'PLAYING';
            lastFrameTime = performance.now();
            requestAnimationFrame(gameLoop);
        }

        document.getElementById('start-btn').addEventListener('click', startGame);
        document.getElementById('retry-btn').addEventListener('click', startGame);

        function triggerMistake(x, y, reason = 'TISSUE TRAUMA!') {
            mistakes++;
            totalActions++;
            combo = 0;
            vitals = Math.max(0, vitals - (difficultyMode === 'relaxed' ? 2.5 : 5.0));
            bloodLossMl += difficultyMode === 'relaxed' ? 15 : 30;
            screenShake = 15;
            audio.playError();
            spawnPopup(x, y, reason, '#f43f5e');
            spawnParticles(x, y, '#e11d48', 14, 160);
        }

        function rewardAction(x, y, basePoints, label = 'HEMOSTASIS!') {
            combo++;
            totalActions++;
            cleanActions++;
            if (combo > maxCombo) maxCombo = combo;

            const multiplier = 1 + Math.floor(combo / 3) * 0.25;
            score += Math.round(basePoints * multiplier);

            healDoseCount++;
            const decayFactor = Math.max(0.22, Math.pow(0.76, healDoseCount - 1));
            const baseHealDose = 8.5;
            const actualHeal = +(baseHealDose * decayFactor).toFixed(1);
            vitals = Math.min(100, vitals + actualHeal);

            if (!focusActive) {
                focusGauge = Math.min(100, focusGauge + 18);
            }

            audio.playSuccess();
            const doseTag = ` +${actualHeal}% (${Math.round(decayFactor * 100)}%)`;
            spawnPopup(x, y, (combo >= 3 ? `${label} x${combo}` : label) + doseTag, '#34d399');
        }

        function getEventPos(e) {
            const rect = canvas.getBoundingClientRect();
            const clientX = e.touches ? e.touches[0].clientX : e.clientX;
            const clientY = e.touches ? e.touches[0].clientY : e.clientY;
            return { x: clientX - rect.left, y: clientY - rect.top };
        }

        function handlePointerDown(e) {
            if (gameState !== 'PLAYING') return;
            audio.init();
            const pos = getEventPos(e);
            pointer.isDown = true;
            pointer.x = pos.x;
            pointer.y = pos.y;
            pointer.prevX = pos.x;
            pointer.prevY = pos.y;

            if (currentTool === 'forceps') {
                for (let i = shrapnels.length - 1; i >= 0; i--) {
                    const s = shrapnels[i];
                    if (s.scanProgress < 0.9) continue;
                    const dist = Math.hypot(pos.x - s.x, pos.y - s.y);
                    if (dist <= s.size + 15) {
                        pointer.grabbedShrapnel = s;
                        spawnParticles(s.x, s.y, '#22d3ee', 8, 90);
                        break;
                    }
                }
            }

            if (currentTool === 'suture') {
                processSutureInput();
            }
        }

        function handlePointerMove(e) {
            if (gameState !== 'PLAYING') return;
            const pos = getEventPos(e);
            pointer.prevX = pointer.x;
            pointer.prevY = pointer.y;
            pointer.x = pos.x;
            pointer.y = pos.y;

            if (pointer.isDown && currentTool === 'forceps' && pointer.grabbedShrapnel) {
                const s = pointer.grabbedShrapnel;
                const dx = pointer.x - pointer.prevX;
                const dy = pointer.y - pointer.prevY;
                const moveDist = Math.hypot(dx, dy);

                if (moveDist > 2) {
                    const maxSpeed = focusActive ? 55 : 34;
                    if (moveDist > maxSpeed) {
                        triggerMistake(s.x, s.y, '牽引過猛！撕裂血管');
                        pointer.grabbedShrapnel = null;
                        return;
                    }

                    const dragAngle = Math.atan2(dy, dx);
                    let angleDiff = Math.abs(dragAngle - s.extractAngle);
                    if (angleDiff > Math.PI) angleDiff = Math.PI * 2 - angleDiff;

                    if (angleDiff > 1.05 && moveDist > 5) {
                        if (!s.angleWarnCooldown || performance.now() - s.angleWarnCooldown > 650) {
                            s.angleWarnCooldown = performance.now();
                            triggerMistake(s.x, s.y, '角度偏離！請沿箭頭方向');
                            pointer.grabbedShrapnel = null;
                            return;
                        }
                    }
                }

                s.x = pointer.x;
                s.y = pointer.y;

                const normDist = Math.pow((s.x - woundCenter.x) / woundRadius.rx, 2) +
                                 Math.pow((s.y - woundCenter.y) / woundRadius.ry, 2);
                if (normDist > 0.90) {
                    const idx = shrapnels.indexOf(s);
                    if (idx !== -1) shrapnels.splice(idx, 1);
                    pointer.grabbedShrapnel = null;
                    spawnParticles(s.x, s.y, '#22d3ee', 20, 160);
                    rewardAction(s.x, s.y, 650, 'EXTRACTED! 異物摘除');
                }
            }

            if (pointer.isDown && currentTool === 'suture') {
                processSutureInput();
            }
        }

        function handlePointerUp() {
            pointer.isDown = false;
            pointer.grabbedShrapnel = null;
        }

        canvas.addEventListener('mousedown', handlePointerDown);
        window.addEventListener('mousemove', handlePointerMove);
        window.addEventListener('mouseup', handlePointerUp);

        canvas.addEventListener('touchstart', (e) => {
            e.preventDefault();
            handlePointerDown(e);
        }, { passive: false });
        window.addEventListener('touchmove', (e) => {
            if (gameState === 'PLAYING') e.preventDefault();
            handlePointerMove(e);
        }, { passive: false });
        window.addEventListener('touchend', handlePointerUp);

        function processSutureInput() {
            lacerations.forEach(lac => {
                if (lac.currentNode >= lac.nodes.length) return;
                const node = lac.nodes[lac.currentNode];
                const dist = Math.hypot(pointer.x - node.x, pointer.y - node.y);

                if (dist <= 24) {
                    if (bleeders.length > 0 || bloodPools.length > 0 || shrapnels.length > 0 || infections.length > 0) {
                        if (!lac.warnTime || performance.now() - lac.warnTime > 900) {
                            lac.warnTime = performance.now();
                            triggerMistake(node.x, node.y, '必須先止血並吸除積血！');
                        }
                        return;
                    }

                    node.stitched = true;
                    lac.currentNode++;
                    const isExact = dist <= 12;
                    audio.playStitch(isExact);
                    spawnParticles(node.x, node.y, '#34d399', 8, 95);

                    if (lac.currentNode >= lac.nodes.length) {
                        rewardAction(node.x, node.y, 950, 'PERFECT SUTURE!');
                    } else {
                        score += isExact ? 120 : 60;
                        spawnPopup(node.x, node.y - 16, isExact ? 'PERFECT!' : 'GOOD', isExact ? '#34d399' : '#a7f3d0');
                    }
                }
            });
        }

        function update(dt) {
            const timeScale = focusActive ? 0.35 : 1.0;
            const modeFactor = difficultyMode === 'relaxed' ? 0.45 : 1.0;

            if (focusActive) {
                focusTimer -= dt;
                if (focusTimer <= 0) {
                    focusActive = false;
                    document.getElementById('focus-vignette').classList.add('hidden');
                }
            }

            pulsePhase += dt * (vitals < 35 ? 8.5 : 4.2) * (focusActive ? 0.5 : 1);
            if (screenShake > 0) screenShake = Math.max(0, screenShake - dt * 45);

            timeLeft -= dt * timeScale;
            if (timeLeft <= 0) {
                timeLeft = 0;
                endGame(false, '手術超過黃金搶救時限！');
                return;
            }

            const poolInterval = difficultyMode === 'relaxed' ? 5.2 : 3.4;
            bleeders.forEach(b => {
                b.poolTimer += dt * timeScale;
                if (b.poolTimer >= poolInterval) {
                    b.poolTimer = 0;
                    let merged = false;
                    for (let pool of bloodPools) {
                        if (Math.hypot(pool.x - b.x, pool.y - b.y) < 65) {
                            pool.radius = Math.min(82, pool.radius + (difficultyMode === 'relaxed' ? 5 : 8));
                            merged = true;
                            break;
                        }
                    }
                    if (!merged && bloodPools.length < 6) {
                        bloodPools.push({ x: b.x + (Math.random() - 0.5) * 20, y: b.y + (Math.random() - 0.5) * 20, radius: 22 });
                    }
                }
            });

            const bleedRate = bleeders.length * 1.05 * modeFactor;
            const poolRate = bloodPools.length * 0.35 * modeFactor;
            const sepsisRate = infections.length * 0.7 * modeFactor;

            vitals -= (bleedRate + poolRate + sepsisRate) * dt * timeScale;
            bloodLossMl += (bleeders.length * 8.2 + bloodPools.length * 1.9) * modeFactor * dt * timeScale;

            const targetSpo2 = Math.max(72, Math.min(99, 99 - bloodPools.length * 2.5 - (100 - vitals) * 0.12));
            spo2 += (targetSpo2 - spo2) * dt * 2;

            if (vitals <= 0) {
                vitals = 0;
                endGame(false, '失血性休克，患者心跳停止...');
                return;
            }

            const isUsingCautery = pointer.isDown && currentTool === 'cautery' && !cauteryOverheated;
            if (isUsingCautery) {
                cauteryHeat = Math.min(100, cauteryHeat + dt * (focusActive ? 22 : 42));
                if (cauteryHeat >= 100) {
                    cauteryOverheated = true;
                    charMarks.push({ x: pointer.x, y: pointer.y, radius: 18 });
                    triggerMistake(pointer.x, pointer.y, '電燒過熱灼傷！');
                }
            } else {
                cauteryHeat = Math.max(0, cauteryHeat - dt * 48);
                if (cauteryOverheated && cauteryHeat <= 25) {
                    cauteryOverheated = false;
                }
            }

            if (pointer.isDown) {
                if (currentTool === 'scanner') {
                    audio.playScanner();
                    shrapnels.forEach(s => {
                        if (s.scanProgress < 1) {
                            const dist = Math.hypot(pointer.x - s.x, pointer.y - s.y);
                            if (dist <= 52) {
                                s.scanProgress = Math.min(1, s.scanProgress + dt * 1.35);
                                if (s.scanProgress >= 1) {
                                    spawnParticles(s.x, s.y, '#38bdf8', 14, 120);
                                    rewardAction(s.x, s.y, 250, 'TARGET LOCATED!');
                                }
                            }
                        }
                    });
                } else if (currentTool === 'suction') {
                    audio.playSuction();
                    spawnParticles(pointer.x, pointer.y, '#fb7185', 2, 65);
                    for (let i = bloodPools.length - 1; i >= 0; i--) {
                        const pool = bloodPools[i];
                        const dist = Math.hypot(pointer.x - pool.x, pointer.y - pool.y);
                        if (dist <= pool.radius + 16) {
                            pool.radius -= dt * 36;
                            if (pool.radius <= 8) {
                                bloodPools.splice(i, 1);
                                rewardAction(pool.x, pool.y, 220, 'FIELD CLEAR!');
                            }
                        }
                    }
                } else if (currentTool === 'cautery' && !cauteryOverheated) {
                    audio.playCautery(cauteryHeat / 100);
                    spawnParticles(pointer.x, pointer.y, '#fbbf24', 3, 85);
                    for (let i = bleeders.length - 1; i >= 0; i--) {
                        const b = bleeders[i];
                        const dist = Math.hypot(pointer.x - b.x, pointer.y - b.y);
                        if (dist <= b.radius + 14) {
                            b.progress += dt * 1.3;
                            if (b.progress >= 1) {
                                bleeders.splice(i, 1);
                                spawnParticles(b.x, b.y, '#fbbf24', 18, 140);
                                rewardAction(b.x, b.y, 400, 'HEMOSTASIS!');
                            }
                        }
                    }
                } else if (currentTool === 'antibiotic') {
                    audio.playAntibiotic();
                    spawnParticles(pointer.x, pointer.y, '#c084fc', 3, 80);
                    for (let i = infections.length - 1; i >= 0; i--) {
                        const inf = infections[i];
                        const dist = Math.hypot(pointer.x - inf.x, pointer.y - inf.y);
                        if (dist <= inf.radius + 16) {
                            inf.progress += dt * 1.15;
                            if (inf.progress >= 1) {
                                infections.splice(i, 1);
                                spawnParticles(inf.x, inf.y, '#a855f7', 16, 130);
                                rewardAction(inf.x, inf.y, 450, 'STERILIZED!');
                            }
                        }
                    }
                }
            }

            const allStitched = lacerations.every(l => l.currentNode >= l.nodes.length);
            if (bleeders.length === 0 && bloodPools.length === 0 && shrapnels.length === 0 && infections.length === 0 && allStitched) {
                if (currentPhase < 3) {
                    setupPhase(currentPhase + 1);
                } else {
                    endGame(true);
                    return;
                }
            }

            for (let i = particles.length - 1; i >= 0; i--) {
                const p = particles[i];
                p.x += p.vx * dt;
                p.y += p.vy * dt;
                p.life -= dt;
                if (p.life <= 0) particles.splice(i, 1);
            }

            for (let i = popups.length - 1; i >= 0; i--) {
                const pop = popups[i];
                pop.y += pop.vy * dt;
                pop.life -= dt;
                if (pop.life <= 0) popups.splice(i, 1);
            }

            updateHUD(dt);
        }

        function updateHUD(dt) {
            const vitalsInt = Math.ceil(vitals);
            const vitalsText = document.getElementById('vitals-text');
            const vitalsBar = document.getElementById('vitals-bar');
            const bpmText = document.getElementById('bpm-text');
            const topHud = document.getElementById('top-hud');
            const vignette = document.getElementById('danger-vignette');

            vitalsText.textContent = `${vitalsInt}%`;
            vitalsBar.style.width = `${vitals}%`;

            const sysBP = Math.max(65, Math.round(118 - (100 - vitals) * 0.48 + bleeders.length * 4));
            const diaBP = Math.max(40, Math.round(76 - (100 - vitals) * 0.32));
            document.getElementById('bp-text').textContent = `${sysBP}/${diaBP}`;
            document.getElementById('spo2-text').textContent = `${Math.round(spo2)}%`;
            document.getElementById('bloodloss-text').textContent = Math.round(bloodLossMl);

            const bpm = Math.round((74 + (100 - vitals) * 1.1 + bleeders.length * 7) * (focusActive ? 0.65 : 1));
            bpmText.textContent = bpm;

            if (vitals < 30) {
                vitalsText.className = 'font-hud text-sm sm:text-lg font-black text-rose-500 animate-pulse';
                vitalsBar.className = 'h-full bg-gradient-to-r from-rose-600 to-amber-500 rounded-full transition-all duration-150';
                bpmText.className = 'font-hud text-lg sm:text-2xl font-black text-rose-500 animate-pulse leading-none';
                topHud.classList.add('hud-panel-danger');
                vignette.classList.remove('hidden');
            } else if (vitals < 60) {
                vitalsText.className = 'font-hud text-sm sm:text-lg font-black text-amber-400';
                vitalsBar.className = 'h-full bg-gradient-to-r from-amber-500 to-yellow-400 rounded-full transition-all duration-150';
                bpmText.className = 'font-hud text-lg sm:text-2xl font-black text-amber-400 leading-none';
                topHud.classList.remove('hud-panel-danger');
                vignette.classList.add('hidden');
            } else {
                vitalsText.className = 'font-hud text-sm sm:text-lg font-black text-emerald-400';
                vitalsBar.className = 'h-full bg-gradient-to-r from-emerald-500 to-cyan-400 rounded-full transition-all duration-150';
                bpmText.className = 'font-hud text-lg sm:text-2xl font-black text-emerald-400 leading-none';
                topHud.classList.remove('hud-panel-danger');
                vignette.classList.add('hidden');
            }

            document.getElementById('blood-pool-warning').classList.toggle('hidden', bloodPools.length < 2);

            const heatHud = document.getElementById('heat-hud');
            if (currentTool === 'cautery' || cauteryHeat > 5) {
                heatHud.classList.remove('hidden');
                document.getElementById('heat-bar').style.width = `${cauteryHeat}%`;
                const heatPercent = document.getElementById('heat-percent');
                heatPercent.textContent = cauteryOverheated ? 'OVERHEAT!' : `${Math.round(cauteryHeat)}%`;
                heatPercent.className = cauteryOverheated ? 'font-hud text-rose-500 animate-pulse' : 'font-hud text-white';
            } else {
                heatHud.classList.add('hidden');
            }

            const focusBtn = document.getElementById('focus-btn');
            const focusFill = document.getElementById('focus-fill');
            const focusStatus = document.getElementById('focus-status-text');
            if (focusActive) {
                focusFill.style.width = `${(focusTimer / 6.0) * 100}%`;
                focusStatus.textContent = `ACTIVE ${focusTimer.toFixed(1)}s`;
                focusBtn.classList.add('focus-btn-ready');
                focusBtn.disabled = true;
            } else {
                focusFill.style.width = `${focusGauge}%`;
                if (focusGauge >= 100) {
                    focusStatus.textContent = 'READY [SPACE]';
                    focusBtn.classList.add('focus-btn-ready');
                    focusBtn.disabled = false;
                } else {
                    focusStatus.textContent = `${Math.round(focusGauge)}%`;
                    focusBtn.classList.remove('focus-btn-ready');
                    focusBtn.disabled = true;
                }
            }

            const mins = Math.floor(timeLeft / 60);
            const secs = Math.floor(timeLeft % 60);
            document.getElementById('timer-text').textContent = `${String(mins).padStart(2, '0')}:${String(secs).padStart(2, '0')}`;
            document.getElementById('score-text').textContent = String(score).padStart(5, '0');

            const comboBadge = document.getElementById('combo-badge');
            if (combo >= 2) {
                comboBadge.textContent = `x${combo}`;
                comboBadge.classList.remove('hidden');
            } else {
                comboBadge.classList.add('hidden');
            }

            const ecgW = ecgCanvas.getBoundingClientRect().width;
            const ecgH = ecgCanvas.getBoundingClientRect().height;
            const midY = ecgH / 2;
            const beatInterval = 60 / bpm;

            ecgTimer += dt;
            let newY = midY;
            const cyclePos = ecgTimer % beatInterval;

            if (cyclePos < 0.04) newY = midY - ecgH * 0.14;
            else if (cyclePos < 0.08) newY = midY + ecgH * 0.18;
            else if (cyclePos < 0.13) {
                newY = midY - ecgH * 0.43;
                if (!ecgCanvas.beatTriggered) {
                    ecgCanvas.beatTriggered = true;
                    audio.playHeartbeat(vitals < 30, focusActive);
                }
            } else if (cyclePos < 0.18) newY = midY + ecgH * 0.28;
            else {
                ecgCanvas.beatTriggered = false;
                newY = midY + Math.sin(ecgTimer * 14) * 1.1;
            }

            ecgPoints.push(newY);
            if (ecgPoints.length > ecgW) ecgPoints.shift();

            if (ecgCtx) {
                ecgCtx.clearRect(0, 0, ecgW, ecgH);
                ecgCtx.strokeStyle = focusActive ? '#38bdf8' : (vitals < 30 ? '#f43f5e' : '#34d399');
                ecgCtx.lineWidth = 2;
                ecgCtx.beginPath();
                for (let i = 0; i < ecgPoints.length; i++) {
                    if (i === 0) ecgCtx.moveTo(i, ecgPoints[i]);
                    else ecgCtx.lineTo(i, ecgPoints[i]);
                }
                ecgCtx.stroke();
            }
        }

        function render() {
            if (!ctx) return;
            ctx.save();
            ctx.clearRect(0, 0, width, height);

            if (screenShake > 0) {
                ctx.translate((Math.random() - 0.5) * screenShake, (Math.random() - 0.5) * screenShake);
            }

            ctx.fillStyle = '#06151c';
            ctx.fillRect(0, 0, width, height);

            ctx.strokeStyle = 'rgba(20, 184, 166, 0.07)';
            ctx.lineWidth = 1;
            for (let x = 0; x < width; x += 36) {
                ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, height); ctx.stroke();
            }
            for (let y = 0; y < height; y += 36) {
                ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(width, y); ctx.stroke();
            }

            const pulseOffset = Math.sin(pulsePhase) * 3.5;
            const rx = woundRadius.rx + pulseOffset;
            const ry = woundRadius.ry + pulseOffset * 0.7;

            ctx.beginPath();
            ctx.ellipse(woundCenter.x, woundCenter.y, rx + 34, ry + 34, 0, 0, Math.PI * 2);
            ctx.fillStyle = '#0d2b36';
            ctx.fill();
            ctx.lineWidth = 3;
            ctx.strokeStyle = 'rgba(45, 212, 191, 0.35)';
            ctx.stroke();

            ctx.beginPath();
            ctx.ellipse(woundCenter.x, woundCenter.y, rx + 20, ry + 20, 0, 0, Math.PI * 2);
            ctx.fillStyle = '#d99b82';
            ctx.fill();

            ctx.beginPath();
            ctx.ellipse(woundCenter.x, woundCenter.y, rx + 10, ry + 10, 0, 0, Math.PI * 2);
            ctx.fillStyle = '#eab308';
            ctx.fill();

            ctx.save();
            const muscleGrad = ctx.createRadialGradient(
                woundCenter.x - rx * 0.2, woundCenter.y - ry * 0.2, rx * 0.1,
                woundCenter.x, woundCenter.y, rx
            );
            muscleGrad.addColorStop(0, '#991b1b');
            muscleGrad.addColorStop(0.65, '#7f1d1d');
            muscleGrad.addColorStop(1, '#450a0a');

            ctx.beginPath();
            ctx.ellipse(woundCenter.x, woundCenter.y, rx, ry, 0, 0, Math.PI * 2);
            ctx.fillStyle = muscleGrad;
            ctx.fill();

            ctx.lineWidth = 6;
            ctx.strokeStyle = 'rgba(69, 10, 10, 0.85)';
            ctx.stroke();

            ctx.clip();

            muscleFibers.forEach(fiber => {
                ctx.beginPath();
                const y = woundCenter.y + fiber.yOffset + Math.sin(pulsePhase + fiber.yOffset * 0.05) * 2;
                ctx.moveTo(woundCenter.x - rx, y);
                ctx.quadraticCurveTo(woundCenter.x, y + fiber.curve, woundCenter.x + rx, y);
                ctx.strokeStyle = `rgba(254, 202, 202, ${fiber.alpha})`;
                ctx.lineWidth = 3;
                ctx.stroke();
            });

            const specGrad = ctx.createRadialGradient(
                woundCenter.x - rx * 0.35, woundCenter.y - ry * 0.35, 5,
                woundCenter.x - rx * 0.35, woundCenter.y - ry * 0.35, rx * 0.55
            );
            specGrad.addColorStop(0, 'rgba(255, 255, 255, 0.14)');
            specGrad.addColorStop(1, 'rgba(255, 255, 255, 0)');
            ctx.fillStyle = specGrad;
            ctx.fillRect(woundCenter.x - rx, woundCenter.y - ry, rx * 2, ry * 2);

            charMarks.forEach(c => {
                ctx.beginPath();
                ctx.arc(c.x, c.y, c.radius, 0, Math.PI * 2);
                ctx.fillStyle = 'rgba(15, 23, 42, 0.65)';
                ctx.fill();
            });

            lacerations.forEach(lac => drawLaceration(lac));
            infections.forEach(inf => drawInfection(inf));

            bloodPools.forEach(pool => {
                const poolGrad = ctx.createRadialGradient(pool.x, pool.y, 4, pool.x, pool.y, pool.radius);
                poolGrad.addColorStop(0, 'rgba(136, 19, 55, 0.92)');
                poolGrad.addColorStop(0.75, 'rgba(159, 18, 57, 0.78)');
                poolGrad.addColorStop(1, 'rgba(159, 18, 57, 0)');
                ctx.beginPath();
                ctx.arc(pool.x, pool.y, pool.radius, 0, Math.PI * 2);
                ctx.fillStyle = poolGrad;
                ctx.fill();
            });

            bleeders.forEach(b => drawBleeder(b));

            ctx.restore();

            drawRetractors(rx, ry);
            shrapnels.forEach(s => drawShrapnel(s));

            particles.forEach(p => {
                ctx.globalAlpha = Math.max(0, p.life * 1.6);
                ctx.fillStyle = p.color;
                ctx.beginPath();
                ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
                ctx.fill();
            });
            ctx.globalAlpha = 1;

            const spotlight = ctx.createRadialGradient(
                woundCenter.x, woundCenter.y, rx * 0.85,
                woundCenter.x, woundCenter.y, Math.max(width, height) * 0.75
            );
            spotlight.addColorStop(0, 'rgba(0,0,0,0)');
            spotlight.addColorStop(1, 'rgba(2, 6, 15, 0.55)');
            ctx.fillStyle = spotlight;
            ctx.fillRect(0, 0, width, height);

            drawToolCursor();

            popups.forEach(pop => {
                ctx.save();
                ctx.globalAlpha = Math.min(1, pop.life * 1.4);
                ctx.font = '900 18px "Orbitron", "Noto Sans TC", sans-serif';
                ctx.textAlign = 'center';
                ctx.fillStyle = '#020617';
                ctx.fillText(pop.text, pop.x + 1.5, pop.y + 1.5);
                ctx.fillStyle = pop.color;
                ctx.fillText(pop.text, pop.x, pop.y);
                ctx.restore();
            });

            ctx.restore();
        }

        function drawRetractors(rx, ry) {
            const angles = [0, Math.PI / 2, Math.PI, (Math.PI * 3) / 2];
            angles.forEach(ang => {
                const x = woundCenter.x + Math.cos(ang) * (rx + 12);
                const y = woundCenter.y + Math.sin(ang) * (ry + 12);
                ctx.save();
                ctx.translate(x, y);
                ctx.rotate(ang);
                ctx.fillStyle = '#94a3b8';
                ctx.strokeStyle = '#334155';
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.roundRect(-8, -18, 28, 36, 4);
                ctx.fill();
                ctx.stroke();
                ctx.restore();
            });
        }

        function drawBleeder(b) {
            const pulse = Math.sin(pulsePhase * 1.8) * 4;
            ctx.beginPath();
            ctx.arc(b.x, b.y, b.radius + pulse + 5, 0, Math.PI * 2);
            ctx.fillStyle = 'rgba(244, 63, 94, 0.35)';
            ctx.fill();

            ctx.beginPath();
            ctx.arc(b.x, b.y, b.radius * (1 - b.progress * 0.45), 0, Math.PI * 2);
            ctx.fillStyle = '#f43f5e';
            ctx.fill();
            ctx.lineWidth = 2;
            ctx.strokeStyle = '#ffe4e6';
            ctx.stroke();

            if (b.progress > 0) {
                ctx.beginPath();
                ctx.arc(b.x, b.y, b.radius + 8, -Math.PI / 2, -Math.PI / 2 + Math.PI * 2 * b.progress);
                ctx.strokeStyle = '#fbbf24';
                ctx.lineWidth = 4;
                ctx.stroke();
            }
        }

        function drawShrapnel(s) {
            if (s.scanProgress < 1) {
                ctx.save();
                ctx.beginPath();
                ctx.arc(s.x, s.y, s.size + 4, 0, Math.PI * 2);
                ctx.strokeStyle = `rgba(56, 189, 248, ${0.2 + s.scanProgress * 0.6})`;
                ctx.setLineDash([4, 4]);
                ctx.lineWidth = 2;
                ctx.stroke();

                if (s.scanProgress > 0) {
                    ctx.beginPath();
                    ctx.arc(s.x, s.y, s.size + 8, -Math.PI / 2, -Math.PI / 2 + Math.PI * 2 * s.scanProgress);
                    ctx.setLineDash([]);
                    ctx.strokeStyle = '#38bdf8';
                    ctx.lineWidth = 3;
                    ctx.stroke();
                }
                ctx.restore();
                return;
            }

            ctx.save();
            ctx.translate(s.x, s.y);

            const arrowLen = 48;
            const ax = Math.cos(s.extractAngle) * arrowLen;
            const ay = Math.sin(s.extractAngle) * arrowLen;

            ctx.beginPath();
            ctx.moveTo(0, 0);
            ctx.lineTo(ax, ay);
            ctx.strokeStyle = pointer.grabbedShrapnel === s ? '#34d399' : 'rgba(52, 211, 153, 0.75)';
            ctx.lineWidth = 3;
            ctx.stroke();

            ctx.save();
            ctx.translate(ax, ay);
            ctx.rotate(s.extractAngle);
            ctx.beginPath();
            ctx.moveTo(0, 0);
            ctx.lineTo(-10, -5);
            ctx.lineTo(-10, 5);
            ctx.closePath();
            ctx.fillStyle = '#34d399';
            ctx.fill();
            ctx.restore();

            ctx.rotate(s.angle);
            if (pointer.grabbedShrapnel === s) {
                ctx.shadowColor = '#22d3ee';
                ctx.shadowBlur = 18;
            }

            ctx.beginPath();
            ctx.moveTo(-s.size, -s.size * 0.4);
            ctx.lineTo(0, -s.size * 0.9);
            ctx.lineTo(s.size * 0.9, -s.size * 0.2);
            ctx.lineTo(s.size * 0.5, s.size * 0.8);
            ctx.lineTo(-s.size * 0.6, s.size * 0.6);
            ctx.closePath();

            const metalGrad = ctx.createLinearGradient(-s.size, -s.size, s.size, s.size);
            metalGrad.addColorStop(0, '#f1f5f9');
            metalGrad.addColorStop(0.5, '#64748b');
            metalGrad.addColorStop(1, '#1e293b');

            ctx.fillStyle = metalGrad;
            ctx.fill();
            ctx.lineWidth = 2;
            ctx.strokeStyle = pointer.grabbedShrapnel === s ? '#22d3ee' : '#cbd5e1';
            ctx.stroke();

            ctx.restore();
        }

        function drawInfection(inf) {
            const r = (inf.radius + Math.cos(pulsePhase) * 3) * (1 - inf.progress * 0.4);
            const grad = ctx.createRadialGradient(inf.x, inf.y, 2, inf.x, inf.y, r);
            grad.addColorStop(0, 'rgba(168, 85, 247, 0.88)');
            grad.addColorStop(0.7, 'rgba(126, 34, 206, 0.52)');
            grad.addColorStop(1, 'rgba(88, 28, 135, 0)');

            ctx.beginPath();
            ctx.arc(inf.x, inf.y, r, 0, Math.PI * 2);
            ctx.fillStyle = grad;
            ctx.fill();

            if (inf.progress > 0) {
                ctx.beginPath();
                ctx.arc(inf.x, inf.y, inf.radius + 4, -Math.PI / 2, -Math.PI / 2 + Math.PI * 2 * inf.progress);
                ctx.strokeStyle = '#d8b4fe';
                ctx.lineWidth = 3.5;
                ctx.stroke();
            }
        }

        function drawLaceration(lac) {
            const isClosed = lac.currentNode >= lac.nodes.length;
            ctx.save();
            ctx.beginPath();
            ctx.moveTo(lac.x1, lac.y1);
            ctx.lineTo(lac.x2, lac.y2);
            ctx.lineCap = 'round';
            ctx.lineWidth = isClosed ? 3 : 14;
            ctx.strokeStyle = isClosed ? '#fca5a5' : '#3b0707';
            ctx.stroke();

            lac.nodes.forEach((node, i) => {
                if (node.stitched) {
                    ctx.beginPath();
                    ctx.moveTo(node.x - 5, node.y - 12);
                    ctx.lineTo(node.x + 5, node.y + 12);
                    ctx.strokeStyle = '#34d399';
                    ctx.lineWidth = 2.5;
                    ctx.stroke();
                } else if (i === lac.currentNode) {
                    ctx.beginPath();
                    ctx.arc(node.x, node.y, 9 + Math.sin(pulsePhase * 2) * 2, 0, Math.PI * 2);
                    ctx.strokeStyle = '#22d3ee';
                    ctx.lineWidth = 2.5;
                    ctx.stroke();

                    ctx.beginPath();
                    ctx.arc(node.x, node.y, 3, 0, Math.PI * 2);
                    ctx.fillStyle = '#22d3ee';
                    ctx.fill();
                } else {
                    ctx.beginPath();
                    ctx.arc(node.x, node.y, 4, 0, Math.PI * 2);
                    ctx.fillStyle = 'rgba(255, 255, 255, 0.35)';
                    ctx.fill();
                }
            });
            ctx.restore();
        }

        function drawToolCursor() {
            if (pointer.x === 0 && pointer.y === 0) return;
            ctx.save();

            if (currentTool === 'scanner') {
                ctx.strokeStyle = '#38bdf8';
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.arc(pointer.x, pointer.y, pointer.isDown ? 46 : 32, 0, Math.PI * 2);
                ctx.stroke();
                if (pointer.isDown) {
                    ctx.fillStyle = 'rgba(56, 189, 248, 0.14)';
                    ctx.fill();
                }
            } else if (currentTool === 'forceps') {
                ctx.strokeStyle = '#22d3ee';
                ctx.lineWidth = 2.5;
                ctx.beginPath();
                ctx.arc(pointer.x, pointer.y, 13, 0, Math.PI * 2);
                ctx.stroke();
            } else if (currentTool === 'suction') {
                ctx.strokeStyle = '#fb7185';
                ctx.lineWidth = 2.5;
                ctx.beginPath();
                ctx.arc(pointer.x, pointer.y, pointer.isDown ? 12 : 18, 0, Math.PI * 2);
                ctx.stroke();
            } else if (currentTool === 'cautery') {
                ctx.strokeStyle = cauteryOverheated ? '#f43f5e' : '#fbbf24';
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.arc(pointer.x, pointer.y, pointer.isDown ? 9 : 15, 0, Math.PI * 2);
                ctx.stroke();

                if (pointer.isDown && !cauteryOverheated) {
                    ctx.beginPath();
                    ctx.moveTo(width * 0.88, 0);
                    ctx.lineTo(pointer.x, pointer.y);
                    ctx.strokeStyle = 'rgba(251, 191, 36, 0.8)';
                    ctx.lineWidth = 3.5;
                    ctx.stroke();
                }
            } else if (currentTool === 'antibiotic') {
                ctx.strokeStyle = '#c084fc';
                ctx.setLineDash([4, 4]);
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.arc(pointer.x, pointer.y, 24, 0, Math.PI * 2);
                ctx.stroke();
                ctx.setLineDash([]);
            } else if (currentTool === 'suture') {
                ctx.strokeStyle = '#34d399';
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.moveTo(pointer.x - 9, pointer.y - 9);
                ctx.lineTo(pointer.x + 9, pointer.y + 9);
                ctx.moveTo(pointer.x + 9, pointer.y - 9);
                ctx.lineTo(pointer.x - 9, pointer.y + 9);
                ctx.stroke();
            }

            drawMiniCursorBadge(pointer.x + 22, pointer.y + 22);
            ctx.restore();
        }

        function drawMiniCursorBadge(bx, by) {
            const toolConfigs = {
                scanner:    { color: '#38bdf8', bg: 'rgba(8, 47, 73, 0.92)' },
                forceps:    { color: '#22d3ee', bg: 'rgba(8, 51, 68, 0.92)' },
                suction:    { color: '#fb7185', bg: 'rgba(76, 5, 25, 0.92)' },
                cautery:    { color: cauteryOverheated ? '#f43f5e' : '#fbbf24', bg: 'rgba(69, 26, 3, 0.92)' },
                antibiotic: { color: '#c084fc', bg: 'rgba(59, 7, 100, 0.92)' },
                suture:     { color: '#34d399', bg: 'rgba(6, 78, 59, 0.92)' }
            };
            const cfg = toolConfigs[currentTool];
            if (!cfg) return;

            ctx.save();
            ctx.translate(bx, by);
            ctx.fillStyle = cfg.bg;
            ctx.strokeStyle = cfg.color;
            ctx.lineWidth = 1.5;
            ctx.beginPath();
            ctx.roundRect(-14, -14, 28, 28, 7);
            ctx.fill();
            ctx.stroke();

            ctx.strokeStyle = cfg.color;
            ctx.fillStyle = cfg.color;
            ctx.lineWidth = 1.8;
            ctx.lineCap = 'round';

            if (currentTool === 'scanner') {
                ctx.fillRect(-4, -8, 8, 6);
                ctx.beginPath();
                ctx.arc(0, 1, 6, 0.25 * Math.PI, 0.75 * Math.PI);
                ctx.stroke();
            } else if (currentTool === 'forceps') {
                ctx.beginPath();
                ctx.moveTo(7, -7);
                ctx.lineTo(-6, 7);
                ctx.moveTo(7, -7);
                ctx.lineTo(-8, 4);
                ctx.stroke();
            } else if (currentTool === 'suction') {
                ctx.beginPath();
                ctx.moveTo(6, -7);
                ctx.quadraticCurveTo(8, -1, 2, 2);
                ctx.lineTo(-6, 7);
                ctx.stroke();
            } else if (currentTool === 'cautery') {
                ctx.beginPath();
                ctx.moveTo(7, -7);
                ctx.lineTo(-7, 7);
                ctx.stroke();
            } else if (currentTool === 'antibiotic') {
                ctx.beginPath();
                ctx.moveTo(6, -6);
                ctx.lineTo(-2, 2);
                ctx.lineWidth = 3.5;
                ctx.stroke();
            } else if (currentTool === 'suture') {
                ctx.beginPath();
                ctx.arc(0, 0, 6.5, Math.PI * 0.85, Math.PI * 1.9);
                ctx.stroke();
            }

            ctx.restore();
        }

        function endGame(isVictory, failReason = '') {
            gameState = 'ENDED';
            focusActive = false;
            document.getElementById('danger-vignette').classList.add('hidden');
            document.getElementById('focus-vignette').classList.add('hidden');

            const endModal = document.getElementById('end-modal');
            const endTitle = document.getElementById('end-title');
            const endRank = document.getElementById('end-rank');

            endModal.classList.remove('hidden');
            endModal.classList.add('flex');

            const accuracy = totalActions > 0 ? Math.round((cleanActions / totalActions) * 100) : 100;
            const elapsed = TOTAL_TIME - timeLeft;
            const mins = Math.floor(elapsed / 60);
            const secs = Math.floor(elapsed % 60);

            if (isVictory) {
                audio.playSuccess();
                score += Math.round(timeLeft * 30 + vitals * 25);
                endTitle.textContent = '手術圓滿成功！患者生命徵象穩定';

                let rank = 'C';
                let rankClass = 'text-slate-300';
                if (vitals >= 80 && mistakes === 0 && accuracy >= 95) {
                    rank = 'S';
                    rankClass = 'text-cyan-400 drop-shadow-[0_0_25px_rgba(34,211,238,0.95)]';
                } else if (vitals >= 60 && mistakes <= 2) {
                    rank = 'A';
                    rankClass = 'text-emerald-400 drop-shadow-[0_0_20px_rgba(52,211,153,0.8)]';
                } else if (vitals >= 30) {
                    rank = 'B';
                    rankClass = 'text-amber-400';
                }
                lastRank = rank;
                endRank.textContent = rank;
                endRank.className = `font-hud text-5xl sm:text-6xl font-black ${rankClass}`;
            } else {
                audio.playError();
                endTitle.textContent = failReason || '手術失敗！患者搶救無效';
                lastRank = 'F';
                endRank.textContent = 'F';
                endRank.className = 'font-hud text-5xl sm:text-6xl font-black text-rose-500 drop-shadow-[0_0_20px_rgba(244,63,94,0.8)]';
            }

            document.getElementById('end-accuracy').textContent = `${accuracy}%`;
            document.getElementById('end-combo').textContent = maxCombo;
            document.getElementById('end-bloodloss').textContent = `${Math.round(bloodLossMl)} mL`;
            document.getElementById('end-score').textContent = score.toLocaleString();
            document.getElementById('end-vitals').textContent = `${Math.ceil(vitals)}%`;
            document.getElementById('end-time').textContent = `${String(mins).padStart(2, '0')}:${String(secs).padStart(2, '0')}`;
            document.getElementById('end-mistakes').textContent = mistakes;
        }

        function gameLoop(now) {
            if (gameState !== 'PLAYING') return;
            const dt = Math.min(0.05, (now - lastFrameTime) / 1000);
            lastFrameTime = now;

            update(dt);
            render();

            requestAnimationFrame(gameLoop);
        }
    </script>
</body>
</html>
