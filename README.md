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
      text-align: center;
    }
    #gameCanvas {
      background: #0b3d0b;
      display: block;
      margin: 20px auto;
      border: 3px solid #55ff55;
      border-radius: 10px;
    }
    #scoreboard, #highScore {
      font-size: 20px;
      margin: 5px 0;
    }
    #gameOver {
      font-size: 28px;
      color: #ff5555;
      display: none;
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
    #controls {
      display: flex;
      justify-content: center;
      margin-top: 10px;
    }
    .control-btn {
      width: 80px;
      height: 80px;
      margin: 0 15px;
      font-size: 40px;
      border-radius: 50%;
      background: #444;
      color: #fff;
      border: 2px solid #aaa;
    }
  </style>
</head>
<body>

<canvas id="gameCanvas" width="400" height="600"></canvas>
<div id="scoreboard">Score: 0 | Lives: 3</div>
<div id="highScore">High Score: 0</div>
<div id="gameOver">GAME OVER</div>
<button id="restartBtn">Play Again</button>

<div id="controls">
  <button class="control-btn" id="leftBtn">⬅️</button>
  <button class="control-btn" id="rightBtn">➡️</button>
</div>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const basketWidth = 80, basketHeight = 20;
let basketX = (canvas.width - basketWidth) / 2;
const blobRadius = 15;
let blobs = [];
let score = 0, lives = 3, blobSpeed = 3, gameOver = false;
let rightPressed = false, leftPressed = false;
let highScore = localStorage.getItem("flemHighScore") || 0;

// --- SOUND EFFECTS
const catchSound = new Audio("https://freesound.org/data/previews/541/541744_1217876-lq.mp3");
const powerupSound = new Audio("https://freesound.org/data/previews/352/352661_4019027-lq.mp3");

function createBlob() {
  const x = Math.random() * (canvas.width - blobRadius * 2) + blobRadius;
  const type = Math.random() < 0.1 ? 'powerup' : 'normal';
  blobs.push({ x, y: -blobRadius, radius: blobRadius, type });
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
    if (blob.type === 'powerup') {
      ctx.fillStyle = '#FFD700'; // gold
    } else {
      const gradient = ctx.createRadialGradient(blob.x, blob.y, blob.radius/2, blob.x, blob.y, blob.radius);
      gradient.addColorStop(0, '#a3ff88');
      gradient.addColorStop(1, '#116611');
      ctx.fillStyle = gradient;
    }
    ctx.beginPath();
    ctx.arc(blob.x, blob.y, blob.radius, 0, Math.PI * 2);
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
      if (blob.type === 'powerup') {
        lives++;
        powerupSound.play();
      } else {
        score++;
        catchSound.play();
        if (score % 5 === 0) blobSpeed = Math.min(blobSpeed + (Math.random() < 0.7 ? 0.5 : -0.3), 8);
      }
      blobs.splice(index, 1);
      updateScoreboard();
    } else if (blob.y - blob.radius > canvas.height) {
      if (blob.type === 'normal') lives--;
      blobs.splice(index, 1);
      updateScoreboard();
      if (lives <= 0) endGame();
    }
  });
}

function updateScoreboard() {
  document.getElementById('scoreboard').textContent = `Score: ${score} | Lives: ${lives}`;
  if (score > highScore) {
    highScore = score;
    localStorage.setItem("flemHighScore", highScore);
  }
  document.getElementById('highScore').textContent = `High Score: ${highScore}`;
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

// Keyboard
document.addEventListener('keydown', e => {
  if (e.key === 'ArrowRight') rightPressed = true;
  if (e.key === 'ArrowLeft') leftPressed = true;
});
document.addEventListener('keyup', e => {
  if (e.key === 'ArrowRight') rightPressed = false;
  if (e.key === 'ArrowLeft') leftPressed = false;
});

// Mobile buttons
document.getElementById('leftBtn').addEventListener('touchstart', () => leftPressed = true);
document.getElementById('leftBtn').addEventListener('touchend', () => leftPressed = false);
document.getElementById('rightBtn').addEventListener('touchstart', () => rightPressed = true);
document.getElementById('rightBtn').addEventListener('touchend', () => rightPressed = false);

document.getElementById('restartBtn').addEventListener('click', restartGame);
setInterval(() => { if (!gameOver) createBlob(); }, 1000);
updateScoreboard();
gameLoop();
</script>

</body>
</html>
