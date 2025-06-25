<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>$FLEM Catch Game</title>
  <style>
    body {
      margin: 0;
      background: #111;
      color: #fff;
      font-family: 'Comic Sans MS', cursive;
      overflow: hidden;
    }
    #gameCanvas {
      background: #0b3d0b;
      display: block;
      margin: 0 auto;
      border: 3px solid #55ff55;
      border-radius: 10px;
    }
    #scoreboard, #leaderboard {
      text-align: center;
      font-size: 20px;
      margin: 10px auto;
    }
    #gameOver {
      text-align: center;
      font-size: 28px;
      color: #ff5555;
      display: none;
    }
    #restartBtn {
      display: none;
      margin: 10px auto;
      padding: 10px 25px;
      font-size: 18px;
      background: #55ff55;
      color: #000;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      display: block;
    }
    #controls {
      position: absolute;
      bottom: 10px;
      left: 0;
      right: 0;
      text-align: center;
    }
    .ctrl-btn {
      font-size: 32px;
      margin: 0 20px;
      padding: 10px 20px;
      background: #333;
      border: 2px solid #aaa;
      border-radius: 10px;
      color: #fff;
    }
  </style>
</head>
<body>

<canvas id="gameCanvas" width="400" height="600"></canvas>
<div id="scoreboard">Score: 0 | Lives: 3</div>
<div id="gameOver">GAME OVER</div>
<button id="restartBtn">Play Again</button>
<div id="leaderboard"></div>

<!-- Mobile Controls -->
<div id="controls">
  <button class="ctrl-btn" id="leftBtn">⬅️</button>
  <button class="ctrl-btn" id="rightBtn">➡️</button>
</div>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

const basketWidth = 80, basketHeight = 20;
let basketX = (canvas.width - basketWidth) / 2;

let blobs = [];
let score = 0, lives = 3;
let rightPressed = false, leftPressed = false;
let blobSpeed = 2.5;
let gameOver = false;
let maxSpeed = 7;

// Sound
const catchSound = new Audio('https://actions.google.com/sounds/v1/cartoon/clang_and_wobble.ogg');
const powerupSound = new Audio('https://actions.google.com/sounds/v1/cartoon/concussive_hit_guitar_boing.ogg');

// Power-up types
const powerUps = [];

function createBlob() {
  const isPowerUp = Math.random() < 0.1;
  const x = Math.random() * (canvas.width - 30) + 15;
  if (isPowerUp) {
    const type = Math.random() < 0.5 ? 'life' : 'slow';
    powerUps.push({ x, y: -15, radius: 12, type });
  } else {
    blobs.push({ x, y: -15, radius: 15 });
  }
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
    const gradient = ctx.createRadialGradient(blob.x, blob.y, blob.radius/2, blob.x, blob.y, blob.radius);
    gradient.addColorStop(0, '#a3ff88');
    gradient.addColorStop(1, '#116611');
    ctx.fillStyle = gradient;
    ctx.beginPath();
    ctx.arc(blob.x, blob.y, blob.radius, 0, Math.PI * 2);
    ctx.fill();
  });
}

function drawPowerUps() {
  powerUps.forEach(p => {
    ctx.fillStyle = p.type === 'life' ? '#ff0' : '#0ff';
    ctx.beginPath();
    ctx.arc(p.x, p.y, p.radius, 0, Math.PI * 2);
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
      score++;
      catchSound.play();
      blobs.splice(index, 1);
      updateScoreboard();
      if (score % 5 === 0 && blobSpeed < maxSpeed) blobSpeed += 0.3;
    } else if (blob.y - blob.radius > canvas.height) {
      lives--;
      blobs.splice(index, 1);
      updateScoreboard();
      if (lives <= 0) endGame();
    }
  });
}

function updatePowerUps() {
  powerUps.forEach((p, index) => {
    p.y += blobSpeed * 0.8;
    if (
      p.y + p.radius > canvas.height - basketHeight - 10 &&
      p.x > basketX &&
      p.x < basketX + basketWidth
    ) {
      powerupSound.play();
      if (p.type === 'life') lives++;
      if (p.type === 'slow') blobSpeed = Math.max(blobSpeed - 1.5, 1.5);
      powerUps.splice(index, 1);
      updateScoreboard();
    } else if (p.y > canvas.height) {
      powerUps.splice(index, 1);
    }
  });
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

function updateScoreboard() {
  document.getElementById('scoreboard').textContent = `Score: ${score} | Lives: ${lives}`;
}

function endGame() {
  gameOver = true;
  document.getElementById('gameOver').style.display = 'block';
  document.getElementById('restartBtn').style.display = 'block';
  updateLeaderboard();
}

function restartGame() {
  score = 0; lives = 3; blobSpeed = 2.5; blobs = []; powerUps.length = 0;
  gameOver = false;
  document.getElementById('gameOver').style.display = 'none';
  document.getElementById('restartBtn').style.display = 'none';
  updateScoreboard();
  gameLoop();
}

function updateLeaderboard() {
  let topScores = JSON.parse(localStorage.getItem('flemLeaderboard')) || [];
  topScores.push(score);
  topScores = topScores.sort((a,b) => b-a).slice(0, 5);
  localStorage.setItem('flemLeaderboard', JSON.stringify(topScores));

  const html = topScores.map((s,i) => `<div>#${i+1}: ${s}</div>`).join('');
  document.getElementById('leaderboard').innerHTML = `<h3>Top Scores</h3>${html}`;
}

// Controls
document.addEventListener('keydown', e => {
  if (e.key === 'ArrowRight') rightPressed = true;
  if (e.key === 'ArrowLeft') leftPressed = true;
});
document.addEventListener('keyup', e => {
  if (e.key === 'ArrowRight') rightPressed = false;
  if (e.key === 'ArrowLeft') leftPressed = false;
});
document.getElementById('restartBtn').addEventListener('click', restartGame);
document.getElementById('leftBtn').addEventListener('touchstart', () => leftPressed = true);
document.getElementById('leftBtn').addEventListener('touchend', () => leftPressed = false);
document.getElementById('rightBtn').addEventListener('touchstart', () => rightPressed = true);
document.getElementById('rightBtn').addEventListener('touchend', () => rightPressed = false);

// Spawning
setInterval(() => {
  if (!gameOver) createBlob();
}, 900);

updateScoreboard();
updateLeaderboard();
gameLoop();
</script>

</body>
</html>
