# index.html
<!DOCTYPE html>
<html>
<head>
  <title>Space Shooter</title>
  <style>
    body {
      margin: 0;
      background: #050816;
      color: white;
      font-family: Arial;
      text-align: center;
    }

    h1 {
      color: #4deeea;
    }

    canvas {
      background: #050816;
      border: 3px solid #4deeea;
      max-width: 95%;
    }

    #score {
      font-size: 22px;
      margin: 10px;
    }

    button {
      padding: 12px 25px;
      font-size: 18px;
      cursor: pointer;
      border: none;
      border-radius: 8px;
    }
  </style>
</head>

<body>

<h1>🚀 Space Shooter</h1>

<div id="score">Score: 0</div>

<canvas id="game" width="800" height="600"></canvas>

<br><br>

<button onclick="restart()">🔄 Restart</button>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

let score = 0;
let gameOver = false;

const player = {
  x: 375,
  y: 520,
  width: 50,
  height: 40,
  speed: 6
};

let bullets = [];
let enemies = [];
let keys = {};

// Keyboard controls
document.addEventListener("keydown", function(e) {
  keys[e.key.toLowerCase()] = true;

  if (e.code === "Space") {
    shoot();
    e.preventDefault();
  }
});

document.addEventListener("keyup", function(e) {
  keys[e.key.toLowerCase()] = false;
});

// Shoot
function shoot() {
  if (gameOver) return;

  bullets.push({
    x: player.x + player.width / 2 - 3,
    y: player.y,
    width: 6,
    height: 15,
    speed: 9
  });
}

// Create enemy
function createEnemy() {
  if (gameOver) return;

  enemies.push({
    x: Math.random() * (canvas.width - 40),
    y: -40,
    width: 40,
    height: 40,
    speed: 2
  });
}

// Collision
function collision(a, b) {
  return (
    a.x < b.x + b.width &&
    a.x + a.width > b.x &&
    a.y < b.y + b.height &&
    a.y + a.height > b.y
  );
}

// Update
function update() {

  if (gameOver) return;

  // Move player
  if (keys["arrowleft"] || keys["a"]) {
    player.x -= player.speed;
  }

  if (keys["arrowright"] || keys["d"]) {
    player.x += player.speed;
  }

  if (keys["arrowup"] || keys["w"]) {
    player.y -= player.speed;
  }

  if (keys["arrowdown"] || keys["s"]) {
    player.y += player.speed;
  }

  // Keep player inside screen
  player.x = Math.max(
    0,
    Math.min(canvas.width - player.width, player.x)
  );

  player.y = Math.max(
    0,
    Math.min(canvas.height - player.height, player.y)
  );

  // Move bullets
  bullets.forEach(bullet => {
    bullet.y -= bullet.speed;
  });

  bullets = bullets.filter(bullet => bullet.y > -20);

  // Move enemies
  enemies.forEach(enemy => {
    enemy.y += enemy.speed;
  });

  // Check enemies
  for (let enemy of enemies) {

    if (enemy.y > canvas.height) {
      endGame();
    }

    if (collision(player, enemy)) {
      endGame();
    }
  }

  // Bullet vs enemy
  for (let i = bullets.length - 1; i >= 0; i--) {

    for (let j = enemies.length - 1; j >= 0; j--) {

      if (collision(bullets[i], enemies[j])) {

        bullets.splice(i, 1);
        enemies.splice(j, 1);

        score += 10;

        document.getElementById("score").textContent =
          "Score: " + score;

        break;
      }
    }
  }
}

// Draw
function draw() {

  ctx.fillStyle = "#050816";
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  // Stars
  ctx.fillStyle = "white";

  for (let i = 0; i < 80; i++) {
    let x = (i * 97) % canvas.width;
    let y = (i * 53) % canvas.height;

    ctx.fillRect(x, y, 2, 2);
  }

  // Player
  ctx.fillStyle = "#4deeea";

  ctx.beginPath();

  ctx.moveTo(
    player.x + player.width / 2,
    player.y
  );

  ctx.lineTo(
    player.x,
    player.y + player.height
  );

  ctx.lineTo(
    player.x + player.width / 2,
    player.y + 30
  );

  ctx.lineTo(
    player.x + player.width,
    player.y + player.height
  );

  ctx.closePath();

  ctx.fill();

  // Bullets
  ctx.fillStyle = "yellow";

  bullets.forEach(bullet => {
    ctx.fillRect(
      bullet.x,
      bullet.y,
      bullet.width,
      bullet.height
    );
  });

  // Enemies
  enemies.forEach(enemy => {

    ctx.fillStyle = "#ff3366";

    ctx.beginPath();

    ctx.arc(
      enemy.x + 20,
      enemy.y + 20,
      20,
      0,
      Math.PI * 2
    );

    ctx.fill();

    ctx.fillStyle = "white";

    ctx.fillRect(
      enemy.x + 10,
      enemy.y + 12,
      6,
      6
    );

    ctx.fillRect(
      enemy.x + 24,
      enemy.y + 12,
      6,
      6
    );
  });

  // Game over
  if (gameOver) {

    ctx.fillStyle = "rgba(0,0,0,0.75)";
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    ctx.fillStyle = "white";
    ctx.font = "50px Arial";
    ctx.textAlign = "center";

    ctx.fillText(
      "GAME OVER",
      canvas.width / 2,
      canvas.height / 2
    );

    ctx.font = "25px Arial";

    ctx.fillText(
      "Score: " + score,
      canvas.width / 2,
      canvas.height / 2 + 50
    );
  }
}

// Game over
function endGame() {
  gameOver = true;
}

// Restart
function restart() {

  score = 0;
  gameOver = false;

  player.x = 375;
  player.y = 520;

  bullets = [];
  enemies = [];

  document.getElementById("score").textContent =
    "Score: 0";
}

// Spawn enemies
setInterval(function() {
  if (!gameOver) {
    createEnemy();
  }
}, 1000);

// Game loop
function gameLoop() {
  update();
  draw();
  requestAnimationFrame(gameLoop);
}

gameLoop();
</script>

</body>
</html>
