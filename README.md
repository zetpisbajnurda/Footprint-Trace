<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Digital Trace | OSINT Terminal</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        :root {
            --bg-color: #050505;
            --accent-color: #00ff41;
            --danger-color: #ff3e3e;
            --text-color: #e0e0e0;
            --card-bg: #111;
        }

        body {
            background-color: var(--bg-color);
            color: var(--text-color);
            font-family: 'Courier New', monospace;
            margin: 0;
            overflow-x: hidden;
            line-height: 1.6;
        }

        /* Анимация сканирования */
        .scanline {
            width: 100%;
            height: 100px;
            background: linear-gradient(0deg, rgba(0, 255, 65, 0) 0%, rgba(0, 255, 65, 0.1) 50%, rgba(0, 255, 65, 0) 100%);
            position: fixed;
            top: -100px;
            left: 0;
            z-index: 10;
            pointer-events: none;
            animation: moveScan 3s linear infinite;
        }

        @keyframes moveScan {
            0% { top: -100px; }
            100% { top: 100vh; }
        }

        header {
            padding: 30px;
            text-align: center;
            border-bottom: 1px solid #222;
        }

        .logo {
            font-size: 2rem;
            font-weight: bold;
            letter-spacing: 5px;
            text-shadow: 0 0 10px var(--accent-color);
        }

        .container {
            max-width: 900px;
            margin: 0 auto;
            padding: 40px 20px;
        }

        /* Терминал ввода */
        .terminal-box {
            background: #000;
            border: 1px solid var(--accent-color);
            padding: 20px;
            box-shadow: 0 0 20px rgba(0, 255, 65, 0.2);
            margin-bottom: 40px;
        }

        .input-line {
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .input-line span { color: var(--accent-color); }

        input {
            background: transparent;
            border: none;
            outline: none;
            color: var(--accent-color);
            font-family: inherit;
            font-size: 1.2rem;
            width: 100%;
        }

        button {
            background: var(--accent-color);
            color: #000;
            border: none;
            padding: 10px 25px;
            font-weight: bold;
            cursor: pointer;
            margin-top: 20px;
            width: 100%;
            transition: 0.3s;
        }

        button:hover {
            box-shadow: 0 0 30px var(--accent-color);
        }

        /* Лог консоли */
        #console-log {
            height: 150px;
            overflow-y: auto;
            background: #080808;
            padding: 10px;
            margin-top: 20px;
            font-size: 0.8rem;
            border: 1px dashed #333;
            color: #888;
        }

        /* Сетка результатов */
        .results {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
            opacity: 0;
            transition: 1s;
        }

        .visible { opacity: 1; }

        .card {
            background: var(--card-bg);
            padding: 20px;
            border: 1px solid #222;
            position: relative;
            overflow: hidden;
        }

        .card h3 { 
            margin-top: 0; 
            color: var(--accent-color);
            font-size: 1rem;
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .card .data { font-size: 0.9rem; color: #aaa; }

        .status-searching { color: orange; animation: blink 1s infinite; }
        @keyframes blink { 50% { opacity: 0; } }

        .found { color: var(--accent-color); font-weight: bold; }
        .danger { color: var(--danger-color); font-weight: bold; }

    </style>
</head>
<body>

    <div class="scanline"></div>

    <header>
        <div class="logo">DIGITAL_TRACE v1.0</div>
    </header>

    <div class="container">
        <div class="terminal-box">
            <div class="input-line">
                <span>[SYSTEM@USER]:~#</span>
                <input type="text" id="targetInput" placeholder="Введите никнейм, почту или домен..." autocomplete="off">
            </div>
            <button onclick="startDeepScan()">ЗАПУСТИТЬ ГЛУБОКОЕ СКАНИРОВАНИЕ</button>
            
            <div id="console-log">
                > Ожидание ввода цели...
            </div>
        </div>

        <div class="results" id="resultsGrid">
            <div class="card">
                <h3><i class="fas fa-search"></i> SHERLOCK_MODULE</h3>
                <div class="data" id="res-sherlock">Статус: IDLE</div>
            </div>
            <div class="card">
                <h3><i class="fas fa-user-secret"></i> SOCIAL_ENGINEERING</h3>
                <div class="data" id="res-social">Статус: IDLE</div>
            </div>
            <div class="card">
                <h3><i class="fas fa-database"></i> LEAK_CHECKER</h3>
                <div class="data" id="res-leaks">Статус: IDLE</div>
            </div>
            <div class="card">
                <h3><i class="fas fa-network-wired"></i> IP_TRACKER</h3>
                <div class="data" id="res-ip">Статус: IDLE</div>
            </div>
        </div>
    </div>

    <script>
        const logBox = document.getElementById('console-log');
        
        function addLog(text, color = "#888") {
            const entry = document.createElement('div');
            entry.style.color = color;
            entry.innerHTML = `> ${text}`;
            logBox.appendChild(entry);
            logBox.scrollTop = logBox.scrollHeight;
        }

        async function startDeepScan() {
            const target = document.getElementById('targetInput').value;
            if(!target) {
                addLog("ОШИБКА: Цель не определена", "#ff3e3e");
                return;
            }

            // Очистка и подготовка
            logBox.innerHTML = "";
            document.getElementById('resultsGrid').classList.add('visible');
            
            const modules = [
                { id: 'res-sherlock', name: 'Sherlock', cmd: `python3 sherlock.py ${target}` },
                { id: 'res-social', name: 'SocialScan', cmd: `socialscan --target ${target}` },
                { id: 'res-leaks', name: 'BreachAnalyzer', cmd: `search-leaks -q ${target}` },
                { id: 'res-ip', name: 'WhoisLookup', cmd: `whois ${target}` }
            ];

            for (let mod of modules) {
                document.getElementById(mod.id).innerHTML = `<span class="status-searching">Выполнение: ${mod.cmd}...</span>`;
                addLog(`Запуск модуля ${mod.name}...`, "#00ff41");
                
                // Имитация задержки сети
                await new Promise(resolve => setTimeout(resolve, Math.random() * 1500 + 500));
                
                updateModuleResult(mod, target);
            }
            
            addLog("СКАНИРОВАНИЕ ЗАВЕРШЕНО. Отчет сформирован.", "#00ff41");
        }

        function updateModuleResult(mod, target) {
            let resHtml = "";
            if(mod.id === 'res-sherlock') {
                resHtml = `Найдено совпадений: <span class="found">GitHub, Steam, Reddit</span>`;
            } else if(mod.id === 'res-social') {
                resHtml = `Вероятное имя: <span class="found">Александр В.</span><br>Локация: <span class="found">Москва, РФ</span>`;
            } else if(mod.id === 'res-leaks') {
                resHtml = `Пароль найден в базе <span class="danger">"Collection #1"</span>!`;
            } else {
                resHtml = `IP-адрес: <span class="found">192.168.1.1</span><br>Провайдер: <span class="found">ISP Global</span>`;
            }
            document.getElementById(mod.id).innerHTML = resHtml;
        }
    </script>
</body>
</html>
