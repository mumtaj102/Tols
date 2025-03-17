index.html<!DOCTYPE html>
<html>
<head>
    <title>Flappy Bird</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            padding: 0;
            background-color: #333;
            touch-action: manipulation;
        }

        #game-container {
            position: relative;
            width: 400px;
            height: 600px;
            max-width: 100%;
            max-height: 90vh;
        }

        #game-canvas {
            width: 100%;
            height: 100%;
        }

        .btn {
            position: absolute;
            left: 50%;
            transform: translateX(-50%);
            padding: 10px 20px;
            font-size: 20px;
            cursor: pointer;
            background: #4CAF50;
            border: none;
            border-radius: 5px;
            color: white;
            z-index: 2;
        }

        #start-btn {
            top: 50%;
            display: block;
        }

        #restart-btn {
            top: 60%;
            display: none;
        }

        #score {
            position: absolute;
            top: 20px;
            left: 50%;
            transform: translateX(-50%);
            color: white;
            font-size: 40px;
            font-family: Arial;
            font-weight: bold;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
            z-index: 1;
        }
    </style>
</head>
<body>
    <div id="game-container">
        <div id="score">0</div>
        <button id="start-btn" class="btn">Start Game</button>
        <button id="restart-btn" class="btn">Restart</button>
        <canvas id="game-canvas"></canvas>
    </div>

    <script>
        const canvas = document.getElementById('game-canvas');
        const ctx = canvas.getContext('2d');
        const startBtn = document.getElementById('start-btn');
        const restartBtn = document.getElementById('restart-btn');
        const scoreElement = document.getElementById('score');

        // Set canvas size
        const setCanvasSize = () => {
            canvas.width = canvas.offsetWidth;
            canvas.height = canvas.offsetHeight;
        };
        setCanvasSize();
        window.addEventListener('resize', setCanvasSize);

        // Load images
        const images = {
            bird: new Image(),
            pipe: new Image(),
            background: new Image()
        };

        images.bird.src = 'https://iili.io/2mONmdl.png';
        images.pipe.src = 'https://i.postimg.cc/pdBBVgZs/Pipe.png';
        images.background.src = 'https://iili.io/2mORBt9.png';

        // Game variables
        let bird = {
            x: 50,
            y: 300,
            velocity: 0,
            gravity: 0.5,
            jump: -8,
            width: 40,
            height: 30
        };

        let pipes = [];
        let score = 0;
        let gameLoop;
        let isGameActive = false;
        const pipeGap = 200;
        const pipeWidth = 60;
        const pipeSpacing = 300;

        // Event listeners
        const handleJump = () => {
            if (isGameActive) {
                bird.velocity = bird.jump;
            }
        };

        document.addEventListener('keydown', (e) => {
            if (e.code === 'Space' || e.code === 'ArrowUp') {
                handleJump();
            }
        });

        canvas.addEventListener('click', handleJump);
        canvas.addEventListener('touchstart', (e) => {
            e.preventDefault();
            handleJump();
        });

        startBtn.addEventListener('click', startGame);
        restartBtn.addEventListener('click', startGame);

        function startGame() {
            // Reset game state
            bird.y = canvas.height / 2;
            bird.velocity = 0;
            pipes = [];
            score = 0;
            scoreElement.textContent = '0';
            isGameActive = true;
            startBtn.style.display = 'none';
            restartBtn.style.display = 'none';
            
            // Start game loop
            gameLoop = requestAnimationFrame(update);
        }

        function createPipe() {
            const gapPosition = Math.random() * (canvas.height - pipeGap - 100) + 50;
            pipes.push({
                x: canvas.width,
                topHeight: gapPosition,
                bottomY: gapPosition + pipeGap,
                passed: false
            });
        }

        function checkCollision(pipe) {
            // Bird collision with pipes
            const birdLeft = bird.x;
            const birdRight = bird.x + bird.width;
            const birdTop = bird.y;
            const birdBottom = bird.y + bird.height;

            const pipeLeft = pipe.x;
            const pipeRight = pipe.x + pipeWidth;
            const topPipeBottom = pipe.topHeight;
            const bottomPipeTop = pipe.bottomY;

            // Check collision with top pipe
            if (birdRight > pipeLeft && birdLeft < pipeRight &&
                birdTop < topPipeBottom) {
                return true;
            }

            // Check collision with bottom pipe
            if (birdRight > pipeLeft && birdLeft < pipeRight &&
                birdBottom > bottomPipeTop) {
                return true;
            }

            // Check boundaries
            if (birdBottom > canvas.height || birdTop < 0) {
                return true;
            }

            return false;
        }

        function update() {
            if (!isGameActive) return;

            // Update bird
            bird.velocity += bird.gravity;
            bird.y += bird.velocity;

            // Update pipes
            if (pipes.length === 0 || pipes[pipes.length - 1].x < canvas.width - pipeSpacing) {
                createPipe();
            }

            for (let i = pipes.length - 1; i >= 0; i--) {
                pipes[i].x -= 2;

                // Score counting
                if (!pipes[i].passed && pipes[i].x + pipeWidth < bird.x) {
                    pipes[i].passed = true;
                    score++;
                    scoreElement.textContent = score;
                }

                // Collision detection
                if (checkCollision(pipes[i])) {
                    gameOver();
                    return;
                }

                // Remove off-screen pipes
                if (pipes[i].x + pipeWidth < 0) {
                    pipes.splice(i, 1);
                }
            }

            draw();
            gameLoop = requestAnimationFrame(update);
        }

        function draw() {
            // Clear canvas
            ctx.drawImage(images.background, 0, 0, canvas.width, canvas.height);

            // Draw bird
            ctx.drawImage(images.bird, bird.x, bird.y, bird.width, bird.height);

            // Draw pipes
            pipes.forEach(pipe => {
                // Top pipe (flipped)
                ctx.save();
                ctx.scale(1, -1);
                ctx.drawImage(images.pipe, pipe.x, -pipe.topHeight, pipeWidth, pipe.topHeight);
                ctx.restore();

                // Bottom pipe
                ctx.drawImage(images.pipe, pipe.x, pipe.bottomY, pipeWidth, canvas.height - pipe.bottomY);
            });
        }

        function gameOver() {
            isGameActive = false;
            cancelAnimationFrame(gameLoop);
            restartBtn.style.display = 'block';
        }

        // Wait for images to load
        Promise.all(Object.values(images).map(img => {
            if (!img.complete) {
                return new Promise(resolve => {
                    img.onload = resolve;
                });
            }
        })).then(() => {
            startBtn.disabled = false;
            draw();
        });
    </script>
</body>
</html>
