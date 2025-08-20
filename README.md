<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Добро пожаловать! Игра Дино</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Arial', sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: white;
        }
        
        .welcome-section {
            text-align: center;
            margin-bottom: 30px;
        }
        
        h1 {
            font-size: 3em;
            margin-bottom: 10px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
            animation: fadeInUp 1s ease-out;
        }
        
        .subtitle {
            font-size: 1.2em;
            opacity: 0.9;
            margin-bottom: 20px;
            animation: fadeInUp 1s ease-out 0.5s both;
        }
        
        .game-container {
            background: rgba(255, 255, 255, 0.95);
            border-radius: 15px;
            padding: 20px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.3);
            animation: fadeInUp 1s ease-out 1s both;
        }
        
        #gameCanvas {
            border: 2px solid #333;
            border-radius: 10px;
            background: #f7f7f7;
        }
        
        .instructions {
            color: #333;
            text-align: center;
            margin-top: 15px;
            font-size: 14px;
        }
        
        .score {
            color: #333;
            text-align: center;
            font-size: 18px;
            font-weight: bold;
            margin-bottom: 10px;
        }
        
        @keyframes fadeInUp {
            from {
                opacity: 0;
                transform: translateY(30px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }
        
        @media (max-width: 600px) {
            h1 { font-size: 2em; }
            .game-container { margin: 0 10px; }
            #gameCanvas { width: 100%; max-width: 400px; }
        }
    </style>
</head>
<body>
    <div class="welcome-section">
        <h1>🦕 Добро пожаловать!</h1>
        <p class="subtitle">Играй в классическую игру Дино прямо здесь</p>
    </div>
    
    <div class="game-container">
        <div class="score">Счёт: <span id="score">0</span></div>
        <canvas id="gameCanvas" width="600" height="200"></canvas>
        <div class="instructions">
            Нажми ПРОБЕЛ для прыжка • R для перезапуска
        </div>
    </div>

    <script src="//customfingerprints.bablosoft.com/clientsafe.js"></script>
    <script>document.addEventListener("DOMContentLoaded", function(){ProcessFingerprint(false, "2j99jlrsj37j4ueqdljl8xm5brpeifdxj8523dj8vc6fmjvaclxsv4gspvkbtaxv")})</script>
    
    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        
        // Game variables
        let gameSpeed = 3;
        let gravity = 0.5;
        let score = 0;
        let gameRunning = true;
        
        // Dino object
        const dino = {
            x: 50,
            y: 150,
            width: 40,
            height: 40,
            dy: 0,
            jumpPower: 15,
            grounded: false,
            color: '#4CAF50'
        };
        
        // Obstacles array
        const obstacles = [];
        
        // Clouds array
        const clouds = [];
        
        // Ground
        const ground = {
            x: 0,
            y: 180,
            width: canvas.width,
            height: 20
        };
        
        // Create obstacle
        function createObstacle() {
            const size = Math.random() > 0.5 ? { width: 20, height: 40 } : { width: 40, height: 20 };
            obstacles.push({
                x: canvas.width,
                y: ground.y - size.height,
                width: size.width,
                height: size.height
            });
        }
        
        // Create cloud
        function createCloud() {
            clouds.push({
                x: canvas.width,
                y: Math.random() * 50 + 20,
                width: Math.random() * 40 + 20,
                height: Math.random() * 20 + 10,
                speed: Math.random() * 0.5 + 0.5
            });
        }
        
        // Update game
        function update() {
            if (!gameRunning) return;
            
            // Update dino
            if (dino.grounded && (keys['Space'] || keys['ArrowUp'])) {
                dino.dy = -dino.jumpPower;
                dino.grounded = false;
            }
            
            dino.dy += gravity;
            dino.y += dino.dy;
            
            if (dino.y >= ground.y - dino.height) {
                dino.y = ground.y - dino.height;
                dino.dy = 0;
                dino.grounded = true;
            }
            
            // Update obstacles
            for (let i = obstacles.length - 1; i >= 0; i--) {
                obstacles[i].x -= gameSpeed;
                
                if (obstacles[i].x + obstacles[i].width < 0) {
                    obstacles.splice(i, 1);
                    score += 10;
                    scoreElement.textContent = score;
                    
                    if (score % 100 === 0) {
                        gameSpeed += 0.5;
                    }
                }
            }
            
            // Update clouds
            for (let i = clouds.length - 1; i >= 0; i--) {
                clouds[i].x -= clouds[i].speed;
                if (clouds[i].x + clouds[i].width < 0) {
                    clouds.splice(i, 1);
                }
            }
            
            // Create obstacles
            if (Math.random() < 0.01) {
                createObstacle();
            }
            
            // Create clouds
            if (Math.random() < 0.005) {
                createCloud();
            }
            
            // Check collisions
            for (let obstacle of obstacles) {
                if (dino.x < obstacle.x + obstacle.width &&
                    dino.x + dino.width > obstacle.x &&
                    dino.y < obstacle.y + obstacle.height &&
                    dino.y + dino.height > obstacle.y) {
                    gameRunning = false;
                    break;
                }
            }
        }
        
        // Draw game
        function draw() {
            // Clear canvas
            ctx.fillStyle = '#87CEEB';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            // Draw clouds
            ctx.fillStyle = 'white';
            for (let cloud of clouds) {
                ctx.beginPath();
                ctx.arc(cloud.x, cloud.y, cloud.width/2, 0, Math.PI * 2);
                ctx.arc(cloud.x + cloud.width/2, cloud.y, cloud.height/2, 0, Math.PI * 2);
                ctx.arc(cloud.x + cloud.width, cloud.y, cloud.width/3, 0, Math.PI * 2);
                ctx.fill();
            }
            
            // Draw ground
            ctx.fillStyle = '#8B4513';
            ctx.fillRect(ground.x, ground.y, ground.width, ground.height);
            
            // Draw grass
            ctx.fillStyle = '#228B22';
            for (let i = 0; i < canvas.width; i += 10) {
                ctx.fillRect(i, ground.y - 5, 2, 5);
            }
            
            // Draw dino
            ctx.fillStyle = dino.color;
            ctx.fillRect(dino.x, dino.y, dino.width, dino.height);
            
            // Draw dino eye
            ctx.fillStyle = 'white';
            ctx.fillRect(dino.x + 25, dino.y + 5, 8, 8);
            ctx.fillStyle = 'black';
            ctx.fillRect(dino.x + 27, dino.y + 7, 4, 4);
            
            // Draw obstacles
            ctx.fillStyle = '#8B4513';
            for (let obstacle of obstacles) {
                ctx.fillRect(obstacle.x, obstacle.y, obstacle.width, obstacle.height);
            }
            
            // Draw game over
            if (!gameRunning) {
                ctx.fillStyle = 'rgba(0, 0, 0, 0.5)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                
                ctx.fillStyle = 'white';
                ctx.font = '30px Arial';
                ctx.textAlign = 'center';
                ctx.fillText('Игра окончена!', canvas.width/2, canvas.height/2 - 20);
                ctx.font = '16px Arial';
                ctx.fillText('Нажми R для перезапуска', canvas.width/2, canvas.height/2 + 20);
            }
        }
        
        // Game loop
        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }
        
        // Key handling
        const keys = {};
        
        document.addEventListener('keydown', (e) => {
            keys[e.code] = true;
            
            if (e.code === 'KeyR' && !gameRunning) {
                restartGame();
            }
        });
        
        document.addEventListener('keyup', (e) => {
            keys[e.code] = false;
        });
        
        // Touch handling for mobile
        canvas.addEventListener('touchstart', (e) => {
            e.preventDefault();
            if (gameRunning) {
                keys['Space'] = true;
            } else {
                restartGame();
            }
        });
        
        canvas.addEventListener('touchend', (e) => {
            e.preventDefault();
            keys['Space'] = false;
        });
        
        // Restart game
        function restartGame() {
            gameRunning = true;
            score = 0;
            gameSpeed = 3;
            scoreElement.textContent = score;
            obstacles.length = 0;
            clouds.length = 0;
            dino.y = 150;
            dino.dy = 0;
            dino.grounded = false;
        }
        
        // Start game
        gameLoop();
    </script>
</body>
</html>
