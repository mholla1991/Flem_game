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
    overflow: hidden;
    -webkit-user-select:none;
    -moz-user-select:none;
    -ms-user-select:none;
    user-select:none;
  }
  #gameCanvas {
    background: #0b3d0b;
    display: block;
    margin: 20px auto 5px;
    border: 3px solid #55ff55;
    border-radius: 10px;
    touch-action: none;
  }
  #scoreboard, #leaderboard {
    text-align: center;
    font-size: 20px;
    margin: 5px auto;
    max-width: 400px;
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
    margin: 10px auto 20px;
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
  #instructions {
    text-align:center;
    max-width:400px;
    margin: 0 auto 10px;
    font-size: 14px;
    color: #aaa;
  }
</style>
</head>
<body>

<canvas id="gameCanvas" width="400" height="600"></canvas>
<div id="scoreboard">Score: 0 | Lives: 3</div>
<div id="leaderboard"><strong>Leaderboard:</strong><br><em>Loading...</em></div>
<div id="gameOver">GAME OVER</div>
<button id="restartBtn">Play Again</button>
<div id="instructions">Use Left/Right arrows or tap left/right side to move basket</div>

<script>
(() => {
  const canvas = document.getElementById('gameCanvas');
  const ctx = canvas.getContext('2d');
  const scoreboard = document.getElementById('scoreboard');
  const leaderboardEl = document.getElementById('leaderboard');
  const gameOverEl = document.getElementById('gameOver');
  const restartBtn = document.getElementById('restartBtn');

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
  const maxBlobSpeed = 7;

  let gameOver = false;

  // Power-ups: 10% chance blob is power-up
  const powerUpRadius = 20;
  const powerUpDuration = 7000; // ms
  let powerUpActive = false;
  let powerUpTimeout;

  // Sounds
  const catchSound = new Audio('https://actions.google.com/sounds/v1/cartoon/pop.ogg');
  const missSound = new Audio('https://actions.google.com/sounds/v1/cartoon/clang_and_wobble.ogg');
  const powerUpSound = new Audio('https://actions.google.com/sounds/v1/cartoon/boing.ogg');

  // Leaderboard stored in localStorage
  const maxLeaders = 5;
  function loadLeaderboard() {
    const stored = localStorage.getItem('flemLeaderboard');
    if (stored) return JSON.parse(stored);
    return [];
  }
  function saveLeaderboard(leaders) {
    localStorage.setItem('flemLeaderboard', JSON.stringify(leaders));
  }
  function updateLeaderboard(score) {
    let leaders = loadLeaderboard();
    leaders.push(score);
    leaders.sort((a,b) => b-a);
    if(leaders.length > maxLeaders) leaders = leaders.slice(0, maxLeaders);
    saveLeaderboard(leaders);
    renderLeaderboard(leaders);
  }
  function renderLeaderboard(leaders) {
    if(leaders.length === 0) {
      leaderboardEl.innerHTML = "<strong>Leaderboard:</strong><br><em>No scores yet</em>";
      return;
    }
    leaderboardEl.innerHTML = "<strong>Leaderboard:</strong><br>" +
      leaders.map((s,i) => `${i+1}. ${s}`).join('<br>');
  }

  // Create a new blob
  function createBlob() {
    const x = Math.random() * (canvas.width - blobRadius * 2) + blobRadius;
    const isPowerUp = Math.random() < 0.1; // 10% chance
    blobs.push({
      x,
      y: -blobRadius,
      radius: isPowerUp ? powerUpRadius : blobRadius,
      powerUp: isPowerUp,
      speed: blobSpeed * (0.8 + Math.random() * 0.4) // +/- 20% random speed
    });
  }

  // Draw basket with glow effect
  function drawBasket() {
    ctx.fillStyle = powerUpActive ? '#ffff55' : '#55ff55';
    ctx.fillRect(basketX, canvas.height - basketHeight - 10, basketWidth, basketHeight);
    ctx.strokeStyle = powerUpActive ? '#ffffaa' : '#aaffaa';
    ctx.lineWidth = 3;
    ctx.strokeRect(basketX, canvas.height - basketHeight - 10, basketWidth, basketHeight);
  }

  // Draw blobs
  function drawBlobs() {
    blobs.forEach(blob => {
      if(blob.powerUp) {
        // Yellowish power-up blob with pulsing effect
        const pulse = 0.5 + 0.5 * Math.sin(Date.now() / 300);
        const grad = ctx.createRadialGradient(blob.x, blob.y, blob.radius/3, blob.x, blob.y, blob.radius);
        grad.addColorStop(0, `rgba(255,255,100,${pulse})`);
        grad.addColorStop(1, `rgba(150,150,0,${pulse})`);
        ctx.fillStyle = grad;
        ctx.beginPath();
        ctx.arc(blob.x, blob.y, blob.radius, 0, Math.PI * 2);
        ctx.fill();

        ctx.strokeStyle = `rgba(255,255,150,${pulse})`;
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.arc(blob.x, blob.y, blob.radius + 3, 0, Math.PI * 2);
        ctx.stroke();
      } else {
        // Normal green blob
        const gradient = ctx.createRadialGradient(blob.x, blob.y, blob.radius/2, blob.x, blob.y, blob.radius);
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
      }
    });
  }

  // Move basket based on keys and touch
  function moveBasket() {
    const speed = powerUpActive ? 12 : 7;
    if (rightPressed && basketX < canvas.width - basketWidth) {
      basketX += speed;
    }
    if (leftPressed && basketX > 0) {
      basketX -= speed;
    }
    // Clamp basket inside canvas
    basketX = Math.min(Math.max(basketX, 0), canvas.width - basketWidth);
  }

  // Update blobs position and handle catches/misses
  function updateBlobs() {
    blobs.forEach((blob, index) => {
      blob.y += blob.speed;
      // Catch blob?
      if (
        blob.y + blob.radius > canvas.height - basketHeight - 10 &&
        blob.x > basketX &&
        blob.x < basketX + basketWidth
      ) {
        if(blob.powerUp) {
          activatePowerUp();
          powerUpSound.currentTime = 0;
          powerUpSound.play();
        } else {
          catchSound.currentTime = 0;
          catchSound.play();
          score++;
          // Speed variation with random easing:
          if (score % 5 === 0) {
            blobSpeed = Math.min(maxBlobSpeed, blobSpeed + (Math.random()*0.8 + 0.2));
          }
        }
        blobs.splice(index, 1);
        updateScoreboard();
      } else if (blob.y - blob.radius > canvas.height) {
        if(!blob.powerUp){
          lives--;
          missSound.currentTime = 0;
          missSound.play();
          updateScoreboard();
          if (lives <= 0) {
            endGame();
          }
        }
        blobs.splice(index, 1);
      }
    });
  }

  // Activate power-up
  function activatePowerUp() {
    powerUpActive = true;
    clearTimeout(powerUpTimeout);
    powerUpTimeout = setTimeout(() => {
      powerUpActive = false;
