# kisangcheong
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI 기상 예보 서비스</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@300;400;500;700&display=swap');
        
        body {
            font-family: 'Noto Sans KR', sans-serif;
            background: linear-gradient(to bottom, #1e3a8a, #3b82f6);
            min-height: 100vh;
            color: white;
        }

        .glass-card {
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.2);
            border-radius: 1.5rem;
        }

        .weather-icon-pulse {
            animation: pulse 2s infinite;
        }

        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.05); }
            100% { transform: scale(1); }
        }

        .loading-spinner {
            border: 3px solid rgba(255, 255, 255, 0.3);
            border-radius: 50%;
            border-top: 3px solid #fff;
            width: 24px;
            height: 24px;
            animation: spin 1s linear infinite;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="p-4 md:p-8">
    <div class="max-w-4xl mx-auto">
        <!-- Header & Search -->
        <header class="flex flex-col md:flex-row justify-between items-center mb-8 gap-4">
            <h1 class="text-3xl font-bold flex items-center gap-2">
                <i class="fas fa-cloud-sun text-yellow-400"></i>
                SkyCast AI
            </h1>
            <div class="relative w-full md:w-96">
                <input type="text" id="cityInput" placeholder="도시 이름을 입력하세요 (예: 서울, Tokyo...)" 
                       class="w-full px-5 py-3 rounded-full bg-white/20 border border-white/30 text-white placeholder-white/60 focus:outline-none focus:ring-2 focus:ring-white/50 transition-all">
                <button onclick="fetchWeather()" class="absolute right-2 top-2 p-2 px-4 bg-white/30 hover:bg-white/40 rounded-full transition-colors">
                    <i class="fas fa-search"></i>
                </button>
            </div>
        </header>

        <!-- Error Message Box -->
        <div id="errorBox" class="hidden mb-6 p-4 bg-red-500/50 border border-red-400 rounded-xl text-center">
            도시를 찾을 수 없습니다. 다시 시도해주세요.
        </div>

        <main id="weatherContent" class="grid grid-cols-1 md:grid-cols-3 gap-6">
            <!-- Main Current Weather -->
            <section class="md:col-span-2 glass-card p-8 flex flex-col justify-between min-h-[400px]">
                <div class="flex justify-between items-start">
                    <div>
                        <h2 id="cityName" class="text-4xl font-bold mb-1">서울</h2>
                        <p id="currentDate" class="text-white/70">2024년 5월 20일 월요일</p>
                    </div>
                    <div class="text-right">
                        <div id="weatherIconContainer" class="text-6xl mb-2 weather-icon-pulse">
                            <i class="fas fa-sun text-yellow-400"></i>
                        </div>
                        <p id="weatherDesc" class="text-xl font-medium">맑음</p>
                    </div>
                </div>

                <div class="flex items-end gap-4 mt-8">
                    <span id="currentTemp" class="text-8xl font-light">24°</span>
                    <div class="mb-4">
                        <p id="tempRange" class="text-lg">최고 26° / 최저 18°</p>
                        <p class="text-white/60">체감 온도 <span id="feelsLike">25°</span></p>
                    </div>
                </div>

                <!-- AI Analysis Summary -->
                <div class="mt-8 p-4 bg-white/10 rounded-xl">
                    <h3 class="text-sm font-bold uppercase tracking-wider mb-2 flex items-center gap-2">
                        <i class="fas fa-robot text-blue-300"></i> AI 기상 분석
                    </h3>
                    <p id="aiSummary" class="text-sm leading-relaxed text-white/90">
                        데이터를 불러오는 중입니다...
                    </p>
                </div>
            </section>

            <!-- Side Stats -->
            <section class="glass-card p-6 flex flex-col gap-6">
                <div class="flex items-center gap-4">
                    <div class="w-12 h-12 rounded-full bg-blue-400/30 flex items-center justify-center text-xl">
                        <i class="fas fa-droplet"></i>
                    </div>
                    <div>
                        <p class="text-sm text-white/60">습도</p>
                        <p id="humidity" class="text-xl font-bold">45%</p>
                    </div>
                </div>
                <div class="flex items-center gap-4">
                    <div class="w-12 h-12 rounded-full bg-yellow-400/30 flex items-center justify-center text-xl">
                        <i class="fas fa-wind"></i>
                    </div>
                    <div>
                        <p class="text-sm text-white/60">풍속</p>
                        <p id="windSpeed" class="text-xl font-bold">3.2 m/s</p>
                    </div>
                </div>
                <div class="flex items-center gap-4">
                    <div class="w-12 h-12 rounded-full bg-purple-400/30 flex items-center justify-center text-xl">
                        <i class="fas fa-umbrella"></i>
                    </div>
                    <div>
                        <p class="text-sm text-white/60">강수 확률</p>
                        <p id="rainProb" class="text-xl font-bold">10%</p>
                    </div>
                </div>
                <div class="flex items-center gap-4">
                    <div class="w-12 h-12 rounded-full bg-orange-400/30 flex items-center justify-center text-xl">
                        <i class="fas fa-sun"></i>
                    </div>
                    <div>
                        <p class="text-sm text-white/60">자외선 지수</p>
                        <p id="uvIndex" class="text-xl font-bold">보통 (4)</p>
                    </div>
                </div>

                <!-- AI Generated Visual -->
                <div class="mt-auto overflow-hidden rounded-xl h-32 relative group cursor-pointer" onclick="generateWeatherImage()">
                    <div id="imageLoading" class="hidden absolute inset-0 bg-black/40 flex items-center justify-center z-10">
                        <div class="loading-spinner"></div>
                    </div>
                    <img id="weatherVisual" src="https://images.unsplash.com/photo-1592210633468-975c78762d98?auto=format&fit=crop&w=400&q=80" 
                         alt="Weather Visual" class="w-full h-full object-cover transition-transform duration-500 group-hover:scale-110">
                    <div class="absolute bottom-2 right-2 bg-black/50 px-2 py-1 rounded text-[10px]">AI 이미지 생성</div>
                </div>
            </section>

            <!-- Forecast -->
            <section class="md:col-span-3 glass-card p-6">
                <h3 class="text-xl font-bold mb-6">주간 예보</h3>
                <div id="forecastList" class="grid grid-cols-2 sm:grid-cols-4 md:grid-cols-7 gap-4">
                    <!-- Forecast items will be injected here -->
                </div>
            </section>
        </main>
    </div>

    <script>
        const apiKey = ""; // API Key placeholder for environment
        const modelId = "gemini-2.5-flash-preview-09-2025";
        
        const weatherIcons = {
            'Clear': 'fa-sun text-yellow-400',
            'Clouds': 'fa-cloud text-gray-300',
            'Rain': 'fa-cloud-showers-heavy text-blue-400',
            'Snow': 'fa-snowflake text-white',
            'Thunderstorm': 'fa-bolt text-yellow-500',
            'Drizzle': 'fa-cloud-rain text-blue-300',
            'Mist': 'fa-smog text-gray-400'
        };

        // Mock data for initial load (in real app, we use OpenWeather API or similar)
        const mockData = {
            city: "서울",
            temp: 24,
            maxTemp: 27,
            minTemp: 16,
            desc: "대체로 맑음",
            condition: "Clear",
            humidity: 42,
            wind: 2.1,
            rain: 5,
            feelsLike: 23,
            uv: "보통 (5)"
        };

        async function fetchWeather() {
            const city = document.getElementById('cityInput').value || "서울";
            const errorBox = document.getElementById('errorBox');
            errorBox.classList.add('hidden');

            try {
                // In a real implementation, you'd fetch from OpenWeatherMap here.
                // For this interactive demo, we'll use Gemini to "simulate" a weather report 
                // and provide analysis for the entered city.
                
                updateUI(mockData, city); // Default UI update
                await getAIAnalysis(city);
            } catch (error) {
                errorBox.classList.remove('hidden');
                console.error(error);
            }
        }

        async function getAIAnalysis(city) {
            const summaryEl = document.getElementById('aiSummary');
            summaryEl.innerHTML = '<div class="loading-spinner scale-50"></div> 분석 중...';

            const prompt = `현재 ${city}의 날씨가 섭씨 24도, 습도 42%, 풍속 2.1m/s인 맑은 날씨라고 가정하고, 사용자에게 친절하고 위트 있는 일기예보 요약(3문장 내외)과 함께 옷차림 추천을 한국어로 작성해줘.`;

            let retries = 0;
            const maxRetries = 5;
            
            while (retries < maxRetries) {
                try {
                    const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/${modelId}:generateContent?key=${apiKey}`, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify({
                            contents: [{ parts: [{ text: prompt }] }]
                        })
                    });

                    const result = await response.json();
                    const aiText = result.candidates?.[0]?.content?.parts?.[0]?.text;
                    if (aiText) {
                        summaryEl.innerText = aiText;
                        return;
                    }
                } catch (e) {
                    retries++;
                    await new Promise(r => setTimeout(r, Math.pow(2, retries) * 1000));
                }
            }
            summaryEl.innerText = "분석을 가져오는 데 실패했습니다. 잠시 후 다시 시도하세요.";
        }

        async function generateWeatherImage() {
            const loader = document.getElementById('imageLoading');
            const visualImg = document.getElementById('weatherVisual');
            const city = document.getElementById('cityName').innerText;
            const desc = document.getElementById('weatherDesc').innerText;

            loader.classList.remove('hidden');

            try {
                const prompt = `A beautiful cinematic high-resolution photograph of the skyline of ${city} during ${desc} weather, professional photography, realistic lighting.`;
                
                const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/imagen-4.0-generate-001:predict?key=${apiKey}`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        instances: { prompt: prompt },
                        parameters: { sampleCount: 1 }
                    })
                });

                const result = await response.json();
                if (result.predictions?.[0]?.bytesBase64Encoded) {
                    visualImg.src = `data:image/png;base64,${result.predictions[0].bytesBase64Encoded}`;
                }
            } catch (error) {
                console.error("Image generation failed:", error);
            } finally {
                loader.classList.add('hidden');
            }
        }

        function updateUI(data, cityName) {
            document.getElementById('cityName').innerText = cityName;
            document.getElementById('currentTemp').innerText = `${data.temp}°`;
            document.getElementById('weatherDesc').innerText = data.desc;
            document.getElementById('tempRange').innerText = `최고 ${data.maxTemp}° / 최저 ${data.minTemp}°`;
            document.getElementById('feelsLike').innerText = `${data.feelsLike}°`;
            document.getElementById('humidity').innerText = `${data.humidity}%`;
            document.getElementById('windSpeed').innerText = `${data.wind} m/s`;
            document.getElementById('rainProb').innerText = `${data.rain}%`;
            
            const iconClass = weatherIcons[data.condition] || 'fa-cloud';
            document.getElementById('weatherIconContainer').innerHTML = `<i class="fas ${iconClass}"></i>`;

            const date = new Date();
            const options = { year: 'numeric', month: 'long', day: 'numeric', weekday: 'long' };
            document.getElementById('currentDate').innerText = date.toLocaleDateString('ko-KR', options);

            renderForecast();
        }

        function renderForecast() {
            const forecastList = document.getElementById('forecastList');
            const days = ['내일', '수', '목', '금', '토', '일', '월'];
            const temps = [25, 23, 21, 19, 22, 24, 26];
            const conditions = ['Clear', 'Clouds', 'Rain', 'Rain', 'Clouds', 'Clear', 'Clear'];

            forecastList.innerHTML = days.map((day, i) => `
                <div class="flex flex-col items-center p-3 rounded-2xl bg-white/5 hover:bg-white/10 transition-colors">
                    <p class="text-sm font-medium mb-2">${day}</p>
                    <i class="fas ${weatherIcons[conditions[i]] || 'fa-sun'} text-xl mb-3"></i>
                    <p class="text-lg font-bold">${temps[i]}°</p>
                    <p class="text-[10px] text-white/50">${temps[i]-5}°</p>
                </div>
            `).join('');
        }

        // Initialize
        window.onload = () => {
            fetchWeather();
            // Optional: Auto-generate image on load
            // generateWeatherImage(); 
        };

        // Enter key support for search
        document.getElementById('cityInput').addEventListener('keypress', (e) => {
            if (e.key === 'Enter') fetchWeather();
        });
    </script>
</body>
</html>
