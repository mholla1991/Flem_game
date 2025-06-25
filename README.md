<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>$FLEM Catch Game</title>
  <style>
    body {
      margin: 0;
      background: #111;
      color: #fff;
      font-family: 'Comic Sans MS', cursive, sans-serif;
      overflow: hidden;
      text-align: center;
    }

    #gameCanvas {
      background: #0b3d0b;
      border: 3px solid #55ff55;
      border-radius: 10px;
      display: block;
      margin: 10px auto;
    }

    #scoreboard, #leaderboard {
      font-size: 20px;
      margin-top: 10px;
    }

    #gameOver {
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
    }

    #restartBtn:hover {
      background: #44dd44;
    }
  </style>
</head>
<body>

<canvas id="gameCanvas" width="400" height="600"></canvas>
<div id="scoreboard">Score: 0 | Lives: 3</div>
<div id="leaderboard">High Score: 0</div>
<div id="gameOver">GAME OVER</div>
<button id="restartBtn">Play Again</button>

<script>
  const canvas = document.getElementById('gameCanvas');
  const ctx = canvas.getContext('2d');
  let basketX = (canvas.width - 80) / 2;
  let rightPressed = false, leftPressed = false, touchX = null;
  const basketWidth = 80, basketHeight = 20, blobRadius = 15;
  let blobs = [], score = 0, lives = 3, blobSpeed = 3, gameOver = false;
  let highScore = parseInt(localStorage.getItem('highScore')) || 0;
  let powerUps = [], powerUpActive = false, powerUpEndTime = 0;

  const catchSound = new Audio("https://cdn.pixabay.com/download/audio/2022/03/15/audio_c9c49a6141.mp3?filename=video-game-win-201408.mp3");
  const missSound = new Audio("https://cdn.pixabay.com/download/audio/2022/03/28/audio_ba67d49d83.mp3?filename=videogame-death-sound-43894.mp3");

  function drawBasket() {
    ctx.fillStyle = '#55ff55';
    ctx.fillRect(basketX, canvas.height - basketHeight - 10, basketWidth, basketHeight);
    ctx.strokeStyle = '#aaffaa';
    ctx.lineWidth = 3;
    ctx.strokeRect(basketX, canvas.height - basketHeight - 10, basketWidth, basketHeight);
  }

  function drawBlobs() {
    blobs.forEach(blob => {
      const grad = ctx.createRadialGradient(blob.x, blob.y, blob.radius / 2, blob.x, blob.y, blob.radius);
      grad.addColorStop(0, '#a3ff88');
      grad.addColorStop(1, '#116611');
      ctx.fillStyle = grad;
      ctx.beginPath();
      ctx.arc(blob.x, blob.y, blob.radius, 0, Math.PI * 2);
      ctx.fill();
    });
  }

  function drawPowerUps() {
    powerUps.forEach(pu => {
      ctx.fillStyle = pu.type === 'slow' ? 'cyan' : 'yellow';
      ctx.beginPath();
      ctx.arc(pu.x, pu.y, 10, 0, Math.PI * 2);
      ctx.fill();
    });
  }

  function moveBasket() {
    const speed = 6;
    if (rightPressed && basketX < canvas.width - basketWidth) basketX += speed;
    if (leftPressed && basketX > 0) basketX -= speed;
  }

  function updateBlobs() {
    blobs.forEach((blob, i) => {
      blob.y += blobSpeed;
      if (
        blob.y + blob.radius > canvas.height - basketHeight - 10 &&
        blob.x > basketX &&
        blob.x < basketX + basketWidth
      ) {
        catchSound.play();
        score++;
        blobs.splice(i, 1);
        if (score % 5 === 0 && blobSpeed < 8) blobSpeed += 0.5;
        updateScoreboard();
      } else if (blob.y - blob.radius > canvas.height) {
        missSound.play();
        lives--;
        blobs.splice(i, 1);
        updateScoreboard();
        if (lives <= 0) endGame();
      }
    });
  }

  function updatePowerUps() {
    powerUps.forEach((pu, i) => {
      pu.y += 2;
      if (
        pu.y + 10 > canvas.height - basketHeight - 10 &&
        pu.x > basketX &&
        pu.x < basketX + basketWidth
      ) {
        powerUps.splice(i, 1);
        activatePowerUp(pu.type);
      } else if (pu.y > canvas.height) {
        powerUps.splice(i, 1);
      }
    });
  }

  function activatePowerUp(type) {
    if (type === 'slow') {
      powerUpActive = true;
      blobSpeed = Math.max(1, blobSpeed - 2);
      powerUpEndTime = Date.now() + 5000;
    } else if (type === 'bonus') {
      score += 5;
      updateScoreboard();
    }
  }

  function updateScoreboard() {
    document.getElementById('scoreboard').textContent = `Score: ${score} | Lives: ${lives}`;
    document.getElementById('leaderboard').textContent = `High Score: ${Math.max(highScore, score)}`;
  }

  function endGame() {
    gameOver = true;
    document.getElementById('gameOver').style.display = 'block';
    document.getElementById('restartBtn').style.display = 'block';
    if (score > highScore) {
      highScore = score;
      localStorage.setItem('highScore', highScore);
    }
  }

  function restartGame() {
    score = 0; lives = 3; blobSpeed = 3; gameOver = false;
    blobs = []; powerUps = [];
    document.getElementById('gameOver').style.display = 'none';
    document.getElementById('restartBtn').style.display = 'none';
    updateScoreboard();
    gameLoop();
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
    if (powerUpActive && Date.now() > powerUpEndTime) {
      powerUpActive = false;
      blobSpeed = Math.min(blobSpeed + 2, 8);
    }
    requestAnimationFrame(gameLoop);
  }

  function createBlob() {
    const x = Math.random() * (canvas.width - blobRadius * 2) + blobRadius;
    blobs.push({ x: x, y: -blobRadius, radius: blobRadius });
  }

  function createPowerUp() {
    const x = Math.random() * (canvas.width - 20) + 10;
    const type = Math.random() > 0.5 ? 'slow' : 'bonus';
    powerUps.push({ x: x, y: -10, type });
  }

  // Keyboard
  document.addEventListener('keydown', e => {
    if (e.key === 'ArrowRight') rightPressed = true;
    if (e.key === 'ArrowLeft') leftPressed = true;
  });
  document.addEventListener('keyup', e => {
    if (e.key === 'ArrowRight') rightPressed = false;
    if (e.key === 'ArrowLeft') leftPressed = false;
  });

  // Touch Controls
  canvas.addEventListener('touchstart', e => {
    touchX = e.touches[0].clientX;
  });
  canvas.addEventListener('touchmove', e => {
    const dx = e.touches[0].clientX - touchX;
    basketX += dx * 0.2;
    basketX = Math.max(0, Math.min(canvas.width - basketWidth, basketX));
    touchX = e.touches[0].clientX;
  });

  document.getElementById('restartBtn').addEventListener('click', restartGame);

  setInterval(() => {
    if (!gameOver) createBlob();
  }, 800);

  setInterval(() => {
    if (!gameOver && Math.random() < 0.3) createPowerUp();
  }, 5000);

  updateScoreboard();
  gameLoop();
</script>
</body>
</html>

