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
    }
    canvas {
      background: #0b3d0b;
      display: block;
      margin: 20px auto;
      border: 3px solid #55ff55;
      border-radius: 10px;
    }
    #scoreboard, #leaderboard {
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
    .mobile-controls {
      display: flex;
      justify-content: center;
      margin: 10px;
      gap: 20px;
    }
    .control-btn {
      font-size: 24px;
      padding: 10px 20px;
      border-radius: 10px;
      background: #55ff55;
      color: #000;
      border: none;
    }
  </style>
</head>
<body>

<canvas id="gameCanvas" width="400" height="600"></canvas>
<div id="scoreboard">Score: 0 | Lives: 3</div>
<div id="leaderboard">High Score: 0</div>
<div id="gameOver">GAME OVER</div>
<button id="restartBtn">Play Again</button>

<div class="mobile-controls">
  <button class="control-btn" id="leftBtn">⬅️</button>
  <button class="control-btn" id="rightBtn">➡️</button>
</div>

<script>
  const canvas = document.getElementById('gameCanvas');
  const ctx = canvas.getContext('2d');

  const basketWidth = 80;
  const basketHeight = 20;
  let basketX = (canvas.width - basketWidth) / 2;

  const blobRadius = 15;
  let blobs = [];
  let powerUps = [];

  let score = 0;
  let lives = 3;
  let highScore = localStorage.getItem('highScore') || 0;

  let rightPressed = false;
  let leftPressed = false;

  let blobSpeed = 3;
  let maxBlobSpeed = 7;
  let gameOver = false;

  let doubleScoreActive = false;
  let slowMotionActive = false;

  function createBlob() {
    const x = Math.random() * (canvas.width - blobRadius * 2) + blobRadius;
    blobs.push({ x, y: -blobRadius, radius: blobRadius });
  }

  function createPowerUp() {
    const types = ['slow', 'double'];
    const type = types[Math.floor(Math.random() * types.length)];
    const x = Math.random() * (canvas.width - 20) + 10;
    powerUps.push({ x, y: -20, type });
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

  function drawPowerUps() {
    powerUps.forEach(p => {
      ctx.fillStyle = p.type === 'slow' ? '#00ffff' : '#ffcc00';
      ctx.beginPath();
      ctx.arc(p.x, p.y, 12, 0, Math.PI * 2);
      ctx.fill();
    });
  }

  function moveBasket() {
    const speed = 7;
    if (rightPressed && basketX < canvas.width - basketWidth) basketX += speed;
    if (leftPressed && basketX > 0) basketX -= speed;
  }

  function updateBlobs() {
    blobs.forEach((blob, index) => {
      blob.y += blobSpeed;
      if (
        blob.y + blob.radius > canvas.height - basketHeight - 10 &&
        blob.x > basketX &&
        blob.x < basketX + basketWidth
      ) {
        score += doubleScoreActive ? 2 : 1;
        blobs.splice(index, 1);
        updateUI();
        if (score % 5 === 0 && blobSpeed < maxBlobSpeed) {
          blobSpeed += Math.random() > 0.5 ? 0.5 : -0.2;
          blobSpeed = Math.max(2, Math.min(blobSpeed, maxBlobSpeed));
        }
      } else if (blob.y - blob.radius > canvas.height) {
        lives--;
        blobs.splice(index, 1);
        updateUI();
        if (lives <= 0) endGame();
      }
    });
  }

  function updatePowerUps() {
    powerUps.forEach((p, i) => {
      p.y += 2;
      if (
        p.y + 12 > canvas.height - basketHeight - 10 &&
        p.x > basketX &&
        p.x < basketX + basketWidth
      ) {
        activatePowerUp(p.type);
        powerUps.splice(i, 1);
      } else if (p.y > canvas.height) {
        powerUps.splice(i, 1);
      }
    });
  }

  function activatePowerUp(type) {
    if (type === 'slow') {
      blobSpeed *= 0.5;
      slowMotionActive = true;
      setTimeout(() => {
        blobSpeed *= 2;
        slowMotionActive = false;
      }, 4000);
    }
    if (type === 'double') {
      doubleScoreActive = true;
      setTimeout(() => {
        doubleScoreActive = false;
      }, 4000);
    }
  }

  function updateUI() {
    document.getElementById('scoreboard').textContent = `Score: ${score} | Lives: ${lives}`;
    document.getElementById('leaderboard').textContent = `High Score: ${Math.max(highScore, score)}`;
  }

  function clear() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
  }

  function gameLoop() {
    if (gameOver) return;
    clear();
    drawBasket();
    drawBlobs();
    drawPowerUps();
    moveBasket();
    updateBlobs();
    updatePowerUps();
    requestAnimationFrame(gameLoop);
  }

  function endGame() {
    gameOver = true;
    if (score > highScore) {
      highScore = score;
      localStorage.setItem('highScore', highScore);
    }
    document.getElementById('gameOver').style.display = 'block';
    document.getElementById('restartBtn').style.display = 'block';
    updateUI();
  }

  function restartGame() {
    score = 0;
    lives = 3;
    blobSpeed = 3;
    blobs = [];
    powerUps = [];
    doubleScoreActive = false;
    slowMotionActive = false;
    gameOver = false;
    document.getElementById('gameOver').style.display = 'none';
    document.getElementById('restartBtn').style.display = 'none';
    updateUI();
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

  // Touch controls
  document.getElementById('leftBtn').addEventListener('touchstart', () => leftPressed = true);
  document.getElementById('leftBtn').addEventListener('touchend', () => leftPressed = false);
  document.getElementById('rightBtn').addEventListener('touchstart', () => rightPressed = true);
  document.getElementById('rightBtn').addEventListener('touchend', () => rightPressed = false);

  // Restart button
  document.getElementById('restartBtn').addEventListener('click', restartGame);

  // Intervals
  setInterval(() => {
    if (!gameOver) createBlob();
  }, 1000);
  setInterval(() => {
    if (!gameOver && Math.random() < 0.4) createPowerUp();
  }, 4000);

  updateUI();
  gameLoop();
</script>

</body>
</html>
