server.js
public/
  index.html
// ================================
// MULTIPLAYER TAG GAME (ADVANCED)
// ================================

const express = require("express");
const http = require("http");
const { Server } = require("socket.io");

const app = express();
const server = http.createServer(app);
const io = new Server(server);

app.use(express.static("public"));

// ---------- CONFIG ----------
const TILE = 40;
const MAP_W = 20;
const MAP_H = 15;
const PLAYER_SPEED = 4;
const DASH_SPEED = 8;
const DASH_COOLDOWN = 1500;
const ROUND_TIME = 60;

// ---------- GAME STATE ----------
let players = {};
let map = generateMaze();
let taggerId = null;
let roundTimer = ROUND_TIME;

// ---------- MAZE GENERATION ----------
function generateMaze() {
  let maze = Array.from({ length: MAP_H }, () =>
    Array.from({ length: MAP_W }, () => 1)
  );

  function carve(x, y) {
    const dirs = [[2,0],[-2,0],[0,2],[0,-2]].sort(() => Math.random() - 0.5);
    for (let [dx, dy] of dirs) {
      const nx = x + dx, ny = y + dy;
      if (
        ny > 0 && ny < MAP_H - 1 &&
        nx > 0 && nx < MAP_W - 1 &&
        maze[ny][nx]
      ) {
        maze[ny][nx] = 0;
        maze[y + dy / 2][x + dx / 2] = 0;
        carve(nx, ny);
      }
    }
  }

  maze[1][1] = 0;
  carve(1, 1);
  return maze;
}

// ---------- ROUND TIMER ----------
setInterval(() => {
  roundTimer--;

  if (roundTimer <= 0) {
    map = generateMaze();
    roundTimer = ROUND_TIME;
    io.emit("newRound", map);
  }

  io.emit("timer", roundTimer);
}, 1000);

// ---------- SOCKET.IO ----------
io.on("connection", socket => {
  players[socket.id] = {
    x: 60,
    y: 60,
    score: 0,
    isTagger: false,
    lastDash: 0
  };

  if (!taggerId) {
    taggerId = socket.id;
    players[socket.id].isTagger = true;
  }

  socket.emit("init", {
    id: socket.id,
    map,
    tagger: taggerId
  });

  io.emit("state", players);

  socket.on("move", dir => {
    const p = players[socket.id];
    if (!p) return;

    let speed = PLAYER_SPEED;

    if (dir === "dash" && Date.now() - p.lastDash > DASH_COOLDOWN) {
      speed = DASH_SPEED;
      p.lastDash = Date.now();
    }

    if (dir === "up") p.y -= speed;
    if (dir === "down") p.y += speed;
    if (dir === "left") p.x -= speed;
    if (dir === "right") p.x += speed;

    // TAG LOGIC
    if (p.isTagger) {
      for (let id in players) {
        if (id !== socket.id) {
          const o = players[id];
          const d = Math.hypot(p.x - o.x, p.y - o.y);
          if (d < 18) {
            p.isTagger = false;
            o.isTagger = true;
            taggerId = id;
            p.score++;
          }
        }
      }
    }

    io.emit("state", players);
  });

  socket.on("disconnect", () => {
    delete players[socket.id];

    if (socket.id === taggerId) {
      taggerId = Object.keys(players)[0];
      if (taggerId) players[taggerId].isTagger = true;
    }

    io.emit("state", players);
  });
});

// ---------- START SERVER ----------
const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
  console.log("Server running on port", PORT);
});
// ================================
// MULTIPLAYER TAG GAME (ADVANCED)
// ================================

const express = require("express");
const http = require("http");
const { Server } = require("socket.io");

const app = express();
const server = http.createServer(app);
const io = new Server(server);

app.use(express.static("public"));

// ---------- CONFIG ----------
const TILE = 40;
const MAP_W = 20;
const MAP_H = 15;
const PLAYER_SPEED = 4;
const DASH_SPEED = 8;
const DASH_COOLDOWN = 1500;
const ROUND_TIME = 60;

// ---------- GAME STATE ----------
let players = {};
let map = generateMaze();
let taggerId = null;
let roundTimer = ROUND_TIME;

// ---------- MAZE GENERATION ----------
function generateMaze() {
  let maze = Array.from({ length: MAP_H }, () =>
    Array.from({ length: MAP_W }, () => 1)
  );

  function carve(x, y) {
    const dirs = [[2,0],[-2,0],[0,2],[0,-2]].sort(() => Math.random() - 0.5);
    for (let [dx, dy] of dirs) {
      const nx = x + dx, ny = y + dy;
      if (
        ny > 0 && ny < MAP_H - 1 &&
        nx > 0 && nx < MAP_W - 1 &&
        maze[ny][nx]
      ) {
        maze[ny][nx] = 0;
        maze[y + dy / 2][x + dx / 2] = 0;
        carve(nx, ny);
      }
    }
  }

  maze[1][1] = 0;
  carve(1, 1);
  return maze;
}

// ---------- ROUND TIMER ----------
setInterval(() => {
  roundTimer--;

  if (roundTimer <= 0) {
    map = generateMaze();
    roundTimer = ROUND_TIME;
    io.emit("newRound", map);
  }

  io.emit("timer", roundTimer);
}, 1000);

// ---------- SOCKET.IO ----------
io.on("connection", socket => {
  players[socket.id] = {
    x: 60,
    y: 60,
    score: 0,
    isTagger: false,
    lastDash: 0
  };

  if (!taggerId) {
    taggerId = socket.id;
    players[socket.id].isTagger = true;
  }

  socket.emit("init", {
    id: socket.id,
    map,
    tagger: taggerId
  });

  io.emit("state", players);

  socket.on("move", dir => {
    const p = players[socket.id];
    if (!p) return;

    let speed = PLAYER_SPEED;

    if (dir === "dash" && Date.now() - p.lastDash > DASH_COOLDOWN) {
      speed = DASH_SPEED;
      p.lastDash = Date.now();
    }

    if (dir === "up") p.y -= speed;
    if (dir === "down") p.y += speed;
    if (dir === "left") p.x -= speed;
    if (dir === "right") p.x += speed;

    // TAG LOGIC
    if (p.isTagger) {
      for (let id in players) {
        if (id !== socket.id) {
          const o = players[id];
          const d = Math.hypot(p.x - o.x, p.y - o.y);
          if (d < 18) {
            p.isTagger = false;
            o.isTagger = true;
            taggerId = id;
            p.score++;
          }
        }
      }
    }

    io.emit("state", players);
  });

  socket.on("disconnect", () => {
    delete players[socket.id];

    if (socket.id === taggerId) {
      taggerId = Object.keys(players)[0];
      if (taggerId) players[taggerId].isTagger = true;
    }

    io.emit("state", players);
  });
});

// ---------- START SERVER ----------
const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
  console.log("Server running on port", PORT);
});
