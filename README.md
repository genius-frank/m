```html name=index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Simple Pong Game</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h1>Simple Pong Game</h1>
    <canvas id="pong" width="700" height="400"></canvas>
    <script src="script.js"></script>
</body>
</html>
```

```css name=style.css
body {
    text-align: center;
    background: #232323;
    color: #fff;
    font-family: Arial, sans-serif;
}

canvas {
    background: #111;
    display: block;
    margin: 20px auto;
    border: 2px solid #fff;
    border-radius: 8px;
}
```

```javascript name=script.js
const canvas = document.getElementById('pong');
const ctx = canvas.getContext('2d');

// Game constants
const PADDLE_WIDTH = 10;
const PADDLE_HEIGHT = 80;
const BALL_RADIUS = 10;
const PLAYER_X = 10;
const AI_X = canvas.width - PADDLE_WIDTH - 10;
const PADDLE_SPEED = 6;
const BALL_SPEED = 5;

// Game state
let playerY = (canvas.height - PADDLE_HEIGHT) / 2;
let aiY = (canvas.height - PADDLE_HEIGHT) / 2;
let ballX = canvas.width / 2;
let ballY = canvas.height / 2;
let ballVelX = BALL_SPEED * (Math.random() > 0.5 ? 1 : -1);
let ballVelY = BALL_SPEED * (Math.random() * 2 - 1);
let playerScore = 0;
let aiScore = 0;

// Mouse control
canvas.addEventListener('mousemove', function(e) {
    const rect = canvas.getBoundingClientRect();
    const mouseY = e.clientY - rect.top;
    playerY = mouseY - PADDLE_HEIGHT / 2;
    // Clamp paddle within canvas
    if (playerY < 0) playerY = 0;
    if (playerY > canvas.height - PADDLE_HEIGHT) playerY = canvas.height - PADDLE_HEIGHT;
});

// Draw functions
function drawRect(x, y, w, h, color = '#fff') {
    ctx.fillStyle = color;
    ctx.fillRect(x, y, w, h);
}

function drawCircle(x, y, r, color = '#fff') {
    ctx.fillStyle = color;
    ctx.beginPath();
    ctx.arc(x, y, r, 0, Math.PI * 2);
    ctx.closePath();
    ctx.fill();
}

function drawText(text, x, y, size = 32) {
    ctx.fillStyle = "#fff";
    ctx.font = `${size}px Arial`;
    ctx.fillText(text, x, y);
}

// AI paddle movement
function moveAI() {
    // Simple AI: move toward the ball, but not perfectly, for challenge
    const target = ballY - (PADDLE_HEIGHT / 2);
    if (aiY + PADDLE_HEIGHT / 2 < ballY - 10) {
        aiY += PADDLE_SPEED * 0.6;
    } else if (aiY + PADDLE_HEIGHT / 2 > ballY + 10) {
        aiY -= PADDLE_SPEED * 0.6;
    }
    // Clamp paddle within canvas
    if (aiY < 0) aiY = 0;
    if (aiY > canvas.height - PADDLE_HEIGHT) aiY = canvas.height - PADDLE_HEIGHT;
}

// Ball and collision logic
function resetBall() {
    ballX = canvas.width / 2;
    ballY = canvas.height / 2;
    ballVelX = BALL_SPEED * (Math.random() > 0.5 ? 1 : -1);
    ballVelY = BALL_SPEED * (Math.random() * 2 - 1);
}

function ballCollidesWithPaddle(paddleX, paddleY) {
    return (
        ballX - BALL_RADIUS < paddleX + PADDLE_WIDTH &&
        ballX + BALL_RADIUS > paddleX &&
        ballY + BALL_RADIUS > paddleY &&
        ballY - BALL_RADIUS < paddleY + PADDLE_HEIGHT
    );
}

function update() {
    // Move ball
    ballX += ballVelX;
    ballY += ballVelY;

    // Top and bottom collision
    if (ballY - BALL_RADIUS < 0 || ballY + BALL_RADIUS > canvas.height) {
        ballVelY = -ballVelY;
        ballY = Math.max(BALL_RADIUS, Math.min(canvas.height - BALL_RADIUS, ballY));
    }

    // Player paddle collision
    if (ballCollidesWithPaddle(PLAYER_X, playerY)) {
        ballX = PLAYER_X + PADDLE_WIDTH + BALL_RADIUS;
        ballVelX = -ballVelX;
        // Add some "spin" based on where the ball hit the paddle
        let collidePoint = ballY - (playerY + PADDLE_HEIGHT / 2);
        collidePoint = collidePoint / (PADDLE_HEIGHT / 2);
        let angle = collidePoint * (Math.PI / 4);
        let direction = ballVelX > 0 ? 1 : -1;
        ballVelX = direction * BALL_SPEED * Math.cos(angle);
        ballVelY = BALL_SPEED * Math.sin(angle);
    }

    // AI paddle collision
    if (ballCollidesWithPaddle(AI_X, aiY)) {
        ballX = AI_X - BALL_RADIUS;
        ballVelX = -ballVelX;
        let collidePoint = ballY - (aiY + PADDLE_HEIGHT / 2);
        collidePoint = collidePoint / (PADDLE_HEIGHT / 2);
        let angle = collidePoint * (Math.PI / 4);
        let direction = ballVelX > 0 ? 1 : -1;
        ballVelX = direction * BALL_SPEED * Math.cos(angle);
        ballVelY = BALL_SPEED * Math.sin(angle);
    }

    // Left and right wall (score)
    if (ballX - BALL_RADIUS < 0) {
        aiScore++;
        resetBall();
    } else if (ballX + BALL_RADIUS > canvas.width) {
        playerScore++;
        resetBall();
    }

    moveAI();
}

function draw() {
    // Clear
    drawRect(0, 0, canvas.width, canvas.height, '#111');

    // Net
    for (let i = 0; i < canvas.height; i += 20) {
        drawRect(canvas.width / 2 - 1, i, 2, 10, '#fff3');
    }

    // Paddles
    drawRect(PLAYER_X, playerY, PADDLE_WIDTH, PADDLE_HEIGHT, '#0ff');
    drawRect(AI_X, aiY, PADDLE_WIDTH, PADDLE_HEIGHT, '#f00');

    // Ball
    drawCircle(ballX, ballY, BALL_RADIUS, '#fff');

    // Score
    drawText(playerScore, canvas.width / 4, 40);
    drawText(aiScore, canvas.width * 3 / 4, 40);
}

function gameLoop() {
    update();
    draw();
    requestAnimationFrame(gameLoop);
}

// Start the game
gameLoop();
'''
