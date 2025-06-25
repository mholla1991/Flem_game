<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>$FLEM Catch Game</title>
<style>
  body {
    margin: 0;
    background: #1a1a1a;
    color: #fff;
    font-family: 'Comic Sans MS', cursive, sans-serif;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  #gameCanvas {
    background: #0b3d0b;
    border: 3px solid #55ff55;
    border-radius: 10px;
    touch-action: none;
  }
  #scoreboard {
    font-size: 24px;
    margin-top: 10px;
  }
  #gameOver {
    font-size: 32px;
    color: #ff5555;
    display: none;
    margin-top: 20px;
  }
  #restartBtn {
    margin: 10px auto;
    padding: 10px 25px;
    font-size: 20px;
    background: #55ff55;
    color: #000;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    display: none;
  }
  #restartBtn:hover {
    background: #44dd44;
  }
</style>
</head>
<body>

<canvas id="gameCanvas" width="400" height="600"></canvas>
<div id="scoreboard">Score: 0 | Lives: 3</div>
<div id="gameOver">GAME OVER</div>
<button id="restartBtn">Play Again</button>

<script>
  const canvas = document.getElementById('gameCanvas');
  const ctx = canvas.getContext('2d');

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
  let gameOver = false;

  // Touch control variables
  let dragging = false;
  let dragOffsetX = 0;

  function createBlob() {
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
  }

  function updateBlobs() {
    blobs.forEach((blob, index) => {
      blob.y += blobSpeed;
      if (
        blob.y + blob.radius > canvas.height - basketHeight - 10 &&
        blob.x > basketX &&
        blob.x < basketX + basketWidth
      ) {
        score++;
        blobs.splice(index, 1);
        updateScoreboard();
        if (score % 5 === 0 && blobSpeed < 8) blobSpeed += 0.5; // Max speed cap
      } else if (blob.y - blob.radius > canvas.height) {
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
    document.getElementById('gameOver').style.display = 'none';
    document.getElementById('restartBtn').style.display = 'none';
    updateScoreboard();
    gameLoop();
  }

  // Keyboard controls
  document.addEventListener('keydown', e => {
    if (e.key === 'ArrowRight') rightPressed = true;
    if (e.key === 'ArrowLeft') leftPressed = true;
  });

  document.addEventListener('keyup', e => {
    if (e.key === 'ArrowRight') rightPressed = false;
    if (e.key === 'ArrowLeft') leftPressed = false;
  });

  // Touch controls for mobile
  canvas.addEventListener('touchstart', e => {
    const touchX = e.touches[0].clientX - canvas.getBoundingClientRect().left;
    if (
      touchX > basketX &&
      touchX < basketX + basketWidth &&
      e.touches[0].clientY > canvas.height - basketHeight - 30
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
      if (basketX < 0) basketX = 0;
      if (basketX > canvas.width - basketWidth) basketX = canvas.width - basketWidth;
    }
  }, { passive: false });

  canvas.addEventListener('touchend', e => {
    dragging = false;
  });

  // Create blobs at intervals
  setInterval(() => {
    if (!gameOver) createBlob();
  }, 1000);

  // Restart button
  document.getElementById('restartBtn').addEventListener('click', restartGame);

  updateScoreboard();
  gameLoop();
</script>

</body>
</html>
