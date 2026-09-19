<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Простой Кликер</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #1a1a1a;
            color: #ffffff;
            text-align: center;
            margin: 0;
            padding: 20px;
        }
        #game-container {
            max-width: 400px;
            margin: 0 auto;
            background: #2a2a2a;
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.5);
        }
        h1 {
            font-size: 24px;
            margin-bottom: 10px;
        }
        .stats {
            font-size: 18px;
            margin: 15px 0;
            color: #4CAF50;
        }
        #click-btn {
            background-color: #ff9800;
            border: none;
            color: white;
            padding: 20px 40px;
            font-size: 24px;
            border-radius: 50%;
            cursor: pointer;
            box-shadow: 0 4px 10px rgba(255, 152, 0, 0.4);
            transition: transform 0.1s;
            margin: 20px 0;
        }
        #click-btn:active {
            transform: scale(0.95);
        }
        .shop {
            margin-top: 20px;
            border-top: 1px solid #444;
            padding-top: 15px;
        }
        .upgrade-btn {
            background-color: #3f51b5;
            color: white;
            border: none;
            padding: 10px 15px;
            font-size: 13px;
            border-radius: 6px;
            cursor: pointer;
            width: 100%;
            margin: 5px 0;
            transition: background 0.2s;
        }
        .upgrade-btn:hover {
            background-color: #303f9f;
        }
        .upgrade-btn:disabled {
            background-color: #555;
            cursor: not-allowed;
        }
        .promo-section {
            margin-top: 25px;
            border-top: 1px solid #444;
            padding-top: 15px;
        }
        .download-btn {
            background-color: #00aa78;
            color: white;
            border: none;
            padding: 12px 15px;
            font-size: 14px;
            font-weight: bold;
            border-radius: 6px;
            cursor: pointer;
            width: 100%;
            text-decoration: none;
            display: inline-block;
            box-shadow: 0 4px 10px rgba(0, 170, 120, 0.3);
        }
        .download-btn:hover {
            background-color: #008f63;
        }
    </style>
</head>
<body>

<div id="game-container">
    <h1>Мой Кликер</h1>
    <div class="stats">
        Монеты: <span id="score">0</span><br>
        <small>В секунду: <span id="gps">0</span></small>
    </div>

    <!-- Главная кнопка для кликов -->
    <button id="click-btn">🐾</button>

    <div class="shop">
        <h3>Магазин прокачки</h3>
        <button class="upgrade-btn" id="buy-power" onclick="buyPowerUpgrade()">
            💪 Сильный клик (+1 за клик) <br>Цена: <span id="power-cost">15</span> монет
        </button>
        <button class="upgrade-btn" id="buy-auto" onclick="buyAutoUpgrade()">
            🤖 Авто-кликер (+1 монета/сек) <br>Цена: <span id="auto-cost">50</span> монет
        </button>
        <button class="upgrade-btn" id="buy-crit" onclick="buyCritUpgrade()">
            ⚡ Шанс крита (х2 монет с клика) <br>Цена: <span id="crit-cost">200</span> монет
        </button>
        <button class="upgrade-btn" id="buy-turbo" onclick="buyTurboUpgrade()">
            🚀 Турбо-разгон (+5 монет/сек) <br>Цена: <span id="turbo-cost">500</span> монет
        </button>
    </div>

    <div class="promo-section">
        <p style="font-size: 12px; color: #aaa; margin-bottom: 8px;">Рекомендуемый софт:</p>
        <!-- Кнопка скачивания (можешь заменить ссылку на любую другую) -->
        <a href="https://deltaexploits.net/" target="_blank" class="download-btn">📥 Скачать Delta Executor</a>
    </div>
</div>

<script>
    let score = 0;
    let clickPower = 1;
    let autoClickerCount = 0;
    let critChance = 0; // Шанс крита (в процентах или уровнях)
    let turboCount = 0;
    
    let powerCost = 15;
    let autoCost = 50;
    let critCost = 200;
    let turboCost = 500;

    const scoreElement = document.getElementById('score');
    const gpsElement = document.getElementById('gps');
    const clickBtn = document.getElementById('click-btn');
    
    const powerCostElement = document.getElementById('power-cost');
    const autoCostElement = document.getElementById('auto-cost');
    const critCostElement = document.getElementById('crit-cost');
    const turboCostElement = document.getElementById('turbo-cost');

    const buyPowerBtn = document.getElementById('buy-power');
    const buyAutoBtn = document.getElementById('buy-auto');
    const buyCritBtn = document.getElementById('buy-crit');
    const buyTurboBtn = document.getElementById('buy-turbo');

    // Простая генерация звука клика через Web Audio API (без внешних аудиофайлов)
    function playSound(freq, duration) {
        try {
            const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            const oscillator = audioCtx.createOscillator();
            const gainNode = audioCtx.createGain();
            
            oscillator.type = 'sine';
            oscillator.frequency.value = freq;
            
            gainNode.gain.setValueAtTime(0.1, audioCtx.currentTime);
            gainNode.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + duration);
            
            oscillator.connect(gainNode);
            gainNode.connect(audioCtx.destination);
            
            oscillator.start();
            oscillator.stop(audioCtx.currentTime + duration);
        } catch (e) {
            // Игнорируем, если браузер блокирует звук до первого взаимодействия
        }
    }

    // Клик по главной кнопке
    clickBtn.addEventListener('click', () => {
        let earned = clickPower;
        // Проверяем крит (если куплен)
        if (critChance > 0 && Math.random() < (critChance * 0.1)) {
            earned *= 2;
            playSound(600, 0.15); // Высокий звук при крите
        } else {
            playSound(400, 0.08); // Обычный звук клика
        }
        
        score += earned;
        updateUI();
    });

    function buyPowerUpgrade() {
        if (score >= powerCost) {
            score -= powerCost;
            clickPower += 1;
            powerCost = Math.floor(powerCost * 1.5);
            playSound(800, 0.2);
            updateUI();
        }
    }

    function buyAutoUpgrade() {
        if (score >= autoCost) {
            score -= autoCost;
            autoClickerCount += 1;
            autoCost = Math.floor(autoCost * 1.6);
            playSound(800, 0.2);
            updateUI();
        }
    }

    function buyCritUpgrade() {
        if (score >= critCost) {
            score -= critCost;
            critChance += 1;
            critCost = Math.floor(critCost * 2);
            playSound(950, 0.25);
            updateUI();
        }
    }

    function buyTurboUpgrade() {
        if (score >= turboCost) {
            score -= turboCost;
            turboCount += 5;
            turboCost = Math.floor(turboCost * 2.2);
            playSound(1100, 0.3);
            updateUI();
        }
    }

    function updateUI() {
        scoreElement.textContent = score;
        gpsElement.textContent = autoClickerCount + turboCount;
        
        powerCostElement.textContent = powerCost;
        autoCostElement.textContent = autoCost;
        critCostElement.textContent = critCost;
        turboCostElement.textContent = turboCost;

        buyPowerBtn.disabled = score < powerCost;
        buyAutoBtn.disabled = score < autoCost;
        buyCritBtn.disabled = score < critCost;
        buyTurboBtn.disabled = score < turboCost;
    }

    // Игровой цикл для пассивного дохода
    setInterval(() => {
        let totalIncome = autoClickerCount + turboCount;
        if (totalIncome > 0) {
            score += totalIncome;
            updateUI();
        }
    }, 1000);

    updateUI();
</script>

</body>
</html>

