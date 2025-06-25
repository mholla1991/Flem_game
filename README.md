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

    #mobileControls {
      margin: 20px;
    }

    #mobileControls button {
      font-size: 32px;
      padding: 10px 20px;
      margin: 10px;
      background-color: #55ff55;
      border: none;
      border-radius: 10px;
      color: #000;
    }

    #mobileControls button:active {
      background-color: #44dd44;
    }
  </style>
</head>
<body>

<canvas id="gameCanvas" width="400" height="600"></canvas>
<div id="scoreboard">Score: 0 | Lives: 3</div>
<div id="gameOver">GAME OVER</div>
<button id="restartBtn">Play Again</button>

<div id="mobileControls">
  <button id="leftBtn">⬅️</button>
  <button id="rightBtn">➡️</button>
</div>

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
  let blobSpeed = 3;
  let maxBlobSpeed = 10;
  let gameOver = false;

  let rightPressed = false;
  let leftPressed = false;

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
        if (score % 5 === 0 && blobSpeed < maxBlobSpeed) {
          blobSpeed += 0.5;
        }
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
    try {
      clear();
      drawBasket();
      drawBlobs();
      moveBasket();
      updateBlobs();
    } catch (e) {
      console.error(e);
      endGame();
    }
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

  // Mobile controls
  document.getElementById('leftBtn').addEventListener('touchstart', e => {
    e.preventDefault(); leftPressed = true;
  });
  document.getElementById('leftBtn').addEventListener('touchend', e => {
    e.preventDefault(); leftPressed = false;
  });
  document.getElementById('rightBtn').addEventListener('touchstart', e => {
    e.preventDefault(); rightPressed = true;
  });
  document.getElementById('rightBtn').addEventListener('touchend', e => {
    e.preventDefault(); rightPressed = false;
  });

  // Blob spawn control
  setInterval(() => {
    if (!gameOver && blobs.length < 7) createBlob();
  }, 1200);

  document.getElementById('restartBtn').addEventListener('click', restartGame);

  updateScoreboard();
  gameLoop();
</script>

</body>
</html>
