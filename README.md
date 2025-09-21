<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Yılan Oyunu</title>
    <style>
        /* Genel Sayfa Stilleri */
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #2c3e50; /* Koyu mavi-gri arka plan */
            font-family: 'Arial', sans-serif;
            color: #ecf0f1; /* Açık renk yazı */
            touch-action: manipulation; /* Mobil'de çift tıklama zoom'unu engeller */
        }

        /* Oyunun Ana Konteyneri */
        .game-container {
            text-align: center;
            background-color: #34495e; /* Daha açık mavi-gri */
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.4);
        }

        h1 {
            margin-top: 0;
            color: #e74c3c; /* Kırmızı başlık */
        }

        #score {
            font-size: 1.5em;
            margin-bottom: 15px;
        }

        /* Oyun Alanı (Canvas) */
        #game-canvas {
            background-color: #16a085; /* Yeşil oyun alanı */
            border: 5px solid #2c3e50;
            border-radius: 10px;
        }

        /* Mobil Kontrol Butonları */
        #mobile-controls {
            display: none; /* Varsayılan olarak gizli, sadece mobil'de görünecek */
            margin-top: 20px;
            display: grid;
            grid-template-columns: repeat(3, 80px);
            grid-template-rows: repeat(2, 80px);
            gap: 10px;
            justify-content: center;
        }

        #mobile-controls button {
            font-size: 2em;
            border: none;
            border-radius: 10px;
            background-color: #e74c3c;
            color: white;
            cursor: pointer;
            box-shadow: 0 4px #c0392b;
            transition: all 0.1s ease;
        }

        #mobile-controls button:active {
            transform: translateY(2px);
            box-shadow: 0 2px #c0392b;
        }

        /* Butonların D-Pad şeklinde yerleşimi */
        #up-btn { grid-column: 2; grid-row: 1; }
        #left-btn { grid-column: 1; grid-row: 2; }
        #down-btn { grid-column: 2; grid-row: 2; }
        #right-btn { grid-column: 3; grid-row: 2; }
        
        /* Oyun Bitti Ekranı */
        #game-over-screen {
            display: none; /* Başlangıçta gizli */
            position: absolute;
            background-color: rgba(44, 62, 80, 0.9);
            padding: 30px;
            border-radius: 15px;
            text-align: center;
        }

        #game-over-screen h2 {
            color: #e74c3c;
            margin-top: 0;
        }

        #restart-btn {
            padding: 15px 30px;
            font-size: 1.2em;
            background-color: #e74c3c;
            color: white;
            border: none;
            border-radius: 10px;
            cursor: pointer;
            margin-top: 10px;
        }

    </style>
</head>
<body>

    <div class="game-container">
        <h1>Yılan Oyunu</h1>
        <div id="score">Skor: 0</div>
        <canvas id="game-canvas" width="400" height="400"></canvas>
        <div id="mobile-controls">
            <button id="up-btn">↑</button>
            <button id="left-btn">←</button>
            <button id="down-btn">↓</button>
            <button id="right-btn">→</button>
        </div>
    </div>
    
    <div id="game-over-screen">
        <h2>Oyun Bitti!</h2>
        <p id="final-score"></p>
        <button id="restart-btn">Yeniden Oyna</button>
    </div>

    <script>
        // DOM Elementlerini seçme
        const canvas = document.getElementById('game-canvas');
        const ctx = canvas.getContext('2d');
        const scoreDisplay = document.getElementById('score');
        const gameOverScreen = document.getElementById('game-over-screen');
        const finalScoreDisplay = document.getElementById('final-score');
        const restartBtn = document.getElementById('restart-btn');

        // Mobil kontrol butonları
        const upBtn = document.getElementById('up-btn');
        const downBtn = document.getElementById('down-btn');
        const leftBtn = document.getElementById('left-btn');
        const rightBtn = document.getElementById('right-btn');
        
        // Oyun Ayarları
        const gridSize = 20; // Her bir kare 20x20 piksel
        const canvasSize = canvas.width;
        let snake, food, score, direction, gameInterval, isGameOver;

        // Oyunu başlatan veya yeniden başlatan fonksiyon
        function startGame() {
            // Başlangıç değerlerini ayarla
            snake = [{ x: 10 * gridSize, y: 10 * gridSize }]; // Yılan ortada başlar
            food = {};
            score = 0;
            direction = 'right'; // İlk yön
            isGameOver = false;

            // Ekranları ayarla
            scoreDisplay.textContent = 'Skor: 0';
            gameOverScreen.style.display = 'none';

            generateFood();
            
            // Eğer daha önce çalışan bir oyun döngüsü varsa temizle
            if (gameInterval) {
                clearInterval(gameInterval);
            }
            // Oyun döngüsünü başlat
            gameInterval = setInterval(gameLoop, 120); // Hızı buradan ayarlayabilirsin (düşük sayı = daha hızlı)
        }

        // Ana oyun döngüsü
        function gameLoop() {
            if (isGameOver) return;
            update();
            draw();
        }

        // Yılanı ve yemi çizen fonksiyon
        function draw() {
            // Arka planı temizle
            ctx.fillStyle = '#16a085';
            ctx.fillRect(0, 0, canvasSize, canvasSize);

            // Yılanı çiz
            ctx.fillStyle = '#f1c40f'; // Sarı yılan
            snake.forEach(segment => {
                ctx.fillRect(segment.x, segment.y, gridSize, gridSize);
                ctx.strokeStyle = '#2c3e50'; // Segment araları
                ctx.strokeRect(segment.x, segment.y, gridSize, gridSize);
            });

            // Yemi çiz
            ctx.fillStyle = '#e74c3c'; // Kırmızı yem
            ctx.fillRect(food.x, food.y, gridSize, gridSize);
            ctx.strokeStyle = '#c0392b';
            ctx.strokeRect(food.x, food.y, gridSize, gridSize);
        }

        // Oyunun durumunu güncelleyen fonksiyon
        function update() {
            // Yılanın yeni başını oluştur
            const head = { x: snake[0].x, y: snake[0].y };
            
            switch (direction) {
                case 'up': head.y -= gridSize; break;
                case 'down': head.y += gridSize; break;
                case 'left': head.x -= gridSize; break;
                case 'right': head.x += gridSize; break;
            }

            // Çarpışma kontrolü
            if (hasCollision(head)) {
                endGame();
                return;
            }

            // Yeni başı yılanın önüne ekle
            snake.unshift(head);

            // Yem yendi mi?
            if (head.x === food.x && head.y === food.y) {
                score++;
                scoreDisplay.textContent = 'Skor: ' + score;
                generateFood(); // Yeni yem oluştur
            } else {
                snake.pop(); // Yem yenmediyse kuyruğu kısalt
            }
        }
        
        // Çarpışma oldu mu diye kontrol eden fonksiyon
        function hasCollision(head) {
            // Duvarlara çarpma
            if (head.x < 0 || head.x >= canvasSize || head.y < 0 || head.y >= canvasSize) {
                return true;
            }
            // Kendine çarpma
            for (let i = 1; i < snake.length; i++) {
                if (head.x === snake[i].x && head.y === snake[i].y) {
                    return true;
                }
            }
            return false;
        }

        // Oyunu bitiren fonksiyon
        function endGame() {
            isGameOver = true;
            clearInterval(gameInterval); // Oyun döngüsünü durdur
            finalScoreDisplay.textContent = 'Skorun: ' + score;
            gameOverScreen.style.display = 'flex';
        }

        // Rastgele bir konumda yem oluşturan fonksiyon
        function generateFood() {
            let foodX, foodY, onSnake;
            do {
                onSnake = false;
                foodX = Math.floor(Math.random() * (canvasSize / gridSize)) * gridSize;
                foodY = Math.floor(Math.random() * (canvasSize / gridSize)) * gridSize;
                // Yemin yılanın üzerinde olup olmadığını kontrol et
                for (const segment of snake) {
                    if (segment.x === foodX && segment.y === foodY) {
                        onSnake = true;
                        break;
                    }
                }
            } while (onSnake); // Yılanın üzerindeyse yeni koordinat üret

            food = { x: foodX, y: foodY };
        }

        // Yön değiştirmeyi yöneten fonksiyon
        function changeDirection(newDirection) {
            const goingUp = direction === 'up';
            const goingDown = direction === 'down';
            const goingLeft = direction === 'left';
            const goingRight = direction === 'right';

            // Yılanın kendi içine dönmesini engelleme
            if (newDirection === 'up' && !goingDown) direction = 'up';
            if (newDirection === 'down' && !goingUp) direction = 'down';
            if (newDirection === 'left' && !goingRight) direction = 'left';
            if (newDirection === 'right' && !goingLeft) direction = 'right';
        }

        // Klavye dinleyicisi
        document.addEventListener('keydown', (e) => {
            switch (e.key) {
                case 'ArrowUp': case 'w': changeDirection('up'); break;
                case 'ArrowDown': case 's': changeDirection('down'); break;
                case 'ArrowLeft': case 'a': changeDirection('left'); break;
                case 'ArrowRight': case 'd': changeDirection('right'); break;
            }
        });
        
        // Mobil buton dinleyicileri
        upBtn.addEventListener('click', () => changeDirection('up'));
        downBtn.addEventListener('click', () => changeDirection('down'));
        leftBtn.addEventListener('click', () => changeDirection('left'));
        rightBtn.addEventListener('click', () => changeDirection('right'));

        // Yeniden başlatma butonu
        restartBtn.addEventListener('click', startGame);

        // İlk oyunu başlat
        startGame();

    </script>
</body>
</html>
