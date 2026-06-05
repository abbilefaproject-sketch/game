<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Mapel Dodge — by Abbilefa</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    background: #080e1e;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    font-family: 'Consolas', monospace;
    overflow: hidden;
  }
  canvas {
    display: block;
    image-rendering: pixelated;
  }
</style>
</head>
<body>
<canvas id="game"></canvas>
<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');

const W = 480, H = 700;
canvas.width = W;
canvas.height = H;

// Scale canvas to fit screen
function resize() {
  const scale = Math.min(window.innerWidth / W, window.innerHeight / H);
  canvas.style.width = W * scale + 'px';
  canvas.style.height = H * scale + 'px';
}
resize();
window.addEventListener('resize', resize);

// COLORS
const C = {
  BG:         '#080e1e',
  BG2:        '#0c1632',
  PANEL:      '#0f1c41',
  BORDER:     '#1e50b4',
  ACCENT:     '#3c8cff',
  ACCENT2:    '#64c8ff',
  WHITE:      '#dcebff',
  DIM:        '#506ea0',
  GOOD:       '#32dc82',
  GOOD_DIM:   '#145032',
  DANGER:     '#ff4650',
  DANGER_DIM: '#501019',
  GOLD:       '#ffc832',
  HEART:      '#ff5064',
};

// MAPEL DATA
const MAPEL_LIST = [
  ["MATEMATIKA",  "#3c82ff"],
  ["FISIKA",      "#64b4ff"],
  ["BIOLOGI",     "#32dc82"],
  ["KIMIA",       "#b464ff"],
  ["GEOGRAFI",    "#ffaa3c"],
  ["SOSIOLOGI",   "#ffdc3c"],
  ["EKONOMI",     "#3cd2b4"],
  ["SEJARAH",     "#ff823c"],
  ["PKN",         "#ff5064"],
  ["SENI RUPA",   "#ff78c8"],
  ["PEND. AGAMA", "#ffc850"],
  ["INFORMATIKA", "#50dcff"],
  ["OLAHRAGA",    "#82ff82"],
];

// STARS
const stars = Array.from({length:90}, () => ({
  x: Math.random()*W,
  y: Math.random()*H,
  size: 0.3 + Math.random()*1.2,
  phase: Math.random()*Math.PI*2
}));

// PARTICLES
let particles = [];
function spawnParticles(x, y, color, count=12) {
  for(let i=0;i<count;i++) {
    const angle = Math.random()*Math.PI*2;
    const speed = 2 + Math.random()*4;
    particles.push({
      x, y,
      vx: Math.cos(angle)*speed,
      vy: Math.sin(angle)*speed,
      life: 1.0,
      color,
      size: 3 + Math.random()*4
    });
  }
}
function updateDrawParticles() {
  particles = particles.filter(p => {
    p.x += p.vx; p.y += p.vy; p.vy += 0.15; p.life -= 0.04;
    if(p.life <= 0) return false;
    const alpha = p.life;
    const hex = p.color;
    const r = parseInt(hex.slice(1,3),16);
    const g = parseInt(hex.slice(3,5),16);
    const b = parseInt(hex.slice(5,7),16);
    ctx.beginPath();
    ctx.arc(p.x, p.y, Math.max(1, p.size*alpha), 0, Math.PI*2);
    ctx.fillStyle = `rgba(${r},${g},${b},${alpha})`;
    ctx.fill();
    return true;
  });
}

// FLOAT TEXTS
let floatTexts = [];
function spawnFloatText(x, y, text, color) {
  floatTexts.push({x, y, text, color, life: 1.0, vy: -2.5});
}
function updateDrawFloatTexts() {
  floatTexts = floatTexts.filter(ft => {
    ft.y += ft.vy; ft.life -= 0.025;
    if(ft.life <= 0) return false;
    const alpha = Math.min(1, ft.life*2);
    const hex = ft.color;
    const r = parseInt(hex.slice(1,3),16);
    const g = parseInt(hex.slice(3,5),16);
    const b = parseInt(hex.slice(5,7),16);
    ctx.font = 'bold 22px Consolas, monospace';
    ctx.textAlign = 'center';
    ctx.fillStyle = `rgba(${r},${g},${b},${alpha})`;
    ctx.fillText(ft.text, ft.x, ft.y);
    return true;
  });
}

// HELPERS
function lerpColor(c1hex, c2hex, t) {
  const r1=parseInt(c1hex.slice(1,3),16), g1=parseInt(c1hex.slice(3,5),16), b1=parseInt(c1hex.slice(5,7),16);
  const r2=parseInt(c2hex.slice(1,3),16), g2=parseInt(c2hex.slice(3,5),16), b2=parseInt(c2hex.slice(5,7),16);
  const r=Math.round(r1+(r2-r1)*t), g=Math.round(g1+(g2-g1)*t), b=Math.round(b1+(b2-b1)*t);
  return `rgb(${r},${g},${b})`;
}

function drawRoundedRect(x, y, w, h, r, fill, strokeColor=null, lineW=0) {
  ctx.beginPath();
  ctx.moveTo(x+r, y);
  ctx.lineTo(x+w-r, y);
  ctx.quadraticCurveTo(x+w, y, x+w, y+r);
  ctx.lineTo(x+w, y+h-r);
  ctx.quadraticCurveTo(x+w, y+h, x+w-r, y+h);
  ctx.lineTo(x+r, y+h);
  ctx.quadraticCurveTo(x, y+h, x, y+h-r);
  ctx.lineTo(x, y+r);
  ctx.quadraticCurveTo(x, y, x+r, y);
  ctx.closePath();
  if(fill) { ctx.fillStyle=fill; ctx.fill(); }
  if(strokeColor && lineW>0) { ctx.strokeStyle=strokeColor; ctx.lineWidth=lineW; ctx.stroke(); }
}

function drawStars(tick) {
  for(const s of stars) {
    const alpha = 0.5 + 0.5*Math.sin(tick*0.02 + s.phase);
    const brightness = Math.floor(60 + 120*alpha);
    ctx.beginPath();
    ctx.arc(s.x, s.y, Math.max(1, s.size), 0, Math.PI*2);
    ctx.fillStyle = `rgb(${Math.floor(brightness/3)},${Math.floor(brightness/2)},${brightness})`;
    ctx.fill();
  }
}

// BLOCK
const BLOCK_W = 90, BLOCK_H = 46;
class Block {
  constructor(speed, isTarget, mapelIdx) {
    this.w = BLOCK_W; this.h = BLOCK_H;
    this.x = 20 + Math.random()*(W - 20 - BLOCK_W - 20);
    this.y = -this.h - Math.random()*80;
    this.speed = speed;
    this.isTarget = isTarget;
    this.mapelName = MAPEL_LIST[mapelIdx][0];
    this.baseColor = MAPEL_LIST[mapelIdx][1];
    this.alive = true;
    this.wobble = Math.random()*Math.PI*2;
  }
  update(tick) {
    this.wobble += 0.05;
    this.x += Math.sin(this.wobble)*0.4;
    this.y += this.speed;
    if(this.y > H + this.h) this.alive = false;
  }
  draw(tick) {
    const rx = Math.floor(this.x), ry = Math.floor(this.y);
    if(this.isTarget) {
      const pulse = 0.5 + 0.5*Math.sin(tick*0.1);
      const glow = lerpColor(C.GOOD_DIM, C.GOOD, pulse);
      drawRoundedRect(rx-3, ry-3, this.w+6, this.h+6, 12, glow);
      drawRoundedRect(rx, ry, this.w, this.h, 9, '#0f3220', C.GOOD, 2);
    } else {
      // Shadow
      drawRoundedRect(rx+3, ry+4, this.w, this.h, 9, 'rgba(0,0,0,0.6)');
      const bg = lerpColor(C.PANEL, this.baseColor, 0.18);
      drawRoundedRect(rx, ry, this.w, this.h, 9, bg, this.baseColor, 2);
    }
    // Label
    const labelColor = this.isTarget ? C.GOOD : this.baseColor;
    const lines = this.mapelName.split(' ');
    ctx.font = 'bold 13px Consolas, monospace';
    ctx.textAlign = 'center';
    ctx.fillStyle = labelColor;
    if(lines.length === 1) {
      ctx.fillText(lines[0], rx + this.w/2, ry + this.h/2 + 5);
    } else {
      ctx.fillText(lines[0], rx + this.w/2, ry + this.h/2 - 4);
      ctx.fillText(lines[1], rx + this.w/2, ry + this.h/2 + 12);
    }
  }
  getRect() { return {x: this.x, y: this.y, w: this.w, h: this.h}; }
}

// PLAYER
const PLAYER_W = 52, PLAYER_H = 52;
class Player {
  constructor() {
    this.x = W/2 - PLAYER_W/2;
    this.y = H - 110;
    this.speed = 6;
    this.invincible = 0;
    this.trail = [];
  }
  update(keys) {
    if(keys['ArrowLeft'] || keys['a'] || keys['A']) this.x -= this.speed;
    if(keys['ArrowRight'] || keys['d'] || keys['D']) this.x += this.speed;
    this.x = Math.max(8, Math.min(W - PLAYER_W - 8, this.x));
    if(this.invincible > 0) this.invincible--;
    const cx = this.x + PLAYER_W/2, cy = this.y + PLAYER_H/2;
    this.trail.push({x:cx, y:cy});
    if(this.trail.length > 12) this.trail.shift();
  }
  draw(tick) {
    // Trail
    for(let i=0;i<this.trail.length;i++) {
      const alpha = i/this.trail.length;
      const sz = Math.max(2, Math.floor(8*alpha));
      const t = {x: this.trail[i].x, y: this.trail[i].y};
      ctx.beginPath();
      ctx.arc(t.x, t.y, sz, 0, Math.PI*2);
      ctx.fillStyle = `rgba(60,140,255,${alpha*0.4})`;
      ctx.fill();
    }
    if(this.invincible > 0 && Math.floor(this.invincible/4) % 2 === 0) return;

    const rx = Math.floor(this.x), ry = Math.floor(this.y);
    const cx = rx + PLAYER_W/2, cy = ry + PLAYER_H/2;

    // Glow aura
    const pulse = 0.4 + 0.3*Math.sin(tick*0.12);
    const glowR = Math.floor(30 + 10*pulse);
    const grad = ctx.createRadialGradient(cx, cy, 0, cx, cy, glowR);
    grad.addColorStop(0, `rgba(60,140,255,0.25)`);
    grad.addColorStop(1, `rgba(60,140,255,0)`);
    ctx.beginPath();
    ctx.arc(cx, cy, glowR, 0, Math.PI*2);
    ctx.fillStyle = grad;
    ctx.fill();

    // Body
    drawRoundedRect(rx+8, ry+16, 36, 32, 7, C.PANEL, C.ACCENT, 2);
    // Head
    ctx.beginPath(); ctx.arc(cx, ry+13, 13, 0, Math.PI*2);
    ctx.fillStyle = C.ACCENT2; ctx.fill();
    ctx.strokeStyle = C.BORDER; ctx.lineWidth = 2; ctx.stroke();
    // Eyes
    for(const ex of [cx-4, cx+4]) {
      ctx.beginPath(); ctx.arc(ex, ry+11, 3, 0, Math.PI*2);
      ctx.fillStyle = C.BG; ctx.fill();
      ctx.beginPath(); ctx.arc(ex, ry+11, 2, 0, Math.PI*2);
      ctx.fillStyle = C.WHITE; ctx.fill();
    }
    // Straps
    ctx.strokeStyle = C.ACCENT; ctx.lineWidth = 3;
    ctx.beginPath(); ctx.moveTo(rx+16, ry+20); ctx.lineTo(rx+16, ry+44); ctx.stroke();
    ctx.beginPath(); ctx.moveTo(rx+36, ry+20); ctx.lineTo(rx+36, ry+44); ctx.stroke();
  }
  getRect() { return {x: this.x+10, y: this.y+14, w:32, h:36}; }
}

function rectsCollide(a, b) {
  return a.x < b.x+b.w && a.x+a.w > b.x && a.y < b.y+b.h && a.y+a.h > b.y;
}

// HUD
function drawHUD(score, lives, level, targetIdx, tick) {
  drawRoundedRect(0, 0, W, 58, 0, C.PANEL);
  ctx.strokeStyle = C.BORDER; ctx.lineWidth = 2;
  ctx.beginPath(); ctx.moveTo(0,58); ctx.lineTo(W,58); ctx.stroke();

  // Score
  ctx.font = '13px Consolas, monospace'; ctx.textAlign = 'left';
  ctx.fillStyle = C.DIM; ctx.fillText('SKOR', 14, 18);
  ctx.font = 'bold 28px Consolas, monospace';
  ctx.fillStyle = C.GOLD; ctx.fillText(score, 14, 48);

  // Level
  ctx.font = '13px Consolas, monospace'; ctx.textAlign = 'left';
  ctx.fillStyle = C.DIM; ctx.fillText('LV', 200, 18);
  ctx.font = 'bold 28px Consolas, monospace';
  ctx.fillStyle = C.ACCENT2; ctx.fillText(level, 200, 48);

  // Hearts
  for(let i=0;i<3;i++) {
    const hx = W - 30 - i*36;
    ctx.font = 'bold 22px Consolas, monospace';
    ctx.textAlign = 'center';
    ctx.fillStyle = i < lives ? C.HEART : C.DIM;
    ctx.fillText('♥', hx, 42);
  }

  // Target banner
  const [tName, tColor] = MAPEL_LIST[targetIdx];
  const bannerW = 260;
  const bx = W/2 - bannerW/2;
  const by = H - 74;
  const pulse = 0.6 + 0.4*Math.sin(tick*0.08);
  const bgColor = lerpColor(C.GOOD_DIM, '#144628', pulse);
  drawRoundedRect(bx, by, bannerW, 42, 10, bgColor, C.GOOD, 2);

  ctx.font = '13px Consolas, monospace'; ctx.textAlign = 'center';
  ctx.fillStyle = C.DIM; ctx.fillText('▶ TANGKAP ◀', W/2, by+14);
  ctx.font = 'bold 18px Consolas, monospace';
  ctx.fillStyle = C.GOOD;
  const lines = tName.split(' ');
  if(lines.length === 1) {
    ctx.fillText(tName, W/2, by+34);
  } else {
    ctx.fillText(lines[0] + ' ' + lines[1], W/2, by+34);
  }

  ctx.strokeStyle = C.BORDER; ctx.lineWidth = 1;
  ctx.beginPath(); ctx.moveTo(0, H-80); ctx.lineTo(W, H-80); ctx.stroke();

  ctx.font = '13px Consolas, monospace'; ctx.textAlign = 'right';
  ctx.fillStyle = C.DIM; ctx.fillText('by Abbilefa', W-10, H-8);
}

// TITLE SCREEN
function drawTitleScreen(tick) {
  ctx.fillStyle = C.BG; ctx.fillRect(0,0,W,H);
  drawStars(tick);

  // Grid
  ctx.strokeStyle = '#0f193c'; ctx.lineWidth = 1;
  for(let i=0;i<W;i+=40) { ctx.beginPath(); ctx.moveTo(i,0); ctx.lineTo(i,H); ctx.stroke(); }
  for(let j=0;j<H;j+=40) { ctx.beginPath(); ctx.moveTo(0,j); ctx.lineTo(W,j); ctx.stroke(); }

  // Glow circle
  const pulse = 0.3 + 0.2*Math.sin(tick*0.05);
  const gr = ctx.createRadialGradient(W/2,170,0,W/2,170,150);
  gr.addColorStop(0, `rgba(30,80,180,${0.3*pulse})`);
  gr.addColorStop(1, 'rgba(30,80,180,0)');
  ctx.fillStyle = gr;
  ctx.fillRect(W/2-150, 20, 300, 300);

  // Title
  ctx.textAlign = 'center';
  ctx.font = 'bold 54px Impact, sans-serif';
  ctx.fillStyle = C.ACCENT2; ctx.fillText('MAPEL', W/2, 180);
  ctx.fillStyle = C.ACCENT; ctx.fillText('DODGE', W/2, 232);

  // Subtitle line
  ctx.strokeStyle = C.BORDER; ctx.lineWidth = 2;
  ctx.beginPath(); ctx.moveTo(80,255); ctx.lineTo(W-80,255); ctx.stroke();

  ctx.font = '16px Consolas, monospace';
  ctx.fillStyle = C.DIM; ctx.fillText('Hindari mapel yang salah!', W/2, 272);
  ctx.fillStyle = C.WHITE; ctx.fillText('Tangkap mapel yang ditunjuk!', W/2, 294);

  // Controls card
  drawRoundedRect(60, 320, W-120, 168, 14, C.PANEL, C.BORDER, 2);
  ctx.font = '16px Consolas, monospace';
  ctx.fillStyle = C.ACCENT; ctx.fillText('KONTROL', W/2, 345);

  const controls = [
    ['← / A', 'Gerak Kiri'],
    ['→ / D', 'Gerak Kanan'],
    ['SPACE', 'Pause / Lanjut'],
    ['ENTER', 'Mulai / Restart'],
  ];
  for(let i=0;i<controls.length;i++) {
    const y = 368 + i*26;
    ctx.textAlign = 'left';
    ctx.font = '15px Consolas, monospace';
    ctx.fillStyle = C.GOLD; ctx.fillText(controls[i][0], 84, y);
    ctx.fillStyle = C.DIM; ctx.fillText('—', 178, y);
    ctx.fillStyle = C.WHITE; ctx.fillText(controls[i][1], 198, y);
  }

  if(Math.floor(tick/30) % 2 === 0) {
    ctx.textAlign = 'center';
    ctx.font = 'bold 20px Consolas, monospace';
    ctx.fillStyle = C.ACCENT;
    ctx.fillText('[ ENTER untuk Mulai ]', W/2, 518);
  }

  ctx.font = '16px Consolas, monospace';
  ctx.fillStyle = C.DIM; ctx.fillText('by  Abbilefa', W/2, 548);

  // Preview blocks
  for(let i=0;i<4;i++) {
    const offset = [-160,-60,40,140][i];
    const bx = W/2 + offset;
    const by = 590 + Math.floor(10*Math.sin(tick*0.07 + i));
    const [mName, mColor] = MAPEL_LIST[i*3];
    const r=parseInt(mColor.slice(1,3),16), g=parseInt(mColor.slice(3,5),16), b=parseInt(mColor.slice(5,7),16);
    drawRoundedRect(bx-44, by-18, 88, 36, 7, `rgba(${r},${g},${b},0.2)`, mColor, 2);
    const mlines = mName.split(' ');
    ctx.font = '12px Consolas, monospace'; ctx.textAlign='center';
    ctx.fillStyle = mColor;
    if(mlines.length===1) { ctx.fillText(mlines[0], bx, by+5); }
    else { ctx.fillText(mlines[0], bx, by-4); ctx.fillText(mlines[1], bx, by+10); }
  }
}

// GAMEOVER SCREEN
function drawGameoverScreen(score, highscore, tick) {
  ctx.fillStyle = 'rgba(0,0,0,0.7)'; ctx.fillRect(0,0,W,H);
  drawRoundedRect(60, 160, W-120, 340, 18, C.PANEL, C.DANGER, 3);

  ctx.textAlign = 'center';
  ctx.font = 'bold 54px Impact, sans-serif';
  ctx.fillStyle = C.DANGER; ctx.fillText('GAME OVER', W/2, 230);
  ctx.strokeStyle = C.DANGER; ctx.lineWidth = 2;
  ctx.beginPath(); ctx.moveTo(100,260); ctx.lineTo(W-100,260); ctx.stroke();

  ctx.font = '16px Consolas, monospace';
  ctx.fillStyle = C.DIM; ctx.fillText('SKOR KAMU', W/2, 285);
  ctx.font = 'bold 54px Impact, sans-serif';
  ctx.fillStyle = C.GOLD; ctx.fillText(score, W/2, 335);

  ctx.font = '16px Consolas, monospace';
  ctx.fillStyle = C.ACCENT; ctx.fillText(`REKOR TERTINGGI : ${highscore}`, W/2, 368);

  if(score >= highscore && score > 0) {
    const pulse = 0.5 + 0.5*Math.sin(tick*0.15);
    ctx.fillStyle = lerpColor(C.GOLD, C.WHITE, pulse);
    ctx.font = 'bold 20px Consolas, monospace';
    ctx.fillText('★ REKOR BARU! ★', W/2, 398);
  }

  if(Math.floor(tick/30) % 2 === 0) {
    ctx.font = '16px Consolas, monospace';
    ctx.fillStyle = C.ACCENT;
    ctx.fillText('[ ENTER untuk Main Lagi ]', W/2, 444);
  }

  ctx.fillStyle = C.DIM; ctx.fillText('by Abbilefa', W/2, 482);
}

// PAUSE SCREEN
function drawPauseScreen() {
  ctx.fillStyle = 'rgba(0,0,0,0.6)'; ctx.fillRect(0,0,W,H);
  drawRoundedRect(100, 240, W-200, 180, 14, C.PANEL, C.ACCENT, 2);
  ctx.textAlign = 'center';
  ctx.font = 'bold 32px Consolas, monospace';
  ctx.fillStyle = C.ACCENT2; ctx.fillText('⏸  PAUSE', W/2, 308);
  ctx.font = '16px Consolas, monospace';
  ctx.fillStyle = C.DIM; ctx.fillText('Tekan SPACE untuk Lanjut', W/2, 365);
}

// INPUT
const keys = {};
window.addEventListener('keydown', e => {
  keys[e.key] = true;
  // Prevent page scroll
  if(['ArrowLeft','ArrowRight','ArrowUp','ArrowDown',' '].includes(e.key)) e.preventDefault();
});
window.addEventListener('keyup', e => { keys[e.key] = false; });

// Touch controls
let touchStartX = null;
canvas.addEventListener('touchstart', e => {
  touchStartX = e.touches[0].clientX;
  e.preventDefault();
}, {passive:false});
canvas.addEventListener('touchmove', e => {
  if(touchStartX === null) return;
  const dx = e.touches[0].clientX - touchStartX;
  if(Math.abs(dx) > 10) {
    keys['ArrowLeft'] = dx < 0;
    keys['ArrowRight'] = dx > 0;
  }
  e.preventDefault();
}, {passive:false});
canvas.addEventListener('touchend', e => {
  keys['ArrowLeft'] = false; keys['ArrowRight'] = false;
  touchStartX = null;
  e.preventDefault();
}, {passive:false});

// MAIN GAME
let state = 'title';
let score = 0, highscore = 0, lives = 3, level = 1, tick = 0;
let paused = false;
let player = new Player();
let blocks = [];
let targetIdx = Math.floor(Math.random()*MAPEL_LIST.length);
let spawnTimer = 0;
const BASE_SPEED = 2.8;
let blockSpeed = BASE_SPEED;
let spawnInterval = 90;
let blocksPerSpawn = 1;

function resetGame() {
  score = 0; lives = 3; level = 1;
  blocks = []; particles = []; floatTexts = [];
  targetIdx = Math.floor(Math.random()*MAPEL_LIST.length);
  blockSpeed = BASE_SPEED; spawnInterval = 90; blocksPerSpawn = 1; spawnTimer = 0;
  player.x = W/2 - PLAYER_W/2; player.invincible = 0; player.trail = [];
}

// Key press handler (one-shot)
const justPressed = {};
window.addEventListener('keydown', e => {
  if(!justPressed[e.key]) {
    justPressed[e.key] = true;
    handleKeyPress(e.key);
  }
});
window.addEventListener('keyup', e => { justPressed[e.key] = false; });

function handleKeyPress(key) {
  if(state === 'title' && key === 'Enter') { resetGame(); state = 'game'; }
  else if(state === 'game' && key === ' ') { paused = !paused; }
  else if(state === 'game' && key === 'Enter' && paused) { paused = false; }
  else if(state === 'gameover' && key === 'Enter') { resetGame(); state = 'game'; }
}

function gameLoop() {
  tick++;

  // UPDATE
  if(state === 'game' && !paused) {
    player.update(keys);

    // Spawn
    spawnTimer++;
    if(spawnTimer >= spawnInterval) {
      spawnTimer = 0;
      const usedX = [];
      for(let i=0;i<blocksPerSpawn;i++) {
        const isTarget = Math.random() < 0.35;
        let midx = Math.floor(Math.random()*MAPEL_LIST.length);
        if(isTarget) midx = targetIdx;
        else { while(midx === targetIdx) midx = Math.floor(Math.random()*MAPEL_LIST.length); }
        const b = new Block(blockSpeed, isTarget, midx);
        let att = 0;
        while(usedX.some(ux => Math.abs(b.x - ux) < BLOCK_W+10) && att < 10) {
          b.x = 20 + Math.random()*(W-20-BLOCK_W-20); att++;
        }
        usedX.push(b.x);
        blocks.push(b);
      }
    }

    // Collision
    const pRect = player.getRect();
    for(const b of blocks) {
      b.update(tick);
      if(!b.alive) continue;
      if(rectsCollide(b.getRect(), pRect)) {
        if(b.isTarget) {
          score += 10;
          b.alive = false;
          const cx = b.x + b.w/2, cy = b.y + b.h/2;
          spawnParticles(cx, cy, C.GOOD, 15);
          spawnFloatText(cx, cy-20, '+10', C.GOOD);
          let old = targetIdx;
          while(targetIdx === old) targetIdx = Math.floor(Math.random()*MAPEL_LIST.length);
        } else if(player.invincible === 0) {
          lives--; b.alive = false; player.invincible = 90;
          const cx = b.x + b.w/2, cy = b.y + b.h/2;
          spawnParticles(cx, cy, C.DANGER, 10);
          spawnFloatText(cx, cy-20, '-NYAWA', C.DANGER);
          if(lives <= 0) { highscore = Math.max(highscore, score); state = 'gameover'; }
        }
      }
    }
    blocks = blocks.filter(b => b.alive);

    // Level
    const newLevel = 1 + Math.floor(score/50);
    if(newLevel !== level) {
      level = newLevel;
      blockSpeed = BASE_SPEED + (level-1)*0.4;
      spawnInterval = Math.max(35, 90-(level-1)*8);
      blocksPerSpawn = Math.min(4, 1 + Math.floor((level-1)/2));
    }
  }

  // DRAW
  ctx.fillStyle = C.BG; ctx.fillRect(0,0,W,H);
  drawStars(tick);

  // Grid
  ctx.strokeStyle = '#0c1430'; ctx.lineWidth = 1;
  for(let i=0;i<W;i+=40) { ctx.beginPath(); ctx.moveTo(i,0); ctx.lineTo(i,H); ctx.stroke(); }
  for(let j=0;j<H;j+=40) { ctx.beginPath(); ctx.moveTo(0,j); ctx.lineTo(W,j); ctx.stroke(); }

  if(state === 'title') {
    drawTitleScreen(tick);
  } else if(state === 'game') {
    for(const b of blocks) b.draw(tick);
    updateDrawParticles();
    player.draw(tick);
    updateDrawFloatTexts();
    drawHUD(score, lives, level, targetIdx, tick);
    if(paused) drawPauseScreen();
  } else if(state === 'gameover') {
    for(const b of blocks) b.draw(tick);
    player.draw(tick);
    drawHUD(score, lives, level, targetIdx, tick);
    drawGameoverScreen(score, highscore, tick);
  }

  requestAnimationFrame(gameLoop);
}

gameLoop();
</script>
</body>
</html>
