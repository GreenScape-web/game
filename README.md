<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>لعبة الطائر المرفرف</title>
    <meta name="google-site-verification" content="l6FSehriSN6V1o0fUyEB7SLy7Ym0pepQBHWJM0tbyV0" />
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Luckiest+Guy&family=Press+Start+2P&family=Tajawal:wght@400;700&display=swap" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap');
        @import url('https://fonts.googleapis.com/css2?family=Tajawal:wght@400;700&display=swap');

        :root {
            --bg-color: #79a6d8;
            --bird-color: #f1c40f;
            --pipe-color: #2d3436;
            --message-box-bg: rgba(44, 62, 80, 0.9);
            --message-box-border: #ecf0f1;
            --text-color: #ecf0f1;
            --button-color: #27ae60;
            --button-hover-color: #2ecc71;
            --font-family: 'Tajawal', sans-serif;
        }

        body {
            font-family: var(--font-family);
            margin: 0;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background: linear-gradient(135deg, #3498db, #8e44ad);
            color: var(--text-color);
            overflow: hidden;
            text-align: center;
        }

        .container {
            position: relative;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            background: rgba(52, 73, 94, 0.7);
            padding: 2rem;
            border-radius: 20px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            transition: transform 0.3s ease-in-out;
            border: 2px solid #5d6d7e;
        }

        #gameCanvas {
            background-color: var(--bg-color);
            border-radius: 10px;
            box-shadow: inset 0 0 15px rgba(0, 0, 0, 0.3);
            border: 2px solid #34495e;
        }

        .game-info {
            margin-top: 1.5rem;
            display: flex;
            justify-content: space-between;
            width: 100%;
            max-width: 400px;
            padding: 0 1rem;
        }

        .score, .high-score {
            font-size: 1.8rem;
            font-weight: bold;
            color: var(--text-color);
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
        }

        .message-box {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: var(--message-box-bg);
            border-radius: 20px;
            padding: 3rem;
            box-shadow: 0 15px 30px rgba(0, 0, 0, 0.6);
            text-align: center;
            z-index: 1000;
            transition: all 0.4s ease-in-out;
            border: 3px solid var(--message-box-border);
            animation: fadeIn 0.5s ease-out;
        }

        .message-box h1 {
            font-size: 2.5rem;
            margin-top: 0;
            color: #e74c3c;
            text-shadow: 2px 2px 3px rgba(0, 0, 0, 0.3);
        }

        .message-box p {
            font-size: 1.5rem;
            margin: 1.5rem 0;
            color: #bdc3c7;
        }

        .message-box button {
            background-color: var(--button-color);
            color: white;
            border: none;
            padding: 1rem 2rem;
            font-size: 1.2rem;
            border-radius: 10px;
            cursor: pointer;
            transition: background-color 0.3s, transform 0.2s, box-shadow 0.2s;
            box-shadow: 0 6px 10px rgba(0, 0, 0, 0.3);
            font-family: var(--font-family);
        }

        .message-box button:hover {
            background-color: var(--button-hover-color);
            transform: translateY(-4px);
            box-shadow: 0 8px 15px rgba(0, 0, 0, 0.4);
        }
        
        .message-box button:active {
            transform: translateY(0);
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
        }

        .hidden {
            display: none;
            opacity: 0;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translate(-50%, -50%) scale(0.8); }
            to { opacity: 1; transform: translate(-50%, -50%) scale(1); }
        }
    </style>
</head>
<body>
    <div class="container">
        <canvas id="gameCanvas"></canvas>
    </div>
    <div class="game-info">
        <div class="score">النتيجة: <span id="currentScore">0</span></div>
        <div class="high-score">أعلى نتيجة: <span id="highScore">0</span></div>
    </div>
    
    <div id="startScreen" class="message-box">
        <h1>مرحبا بك في لعبة الطائر المرفرف!</h1>
        <p>انقر أو اضغط على الشاشة للعب</p>
        <button id="startButton">بدء اللعبة</button>
    </div>

    <div id="gameOverScreen" class="message-box hidden">
        <h1>انتهت اللعبة!</h1>
        <p>النتيجة النهائية: <span id="finalScore">0</span></p>
        <button id="restartButton">إعادة اللعب</button>
    </div>

    <script>
        // Get the canvas and its context
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        // Game state variables
        let isGameRunning = false;
        let score = 0;
        let highScore = localStorage.getItem('flappyHighScore') || 0;

        // Player (Bird) properties
        const bird = {
            x: 50,
            y: 0, 
            radius: 12,
            gravity: 0.2,
            velocity: 0,
            lift: -4.5,
            color: '#f1c40f',
            strokeColor: '#e67e22',
            strokeWidth: 2,
        };

        // Pipe properties
        let pipes = [];
        const pipeWidth = 45;
        const pipeGap = 130; 
        let frameCount = 0;
        const pipeInterval = 90; 
        const pipeSpeed = 2;
        const pipeColor = '#2d3436';

        // UI elements
        const currentScoreElement = document.getElementById('currentScore');
        const highScoreElement = document.getElementById('highScore');
        const startScreen = document.getElementById('startScreen');
        const gameOverScreen = document.getElementById('gameOverScreen');
        const finalScoreElement = document.getElementById('finalScore');
        const startButton = document.getElementById('startButton');
        const restartButton = document.getElementById('restartButton');

        // --- Game Logic Functions ---

        /**
         * Resizes the canvas to fit the window and initializes the bird's position.
         */
        function resizeCanvas() {
            canvas.width = Math.min(window.innerWidth * 0.8, 400);
            canvas.height = Math.min(window.innerHeight * 0.7, 500);
            bird.y = canvas.height / 2;
        }

        /**
         * Draws the bird on the canvas.
         */
        function drawBird() {
            ctx.beginPath();
            ctx.arc(bird.x, bird.y, bird.radius, 0, Math.PI * 2);
            ctx.fillStyle = bird.color;
            ctx.strokeStyle = bird.strokeColor;
            ctx.lineWidth = bird.strokeWidth;
            ctx.fill();
            ctx.stroke();
            ctx.closePath();
        }

        /**
         * Updates the bird's position based on gravity and velocity.
         */
        function updateBird() {
            bird.velocity += bird.gravity;
            bird.y += bird.velocity;

            // Check for collision with top/bottom of the canvas
            if (bird.y + bird.radius > canvas.height || bird.y - bird.radius < 0) {
                gameOver();
            }
        }

        /**
         * Draws a pipe on the canvas.
         * @param {number} x - The x-coordinate of the pipe.
         * @param {number} height - The height of the top pipe.
         */
        function drawPipe(x, height) {
            // Top pipe
            ctx.fillStyle = pipeColor;
            ctx.fillRect(x, 0, pipeWidth, height);
            // Bottom pipe
            ctx.fillRect(x, height + pipeGap, pipeWidth, canvas.height - (height + pipeGap));
        }

        /**
         * Generates and updates pipes.
         */
        function updatePipes() {
            frameCount++;

            // Add a new pipe every 'pipeInterval' frames
            if (frameCount % pipeInterval === 0) {
                const pipeHeight = Math.random() * (canvas.height - pipeGap - 80) + 40;
                pipes.push({ x: canvas.width, height: pipeHeight });
            }

            // Move existing pipes and remove old ones
            for (let i = pipes.length - 1; i >= 0; i--) {
                pipes[i].x -= pipeSpeed;

                // Check for collision
                if (
                    bird.x + bird.radius > pipes[i].x &&
                    bird.x - bird.radius < pipes[i].x + pipeWidth
                ) {
                    if (
                        bird.y - bird.radius < pipes[i].height ||
                        bird.y + bird.radius > pipes[i].height + pipeGap
                    ) {
                        gameOver();
                    }
                }

                // Check if the bird has passed the pipe to increase the score
                if (pipes[i].x + pipeWidth < bird.x && !pipes[i].passed) {
                    score++;
                    pipes[i].passed = true;
                    currentScoreElement.textContent = score;
                }

                // Remove pipes that have moved off screen
                if (pipes[i].x + pipeWidth < 0) {
                    pipes.splice(i, 1);
                }
            }
        }

        /**
         * The main game loop.
         */
        function gameLoop() {
            if (!isGameRunning) return;

            // Clear the canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Update and draw game elements
            updateBird();
            updatePipes();
            drawBird();
            for (const pipe of pipes) {
                drawPipe(pipe.x, pipe.height);
            }

            // Request the next animation frame
            requestAnimationFrame(gameLoop);
        }

        /**
         * Handles the bird's jump action.
         */
        function birdJump() {
            if (isGameRunning) {
                bird.velocity = bird.lift;
            } else if (gameOverScreen.classList.contains('hidden')) {
                // If game is not running and game over screen is not visible, start the game
                startGame();
            }
        }

        /**
         * Displays a message box with the given text.
         * @param {HTMLElement} element - The message box element to show.
         */
        function showMessageBox(element) {
            element.classList.remove('hidden');
        }

        /**
         * Hides a message box.
         * @param {HTMLElement} element - The message box element to hide.
         */
        function hideMessageBox(element) {
            element.classList.add('hidden');
        }

        /**
         * Starts a new game.
         */
        function startGame() {
            hideMessageBox(startScreen);
            hideMessageBox(gameOverScreen);
            isGameRunning = true;
            score = 0;
            currentScoreElement.textContent = score;
            bird.y = canvas.height / 2;
            bird.velocity = 0;
            pipes = [];
            frameCount = 0;
            gameLoop();
        }

        /**
         * Ends the current game.
         */
        function gameOver() {
            isGameRunning = false;
            finalScoreElement.textContent = score;
            showMessageBox(gameOverScreen);
            if (score > highScore) {
                highScore = score;
                localStorage.setItem('flappyHighScore', highScore);
                highScoreElement.textContent = highScore;
            }
        }

        // --- Event Listeners and Initialization ---

        // Resize the canvas when the window is resized
        window.addEventListener('resize', resizeCanvas);

        // Handle user input
        window.addEventListener('mousedown', birdJump);
        window.addEventListener('touchstart', birdJump);
        window.addEventListener('keydown', (e) => {
            if (e.code === 'Space' || e.code === 'ArrowUp') {
                birdJump();
            }
        });

        // Handle button clicks
        startButton.addEventListener('click', startGame);
        restartButton.addEventListener('click', startGame);

        // Initial setup
        resizeCanvas();
        highScoreElement.textContent = highScore;

    </script>
</body>
</html>
