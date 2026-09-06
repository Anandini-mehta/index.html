# index.html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>🚀 Space Shooter</title>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  background: radial-gradient(circle at center, #101d4a, #02030c);
  color: white;
  font-family: Arial, sans-serif;
  text-align: center;
  overflow: hidden;
}

h1 {
  margin: 10px 0;
  color: #4deeea;
  text-shadow: 0 0 15px #4deeea;
}

#gameContainer {
  position: relative;
  width: 800px;
  max-width: 96vw;
  margin: auto;
}

canvas {
  width: 100%;
  height: auto;
  border: 3px solid #4deeea;
  border-radius: 12px;
  background: #02030c;
  box-shadow: 0 0 30px #4deeea55;
}

#hud {
  position: absolute;
  top: 12px;
  left: 15px;
  right: 15px;
  display: flex;
  justify-content: space-between;
  font-size: 18px;
  font-weight: bold;
  pointer-events: none;
}

#startScreen,
#gameOverScreen {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 20, 0.88);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 10px;
}

#gameOverScreen {
  display: none;
}

.screenTitle {
  font-size: 50px;
  color: #4deeea;
  text-shadow: 0 0 20px #4deeea;
  margin: 10px;
}

.gameOverTitle {
  color: #ff416c;
  text-shadow: 0 0 20px #ff416c;
}

button {
  padding: 13px 28px;
  margin: 8px;
  border: none;
  border-radius: 10px;
  background: #4deeea;
  color: #06101d;
  font-size: 18px;
  font-weight: bold;
  cursor: pointer;
}

button:hover {
  background: white;
  transform: scale(1.05);
}

#controls {
  margin-top: 10px;
}

.controlButton {
  width: 60px;
  height: 50px;
  padding: 0;
  margin: 3px;
  font-size: 22px;
  background: #263b73;
  color: white;
}

.controlButton:active {
  background: #4deeea;
}

#shootButton {
  background: #ff416c;
  color: white;
}
</style>
</head>

<body>

<h1>🚀 SPACE SHOOTER</h1>

<div id="gameContainer">

<canvas id="game" width="800" height="600"></canvas>

<div id="hud">
  <span>🏆 Score: <span id="score">0</span></span>
  <span>❤️ Lives: <span id="lives">3</span></span>
  <span>🔥 Level: <span id="level">1</span></span>
</div>

<div id="startScreen">
  <div class="screenTitle">🚀 SPACE SHOOTER</div>

  <p>Destroy the enemies and survive!</p>
  <p>⌨️ WASD / Arrow Keys = Move</p>
  <p>🚀 Space = Shoot</p>

  <button onclick="startGame()">START GAME</button>
</div>

<div id="gameOverScreen">
  <div class="screenTitle gameOverTitle">💥 GAME OVER</div>

  <p>🏆 Score: <span id="finalScore">0</span></p>
  <p>🥇 High Score: <span id="highScore">0</span></p>

  <button onclick="restartGame()">PLAY AGAIN</button>
</div>

</div>

<div id="controls">

  <div>
    <button class="controlButton" id="up">⬆️</button>
  </div>

  <div>
    <button class="controlButton" id="left">⬅️</button>
    <button class="controlButton" id="down">⬇️</button>
    <button class="controlButton" id="right">➡️</button>
    <button class="controlButton" id="shootButton">🚀</button>
  </div>

</div>

<script>

const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

const scoreText = document.getElementById("score");
const livesText = document.getElementById("lives");
const levelText = document.getElementById("level");

const startScreen = document.getElementById("startScreen");
const gameOverScreen = document.getElementById("gameOverScreen");

let score = 0;
let lives = 3;
let level = 1;
let gameRunning = false;

let bullets = [];
let enemies = [];
let particles = [];
let stars = [];

let enemyTimer = 0;
let bossTimer = 0;

let highScore = Number(localStorage.getItem("spaceHighScore")) || 0;

document.getElementById("highScore").textContent = highScore;

const keys = {};

const player = {
  x: 375,
  y: 510,
  width: 50,
  height: 55,
  speed: 7,
  cooldown: 0
};


/* =========================
   SOUND
========================= */

let audioContext;

function sound(frequency, duration) {

  try {

    if (!audioContext) {
      audioContext = new (window.AudioContext || window.webkitAudioContext)();
    }

    const oscillator = audioContext.createOscillator();
    const gain = audioContext.createGain();

    oscillator.frequency.value = frequency;
    oscillator.type = "square";

    gain.gain.value = 0.05;

    oscillator.connect(gain);
    gain.connect(audioContext.destination);

    oscillator.start();

    oscillator.stop(
      audioContext.currentTime + duration
    );

  } catch (e) {}

}


/* =========================
   KEYBOARD
========================= */

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


/* =========================
   TOUCH CONTROLS
========================= */

function touchControl(key, element) {

  element.addEventListener("touchstart", function(e) {
    e.preventDefault();
    keys[key] = true;
  });

  element.addEventListener("touchend", function(e) {
    e.preventDefault();
    keys[key] = false;
  });

  element.addEventListener("mousedown", function() {
    keys[key] = true;
  });

  element.addEventListener("mouseup", function() {
    keys[key] = false;
  });

}

touchControl("arrowup", document.getElementById("up"));
touchControl("arrowdown", document.getElementById("down"));
touchControl("arrowleft", document.getElementById("left"));
touchControl("arrowright", document.getElementById("right"));

document.getElementById("shootButton").addEventListener("click", shoot);


/* =========================
   STARS
========================= */

for (let i = 0; i < 120; i++) {

  stars.push({
    x: Math.random() * canvas.width,
    y: Math.random() * canvas.height,
    size: Math.random() * 3,
    speed: Math.random() * 3 + 1
  });

}


/* =========================
   START GAME
========================= */

function startGame() {

  score = 0;
  lives = 3;
  level = 1;

  bullets = [];
  enemies = [];
  particles = [];

  player.x = 375;
  player.y = 510;

  scoreText.textContent = score;
  livesText.textContent = lives;
  levelText.textContent = level;

  gameRunning = true;

  startScreen.style.display = "none";
  gameOverScreen.style.display = "none";

  sound(500, 0.1);

}


/* =========================
   RESTART
========================= */

function restartGame() {
  startGame();
}


/* =========================
   SHOOT
========================= */

function shoot() {

  if (!gameRunning) return;

  if (player.cooldown > 0) return;

  bullets.push({

    x: player.x + player.width / 2 - 3,
    y: player.y,
    width: 6,
    height: 18,
    speed: 11

  });

  player.cooldown = 10;

  sound(700, 0.05);

}


/* =========================
   CREATE ENEMY
========================= */

function createEnemy() {

  if (!gameRunning) return;

  enemies.push({

    x: Math.random() * (canvas.width - 40),
    y: -50,
    width: 40,
    height: 40,

    speed: 2 + level * 0.4,

    health: 1,

    boss: false

  });

}


/* =========================
   CREATE BOSS
========================= */

function createBoss() {

  if (!gameRunning) return;

  enemies.push({

    x: canvas.width / 2 - 45,
    y: -100,

    width: 90,
    height: 70,

    speed: 1,

    health: 10,

    boss: true

  });

  sound(100, 0.5);

}


/* =========================
   COLLISION
========================= */

function collision(a, b) {

  return (

    a.x < b.x + b.width &&

    a.x + a.width > b.x &&

    a.y < b.y + b.height &&

    a.y + a.height > b.y

  );

}


/* =========================
   EXPLOSION
========================= */

function explosion(x, y) {

  for (let i = 0; i < 25; i++) {

    particles.push({

      x: x,
      y: y,

      vx: (Math.random() - 0.5) * 8,
      vy: (Math.random() - 0.5) * 8,

      life: 30,

      size: Math.random() * 5 + 2

    });

  }

  sound(120, 0.15);

}


/* =========================
   UPDATE
========================= */

function update() {

  if (!gameRunning) return;


  /* PLAYER */

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


  player.x = Math.max(
    0,
    Math.min(
      canvas.width - player.width,
      player.x
    )
  );

  player.y = Math.max(
    0,
    Math.min(
      canvas.height - player.height,
      player.y
    )
  );


  if (player.cooldown > 0) {
    player.cooldown--;
  }


  /* BULLETS */

  bullets.forEach(bullet => {

    bullet.y -= bullet.speed;

  });

  bullets = bullets.filter(
    bullet => bullet.y > -30
  );


  /* ENEMIES */

  enemies.forEach(enemy => {

    enemy.y += enemy.speed;

  });


  /* ENEMY COLLISION */

  for (let i = enemies.length - 1; i >= 0; i--) {

    const enemy = enemies[i];


    if (enemy.y > canvas.height) {

      enemies.splice(i, 1);

      loseLife();

      continue;

    }


    if (collision(player, enemy)) {

      explosion(
        enemy.x + enemy.width / 2,
        enemy.y + enemy.height / 2
      );

      enemies.splice(i, 1);

      loseLife();

    }

  }


  /* BULLET COLLISION */

  for (
    let i = bullets.length - 1;
    i >= 0;
    i--
  ) {

    for (
      let j = enemies.length - 1;
      j >= 0;
      j--
    ) {

      if (
        collision(
          bullets[i],
          enemies[j]
        )
      ) {

        enemies[j].health--;

        bullets.splice(i, 1);

        if (enemies[j].health <= 0) {

          const enemy = enemies[j];

          explosion(
            enemy.x + enemy.width / 2,
            enemy.y + enemy.height / 2
          );

          if (enemy.boss) {

            score += 100;

          } else {

            score += 10;

          }

          enemies.splice(j, 1);

          scoreText.textContent = score;

          updateLevel();

        }

        break;

      }

    }

  }


  /* ENEMY SPAWNING */

  enemyTimer++;

  const spawnRate =
    Math.max(25, 80 - level * 5);

  if (enemyTimer > spawnRate) {

    createEnemy();

    enemyTimer = 0;

  }


  /* BOSS */

  bossTimer++;

  if (
    bossTimer > 1000 &&
    level >= 3 &&
    !enemies.some(enemy => enemy.boss)
  ) {

    createBoss();

    bossTimer = 0;

  }


  /* PARTICLES */

  particles.forEach(p => {

    p.x += p.vx;
    p.y += p.vy;
    p.life--;

  });

  particles = particles.filter(
    p => p.life > 0
  );


  /* STARS */

  stars.forEach(star => {

    star.y += star.speed;

    if (star.y > canvas.height) {

      star.y = 0;
      star.x = Math.random() * canvas.width;

    }

  });

}


/* =========================
   LOSE LIFE
========================= */

function loseLife() {

  lives--;

  livesText.textContent = lives;

  sound(80, 0.2);

  if (lives <= 0) {

    endGame();

  }

}


/* =========================
   LEVEL
========================= */

function updateLevel() {

  const newLevel =
    Math.floor(score / 100) + 1;

  if (newLevel > level) {

    level = newLevel;

    levelText.textContent = level;

    sound(900, 0.2);

  }

}


/* =========================
   DRAW
========================= */

function draw() {


  /* BACKGROUND */

  ctx.fillStyle = "#02030c";

  ctx.fillRect(
    0,
    0,
    canvas.width,
    canvas.height
  );


  /* STARS */

  stars.forEach(star => {

    ctx.fillStyle = "#ffffff";

    ctx.globalAlpha = 0.4 + star.size / 4;

    ctx.fillRect(
      star.x,
      star.y,
      star.size,
      star.size
    );

  });

  ctx.globalAlpha = 1;


  /* PLAYER */

  ctx.fillStyle = "#4deeea";

  ctx.shadowColor = "#4deeea";
  ctx.shadowBlur = 20;

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
    player.y + 38
  );

  ctx.lineTo(
    player.x + player.width,
    player.y + player.height
  );

  ctx.closePath();

  ctx.fill();

  ctx.shadowBlur = 0;


  /* ENGINE */

  ctx.fillStyle = "#ffcc00";

  ctx.fillRect(
    player.x + 20,
    player.y + 43,
    10,
    15
  );


  /* BULLETS */

  bullets.forEach(bullet => {

    ctx.fillStyle = "#ffff00";

    ctx.shadowColor = "#ffff00";
    ctx.shadowBlur = 15;

    ctx.fillRect(
      bullet.x,
      bullet.y,
      bullet.width,
      bullet.height
    );

  });

  ctx.shadowBlur = 0;


  /* ENEMIES */

  enemies.forEach(enemy => {

    if (enemy.boss) {

      drawBoss(enemy);

    } else {

      drawEnemy(enemy);

    }

  });


  /* PARTICLES */

  particles.forEach(p => {

    ctx.fillStyle =
      Math.random() > 0.5
      ? "#ffcc00"
      : "#ff416c";

    ctx.globalAlpha = p.life / 30;

    ctx.fillRect(
      p.x,
      p.y,
      p.size,
      p.size
    );

  });

  ctx.globalAlpha = 1;

}


/* =========================
   DRAW ENEMY
========================= */

function drawEnemy(enemy) {

  ctx.fillStyle = "#ff416c";

  ctx.shadowColor = "#ff416c";
  ctx.shadowBlur = 15;

  ctx.beginPath();

  ctx.arc(
    enemy.x + 20,
    enemy.y + 20,
    20,
    0,
    Math.PI * 2
  );

  ctx.fill();

  ctx.shadowBlur = 0;


  /* EYES */

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

}


/* =========================
   DRAW BOSS
========================= */

function drawBoss(enemy) {

  ctx.fillStyle = "#9b5cff";

  ctx.shadowColor = "#9b5cff";
  ctx.shadowBlur = 25;

  ctx.beginPath();

  ctx.roundRect(
    enemy.x,
    enemy.y,
    enemy.width,
    enemy.height,
    15
  );

  ctx.fill();

  ctx.shadowBlur = 0;


  /* BOSS EYES */

  ctx.fillStyle = "#ff0000";

  ctx.fillRect(
    enemy.x + 18,
    enemy.y + 20,
    15,
    15
  );

  ctx.fillRect(
    enemy.x + 57,
    enemy.y + 20,
    15,
    15
  );


  /* HEALTH BAR */

  ctx.fillStyle = "#222";

  ctx.fillRect(
    enemy.x,
    enemy.y - 12,
    enemy.width,
    7
  );

  ctx.fillStyle = "#00ff66";

  ctx.fillRect(
    enemy.x,
    enemy.y - 12,
    enemy.width * (enemy.health / 10),
    7
  );

}


/* =========================
   GAME OVER
========================= */

function endGame() {

  gameRunning = false;

  finalScore.textContent = score;

  if (score > highScore) {

    highScore = score;

    localStorage.setItem(
      "spaceHighScore",
      highScore
    );

  }

  document.getElementById(
    "highScore"
  ).textContent = highScore;

  gameOverScreen.style.display = "flex";

}


/* =========================
   GAME LOOP
========================= */

function gameLoop() {

  update();

  draw();

  requestAnimationFrame(gameLoop);

}

gameLoop();

</script>

</body>
</html>
