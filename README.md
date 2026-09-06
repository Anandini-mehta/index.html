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
