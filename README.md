<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no" />
<title>$FLEM Catch Game</title>
<style>
  body {
    margin: 0;
    background: #1a1a1a;
    color: #fff;
    font-family: 'Comic Sans MS', cursive, sans-serif;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-start;
    height: 100vh;
    overflow: hidden;
    -webkit-tap-highlight-color: transparent;
  }
  #gameContainer {
    position: relative;
    max-width: 400px;
    width: 90vw;
  }
  canvas {
    background: #0b3d0b;
    display: block;
    border: 3px solid #55ff55;
    border-radius: 10px;
    width: 100% !important;
    height: auto !important;
    touch-action: none;
    user-select: none;
  }
  #scoreboard {
    text-align: center;
    font-size: 20px;
    margin-top: 10px;
  }
  #gameOver {
    text-align: center;
    font-size: 32px;
    color: #ff5555;
    display: none;
    margin-top: 20px;
  }
  #restartBtn {
    display: none;
    margin: 10px auto;
    padding: 10px 25px;
    font-size: 20px;
    background: #55ff55;
    color: #000;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    display: block;
  }
  #restartBtn:hover {
    background: #44dd44;
  }
  #touchControls {
    margin-top: 15px;
    display: flex;
    justify-content: center;
    gap: 40px;
  }
  .touchBtn {
    background: #55ff55;
    border-radius: 8px;
    padding: 15px 25px;
    font-size: 24px;
    color: #000;
    user-select: none;
    touch-action: manipulation;
    cursor: pointer;
    width: 80px;
    text-align: center;
    font-weight: bold;
    box-shadow: 0 0 8px #44dd44;
  }
  .touchBtn:active {
    background: #44dd44;
    box-shadow: 0 0 12px #22bb22;
  }
</style>
</head>
<body>

<div id="gameContainer">
  <canvas id="gameCanvas" width="400" height="600"></canvas>
  <div id="scoreboard">Score: 0 | Lives: 3</div>
  <div id="gameOver">GAME OVER</div>
  <button id="restartBtn">Play Again</button>
  <div id="touchControls">
    <div id="leftBtn" class="touchBtn">◀️</div>
    <div id="rightBtn" class="touchBtn">▶️</div>
  </div>
</div>

<script>
(() => {
  const canvas = document.getElementById('gameCanvas');
  const ctx = canvas.getContext('2d');
  const scoreboard = document.getElementById('scoreboard');
  const gameOverText = document.getElementById('gameOver');
  const restartBtn = document.getElementById('restartBtn');
  const leftBtn = document.getElementById('leftBtn');
  const rightBtn = document.getElementById('rightBtn');

  const baseWidth = 400;
  const baseHeight = 600;

  // Basket properties
  const basketWidth = 80;
  const basketHeight = 20;
  let basketX = (baseWidth - basketWidth) / 2;

  // Blob properties
  const blobRadius = 15;
  let blobs = [];

  // Game state
  let score = 0;
  let lives = 3;
  let blobSpeed = 3;
  const maxSpeed = 8;
  let gameOver = false;

  // Movement flags
  let rightPressed = false;
  let leftPressed = false;
  let touchLeft = false;
  let touchRight = false;

  // Sounds (simple beep)
  const beepCatch = new Audio('data:audio/wav;base64,UklGRiQAAABXQVZFZm10IBAAAAABAAEAgD4AAAB9AAACABAAZGF0YQAAAAA=');
  const beepMiss = new Audio('data:audio/wav;base64,UklGRiQAAABXQVZFZm10IBAAAAABAAEAgD4AAAB9AAACABAAZGF0YQAAAAA=');

  // Responsive canvas sizing
  function resizeCanvas() {
    const container = document.getElementById('gameContainer');
    const containerWidth = container.clientWidth;
    canvas.style.width = containerWidth + 'px';
    canvas.style.height = (containerWidth * (baseHeight / baseWidth)) + 'px';
  }

  window.addEventListener('resize', resizeCanvas);
  resizeCanvas();

  // Create new blob at random X position
  function createBlob() {
    const x = Math.random() * (baseWidth - blobRadius * 2) + blobRadius;
    blobs.push({ x, y: -blobRadius, radius: blobRadius });
  }

  // Draw basket with glow
  function drawBasket() {
    ctx.fillStyle = '#55ff55';
    ctx.fillRect(basketX, baseHeight - basketHeight - 10, basketWidth, basketHeight);
    ctx.strokeStyle = '#aaffaa';
    ctx.lineWidth = 3;
    ctx.strokeRect(basketX, baseHeight - basketHeight - 10, basketWidth, basketHeight);
  }

  // Draw blobs (green snot-like)
  function drawBlobs() {
    blobs.forEach(blob => {
      const gradient = ctx.createRadialGradient(blob.x, blob.y, blob.radius / 2, blob.x, blob.y, blob.radius);
      gradient.addColorStop(0, '#a3ff88');
      gradient.addColorStop(1, '#116611');
      ctx.fillStyle = gradient;
      ctx.beginPath();
      ctx.arc(blob.x, blob.y, blob.radius, 0, Math.PI * 2);
      ctx.fill();

      // Shine
      ctx.fillStyle = 'rgba(255,255,255,0.6)';
      ctx.beginPath();
      ctx.arc(blob.x - 5, blob.y - 5, blob.radius / 3, 0, Math.PI * 2);
      ctx.fill();
    });
  }

  // Clear canvas for redraw
  function clear() {
    ctx.clearRect(0, 0, baseWidth, baseHeight);
  }

  // Move basket by pressed keys or touch
  function moveBasket() {
    const speed = 7;
    if ((rightPressed || touchRight) && basketX < baseWidth - basketWidth) {
      basketX += speed;
      if (basketX > baseWidth - basketWidth) basketX = baseWidth - basketWidth;
    }
    if ((leftPressed || touchLeft) && basketX > 0) {
      basketX -= speed;
      if (basketX < 0) basketX = 0;
    }
  }

  // Update blob positions & collisions
  function updateBlobs() {
    blobs.forEach((blob, i) => {
      // Move down with random speed variation for difficulty fluctuation
      const variation = (Math.sin(score / 3 + i) * 0.7);
      blob.y += Math.min(blobSpeed + variation, maxSpeed);

      // Check catch
      if (
        blob.y + blob.radius > baseHeight - basketHeight - 10 &&
        blob.x > basketX &&
        blob.x < basketX + basketWidth
      ) {
        score++;
        blobs.splice(i, 1);
        beepCatch.play().catch(() => {}); // play catch sound
        updateScoreboard();
        // Speed adjustment with some random difficulty
        if (score % 5 === 0 && blobSpeed < maxSpeed) {
          blobSpeed += 0.5;
        }
      }
      // Missed blob
      else if (blob.y - blob.radius > baseHeight) {
        lives--;
        blobs.splice(i, 1);
        beepMiss.play().catch(() => {}); // play miss sound
        updateScoreboard();
        if (lives <= 0) {
          endGame();
        }
      }
    });
  }

  // Update scoreboard text
  function updateScoreboard() {
    scoreboard.textContent = `Score: ${score} | Lives: ${lives}`;
  }

  // End game handler
  function endGame() {
    gameOver = true;
    gameOverText.style.display = 'block';
    restartBtn.style.display = 'block';
  }

  // Game loop
  function gameLoop() {
    if (gameOver) return;
    clear();
    drawBasket();
    drawBlobs();
    moveBasket();
    updateBlobs();
    requestAnimationFrame(gameLoop);
  }

  // Restart game reset
  function restartGame() {
    score = 0;
    lives = 3;
    blobSpeed = 3;
    blobs = [];
    basketX = (baseWidth - basketWidth) / 2;
    gameOver = false;
    gameOverText.style.display = 'none';
    restartBtn.style.display = 'none';
    updateScoreboard();
    gameLoop();
  }

  // Keyboard event listeners
  window.addEventListener('keydown', e => {
    if (e.key === 'ArrowRight' || e.key.toLowerCase() === 'd') rightPressed = true;
    if (e.key === 'ArrowLeft' || e.key.toLowerCase() === 'a') leftPressed = true;
  });
  window.addEventListener('keyup', e => {
    if (e.key === 'ArrowRight' || e.key.toLowerCase() === 'd') rightPressed = false;
    if (e.key === 'ArrowLeft' || e.key.toLowerCase() === 'a') leftPressed = false;
  });

  // Touch controls
  leftBtn.addEventListener('touchstart', e => { e.preventDefault(); touchLeft = true; });
  leftBtn.addEventListener('touchend', e => { e.preventDefault(); touchLeft = false; });
  rightBtn.addEventListener('touchstart', e => { e.preventDefault(); touchRight = true; });
  rightBtn.addEventListener('touchend', e => { e.preventDefault(); touchRight = false; });

  // Prevent scrolling on touch buttons
  leftBtn.addEventListener('touchmove', e => e.preventDefault());
  rightBtn.addEventListener('touchmove', e => e.preventDefault());

  // Create blobs periodically
  setInterval(() => {
    if (!gameOver) createBlob();
  }, 900);

  // Restart button
  restartBtn.addEventListener('click', restartGame);

  // Start game
  updateScoreboard();
  gameLoop();
})();
</script>

</body>
</html>
