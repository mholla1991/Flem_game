<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" />
<title>$FLEM Catch Game</title>
<style>
  body {
    margin: 0; padding: 0;
    background: #1a1a1a;
    color: #fff;
    font-family: 'Comic Sans MS', cursive, sans-serif;
    overflow: hidden;
    user-select: none;
    -webkit-user-select: none;
  }
  #gameCanvas {
    display: block;
    background: #0b3d0b;
    border: 3px solid #55ff55;
    border-radius: 10px;
    margin: 10px auto 0 auto;
    touch-action: none;
  }
  #scoreboard {
    text-align: center;
    font-size: 22px;
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
    margin: 15px auto;
    padding: 10px 25px;
    font-size: 20px;
    background: #55ff55;
    color: #000;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    display: block;
    user-select: none;
  }
  #restartBtn:hover {
    background: #44dd44;
  }
</style>
</head>
<body>

<canvas id="gameCanvas"></canvas>
<div id="scoreboard">Score: 0 | Lives: 3</div>
<div id="gameOver">GAME OVER</div>
<button id="restartBtn">Play Again</button>

<script>
(() => {
  const canvas = document.getElementById('gameCanvas');
  const ctx = canvas.getContext('2d');

  // Responsive canvas size
  function resizeCanvas() {
    canvas.width = Math.min(window.innerWidth * 0.95, 400);
    canvas.height = Math.min(window.innerHeight * 0.7, 600);
  }
  resizeCanvas();
  window.addEventListener('resize', () => {
    resizeCanvas();
    // Reset basket position when resized
    basketX = (canvas.width - basketWidth) / 2;
  });

  const basketWidth = 80;
  const basketHeight = 20;
  let basketX = (canvas.width - basketWidth) / 2;

  const blobRadius = 15;
  let blobs = [];

  let score = 0;
  let lives = 3;
  let rightPressed = false;
  let leftPressed = false;
  let blobSpeed = 3;
  const maxBlobSpeed = 8;
  let gameOver = false;

  // Touch dragging
  let dragging = false;
  let dragOffsetX = 0;

  // Limit blobs on screen for performance
  const maxBlobs = 8;

  function createBlob() {
    if (blobs.length >= maxBlobs) return; // Limit blobs
    const x = Math.random() * (canvas.width - blobRadius * 2) + blobRadius;
    blobs.push({ x: x, y: -blobRadius, radius: blobRadius });
  }

  function drawBasket() {
    ctx.fillStyle = '#55ff55';
    ctx.fillRect(basketX, canvas.height - basketHeight - 10, basketWidth, basketHeight);
    ctx.strokeStyle = '#aaffaa';
    ctx.lineWidth = 3;
    ctx.strokeRect(basketX, canvas.height - basketHeight - 10, basketWidth, basketHeight);
  }

  function drawBlobs() {
    blobs.forEach(blob => {
      const gradient = ctx.createRadialGradient(blob.x, blob.y, blob.radius / 2, blob.x, blob.y, blob.radius);
      gradient.addColorStop(0, '#a3ff88');
      gradient.addColorStop(1, '#116611');
      ctx.fillStyle = gradient;
      ctx.beginPath();
      ctx.arc(blob.x, blob.y, blob.radius, 0, Math.PI * 2);
      ctx.fill();

      ctx.fillStyle = 'rgba(255,255,255,0.6)';
      ctx.beginPath();
      ctx.arc(blob.x - 5, blob.y - 5, blob.radius / 3, 0, Math.PI * 2);
      ctx.fill();
    });
  }

  function moveBasket() {
    const speed = 7;
    if (rightPressed && basketX < canvas.width - basketWidth) {
      basketX += speed;
    }
    if (leftPressed && basketX > 0) {
      basketX -= speed;
    }
    // Clamp basket position
    if (basketX < 0) basketX = 0;
    if (basketX > canvas.width - basketWidth) basketX = canvas.width - basketWidth;
  }

  function updateBlobs() {
    blobs.forEach((blob, index) => {
      // Make blob speed randomly vary a bit for challenge easing
      let currentSpeed = blobSpeed + (Math.random() * 1 - 0.5);
      currentSpeed = Math.min(Math.max(currentSpeed, 2), maxBlobSpeed);

      blob.y += currentSpeed;

      // Check if blob caught
      if (
        blob.y + blob.radius > canvas.height - basketHeight - 10 &&
        blob.x > basketX &&
        blob.x < basketX + basketWidth
      ) {
        score++;
        blobs.splice(index, 1);
        updateScoreboard();

        // Speed up every 5 points but cap speed
        if (score % 5 === 0 && blobSpeed < maxBlobSpeed) {
          blobSpeed += 0.5;
        }
      }
      // Check if missed
      else if (blob.y - blob.radius > canvas.height) {
        lives--;
        blobs.splice(index, 1);
        updateScoreboard();

        if (lives <= 0) {
          endGame();
        }
      }
    });
  }

  function updateScoreboard() {
    document.getElementById('scoreboard').textContent = `Score: ${score} | Lives: ${lives}`;
  }

  function clear() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
  }

  function gameLoop() {
    if (gameOver) return;
    clear();
    drawBasket();
    drawBlobs();
    moveBasket();
    updateBlobs();
    requestAnimationFrame(gameLoop);
  }

  function endGame() {
    gameOver = true;
    document.getElementById('gameOver').style.display = 'block';
    document.getElementById('restartBtn').style.display = 'block';
  }

  function restartGame() {
    score = 0;
    lives = 3;
    blobSpeed = 3;
    blobs = [];
    gameOver = false;
    basketX = (canvas.width - basketWidth) / 2;
    document.getElementById('gameOver').style.display = 'none';
    document.getElementById('restartBtn').style.display = 'none';
    updateScoreboard();
    gameLoop();
  }

  // Keyboard controls
  document.addEventListener('keydown', e => {
    if (e.key === 'ArrowRight' || e.key === 'Right') rightPressed = true;
    if (e.key === 'ArrowLeft' || e.key === 'Left') leftPressed = true;
  });
  document.addEventListener('keyup', e => {
    if (e.key === 'ArrowRight' || e.key === 'Right') rightPressed = false;
    if (e.key === 'ArrowLeft' || e.key === 'Left') leftPressed = false;
  });

  // Touch controls for mobile: drag basket
  canvas.addEventListener('touchstart', e => {
    const touchX = e.touches[0].clientX - canvas.getBoundingClientRect().left;
    if (
      touchX > basketX &&
      touchX < basketX + basketWidth &&
      e.touches.length === 1
    ) {
      dragging = true;
      dragOffsetX = touchX - basketX;
    }
  });
  canvas.addEventListener('touchmove', e => {
    if (dragging) {
      e.preventDefault();
      const touchX = e.touches[0].clientX - canvas.getBoundingClientRect().left;
      basketX = touchX - dragOffsetX;
      // Clamp basket position
      if (basketX < 0) basketX = 0;
      if (basketX > canvas.width - basketWidth) basketX = canvas.width - basketWidth;
    }
  }, { passive: false });
  canvas.addEventListener('touchend', e => {
    dragging = false;
  });
  canvas.addEventListener('touchcancel', e => {
    dragging = false;
  });

  // Create blobs at intervals
  setInterval(() => {
    if (!gameOver) createBlob();
  }, 900);

  // Restart button listener
  document.getElementById('restartBtn').addEventListener('click', restartGame);

  // Start game
  updateScoreboard();
  gameLoop();
})();
</script>

</body>
</html>
