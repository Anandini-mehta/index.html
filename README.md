# index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Space Shooter</title>

  <style>
    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      background: #050816;
      color: white;
      font-family: Arial, sans-serif;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      overflow: hidden;
    }

    #game {
      position: relative;
      width: 800px;
      max-width: 95vw;
      height: 600px;
      max-height: 90vh;
      background: linear-gradient(#050816, #101a3a);
      border: 3px solid #4deeea;
      border-radius: 12px;
      overflow: hidden;
      box-shadow: 0 0 30px #4deeea55;
    }

    canvas {
      width: 100%;
      height: 100%;
      display: block;
    }

    #info {
      position: absolute;
      top: 12px;
      left: 15px;
      font-size: 20px;
      font-weight: bold;
      text-shadow: 0 0 8px #4deeea;
      pointer-events: none;
    }

    #gameOver {
      position: absolute;
      inset: 0;
      display: none;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      background: rgba(0, 0, 20, 0.85);
      text-align: center;
    }

    #gameOver h1 {
      color: #ff4d6d;
      font-size: 55px;
      margin: 10px;
    }

    #gameOver p {
      font-size: 24px;
    }

    button {
      padding: 14px 30px;
      border: none;
      border-radius: 8px;
      background: #4deeea;
      color: #06101d;
      font-size: 18px;
      font-weight: bold;
      cursor: pointer;
    }

    button:hover {
      background: white;
    }
  </style>
</head>

<body>

<div id="game">
  <canvas id="canvas" width="800" height="600"></canvas>

  <div id="info">
    Score: <span id="score">0</span>
  </div>

  <div id="gameOver">
    <h1>GAME OVER</h1>
    <p>Final Score: <span id="finalScore">0</span></p>
    <button onclick="restartGame()">Play Again</button>
  </div>
</div>

<script>
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

const scoreElement = document.getElementById("score");
const finalScoreElement = document.getElementById("finalScore");
const gameOverScreen = document.getElementById("gameOver");

let score = 0;
let gameRunning = true;

const keys = {};

document.addEventListener("keydown", (event) => {
  keys[event.key.toLowerCase()] = true;

  if (event.code === "Space") {
    shoot();
    event.preventDefault();
  }
});

document.addEventListener("keyup", (event) => {
  keys[event.key.toLowerCase()] = false;
});

const player = {
  x: 375,
  y: 520,
  width: 50,
  height: 50,
  speed: 7,
  cooldown: 0
};

let bullets = [];
let enemies = [];
let stars = [];

// Create background stars
for (let i = 0; i < 100; i++) {
  stars.push({
    x: Math.random() * canvas.width,
    y: Math.random() * canvas.height,
    size: Math.random() * 2 + 1,
    speed: Math.random() * 2 + 0.5
  });
}

// Draw player spaceship
function drawPlayer() {
  ctx.fillStyle = "#4deeea";

  ctx.beginPath();
  ctx.moveTo(player.x + player.width / 2, player.y);
  ctx.lineTo(player.x, player.y + player.height);
  ctx.lineTo(player.x + player.width / 2, player.y + 38);
  ctx.lineTo(player.x + player.width, player.y + player.height);
  ctx.closePath();
  ctx.fill();

  // Engine glow
  ctx.fillStyle = "#ffcc00";
  ctx.fillRect(
    player.x + 20,
    player.y + 42,
    10,
    10
  );
}

// Shoot bullet
function shoot() {
  if (!gameRunning || player.cooldown > 0) return;

  bullets.push({
    x: player.x + player.width / 2 - 3,
    y: player.y,
    width: 6,
    height: 15,
    speed: 10
  });

  player.cooldown = 12;
}

// Create enemy
function createEnemy() {
  enemies.push({
    x: Math.random() * (canvas.width - 40),
    y: -50,
    width: 40,
    height: 40,
