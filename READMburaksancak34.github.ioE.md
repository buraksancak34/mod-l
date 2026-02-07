<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Destekli Görüşme Paneli</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700;800&display=swap');

        /* WordPress İzolasyonu */
        .wp-app-wrapper {
            font-family: 'Inter', sans-serif;
            background-color: #0f172a; /* Slate 900 */
            color: #f8fafc;
            margin: 0;
            padding: 0;
            min-height: 100vh;
            width: 100%;
            box-sizing: border-box;
            overflow-x: hidden;
        }

        .wp-app-wrapper * {
            box-sizing: border-box;
        }

        /* Glassmorphism Kartlar */
        .glass-panel {
            background: rgba(30, 41, 59, 0.7); /* Slate 800 + Opacity */
            border: 1px solid rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(12px);
            transition: transform 0.3s ease, border-color 0.3s ease;
        }
        
        .glass-panel:hover {
            border-color: rgba(99, 102, 241, 0.5); /* Indigo 500 */
            transform: translateY(-2px);
        }

        /* AI Gradient Butonlar */
        .btn-ai {
            background: linear-gradient(135deg, #4f46e5 0%, #9333ea 100%);
            color: white;
            transition: all 0.3s ease;
            box-shadow: 0 4px 15px rgba(79, 70, 229, 0.4);
        }
        
        .btn-ai:hover {
            box-shadow: 0 6px 20px rgba(79, 70, 229, 0.6);
            transform: scale(1.02);
            filter: brightness(1.1);
        }

        /* Yükleniyor Animasyonu */
        @keyframes shimmer {
            0% { background-position: -200% 0; }
            100% { background-position: 200% 0; }
        }
        .loading-shimmer {
            background: linear-gradient(90deg, #1e293b 25%, #334155 50%, #1e293b 75%);
            background-size: 200% 100%;
            animation: shimmer 1.5s infinite;
            border-radius: 0.5rem;
        }

        /* Admin Sidebar */
        .admin-sidebar {
            transition: transform 0.4s cubic-bezier(0.4, 0, 0.2, 1);
            box-shadow: -10px 0 30px rgba(0,0,0,0.5);
            z-index: 50;
        }

        /* Chart Container */
        .chart-wrapper {
            position: relative;
            width: 100%;
            height: 350px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .custom-input {
            background: #1e293b;
            border: 1px solid #334155;
            color: white;
            padding: 8px;
            border-radius: 6px;
            width: 100%;
            font-size: 14px;
        }

        /* Scrollbar */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: #0f172a; }
        ::-webkit-scrollbar-thumb { background: #334155; border-radius: 10px; }
    </style>

    <!-- Placeholder Comments -->
    <!-- Chosen Palette: Dark Mode (Slate 900 background, Indigo/Violet accents for AI, Emerald for success) -->
    <!-- Application Structure Plan: 
         1. Header: Title and AI Action Buttons.
         2. AI Result Area: Collapsible section for Gemini outputs.
         3. Dashboard Grid:
            - Left: Summary Stats & Radar Chart (360 view).
            - Right/Bottom: Category Cards with progress bars.
         4. Admin Sidebar: Slide-out panel for live data editing.
    -->
    <!-- Visualization & Content Choices:
         - Chart.js Radar Chart: Perfectly suits the "360 degree" request to show balance across categories.
         - Progress Cards: Clear, linear tracking for individual goals.
         - AI Integration: Uses Gemini API for qualitative analysis of the quantitative data.
    -->
    <!-- CONFIRMATION: NO SVG graphics used (Chart.js canvas + Unicode). NO Mermaid JS used. -->
</head>
<body class="wp-app-wrapper">

    <!-- Admin Sidebar -->
    <div id="admin-panel" class="fixed top-0 right-0 h-full w-80 bg-slate-900 border-l border-slate-700 p-6 admin-sidebar translate-x-full overflow-y-auto">
        <div class="flex justify-between items-center mb-6">
            <h2 class="text-xl font-bold text-white flex items-center gap-2">⚙️ Veri Yönetimi</h2>
            <button onclick="toggleAdmin()" class="text-slate-400 hover:text-white text-2xl">&times;</button>
        </div>
        
        <div id="editor-list" class="space-y-4">
            <!-- JS ile doldurulacak -->
        </div>

        <div class="mt-8 pt-6 border-t border-slate-700 space-y-3">
            <button onclick="addNewCategory()" class="w-full py-2 bg-slate-800 hover:bg-slate-700 text-slate-300 rounded-lg text-sm font-bold transition-colors">
                + Yeni Kategori
            </button>
            <button onclick="copyCurrentCode()" class="w-full py-2 bg-indigo-600 hover:bg-indigo-500 text-white rounded-lg text-sm font-bold transition-colors">
                Kodu Kopyala
            </button>
        </div>
    </div>

    <div class="max-w-7xl mx-auto p-4 md:p-8">
        
        <!-- Header & AI Controls -->
        <header class="flex flex-col md:flex-row justify-between items-center mb-10 gap-6">
            <div class="text-center md:text-left">
                <h1 class="text-3xl md:text-4xl font-black text-white tracking-tight mb-2">
                    360° Görüşme Paneli
                </h1>
                <p class="text-slate-400 text-sm md:text-base">
                    Stratejik İletişim Ağı ve İlerleme Takibi
                </p>
            </div>
            
            <div class="flex flex-wrap justify-center gap-3">
                <button onclick="generateAiStrategy()" id="btn-strategy" class="btn-ai px-6 py-3 rounded-full font-bold text-sm flex items-center gap-2">
                    <span>✨ Strateji Üret</span>
                </button>
                <button onclick="generateOutreach()" id="btn-draft" class="bg-slate-800 border border-slate-600 hover:bg-slate-700 text-white px-6 py-3 rounded-full font-bold text-sm transition-all flex items-center gap-2">
                    <span>📝 Taslak Hazırla</span>
                </button>
                <button onclick="toggleAdmin()" class="bg-slate-800 border border-slate-600 hover:bg-slate-700 text-indigo-400 px-4 py-3 rounded-full font-bold text-sm transition-all">
                    🛠️ Düzenle
                </button>
            </div>
        </header>

        <!-- AI Output Section -->
        <div id="ai-container" class="hidden mb-10">
            <div class="glass-panel rounded-2xl p-6 border-l-4 border-indigo-500 relative">
                <button onclick="closeAi()" class="absolute top-4 right-4 text-slate-500 hover:text-white">&times;</button>
                <h3 class="text-indigo-400 font-bold uppercase text-xs tracking-widest mb-4 flex items-center gap-2">
                    <span class="text-lg">✨</span> Gemini AI Analizi
                </h3>
                <div id="ai-content" class="text-slate-300 text-sm leading-relaxed whitespace-pre-wrap"></div>
            </div>
        </div>

        <!-- Dashboard Layout -->
        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8 mb-10">
            
            <!-- Left Column: Radar Chart & Summary -->
            <div class="lg:col-span-1 space-y-6">
                <!-- Summary Stats -->
                <div class="grid grid-cols-2 gap-4">
                    <div class="glass-panel p-4 rounded-xl text-center">
                        <p class="text-slate-500 text-[10px] font-bold uppercase">Genel İlerleme</p>
                        <p id="total-percent" class="text-3xl font-black text-white mt-1">%0</p>
                    </div>
                    <div class="glass-panel p-4 rounded-xl text-center">
                        <p class="text-slate-500 text-[10px] font-bold uppercase">Kalan Hedef</p>
                        <p id="total-remaining" class="text-3xl font-black text-indigo-400 mt-1">0</p>
                    </div>
                </div>

                <!-- Radar Chart -->
                <div class="glass-panel p-4 rounded-2xl flex flex-col items-center">
                    <h3 class="text-slate-400 text-xs font-bold uppercase mb-4 w-full text-center">Ağ Dengesi (360°)</h3>
                    <div class="chart-wrapper">
                        <canvas id="radarChart"></canvas>
                    </div>
                </div>
            </div>

            <!-- Right Column: Category Cards -->
            <div class="lg:col-span-2">
                <div id="cards-grid" class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <!-- Cards will be injected here -->
                </div>
            </div>
        </div>

    </div>

    <script>
        // --- Gemini API Configuration ---
        // Not: Güvenlik için API anahtarınızı buraya ekleyin.
        const apiKey = ""; 

        // --- Data State ---
        let data = [
            { id: 1, title: 'Hocalar', completed: 11, total: 11, color: 'indigo', emoji: '👨‍🏫' },
            { id: 2, title: 'STK (Sivil Toplum)', completed: 5, total: 6, color: 'emerald', emoji: '🌳' },
            { id: 3, title: 'Bürokratlar', completed: 4, total: 4, color: 'blue', emoji: '🏛️' },
            { id: 4, title: 'Siyasi Partiler', completed: 2, total: 6, color: 'violet', emoji: '💼' },
            { id: 5, title: 'Enerji Sektörü', completed: 1, total: 4, color: 'orange', emoji: '⚡' },
            { id: 6, title: 'Gazeteciler', completed: 0, total: 2, color: 'rose', emoji: '🎤' }
        ];

        let radarChart = null;

        // --- Core Functions ---

        function init() {
            renderDashboard();
            renderAdmin();
            initChart();
        }

        function renderDashboard() {
            const grid = document.getElementById('cards-grid');
            grid.innerHTML = '';
            
            let totalDone = 0;
            let totalTarget = 0;

            data.forEach(item => {
                totalDone += item.completed;
                totalTarget += item.total;
                const perc = item.total > 0 ? Math.round((item.completed / item.total) * 100) : 0;
                const isFull = perc >= 100;

                // Progress Icons
                let iconsHtml = '';
                for(let i=0; i<item.total; i++) {
                    const opacity = i < item.completed ? 'opacity-100' : 'opacity-20 grayscale';
                    iconsHtml += `<span class="text-xl ${opacity}">${item.emoji}</span>`;
                }

                const card = document.createElement('div');
                card.className = 'glass-panel p-5 rounded-2xl flex flex-col justify-between';
                card.innerHTML = `
                    <div class="flex justify-between items-start mb-4">
                        <div>
                            <h3 class="font-bold text-white text-base">${item.title}</h3>
                            <p class="text-[10px] text-slate-400 font-bold uppercase mt-1">${item.completed} / ${item.total} KİŞİ</p>
                        </div>
                        <span class="text-2xl font-black ${isFull ? 'text-emerald-400' : 'text-indigo-400'}">${perc}%</span>
                    </div>
                    <div class="flex flex-wrap gap-1 mb-4 min-h-[30px]">${iconsHtml}</div>
                    <div class="w-full bg-slate-800 h-1.5 rounded-full overflow-hidden">
                        <div class="h-full ${isFull ? 'bg-emerald-500' : 'bg-indigo-500'} transition-all duration-1000" style="width: ${perc}%"></div>
                    </div>
                `;
                grid.appendChild(card);
            });

            // Update Summaries
            const overallPerc = totalTarget > 0 ? Math.round((totalDone / totalTarget) * 100) : 0;
            document.getElementById('total-percent').innerText = `%${overallPerc}`;
            document.getElementById('total-remaining').innerText = totalTarget - totalDone;

            // Update Chart if exists
            if (radarChart) updateChart();
        }

        // --- Chart.js Logic ---
        function initChart() {
            const ctx = document.getElementById('radarChart').getContext('2d');
            
            Chart.defaults.color = '#94a3b8';
            Chart.defaults.font.family = 'Inter';

            radarChart = new Chart(ctx, {
                type: 'radar',
                data: getChartData(),
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        r: {
                            angleLines: { color: 'rgba(255, 255, 255, 0.1)' },
                            grid: { color: 'rgba(255, 255, 255, 0.1)' },
                            pointLabels: {
                                font: { size: 11, weight: 'bold' },
                                color: '#f8fafc'
                            },
                            ticks: { display: false, backdropColor: 'transparent' },
                            suggestedMin: 0,
                            suggestedMax: 100
                        }
                    },
                    plugins: {
                        legend: { display: false },
                        tooltip: {
                            backgroundColor: 'rgba(15, 23, 42, 0.9)',
                            titleColor: '#fff',
                            bodyColor: '#cbd5e1',
                            padding: 10,
                            cornerRadius: 8,
                            callbacks: {
                                label: function(context) {
                                    return context.parsed.r + '% Tamamlandı';
                                }
                            }
                        }
                    }
                }
            });
        }

        function getChartData() {
            return {
                labels: data.map(d => d.title),
                datasets: [{
                    label: 'İlerleme',
                    data: data.map(d => d.total > 0 ? Math.round((d.completed / d.total) * 100) : 0),
                    backgroundColor: 'rgba(99, 102, 241, 0.2)', // Indigo transparent
                    borderColor: '#818cf8', // Indigo 400
                    pointBackgroundColor: '#fff',
                    pointBorderColor: '#6366f1',
                    borderWidth: 2,
                    pointRadius: 3
                }]
            };
        }

        function updateChart() {
            radarChart.data = getChartData();
            radarChart.update();
        }

        // --- AI Logic (Gemini API) ---
        async function callGemini(prompt) {
            const container = document.getElementById('ai-container');
            const content = document.getElementById('ai-content');
            
            container.classList.remove('hidden');
            content.innerHTML = `<div class="h-24 w-full loading-shimmer"></div>`; // Loading state

            if (!apiKey) {
                content.innerHTML = `<span class="text-rose-400">⚠️ API Anahtarı eksik. Lütfen kodu düzenleyerek Gemini API anahtarınızı ekleyin.</span>`;
                return;
            }

            const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;
            const payload = {
                contents: [{ parts: [{ text: prompt }] }],
                systemInstruction: { parts: [{ text: "Sen profesyonel bir ağ kurma ve stratejik iletişim danışmanısın. Kullanıcının görüşme verilerini analiz ederek kısa, net, uygulanabilir ve profesyonel Türkçe tavsiyeler ver." }] }
            };

            try {
                const response = await fetch(url, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                
                if (!response.ok) throw new Error('API Bağlantı Hatası');
                
                const result = await response.json();
                const text = result.candidates[0].content.parts[0].text;
                content.innerText = text;
            } catch (error) {
                content.innerText = "Bir hata oluştu. Lütfen bağlantınızı kontrol edin.";
                console.error(error);
            }
        }

        function generateAiStrategy() {
            const prompt = `Mevcut görüşme verilerim şunlar: ${JSON.stringify(data)}. 
            Özellikle ilerleme yüzdesi düşük olan alanlar (Gazeteciler, Enerji Sektörü vb.) için ne yapmalıyım? 
            Mevcut durumu analiz et ve 3 maddelik somut bir eylem planı oluştur.`;
            callGemini(prompt);
        }

        function generateOutreach() {
            const incomplete = data.filter(d => d.completed < d.total).map(d => d.title).join(', ');
            const prompt = `Şu gruplar için henüz görüşmeler tamamlanmadı: ${incomplete}. 
            Bu gruplardaki kişilere göndermek üzere kısa, profesyonel ve ilgi çekici bir e-posta/mesaj taslağı hazırla. 
            Her grup için tonu özelleştir (Örn: Gazeteciler için merak uyandırıcı, Siyasi partiler için resmi).`;
            callGemini(prompt);
        }

        function closeAi() {
            document.getElementById('ai-container').classList.add('hidden');
        }

        // --- Admin/Editor Logic ---
        function toggleAdmin() {
            const panel = document.getElementById('admin-panel');
            panel.classList.toggle('translate-x-full');
            if (!panel.classList.contains('translate-x-full')) renderAdmin();
        }

        function updateData(id, field, value) {
            const idx = data.findIndex(d => d.id === id);
            if (idx === -1) return;
            
            if (field === 'completed' || field === 'total') {
                data[idx][field] = parseInt(value) || 0;
            } else {
                data[idx][field] = value;
            }
            renderDashboard();
        }

        function addNewCategory() {
            data.push({ id: Date.now(), title: 'Yeni', completed: 0, total: 5, color: 'slate', emoji: '📌' });
            renderDashboard();
            renderAdmin();
        }

        function deleteCategory(id) {
            data = data.filter(d => d.id !== id);
            renderDashboard();
            renderAdmin();
        }

        function renderAdmin() {
            const list = document.getElementById('editor-list');
            list.innerHTML = '';
            
            data.forEach(item => {
                const el = document.createElement('div');
                el.className = 'p-4 bg-slate-800/50 border border-slate-700 rounded-xl relative group';
                el.innerHTML = `
                    <button onclick="deleteCategory(${item.id})" class="absolute top-2 right-2 text-rose-500 opacity-0 group-hover:opacity-100 transition-opacity text-xs">Sil</button>
                    <div class="mb-2">
                        <label class="text-[10px] text-slate-500 uppercase font-bold">Kategori & Emoji</label>
                        <div class="flex gap-2">
                            <input type="text" value="${item.emoji}" class="custom-input w-12 text-center" onchange="updateData(${item.id}, 'emoji', this.value)">
                            <input type="text" value="${item.title}" class="custom-input" onchange="updateData(${item.id}, 'title', this.value)">
                        </div>
                    </div>
                    <div class="grid grid-cols-2 gap-2">
                        <div>
                            <label class="text-[10px] text-slate-500 uppercase font-bold">Tamamlanan</label>
                            <input type="number" value="${item.completed}" class="custom-input" onchange="updateData(${item.id}, 'completed', this.value)">
                        </div>
                        <div>
                            <label class="text-[10px] text-slate-500 uppercase font-bold">Hedef</label>
                            <input type="number" value="${item.total}" class="custom-input" onchange="updateData(${item.id}, 'total', this.value)">
                        </div>
                    </div>
                `;
                list.appendChild(el);
            });
        }

        function copyCurrentCode() {
            const currentDataStr = JSON.stringify(data);
            const html = document.documentElement.outerHTML.replace(/let data = \[.*?\];/s, `let data = ${currentDataStr};`);
            
            const textarea = document.createElement('textarea');
            textarea.value = html;
            document.body.appendChild(textarea);
            textarea.select();
            document.execCommand('copy');
            document.body.removeChild(textarea);
            
            alert('Panel kodu kopyalandı! WordPress veya Tiiny.host üzerinde kullanabilirsiniz.');
        }

        // --- Initialization ---
        window.onload = init;
    </script>
</body>
</html>
