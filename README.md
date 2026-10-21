[index (3).html](https://github.com/user-attachments/files/33777825/index.3.html)
<!doctype html>
<html lang="et">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Põgenemistuba VR</title>
<style>
  html,body{margin:0;height:100%;overflow:hidden;background:#000;font-family:system-ui,sans-serif}
  #info{position:fixed;top:10px;left:10px;color:#fff;background:rgba(0,0,0,.65);padding:10px 14px;border-radius:10px;font-size:14px;line-height:1.5;max-width:420px;z-index:5}
  #info b{color:#ffd54f}
</style>
<script type="importmap">
{ "imports": {
  "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
  "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/"
} }
</script>
</head>
<body>
<div id="info">
  <b>🔐 Põgenemistuba VR – Professor Nupu töötuba</b><br>
  Questis: vajuta all <b>ENTER VR</b>.<br>
  Arvutis: <b>klõps</b> = kasuta / võta ese, <b>parem klõps</b> = pane ese maha,
  <b>lohista</b> = vaata ringi, <b>WASD</b> = liigu, <b>Esc</b> = paus.
</div>
<script type="module">
import * as THREE from 'three';
import { VRButton } from 'three/addons/webxr/VRButton.js';
import { mergeGeometries } from 'three/addons/utils/BufferGeometryUtils.js';

/* =========================================================
   PÕHI
   ========================================================= */
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.setSize(innerWidth, innerHeight);
renderer.xr.enabled = true;
renderer.xr.setReferenceSpaceType('local-floor');
renderer.xr.setFoveation(1);
document.body.appendChild(renderer.domElement);
document.body.appendChild(VRButton.createButton(renderer));

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x0b0b0b);
const camera = new THREE.PerspectiveCamera(70, innerWidth / innerHeight, 0.02, 60);
const rig = new THREE.Group();
scene.add(rig); rig.add(camera);
camera.position.set(0, 1.6, 0);

const hemi = new THREE.HemisphereLight(0xfff0dd, 0x45352a, 1.1); scene.add(hemi);
const moon = new THREE.DirectionalLight(0x9fb4ff, 0.7); moon.position.set(6, 3.5, 0.2); scene.add(moon);
const deskLight = new THREE.PointLight(0xffc27a, 0.6, 0, 2); deskLight.position.set(2.62, 1.06, -0.32); scene.add(deskLight);
const ceilLight = new THREE.PointLight(0xfff1dc, 0, 0, 2); ceilLight.position.set(0, 2.42, 0); scene.add(ceilLight);
const outLight = new THREE.PointLight(0xfffbe8, 0, 0, 2); outLight.position.set(0, 1.6, -3.6); scene.add(outLight);

const UP = new THREE.Vector3(0, 1, 0), DOWN = new THREE.Vector3(0, -1, 0), NEG_Z = new THREE.Vector3(0, 0, -1);
const head = new THREE.Vector3(), fwd = new THREE.Vector3();
const tmpV = new THREE.Vector3(), tmpV2 = new THREE.Vector3(), tmpD = new THREE.Vector3(), tmpQ = new THREE.Quaternion();

function mulberry32(a){return function(){a|=0;a=a+0x6D2B79F5|0;let t=Math.imul(a^a>>>15,1|a);t=t+Math.imul(t^t>>>7,61|t)^t;return((t^t>>>14)>>>0)/4294967296;};}
const tr = mulberry32(777);
const rand = Math.random;
const randInt = (a, b) => a + Math.floor(rand() * (b - a + 1));
const pick = a => a[Math.floor(rand() * a.length)];
const shuffle = a => { for (let i = a.length - 1; i > 0; i--) { const j = Math.floor(rand() * (i + 1)); [a[i], a[j]] = [a[j], a[i]]; } return a; };
const clamp = THREE.MathUtils.clamp;
const ease = k => k < 0.5 ? 2 * k * k : 1 - Math.pow(-2 * k + 2, 2) / 2;
const fmtTime = s => { s = Math.floor(s); return `${Math.floor(s / 60)}:${String(s % 60).padStart(2, '0')}`; };
const removeFrom = (a, x) => { const i = a.indexOf(x); if (i >= 0) a.splice(i, 1); };
const FONT = 'system-ui, "Segoe UI", Roboto, Arial, "Noto Color Emoji", "Apple Color Emoji", "Segoe UI Emoji", sans-serif';

/* ---------- heli ---------- */
let actx = null;
function audio(){ if (!actx) actx = new (window.AudioContext || window.webkitAudioContext)(); if (actx.state === 'suspended') actx.resume(); return actx; }
function tone(freq, dur = .15, delayS = 0, type = 'sine', vol = .25){
  const a = audio(), t = a.currentTime + delayS, o = a.createOscillator(), g = a.createGain();
  o.type = type; o.frequency.value = freq;
  g.gain.setValueAtTime(0.0001, t); g.gain.exponentialRampToValueAtTime(vol, t + 0.01); g.gain.exponentialRampToValueAtTime(0.0001, t + dur);
  o.connect(g).connect(a.destination); o.start(t); o.stop(t + dur + 0.03);
}
function sweep(f0, f1, dur, type = 'sine', vol = 0.2, delayS = 0){
  const a = audio(), t = a.currentTime + delayS, o = a.createOscillator(), g = a.createGain();
  o.type = type; o.frequency.setValueAtTime(f0, t); o.frequency.exponentialRampToValueAtTime(f1, t + dur);
  g.gain.setValueAtTime(0.0001, t); g.gain.exponentialRampToValueAtTime(vol, t + 0.02); g.gain.exponentialRampToValueAtTime(0.0001, t + dur);
  o.connect(g).connect(a.destination); o.start(t); o.stop(t + dur + 0.05);
}
function noise(dur = 0.3, vol = 0.2, freq = 2000){
  const a = audio(), n = Math.floor(a.sampleRate * dur), b = a.createBuffer(1, n, a.sampleRate), d = b.getChannelData(0);
  for (let i = 0; i < n; i++) d[i] = (Math.random() * 2 - 1) * (1 - i / n);
  const s = a.createBufferSource(); s.buffer = b;
  const f = a.createBiquadFilter(); f.type = 'bandpass'; f.frequency.value = freq;
  const g = a.createGain(); g.gain.value = vol;
  s.connect(f).connect(g).connect(a.destination); s.start();
}
const sfx = {
  click(){ tone(520, .05, 0, 'triangle', .15); },
  ok(){ tone(660, .12); tone(880, .2, .11); },
  bad(){ tone(170, .25, 0, 'square', .08); },
  pick(){ tone(640, .06, 0, 'triangle', .14); tone(900, .05, .05, 'triangle', .1); },
  drop(){ tone(220, .09, 0, 'triangle', .16); },
  beep(){ tone(1250, .06, 0, 'square', .06); },
  tick(){ tone(1900, .03, 0, 'square', .06); tone(1500, .03, .25, 'square', .06); },
  squeak(){ sweep(900, 1600, .12, 'sine', .18); sweep(1500, 1000, .12, 'sine', .15, .12); },
  unlock(){ tone(420, .05, 0, 'square', .1); tone(640, .05, .07, 'square', .1); tone(880, .25, .16, 'sine', .2); },
  creak(){ sweep(95, 60, 1.5, 'sawtooth', .05); sweep(150, 110, 1.2, 'sawtooth', .03, .3); },
  spark(){ noise(.45, .35, 3200); tone(110, .7, .05, 'sawtooth', .05); },
  rotate(){ sweep(320, 220, .3, 'triangle', .1); },
  bell(){ tone(2093, .9, 0, 'sine', .18); tone(3140, .6, 0, 'sine', .08); },
  reveal(){ sweep(400, 1300, .6, 'sine', .14); tone(1568, .4, .5, 'sine', .1); },
  win(){ [523, 659, 784, 1046, 1318].forEach((f, i) => tone(f, .25, i * .13)); }
};
addEventListener('pointerdown', () => audio(), { once: true });

/* ---------- lõuendid ja paneelid ---------- */
function roundRect(c, x, y, w, h, r){ c.beginPath(); c.moveTo(x + r, y); c.arcTo(x + w, y, x + w, y + h, r); c.arcTo(x + w, y + h, x, y + h, r); c.arcTo(x, y + h, x, y, r); c.arcTo(x, y, x + w, y, r); c.closePath(); }
function splitLines(c, text, maxW){
  const lines = [];
  for (const para of String(text).split('\n')) {
    let line = '';
    for (const w of para.split(' ')) { const t = line ? line + ' ' + w : w; if (line && c.measureText(t).width > maxW) { lines.push(line); line = w; } else line = t; }
    lines.push(line);
  }
  return lines;
}
function wrapText(c, text, x, y, maxW, lh){
  const lines = splitLines(c, text, maxW), y0 = y - (lines.length - 1) * lh / 2;
  lines.forEach((l, i) => c.fillText(l, x, y0 + i * lh));
}
// kirjutab teksti etteantud kasti, vähendades fonti kuni mahub
function fitText(c, text, x, yTop, maxW, maxH, fs0, bold = false){
  let fs = fs0, lines;
  for (; fs > 10; fs -= 1) { c.font = `${bold ? 'bold ' : ''}${fs}px ${FONT}`; lines = splitLines(c, text, maxW); if (lines.length * fs * 1.32 <= maxH) break; }
  wrapText(c, text, x, yTop + maxH / 2, maxW, fs * 1.32);
}
function panelBg(c, W, H, fill = 'rgba(15,28,38,0.94)', r = 36){ roundRect(c, 4, 4, W - 8, H - 8, r); c.fillStyle = fill; c.fill(); c.lineWidth = 6; c.strokeStyle = 'rgba(255,255,255,0.3)'; c.stroke(); }
class CanvasPlane {
  constructor(w, h, ppm = 400, ui = false){
    this.canvas = document.createElement('canvas');
    this.canvas.width = Math.round(w * ppm); this.canvas.height = Math.round(h * ppm);
    this.ctx = this.canvas.getContext('2d');
    this.tex = new THREE.CanvasTexture(this.canvas); this.tex.colorSpace = THREE.SRGBColorSpace; this.tex.anisotropy = 4;
    this.mesh = new THREE.Mesh(new THREE.PlaneGeometry(w, h), new THREE.MeshBasicMaterial({ map: this.tex, transparent: true, depthTest: !ui, depthWrite: !ui }));
    if (ui) this.mesh.renderOrder = 900;
  }
  draw(fn){ const c = this.ctx; c.clearRect(0, 0, this.canvas.width, this.canvas.height); fn(c, this.canvas.width, this.canvas.height); this.tex.needsUpdate = true; return this; }
}
function canvasTex(w, h, draw, repeat = true){
  const c = document.createElement('canvas'); c.width = w; c.height = h; draw(c.getContext('2d'), w, h);
  const t = new THREE.CanvasTexture(c); t.colorSpace = THREE.SRGBColorSpace; t.anisotropy = 8;
  if (repeat) t.wrapS = t.wrapT = THREE.RepeatWrapping;
  return t;
}
function litPlane(w, h, ppm, draw){ return new THREE.Mesh(new THREE.PlaneGeometry(w, h), new THREE.MeshLambertMaterial({ map: canvasTex(Math.round(w * ppm), Math.round(h * ppm), draw, false) })); }
function speckle(c, W, H, n, a){ for (let i = 0; i < n; i++) { c.fillStyle = tr() < 0.5 ? `rgba(0,0,0,${a * tr()})` : `rgba(255,255,255,${a * tr()})`; c.fillRect(tr() * W, tr() * H, 2, 2); } }
function grain(c, x, y, w, h, n, a){
  c.save(); c.beginPath(); c.rect(x, y, w, h); c.clip();
  for (let g = 0; g < n; g++) { c.strokeStyle = `rgba(40,20,6,${a * (0.5 + tr())})`; c.lineWidth = 0.8 + tr() * 1.5; const yy = y + tr() * h; c.beginPath(); c.moveTo(x, yy); c.bezierCurveTo(x + w * 0.3, yy + (tr() - 0.5) * 8, x + w * 0.7, yy + (tr() - 0.5) * 8, x + w, yy + (tr() - 0.5) * 6); c.stroke(); }
  c.restore();
}

/* ---------- tekstuurid ---------- */
const floorTex = canvasTex(512, 512, (c, W, H) => {
  const rows = 6, rh = H / rows;
  for (let r = 0; r < rows; r++) {
    let x = -tr() * W * 0.8;
    while (x < W) {
      const len = W * (0.35 + tr() * 0.5), b = 120 + tr() * 45;
      c.fillStyle = `rgb(${b | 0},${b * 0.66 | 0},${b * 0.4 | 0})`; c.fillRect(x, r * rh, len, rh);
      grain(c, x, r * rh, len, rh, 10, 0.12);
      c.fillStyle = 'rgba(25,12,3,0.6)'; c.fillRect(x, r * rh, 2, rh);
      x += len;
    }
    c.fillStyle = 'rgba(25,12,3,0.7)'; c.fillRect(0, r * rh, W, 2);
  }
});
const wallTex = canvasTex(256, 256, (c, W, H) => {
  c.fillStyle = '#4c7262'; c.fillRect(0, 0, W, H);
  speckle(c, W, H, 1500, 0.08);
  c.strokeStyle = 'rgba(235,215,160,0.22)'; c.lineWidth = 2;
  for (let k = -2; k <= 2; k++) { c.beginPath(); c.moveTo(k * W / 2, 0); c.lineTo(k * W / 2 + W, H); c.stroke(); c.beginPath(); c.moveTo(k * W / 2 + W, 0); c.lineTo(k * W / 2, H); c.stroke(); }
  c.fillStyle = 'rgba(235,215,160,0.3)';
  for (const [x, y] of [[0, 0], [W, 0], [0, H], [W, H], [W / 2, H / 2], [W / 2, 0], [W / 2, H], [0, H / 2], [W, H / 2]]) {
    for (let a = 0; a < 4; a++) { c.beginPath(); c.arc(x + Math.cos(a * Math.PI / 2) * 9, y + Math.sin(a * Math.PI / 2) * 9, 5, 0, 7); c.fill(); }
    c.beginPath(); c.arc(x, y, 4, 0, 7); c.fill();
  }
});
const wainsTex = canvasTex(256, 384, (c, W, H) => {
  c.fillStyle = '#6b4a2e'; c.fillRect(0, 0, W, H); grain(c, 0, 0, W, H, 30, 0.1);
  const m = 26;
  c.fillStyle = 'rgba(0,0,0,0.12)'; c.fillRect(m, m, W - 2 * m, H - 2 * m);
  c.strokeStyle = 'rgba(255,230,190,0.25)'; c.lineWidth = 4; c.beginPath(); c.moveTo(m, H - m); c.lineTo(m, m); c.lineTo(W - m, m); c.stroke();
  c.strokeStyle = 'rgba(0,0,0,0.45)'; c.beginPath(); c.moveTo(W - m, m); c.lineTo(W - m, H - m); c.lineTo(m, H - m); c.stroke();
});
const woodTex = canvasTex(256, 256, (c, W, H) => { c.fillStyle = '#7a5233'; c.fillRect(0, 0, W, H); grain(c, 0, 0, W, H, 40, 0.14); speckle(c, W, H, 400, 0.06); });
const fabricTex = canvasTex(128, 128, (c, W, H) => { c.fillStyle = '#8e2b2b'; c.fillRect(0, 0, W, H); for (let y = 0; y < H; y += 3) { c.fillStyle = 'rgba(0,0,0,0.08)'; c.fillRect(0, y, W, 1); } speckle(c, W, H, 600, 0.12); });
const rugTex = canvasTex(512, 352, (c, W, H) => {
  c.fillStyle = '#7a1f1f'; c.fillRect(0, 0, W, H);
  c.strokeStyle = '#d9b26a'; c.lineWidth = 10; c.strokeRect(14, 14, W - 28, H - 28);
  c.strokeStyle = '#1f3a5a'; c.lineWidth = 14; c.strokeRect(36, 36, W - 72, H - 72);
  c.fillStyle = '#1f3a5a'; c.beginPath(); c.moveTo(W / 2, 62); c.lineTo(W - 120, H / 2); c.lineTo(W / 2, H - 62); c.lineTo(120, H / 2); c.closePath(); c.fill();
  c.fillStyle = '#d9b26a'; c.beginPath(); c.moveTo(W / 2, 100); c.lineTo(W - 180, H / 2); c.lineTo(W / 2, H - 100); c.lineTo(180, H / 2); c.closePath(); c.fill();
  c.fillStyle = '#7a1f1f'; c.beginPath(); c.arc(W / 2, H / 2, 38, 0, 7); c.fill();
  speckle(c, W, H, 3000, 0.12);
}, false);
const paperTex = canvasTex(200, 270, (c, W, H) => {
  c.fillStyle = '#f2e6c8'; c.fillRect(0, 0, W, H);
  c.strokeStyle = 'rgba(60,40,20,0.55)'; c.lineWidth = 2;
  for (let y = 40; y < H - 20; y += 18) { c.beginPath(); c.moveTo(18, y); for (let x = 18; x < W - 18 - tr() * 40; x += 6) c.lineTo(x, y + (tr() - 0.5) * 3); c.stroke(); }
}, false);
const doorTex = canvasTex(256, 540, (c, W, H) => {
  c.fillStyle = '#596067'; c.fillRect(0, 0, W, H); speckle(c, W, H, 2500, 0.1);
  for (const [x, y, w, h] of [[28, 30, 200, 210], [28, 290, 200, 220]]) { c.fillStyle = 'rgba(0,0,0,0.18)'; c.fillRect(x, y, w, h); c.strokeStyle = 'rgba(255,255,255,0.18)'; c.lineWidth = 3; c.strokeRect(x, y, w, h); }
  c.fillStyle = '#3b4045'; for (let y = 12; y < H; y += 44) for (const x of [10, W - 10]) { c.beginPath(); c.arc(x, y, 4, 0, 7); c.fill(); }
  c.fillStyle = '#e8c547'; c.font = `bold 30px ${FONT}`; c.textAlign = 'center'; c.fillText('⚠ VÄLJAPÄÄS', W / 2, 270);
}, false);
const globeTex = canvasTex(1024, 512, (c, W, H) => {
  c.fillStyle = '#2f6fa8'; c.fillRect(0, 0, W, H);
  const gr = mulberry32(42);
  for (let k = 0; k < 9; k++) {
    const cx = gr() * W, cy = H * (0.2 + gr() * 0.6), n = 30 + gr() * 40;
    c.fillStyle = gr() < 0.7 ? '#6f9a4a' : '#c2a46a';
    for (let i = 0; i < n; i++) { const a = gr() * 7, r = gr() * 90; c.beginPath(); c.arc((cx + Math.cos(a) * r * 1.4 + W) % W, cy + Math.sin(a) * r * 0.7, 10 + gr() * 22, 0, 7); c.fill(); }
  }
  c.strokeStyle = 'rgba(255,255,255,0.3)'; c.lineWidth = 2;
  for (let i = 1; i < 6; i++) { c.beginPath(); c.moveTo(0, i * H / 6); c.lineTo(W, i * H / 6); c.stroke(); }
  for (let i = 0; i < 12; i++) { c.beginPath(); c.moveTo(i * W / 12, 0); c.lineTo(i * W / 12, H); c.stroke(); }
  const sx = W * 0.25, sy = H * 0.36, R1 = 30, R2 = 12;
  c.fillStyle = '#e01b1b'; c.strokeStyle = '#fff'; c.lineWidth = 4; c.beginPath();
  for (let i = 0; i < 10; i++) { const a = -Math.PI / 2 + i * Math.PI / 5, r = i % 2 ? R2 : R1; c.lineTo(sx + Math.cos(a) * r, sy + Math.sin(a) * r); }
  c.closePath(); c.fill(); c.stroke();
});
const shadowTex = canvasTex(128, 128, (c, W, H) => { const g = c.createRadialGradient(W / 2, H / 2, 0, W / 2, H / 2, W / 2); g.addColorStop(0, 'rgba(0,0,0,0.55)'); g.addColorStop(1, 'rgba(0,0,0,0)'); c.fillStyle = g; c.fillRect(0, 0, W, H); }, false);

const M = {
  floor: new THREE.MeshLambertMaterial({ map: floorTex }),
  wall: new THREE.MeshLambertMaterial({ map: wallTex }),
  wains: new THREE.MeshLambertMaterial({ map: wainsTex }),
  trim: new THREE.MeshLambertMaterial({ color: 0x4a311c }),
  ceil: new THREE.MeshLambertMaterial({ color: 0xd9d2c3 }),
  wood: new THREE.MeshLambertMaterial({ map: woodTex }),
  woodDark: new THREE.MeshLambertMaterial({ map: woodTex, color: 0x9a7a60 }),
  inner: new THREE.MeshLambertMaterial({ color: 0x3a2618 }),
  metal: new THREE.MeshLambertMaterial({ color: 0x8a9298 }),
  darkMetal: new THREE.MeshLambertMaterial({ color: 0x2d3236 }),
  brass: new THREE.MeshLambertMaterial({ color: 0xc9a13b, emissive: 0x1a1200 }),
  black: new THREE.MeshLambertMaterial({ color: 0x111111 }),
  paper: new THREE.MeshLambertMaterial({ map: paperTex, side: THREE.DoubleSide }),
  fabric: new THREE.MeshLambertMaterial({ map: fabricTex }),
  leaf: new THREE.MeshLambertMaterial({ color: 0x3f7f35, flatShading: true }),
  pot: new THREE.MeshLambertMaterial({ color: 0xb5643c }),
  shade: new THREE.MeshLambertMaterial({ color: 0x2f5a3a, side: THREE.DoubleSide }),
  duck: new THREE.MeshLambertMaterial({ color: 0xffd23a }),
  beak: new THREE.MeshLambertMaterial({ color: 0xff8a1c }),
  shadow: new THREE.MeshBasicMaterial({ map: shadowTex, transparent: true, depthWrite: false })
};

/* ---------- geomeetria abid ---------- */
function boxUV(w, h, d, tx = 1, ty = tx){
  const g = new THREE.BoxGeometry(w, h, d), uv = g.attributes.uv;
  const dims = [[d, h], [d, h], [w, d], [w, d], [w, h], [w, h]];
  for (let f = 0; f < 6; f++) for (let k = 0; k < 4; k++) { const i = f * 4 + k; uv.setXY(i, uv.getX(i) * dims[f][0] / tx, uv.getY(i) * dims[f][1] / ty); }
  return g;
}
function planeUV(w, h, tile = 1){ const g = new THREE.PlaneGeometry(w, h), uv = g.attributes.uv; for (let i = 0; i < uv.count; i++) uv.setXY(i, uv.getX(i) * w / tile, uv.getY(i) * h / tile); return g; }

/* =========================================================
   INTERAKTSIOONI REGISTER
   ========================================================= */
const statics = [];        // seinad, mööbel – peatavad kiire, neile saab asju panna
const interactives = [];   // klõpsatavad / võetavad asjad
const room = new THREE.Group(); scene.add(room);
function S(geo, mat, x = 0, y = 0, z = 0, parent = room){ const m = new THREE.Mesh(geo, mat); m.position.set(x, y, z); parent.add(m); statics.push(m); return m; }
function D(geo, mat, x = 0, y = 0, z = 0, parent = room){ const m = new THREE.Mesh(geo, mat); m.position.set(x, y, z); parent.add(m); return m; }
function makeIA(root, ia){
  root.userData.ia = ia; interactives.push(root);
  if (!ia.ui && !ia.scaleHover) root.traverse(m => { if (m.isMesh && m.material && m.material.emissive) { m.material = m.material.clone(); m.userData.baseEm = m.material.emissive.clone(); } });
  root.userData.bs = root.scale.clone();
  return root;
}
function shown(o){ let last = null; while (o) { if (!o.visible) return false; last = o; o = o.parent; } return last === scene; }

/* ---------- animatsioonid ---------- */
const tweens = [];
function tween(dur, fn, done){ const tw = { t: 0, dur, fn, done }; tweens.push(tw); return tw; }
const delay = (s, fn) => tween(s, () => {}, fn);
function updateTweens(dt){
  for (let i = 0; i < tweens.length; i++) {
    const tw = tweens[i]; tw.t += dt; const k = Math.min(1, tw.t / tw.dur); tw.fn(k);
    if (k >= 1) { tweens.splice(i, 1); i--; if (tw.done) tw.done(); }
  }
}

/* =========================================================
   TUBA (6 × 6 m, kõrgus 3 m)
   ========================================================= */
S(planeUV(6, 6, 1.5).rotateX(-Math.PI / 2), M.floor);
D(new THREE.PlaneGeometry(6, 6).rotateX(Math.PI / 2), M.ceil, 0, 3, 0);
S(planeUV(6, 3).rotateY(Math.PI), M.wall, 0, 1.5, 3);            // lõuna
S(planeUV(6, 3).rotateY(-Math.PI / 2), M.wall, 3, 1.5, 0);       // ida
S(planeUV(6, 3).rotateY(Math.PI / 2), M.wall, -3, 1.5, 0);       // lääs
S(planeUV(2.45, 3), M.wall, -1.775, 1.5, -3);                     // põhi (uks keskel)
S(planeUV(2.45, 3), M.wall, 1.775, 1.5, -3);
S(planeUV(1.1, 0.85), M.wall, 0, 2.575, -3);
// puitpaneelid seinte alaosas
S(boxUV(6, 0.9, 0.02, 0.6, 0.9), M.wains, 0, 0.45, 2.99);
S(boxUV(0.02, 0.9, 6, 0.6, 0.9), M.wains, 2.99, 0.45, 0);
S(boxUV(0.02, 0.9, 6, 0.6, 0.9), M.wains, -2.99, 0.45, 0);
S(boxUV(2.45, 0.9, 0.02, 0.6, 0.9), M.wains, -1.775, 0.45, -2.99);
S(boxUV(2.45, 0.9, 0.02, 0.6, 0.9), M.wains, 1.775, 0.45, -2.99);
for (const [w, d, x, z] of [[6, 0.05, 0, 2.975], [0.05, 6, 2.975, 0], [0.05, 6, -2.975, 0], [2.45, 0.05, -1.775, -2.975], [2.45, 0.05, 1.775, -2.975]]) {
  D(new THREE.BoxGeometry(w, 0.04, d), M.trim, x, 0.92, z);
  D(new THREE.BoxGeometry(w, 0.08, d), M.trim, x, 2.96, z);
}
// ukseraam ja uks
D(new THREE.BoxGeometry(0.05, 2.15, 0.12), M.trim, -0.525, 1.075, -3);
D(new THREE.BoxGeometry(0.05, 2.15, 0.12), M.trim, 0.525, 1.075, -3);
D(new THREE.BoxGeometry(1.1, 0.05, 0.12), M.trim, 0, 2.125, -3);
const doorPivot = new THREE.Group(); doorPivot.position.set(-0.5, 0, -3.0); room.add(doorPivot);
S(new THREE.BoxGeometry(1.0, 2.1, 0.05), new THREE.MeshLambertMaterial({ map: doorTex }), 0.5, 1.05, 0, doorPivot);
D(new THREE.BoxGeometry(0.12, 0.025, 0.04), M.brass, 0.86, 1.0, 0.045, doorPivot);
D(new THREE.CylinderGeometry(0.03, 0.03, 0.01, 16).rotateX(Math.PI / 2), M.brass, 0.9, 1.0, 0.028, doorPivot);
makeIA(doorPivot, { onClick: () => { if (!st.escaped) { sfx.bad(); notify('Uks on lukus 🔒 Sisesta kood klahvistikule.'); } } });
// ukse taga: hele väljapääs
const outside = new THREE.Mesh(new THREE.BoxGeometry(1.3, 2.3, 1.2), new THREE.MeshBasicMaterial({ color: 0xfff6dc, side: THREE.BackSide }));
outside.position.set(0, 1.15, -3.65); room.add(outside);

// aken (ida sein) – öine taevas
{
  const sky = canvasTex(256, 232, (c, W, H) => {
    const g = c.createLinearGradient(0, 0, 0, H); g.addColorStop(0, '#0a1330'); g.addColorStop(1, '#2b4470'); c.fillStyle = g; c.fillRect(0, 0, W, H);
    for (let i = 0; i < 70; i++) { c.fillStyle = `rgba(255,255,255,${0.3 + tr() * 0.7})`; c.fillRect(tr() * W, tr() * H * 0.75, 1.5, 1.5); }
    c.shadowColor = '#fff8d0'; c.shadowBlur = 30; c.fillStyle = '#fff6d5'; c.beginPath(); c.arc(W * 0.7, H * 0.28, H * 0.09, 0, 7); c.fill(); c.shadowBlur = 0;
    c.fillStyle = '#0b1810';
    for (let x = -10; x < W + 20; x += 18 + tr() * 14) { const h = 40 + tr() * 50; c.beginPath(); c.moveTo(x - 14, H); c.lineTo(x, H - h); c.lineTo(x + 14, H); c.fill(); }
  }, false);
  D(new THREE.PlaneGeometry(1.0, 0.9).rotateY(-Math.PI / 2), new THREE.MeshBasicMaterial({ map: sky }), 2.985, 1.75, 0.2);
  for (const y of [1.27, 2.23]) D(new THREE.BoxGeometry(0.08, 0.06, 1.12), M.trim, 2.96, y, 0.2);
  for (const z of [-0.33, 0.73]) D(new THREE.BoxGeometry(0.08, 1.02, 0.06), M.trim, 2.96, 1.75, z);
  D(new THREE.BoxGeometry(0.04, 0.9, 0.03), M.trim, 2.97, 1.75, 0.2);
  D(new THREE.BoxGeometry(0.04, 0.03, 1.0), M.trim, 2.97, 1.75, 0.2);
  S(new THREE.BoxGeometry(0.16, 0.04, 1.2), M.trim, 2.9, 1.28, 0.2);
  for (const z of [-0.52, 0.92]) D(new THREE.BoxGeometry(0.05, 1.45, 0.3), M.fabric, 2.93, 1.75, z);
}
// laelamp
D(new THREE.CylinderGeometry(0.008, 0.008, 0.36, 6), M.black, 0, 2.82, 0);
D(new THREE.ConeGeometry(0.22, 0.18, 24, 1, true), M.shade, 0, 2.58, 0);
const bulbMat = new THREE.MeshLambertMaterial({ color: 0xfff6e0, emissive: 0x222018 });
D(new THREE.SphereGeometry(0.05, 12, 8), bulbMat, 0, 2.52, 0);
// vaip
S(new THREE.PlaneGeometry(3.2, 2.2).rotateX(-Math.PI / 2), new THREE.MeshLambertMaterial({ map: rugTex }), 0, 0.004, 0.5);
// varjud mööbli all
for (const [x, z, w, d] of [[2.55, 0.2, 1.3, 2.0], [2.1, 2.5, 1.3, 0.9], [-2.2, 2.2, 0.8, 0.8], [-2.15, -2.15, 1.2, 1.2], [2.55, -2.55, 0.8, 0.8], [-2.8, -0.5, 0.7, 2.3]]) {
  const m = D(new THREE.PlaneGeometry(w, d).rotateX(-Math.PI / 2), M.shadow, x, 0.007, z); m.renderOrder = 1;
}

/* ---------- maal (lõuna sein) ---------- */
{
  const g = new THREE.Group(); g.position.set(-1.4, 1.65, 2.975); g.rotation.y = Math.PI; room.add(g);
  const pic = litPlane(0.8, 0.6, 320, (c, W, H) => {
    const gr = c.createLinearGradient(0, 0, 0, H); gr.addColorStop(0, '#f2b872'); gr.addColorStop(0.65, '#f6e3b8'); c.fillStyle = gr; c.fillRect(0, 0, W, H);
    c.fillStyle = '#fbf3cf'; c.beginPath(); c.arc(W * 0.72, H * 0.34, H * 0.11, 0, 7); c.fill();
    c.fillStyle = '#7c9a55'; c.beginPath(); c.moveTo(0, H * 0.7); c.quadraticCurveTo(W * 0.3, H * 0.45, W * 0.6, H * 0.68); c.quadraticCurveTo(W * 0.8, H * 0.8, W, H * 0.6); c.lineTo(W, H); c.lineTo(0, H); c.fill();
    c.fillStyle = '#4f6e38'; c.beginPath(); c.moveTo(0, H * 0.85); c.quadraticCurveTo(W * 0.5, H * 0.66, W, H * 0.88); c.lineTo(W, H); c.lineTo(0, H); c.fill();
    c.fillStyle = '#4a3020'; c.fillRect(W * 0.22, H * 0.48, 8, H * 0.2); c.fillStyle = '#3d6b33'; c.beginPath(); c.arc(W * 0.22 + 4, H * 0.46, 30, 0, 7); c.fill();
    speckle(c, W, H, 1500, 0.1);
  });
  pic.position.z = 0.012; g.add(pic);
  D(new THREE.BoxGeometry(0.92, 0.72, 0.02), M.brass, 0, 0, 0, g);
  makeIA(g, { onClick: () => notify('Ilus maal… aga selle taga pole midagi.') });
}

/* ---------- raamaturiiul (lääne sein) ---------- */
S(new THREE.BoxGeometry(0.02, 2.0, 2.0), M.woodDark, -2.97, 1.0, -0.5);
for (const z of [-1.5, 0.5]) S(boxUV(0.36, 2.0, 0.03, 0.6), M.wood, -2.8, 1.0, z);
for (const y of [0.06, 0.46, 0.86, 1.26, 1.66, 1.99]) S(boxUV(0.36, 0.03, 2.0, 0.6), M.wood, -2.8, y, -0.5);
S(boxUV(0.25, 0.18, 0.3, 0.6), M.woodDark, -2.8, 0.165, 0.2);   // väike kast alumisel riiulil
const BOOK_ROWS = [0.475, 0.875, 1.275, 1.675];
const MUTED = ['#5d4037', '#6d4c41', '#4e342e', '#3e2723', '#455a64', '#37474f', '#795548', '#8d6e63', '#4a3b2a', '#5b5b4f', '#33413a', '#6b5b45', '#7b5e57', '#546e7a', '#5a4636', '#704a3a'];
const COLS = [['PUNANE', '#d32f2f'], ['ROHELINE', '#2e9d3e'], ['SININE', '#1e6fd9'], ['KOLLANE', '#f2c21b'], ['VALGE', '#f2f2f2']];
const shelfBooks = new THREE.Group(); room.add(shelfBooks);
function buildBooks(){
  for (const ch of [...shelfBooks.children]) { shelfBooks.remove(ch); removeFrom(statics, ch); removeFrom(interactives, ch); ch.geometry.dispose(); }
  const geos = [], col = new THREE.Color();
  for (let row = 0; row < 4; row++) {
    const keyIdx = row === 0 ? -1 : 3 - row;   // ülemine rida = 1. värv
    const books = []; let z = -1.47;
    while (true) { const t = 0.028 + rand() * 0.035; if (z + t > 0.24) break; books.push({ z: z + t / 2, t, h: 0.2 + rand() * 0.13, d: 0.17 + rand() * 0.08 }); z += t + (rand() < 0.12 ? 0.012 : 0.002); }
    const ki = keyIdx >= 0 ? 2 + Math.floor(rand() * (books.length - 4)) : -1;
    books.forEach((b, i) => {
      const isKey = i === ki, front = isKey ? -2.625 : -2.70;
      const g = new THREE.BoxGeometry(b.d, b.h, b.t).translate(front - b.d / 2, BOOK_ROWS[row] + b.h / 2, b.z);
      if (isKey) {
        const m = new THREE.Mesh(g, new THREE.MeshLambertMaterial({ color: COLS[P.lock[keyIdx]][1] }));
        shelfBooks.add(m); statics.push(m);
        makeIA(m, { onClick: () => notify('See raamat on teistest ettepoole lükatud… 🤔') });
      } else {
        const ng = g.toNonIndexed(); g.dispose();
        col.set(pick(MUTED)).multiplyScalar(0.85 + rand() * 0.3);
        const cols = new Float32Array(ng.attributes.position.count * 3);
        for (let k = 0; k < cols.length; k += 3) { cols[k] = col.r; cols[k + 1] = col.g; cols[k + 2] = col.b; }
        ng.setAttribute('color', new THREE.BufferAttribute(cols, 3)); ng.deleteAttribute('uv'); geos.push(ng);
      }
    });
  }
  const mesh = new THREE.Mesh(mergeGeometries(geos), new THREE.MeshLambertMaterial({ vertexColors: true }));
  shelfBooks.add(mesh); statics.push(mesh);
}

/* ---------- kirjutuslaud + sahtel (ida sein) ---------- */
S(boxUV(0.83, 0.05, 1.6, 0.6), M.wood, 2.535, 0.755, 0.2);
for (const [x, z] of [[2.17, -0.55], [2.9, -0.55], [2.17, 0.95], [2.9, 0.95]]) S(boxUV(0.06, 0.73, 0.06, 0.6), M.woodDark, x, 0.365, z);
for (const [z0, z1] of [[-0.55, -0.11], [0.51, 0.95]]) S(boxUV(0.03, 0.13, z1 - z0, 0.6), M.woodDark, 2.17, 0.665, (z0 + z1) / 2);
for (const z of [-0.11, 0.51]) S(new THREE.BoxGeometry(0.78, 0.17, 0.02), M.inner, 2.55, 0.645, z);
S(new THREE.BoxGeometry(0.78, 0.02, 0.64), M.inner, 2.55, 0.555, 0.2);
S(new THREE.BoxGeometry(0.02, 0.17, 0.64), M.inner, 2.93, 0.645, 0.2);
const drawer = new THREE.Group(); room.add(drawer);
S(boxUV(0.03, 0.16, 0.6, 0.6), M.wood, 2.165, 0.645, 0.2, drawer);
S(new THREE.BoxGeometry(0.66, 0.012, 0.56), M.inner, 2.51, 0.58, 0.2, drawer);
for (const z of [-0.074, 0.474]) S(new THREE.BoxGeometry(0.66, 0.11, 0.012), M.inner, 2.51, 0.64, z, drawer);
S(new THREE.BoxGeometry(0.012, 0.11, 0.56), M.inner, 2.84, 0.64, 0.2, drawer);
D(new THREE.BoxGeometry(0.006, 0.055, 0.26), M.brass, 2.147, 0.672, 0.2, drawer);
{
  const h = new THREE.Group(); h.position.set(2.135, 0.6, 0.2); drawer.add(h);
  D(new THREE.BoxGeometry(0.02, 0.02, 0.14), M.brass, 0, 0, 0, h);
  makeIA(h, { onClick: () => { sfx.bad(); notify('Sahtel on lukus – keera kolme värvirõngast 🎨'); }, enabled: () => !st.drawerOpen });
}
const rings = [];
for (let i = 0; i < 3; i++) {
  const m = new THREE.Mesh(new THREE.CylinderGeometry(0.03, 0.03, 0.026, 20).rotateZ(Math.PI / 2), new THREE.MeshLambertMaterial({ color: 0xffffff }));
  m.position.set(2.131, 0.672, 0.2 + (i - 1) * 0.08); drawer.add(m);
  D(new THREE.BoxGeometry(0.028, 0.006, 0.006), M.black, 0, 0.029, 0, m);
  makeIA(m, { onClick: () => clickRing(i), enabled: () => !st.drawerOpen });
  rings.push(m);
}
// lauatarbed
{
  const lamp = new THREE.Group(); lamp.position.set(2.75, 0.78, -0.4); room.add(lamp);
  D(new THREE.CylinderGeometry(0.07, 0.08, 0.025, 20), M.brass, 0, 0.012, 0, lamp);
  const arm = D(new THREE.CylinderGeometry(0.008, 0.008, 0.36, 8), M.brass, -0.05, 0.18, 0.04, lamp); arm.rotation.set(0.25, 0, 0.3);
  const sh = D(new THREE.ConeGeometry(0.08, 0.1, 20, 1, true), M.shade, -0.12, 0.33, 0.09, lamp); sh.rotation.z = 0.5;
  D(new THREE.SphereGeometry(0.025, 10, 8), new THREE.MeshBasicMaterial({ color: 0xffe2a8 }), -0.12, 0.3, 0.09, lamp);
}
const startNote = new THREE.Mesh(new THREE.PlaneGeometry(0.2, 0.27).rotateX(-Math.PI / 2), M.paper);
startNote.position.set(2.42, 0.783, 0.6); startNote.rotation.y = 0.25; room.add(startNote);
makeIA(startNote, { onClick: () => openReader('✉️ Kiri laual', START_TEXT) });
{
  const bell = new THREE.Group(); bell.position.set(2.36, 0.78, 0.88); room.add(bell);
  D(new THREE.CylinderGeometry(0.055, 0.06, 0.015, 20), M.woodDark, 0, 0.0075, 0, bell);
  D(new THREE.SphereGeometry(0.045, 20, 10, 0, Math.PI * 2, 0, Math.PI / 2), M.brass, 0, 0.015, 0, bell);
  D(new THREE.CylinderGeometry(0.008, 0.008, 0.02, 8), M.brass, 0, 0.065, 0, bell);
  makeIA(bell, { onClick: () => { sfx.bell(); showHint(); } });
  const card = litPlane(0.12, 0.05, 1200, (c, W, H) => { c.fillStyle = '#f2e6c8'; c.fillRect(0, 0, W, H); c.fillStyle = '#5a1a1a'; c.font = `bold ${H * 0.55}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText('🔔 VIHJE', W / 2, H * 0.55); });
  card.position.set(2.3, 0.81, 0.72); card.rotation.y = -Math.PI / 2; room.add(card);
}

/* ---------- kell (ida sein) ---------- */
const clockG = new THREE.Group(); clockG.position.set(2.975, 1.95, -1.8); clockG.rotation.y = -Math.PI / 2; room.add(clockG);
S(new THREE.CylinderGeometry(0.19, 0.19, 0.04, 40).rotateX(Math.PI / 2), M.woodDark, 0, 0, 0, clockG);
{
  const face = new THREE.Mesh(new THREE.CircleGeometry(0.165, 48), new THREE.MeshLambertMaterial({ map: canvasTex(256, 256, (c, W, H) => {
    c.fillStyle = '#f5ecd6'; c.fillRect(0, 0, W, H);
    c.strokeStyle = '#3b2a16'; c.lineWidth = 6; c.beginPath(); c.arc(W / 2, H / 2, W / 2 - 6, 0, 7); c.stroke();
    c.fillStyle = '#2b1d10'; c.font = `bold ${W * 0.1}px Georgia, serif`; c.textAlign = 'center'; c.textBaseline = 'middle';
    for (let n = 1; n <= 12; n++) { const a = n * Math.PI / 6 - Math.PI / 2; c.fillText(String(n), W / 2 + Math.cos(a) * W * 0.36, H / 2 + Math.sin(a) * H * 0.36); }
    for (let n = 0; n < 60; n++) { const a = n * Math.PI / 30, r0 = W * (n % 5 ? 0.45 : 0.43); c.lineWidth = n % 5 ? 1.5 : 3; c.beginPath(); c.moveTo(W / 2 + Math.cos(a) * r0, H / 2 + Math.sin(a) * r0); c.lineTo(W / 2 + Math.cos(a) * W * 0.47, H / 2 + Math.sin(a) * W * 0.47); c.stroke(); }
  }, false) }));
  face.position.z = 0.021; clockG.add(face);
}
const hourHand = new THREE.Group(); hourHand.position.z = 0.024; clockG.add(hourHand);
D(new THREE.BoxGeometry(0.016, 0.085, 0.004).translate(0, 0.035, 0), M.black, 0, 0, 0, hourHand);
const minuteHand = new THREE.Group(); minuteHand.position.z = 0.029; clockG.add(minuteHand);
D(new THREE.BoxGeometry(0.008, 0.13, 0.003).translate(0, 0.055, 0), M.black, 0, 0, 0, minuteHand);
D(new THREE.CylinderGeometry(0.012, 0.012, 0.01, 12).rotateX(Math.PI / 2), M.brass, 0, 0, 0.032, clockG);
makeIA(clockG, { onClick: () => { sfx.tick(); notify('Kell on seisma jäänud… 🕐'); } });

/* ---------- kirst (kagunurk) ---------- */
const chest = new THREE.Group(); chest.position.set(2.1, 0, 2.5); room.add(chest);
S(boxUV(0.9, 0.03, 0.5, 0.6), M.wood, 0, 0.015, 0, chest);
for (const z of [-0.235, 0.235]) S(boxUV(0.9, 0.42, 0.03, 0.6), M.wood, 0, 0.24, z, chest);
for (const x of [-0.435, 0.435]) S(boxUV(0.03, 0.42, 0.44, 0.6), M.wood, x, 0.24, 0, chest);
D(new THREE.PlaneGeometry(0.84, 0.44).rotateX(-Math.PI / 2), M.inner, 0, 0.031, 0, chest);
for (const x of [-0.3, 0.3]) D(new THREE.BoxGeometry(0.05, 0.43, 0.006), M.darkMetal, x, 0.235, -0.253, chest);
const lid = new THREE.Group(); lid.position.set(0, 0.45, 0.25); chest.add(lid);
S(boxUV(0.94, 0.06, 0.52, 0.6), M.wood, 0, 0.03, -0.26, lid);
for (const x of [-0.3, 0.3]) D(new THREE.BoxGeometry(0.05, 0.065, 0.53), M.darkMetal, x, 0.03, -0.26, lid);
const lockPlate = new THREE.Group(); lockPlate.position.set(0, 0.37, -0.256); chest.add(lockPlate);
D(new THREE.BoxGeometry(0.09, 0.11, 0.01), M.brass, 0, 0, 0, lockPlate);
D(new THREE.CircleGeometry(0.009, 12).rotateY(Math.PI), M.black, 0, 0.012, -0.0055, lockPlate);
D(new THREE.PlaneGeometry(0.006, 0.02).rotateY(Math.PI), M.black, 0, -0.004, -0.0055, lockPlate);
const keyInLock = D(new THREE.TorusGeometry(0.018, 0.005, 8, 16), M.brass, 0, 0.012, -0.035, lockPlate); keyInLock.visible = false;
makeIA(lockPlate, { accepts: 'key', onUse: useKey, onEmpty: () => { sfx.bad(); notify('Kirst on lukus 🔒 Vaja on võtit.'); }, enabled: () => !st.chestOpen, wrongMsg: 'See ei sobi lukku.' });
const riddleNote = new THREE.Mesh(new THREE.PlaneGeometry(0.2, 0.26).rotateX(-Math.PI / 2), M.paper);
riddleNote.position.set(0.18, 0.034, 0); riddleNote.rotation.y = -0.3; chest.add(riddleNote);
makeIA(riddleNote, { onClick: () => { st.riddleRead = true; openReader('📜 Mõistatus', P.riddle + '\n\n(Vastus on üks number.)'); }, enabled: () => st.chestOpen });

/* ---------- gloobus (edelanurk) ---------- */
const globe = new THREE.Group(); globe.position.set(-2.2, 0, 2.2); room.add(globe);
S(new THREE.CylinderGeometry(0.2, 0.24, 0.05, 24), M.woodDark, 0, 0.025, 0, globe);
S(new THREE.CylinderGeometry(0.025, 0.03, 0.7, 10), M.woodDark, 0, 0.4, 0, globe);
D(new THREE.TorusGeometry(0.29, 0.012, 8, 48).rotateY(Math.PI / 2), M.brass, 0, 1.0, 0, globe);
D(new THREE.TorusGeometry(0.262, 0.01, 8, 48).rotateX(Math.PI / 2), M.brass, 0, 1.0, 0, globe);
const globeSpin = new THREE.Group(); globeSpin.position.y = 1.0; globe.add(globeSpin);
const globeTop = new THREE.Group(); globeSpin.add(globeTop);
{
  const gMat = new THREE.MeshLambertMaterial({ map: globeTex, side: THREE.DoubleSide });
  const top = new THREE.SphereGeometry(0.25, 40, 12, 0, Math.PI * 2, 0, Math.PI / 2);
  const bot = new THREE.SphereGeometry(0.25, 40, 12, 0, Math.PI * 2, Math.PI / 2, Math.PI / 2);
  const tu = top.attributes.uv, bu = bot.attributes.uv;
  for (let i = 0; i < tu.count; i++) tu.setY(i, 0.5 + tu.getY(i) * 0.5);
  for (let i = 0; i < bu.count; i++) bu.setY(i, bu.getY(i) * 0.5);
  D(top, gMat, 0, 0, 0, globeTop);
  D(bot, gMat, 0, 0, 0, globeSpin);
  D(new THREE.CircleGeometry(0.248, 32).rotateX(-Math.PI / 2), M.woodDark, 0, 0, 0, globeSpin);
  D(new THREE.CircleGeometry(0.248, 32).rotateX(Math.PI / 2), M.woodDark, 0, 0, 0, globeTop);
}
const GLOBE_BASE = 3 * Math.PI / 4;   // punane täht vaatab toa keskele
makeIA(globeSpin, { onClick: clickGlobe, enabled: () => !st.globeOpen });

/* ---------- elektrikilp (põhja sein) ---------- */
const fuseBox = new THREE.Group(); fuseBox.position.set(-1.6, 1.45, -2.93); room.add(fuseBox);
S(new THREE.BoxGeometry(0.4, 0.5, 0.14), M.metal, 0, 0, 0, fuseBox);
{
  const face = litPlane(0.36, 0.46, 700, (c, W, H) => {
    c.fillStyle = '#8c949b'; c.fillRect(0, 0, W, H);
    c.fillStyle = '#f2c21b'; c.fillRect(0, 0, W, H * 0.07); c.fillRect(0, H * 0.93, W, H * 0.07);
    c.fillStyle = '#111'; for (let x = -H; x < W; x += 28) { c.beginPath(); c.moveTo(x, 0); c.lineTo(x + 14, 0); c.lineTo(x + 14 + H * 0.07, H * 0.07); c.lineTo(x + H * 0.07, H * 0.07); c.fill(); }
    c.font = `bold ${H * 0.075}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText('⚡ ELEKTRIKILP', W / 2, H * 0.16);
    c.fillStyle = '#1e2226'; roundRect(c, W * 0.1, H * 0.3, W * 0.8, H * 0.42, 10); c.fill();
    c.strokeStyle = '#c9a13b'; c.lineWidth = 4;
    for (const fx of [0.222, 0.5, 0.778]) c.strokeRect(W * fx - W * 0.07, H * 0.36, W * 0.14, H * 0.3);
    c.fillStyle = '#ff4d4d'; c.font = `bold ${H * 0.1}px ${FONT}`; c.fillText('?', W * 0.778, H * 0.82);
  });
  face.position.z = 0.071; fuseBox.add(face);
}
function buildFuse(){
  const g = new THREE.Group();
  D(new THREE.CylinderGeometry(0.011, 0.011, 0.045, 14).rotateX(Math.PI / 2), new THREE.MeshLambertMaterial({ color: 0xcfe8ff, transparent: true, opacity: 0.6 }), 0, 0, 0, g);
  D(new THREE.CylinderGeometry(0.002, 0.002, 0.045, 4).rotateX(Math.PI / 2), M.darkMetal, 0, 0, 0, g);
  for (const z of [-0.028, 0.028]) D(new THREE.CylinderGeometry(0.013, 0.013, 0.014, 14).rotateX(Math.PI / 2), M.brass, 0, 0, z, g);
  return g;
}
for (const x of [-0.1, 0]) { const f = buildFuse(); f.rotation.x = Math.PI / 2; f.position.set(x, -0.005, 0.086); fuseBox.add(f); }
const insertedFuse = buildFuse(); insertedFuse.rotation.x = Math.PI / 2; insertedFuse.position.set(0.1, -0.005, 0.086); insertedFuse.visible = false; fuseBox.add(insertedFuse);
makeIA(fuseBox, { accepts: 'fuse', onUse: useFuse, onEmpty: () => { sfx.bad(); notify('Elektrikilbis on üks kaitse puudu ⚡'); }, enabled: () => !st.fuseIn, wrongMsg: 'See ei ole kaitse.' });

/* ---------- koodi silt + klahvistik ---------- */
{
  const sign = litPlane(1.3, 0.34, 600, (c, W, H) => {
    const g = c.createLinearGradient(0, 0, 0, H); g.addColorStop(0, '#d8b45a'); g.addColorStop(1, '#9c7a2c'); c.fillStyle = g; roundRect(c, 0, 0, W, H, 24); c.fill();
    c.strokeStyle = '#5a4214'; c.lineWidth = 8; roundRect(c, 10, 10, W - 20, H - 20, 18); c.stroke();
    c.fillStyle = '#3a2a0a'; c.textAlign = 'center'; c.textBaseline = 'middle';
    c.font = `bold ${H * 0.16}px ${FONT}`; c.fillText('KOODI JÄRJEKORD', W / 2, H * 0.2);
    const cells = [['🔦', 'UV-kiri'], ['🦆', 'parte toas'], ['📜', 'mõistatus'], ['🕐', 'kell (tunnid)']];
    cells.forEach(([e, l], i) => {
      const cx = W * (0.14 + i * 0.24);
      c.fillStyle = 'rgba(60,40,10,0.18)'; roundRect(c, cx - W * 0.1, H * 0.34, W * 0.2, H * 0.56, 14); c.fill();
      c.fillStyle = '#3a2a0a'; c.font = `bold ${H * 0.13}px ${FONT}`; c.fillText(`${i + 1}.`, cx - W * 0.07, H * 0.5);
      c.font = `${H * 0.22}px ${FONT}`; c.fillText(e, cx + W * 0.02, H * 0.52);
      c.font = `bold ${H * 0.1}px ${FONT}`; c.fillText(l, cx, H * 0.8);
    });
  });
  sign.position.set(0, 2.56, -2.985); room.add(sign);
}
const keypad = new THREE.Group(); keypad.position.set(0.9, 1.35, -2.975); room.add(keypad);
S(new THREE.BoxGeometry(0.24, 0.36, 0.03), M.darkMetal, 0, 0, 0, keypad);
const kpDisplay = new CanvasPlane(0.2, 0.07, 1200); kpDisplay.mesh.position.set(0, 0.125, 0.0165); keypad.add(kpDisplay.mesh);
const ledMat = new THREE.MeshLambertMaterial({ color: 0x330000, emissive: 0x550000 });
D(new THREE.SphereGeometry(0.007, 10, 8), ledMat, 0.1, 0.163, 0.016, keypad);
const kpButtons = [];
['1', '2', '3', '4', '5', '6', '7', '8', '9', 'C', '0', 'OK'].forEach((lbl, i) => {
  const b = new CanvasPlane(0.058, 0.052, 1200);
  b.draw((c, W, H) => {
    roundRect(c, 4, 4, W - 8, H - 8, 12); c.fillStyle = lbl === 'C' ? '#8e2a2a' : lbl === 'OK' ? '#2a7a35' : '#4a5258'; c.fill();
    c.strokeStyle = 'rgba(255,255,255,0.4)'; c.lineWidth = 3; c.stroke();
    c.fillStyle = '#fff'; c.font = `bold ${H * (lbl === 'OK' ? 0.42 : 0.55)}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText(lbl, W / 2, H * 0.54);
  });
  b.mesh.position.set(((i % 3) - 1) * 0.068, 0.05 - Math.floor(i / 3) * 0.062, 0.0165);
  keypad.add(b.mesh);
  makeIA(b.mesh, { scaleHover: true, onClick: () => keyPress(lbl) });
  kpButtons.push(b.mesh);
});

/* ---------- tugitool ja taim ---------- */
{
  const ch = new THREE.Group(); ch.position.set(-2.15, 0, -2.15); ch.rotation.y = Math.PI / 4; room.add(ch);
  S(new THREE.BoxGeometry(0.7, 0.32, 0.65), M.fabric, 0, 0.16, 0, ch);
  S(new THREE.BoxGeometry(0.6, 0.12, 0.6), M.fabric, 0, 0.38, 0.02, ch);
  S(new THREE.BoxGeometry(0.7, 0.65, 0.15), M.fabric, 0, 0.7, -0.27, ch);
  for (const x of [-0.32, 0.32]) S(new THREE.BoxGeometry(0.1, 0.26, 0.65), M.fabric, x, 0.55, 0, ch);
  const pl = new THREE.Group(); pl.position.set(2.55, 0, -2.55); room.add(pl);
  S(new THREE.CylinderGeometry(0.18, 0.13, 0.35, 16), M.pot, 0, 0.175, 0, pl);
  for (let i = 0; i < 9; i++) { const a = i * 0.7, l = D(new THREE.SphereGeometry(0.1, 8, 6).scale(0.5, 2.6, 0.25), M.leaf, Math.cos(a) * 0.07, 0.55 + (i % 3) * 0.06, Math.sin(a) * 0.07, pl); l.rotation.set(Math.sin(a) * 0.5, a, Math.cos(a) * 0.5); }
}

/* ---------- UV salakiri (lõuna sein) ---------- */
const uvCanvas = new CanvasPlane(1.2, 0.7, 700);
const uvMsg = uvCanvas.mesh;
uvMsg.material.opacity = 0; uvMsg.material.blending = THREE.AdditiveBlending; uvMsg.material.depthWrite = false;
uvMsg.position.set(0.9, 1.55, 2.968); uvMsg.rotation.y = Math.PI; room.add(uvMsg);
function drawUVMsg(){
  uvCanvas.draw((c, W, H) => {
    c.textAlign = 'center'; c.textBaseline = 'middle';
    c.shadowColor = '#b388ff'; c.shadowBlur = 30; c.fillStyle = '#e6d6ff';
    c.font = `bold ${H * 0.4}px ${FONT}`; c.fillText(`🔦 = ${P.d1}`, W / 2, H * 0.35);
    c.shadowBlur = 16; c.fillStyle = '#c6ffef'; c.font = `bold ${H * 0.085}px ${FONT}`;
    wrapText(c, 'Võti peitub maakera sees.\nKeera maakera, kuni punane täht vaatab toa keskele!', W / 2, H * 0.78, W * 0.92, H * 0.11);
  });
}

/* ---------- pardid ---------- */
const DUCK_SPOTS = [
  [2.75, 0.78, -0.15, 2.4], [2.9, 1.30, 0.6, Math.PI], [-2.76, 0.475, 0.36, 1.57], [-2.76, 1.275, 0.38, 1.57],
  [-2.78, 2.005, -0.9, 1.4], [-2.75, 0.075, -0.6, 1.6], [2.6, 0.0, 0.25, 3.0], [-2.12, 0.44, -2.12, 0.8],
  [-2.62, 0.0, 2.62, 0.6], [-1.6, 1.70, -2.93, -Math.PI / 2], [1.45, 0.0, 2.7, 2.0], [2.78, 0.0, 2.86, 2.6],
  [2.2, 0.0, -2.65, 1.2], [-1.35, 0.006, 1.35, -0.6]
];
const ducks = [];
for (let i = 0; i < 8; i++) {
  const d = new THREE.Group(); room.add(d);
  D(new THREE.SphereGeometry(0.05, 14, 10).scale(1.35, 0.9, 1), M.duck, 0, 0.045, 0, d);
  D(new THREE.SphereGeometry(0.032, 12, 10), M.duck, 0.045, 0.1, 0, d);
  D(new THREE.ConeGeometry(0.012, 0.03, 8).rotateZ(-Math.PI / 2), M.beak, 0.085, 0.095, 0, d);
  for (const z of [-0.017, 0.017]) D(new THREE.SphereGeometry(0.005, 6, 6), M.black, 0.068, 0.11, z, d);
  D(new THREE.ConeGeometry(0.02, 0.04, 8).rotateZ(Math.PI / 2 + 0.5), M.duck, -0.07, 0.07, 0, d);
  makeIA(d, { onClick: () => { sfx.squeak(); const y0 = d.position.y; tween(0.3, k => d.position.y = y0 + Math.sin(k * Math.PI) * 0.06); } });
  ducks.push(d);
}

/* ---------- võetavad esemed ---------- */
function finishItem(g, ia){
  g.position.set(0, 0, 0); g.rotation.set(0, 0, 0); g.updateMatrixWorld(true);
  const box = new THREE.Box3().setFromObject(g);
  ia.restOffset = -box.min.y + 0.002; ia.grabbable = true;
  ia.holdPos = ia.holdPos || new THREE.Vector3(); ia.holdRot = ia.holdRot || new THREE.Euler();
  return makeIA(g, ia);
}
const uvObj = new THREE.Group();
D(new THREE.CylinderGeometry(0.018, 0.018, 0.17, 14).rotateX(Math.PI / 2), M.darkMetal, 0, 0, 0.02, uvObj);
D(new THREE.CylinderGeometry(0.028, 0.02, 0.05, 16).rotateX(Math.PI / 2), new THREE.MeshLambertMaterial({ color: 0x4a2a7a }), 0, 0, -0.085, uvObj);
D(new THREE.CircleGeometry(0.025, 16).rotateY(Math.PI), new THREE.MeshLambertMaterial({ color: 0x9b5cff, emissive: 0x6a2cff }), 0, 0, -0.111, uvObj);
D(new THREE.BoxGeometry(0.01, 0.006, 0.02), new THREE.MeshLambertMaterial({ color: 0xc0392b }), 0, 0.019, -0.01, uvObj);
finishItem(uvObj, { item: 'uv', enabled: () => st.drawerOpen, onGrab: () => { if (!st.uvTaken) { st.uvTaken = true; notify('UV-lamp! 🔦 Suuna see seintele.', 3); } } });
const beam = new THREE.Mesh(new THREE.ConeGeometry(0.35, 2.2, 20, 1, true).translate(0, -1.1, 0).rotateX(Math.PI / 2),
  new THREE.MeshBasicMaterial({ color: 0x9b5cff, transparent: true, opacity: 0.13, blending: THREE.AdditiveBlending, depthWrite: false, side: THREE.DoubleSide }));
beam.position.z = -0.112; beam.visible = false; uvObj.add(beam);

const keyObj = new THREE.Group();
D(new THREE.TorusGeometry(0.022, 0.006, 8, 20).rotateX(Math.PI / 2), M.brass, 0, 0, 0.055, keyObj);
D(new THREE.CylinderGeometry(0.005, 0.005, 0.08, 8).rotateX(Math.PI / 2), M.brass, 0, 0, 0, keyObj);
D(new THREE.BoxGeometry(0.016, 0.006, 0.01), M.brass, 0.01, 0, -0.035, keyObj);
D(new THREE.BoxGeometry(0.012, 0.006, 0.008), M.brass, 0.008, 0, -0.018, keyObj);
finishItem(keyObj, { item: 'key', enabled: () => st.globeOpen, onGrab: () => { st.keyTaken = true; } });

const fuseObj = buildFuse();
finishItem(fuseObj, { item: 'fuse', enabled: () => st.chestOpen });

const items = {
  uv: { obj: uvObj, home: drawer, pos: new THREE.Vector3(2.45, 0.586, 0.2), rotY: 0 },
  key: { obj: keyObj, home: globe, pos: new THREE.Vector3(0, 1.0, 0), rotY: 0.6 },
  fuse: { obj: fuseObj, home: chest, pos: new THREE.Vector3(-0.2, 0.031, -0.05), rotY: 0.4 }
};

/* =========================================================
   KASUTAJALIIDES
   ========================================================= */
function makeButton(label, w, h, onClick, { color = '#2e7d32', fs = 0.4 } = {}){
  const p = new CanvasPlane(w, h, 600, true);
  p.draw((c, W, H) => {
    roundRect(c, 5, 5, W - 10, H - 10, Math.min(W, H) * 0.25); c.fillStyle = color; c.fill(); c.lineWidth = 6; c.strokeStyle = 'rgba(255,255,255,.85)'; c.stroke();
    c.fillStyle = '#fff'; c.textAlign = 'center'; c.textBaseline = 'middle'; c.font = `bold ${H * fs}px ${FONT}`; c.fillText(label, W / 2, H * 0.54);
  });
  p.mesh.renderOrder = 910;
  return makeIA(p.mesh, { ui: true, onClick });
}
function placeInFront(g, dist = 1, yOff = -0.1){
  scene.updateMatrixWorld(true);
  camera.getWorldPosition(head); camera.getWorldDirection(fwd); fwd.y = 0;
  if (fwd.lengthSq() < 1e-4) fwd.set(0, 0, -1);
  fwd.normalize();
  g.position.copy(head).addScaledVector(fwd, dist); g.position.y = head.y + yOff;
  g.lookAt(head.x, g.position.y, head.z);
}
// teade
const notice = new CanvasPlane(0.9, 0.12, 700, true);
notice.mesh.renderOrder = 999; notice.mesh.position.set(0, -0.22, -1); notice.mesh.visible = false; camera.add(notice.mesh);
let noticeT = 0;
function notify(msg, dur = 2.5){
  noticeT = dur; notice.mesh.visible = true;
  notice.draw((c, W, H) => {
    panelBg(c, W, H, 'rgba(0,0,0,0.78)', 30); c.fillStyle = '#fff'; c.textAlign = 'center'; c.textBaseline = 'middle';
    let fs = H * 0.38; c.font = `bold ${fs}px ${FONT}`;
    while (c.measureText(msg).width > W * 0.93 && fs > 12) { fs -= 2; c.font = `bold ${fs}px ${FONT}`; }
    c.fillText(msg, W / 2, H / 2);
  });
}
// lugemisvaade (kirjad, vihjed)
const readerUI = new THREE.Group(); readerUI.visible = false; scene.add(readerUI);
const readerPanel = new CanvasPlane(1.0, 0.72, 900, true); readerUI.add(readerPanel.mesh);
makeIA(readerPanel.mesh, { ui: true, noHover: true, onClick: () => closeReader() });
{ const b = makeButton('✖ Sulge', 0.3, 0.09, () => closeReader(), { color: '#455a64', fs: 0.45 }); b.position.set(0, -0.43, 0.01); readerUI.add(b); }
function openReader(title, text, paper = true){
  readerPanel.draw((c, W, H) => {
    if (paper) { roundRect(c, 4, 4, W - 8, H - 8, 24); c.fillStyle = '#f1e4c3'; c.fill(); c.strokeStyle = '#b79a62'; c.lineWidth = 8; c.stroke(); c.fillStyle = '#3b2a16'; }
    else { panelBg(c, W, H); c.fillStyle = '#ffffff'; }
    c.textAlign = 'center'; c.textBaseline = 'middle';
    c.font = `bold ${H * 0.085}px ${FONT}`; c.fillText(title, W / 2, H * 0.11);
    fitText(c, text, W / 2, H * 0.2, W * 0.86, H * 0.76, Math.round(H * 0.058));
  });
  placeInFront(readerUI, 0.9, -0.08); readerUI.visible = true;
}
function closeReader(){ readerUI.visible = false; }
// peamenüü
const menuUI = new THREE.Group(); scene.add(menuUI);
{
  const p = new CanvasPlane(1.3, 0.8, 800, true); menuUI.add(p.mesh); makeIA(p.mesh, { ui: true, noHover: true });
  p.draw((c, W, H) => {
    panelBg(c, W, H); c.textAlign = 'center'; c.textBaseline = 'middle';
    c.fillStyle = '#ffd54f'; c.font = `bold ${H * 0.12}px ${FONT}`; c.fillText('🔐 Põgenemistuba', W / 2, H * 0.13);
    c.fillStyle = '#cfd8dc'; c.font = `${H * 0.06}px ${FONT}`; c.fillText('Professor Nupu töötuba', W / 2, H * 0.25);
    c.fillStyle = '#fff';
    fitText(c, 'Professor lukustas su oma töötuppa. Leia 4-numbriline kood ja ava uks!\n\n🎯 Päästik = kasuta / võta ese\n✊ Külgnupp = haara (lase lahti = pane maha)\n🕹️ Vasak kang = liigu • parem kang = pööra\n⏸ Y / B = paus • X / A = pane ese maha', W / 2, H * 0.32, W * 0.88, H * 0.62, Math.round(H * 0.05));
  });
  const b = makeButton('▶ Alusta', 0.5, 0.14, () => newGame(), { color: '#2e7d32', fs: 0.45 }); b.position.set(0, -0.5, 0.01); menuUI.add(b);
}
// paus
const pauseUI = new THREE.Group(); pauseUI.visible = false; scene.add(pauseUI);
const pausePanel = new CanvasPlane(0.8, 0.26, 700, true); pausePanel.mesh.position.y = 0.36; pauseUI.add(pausePanel.mesh); makeIA(pausePanel.mesh, { ui: true, noHover: true });
const pauseBtns = [
  ['▶ Jätka', '#2e7d32', () => closePause()],
  ['💡 Vihje', '#f57f17', () => { closePause(); showHint(); }],
  ['🔁 Uus mäng', '#1565c0', () => newGame()],
  ['🚪 Välju VR-ist', '#b71c1c', () => exitVR()]
].map(([l, col, fn], i) => { const b = makeButton(l, 0.56, 0.12, fn, { color: col, fs: 0.42 }); b.position.set(0, 0.15 - i * 0.145, 0.01); pauseUI.add(b); return b; });
// võit
const winUI = new THREE.Group(); winUI.visible = false; scene.add(winUI);
const winPanel = new CanvasPlane(1.1, 0.7, 800, true); winPanel.mesh.position.y = 0.12; winUI.add(winPanel.mesh); makeIA(winPanel.mesh, { ui: true, noHover: true });
const winBtns = [['🔁 Uus mäng', '#2e7d32', () => newGame()], ['🚪 Välju VR-ist', '#b71c1c', () => exitVR()]]
  .map(([l, col, fn], i) => { const b = makeButton(l, 0.46, 0.12, fn, { color: col, fs: 0.4 }); b.position.set(-0.25 + i * 0.5, -0.33, 0.01); winUI.add(b); return b; });

/* =========================================================
   MÕISTATUSED
   ========================================================= */
const RIDDLES = [
  ['Oksal istus 8 lindu. 7 lendas minema. Mitu lindu jäi oksale?', 1],
  ['Ämblikul on 8 jalga, putukal 6. Mitu jalga on ämblikul rohkem?', 2],
  ['Kui täna on esmaspäev, mitmes nädalapäev on ülehomme?', 3],
  ['Karbis oli 12 muna. Pooled söödi ära ja siis veel kaks. Mitu muna jäi alles?', 4],
  ['Kolm sõpra jagavad 15 kommi võrdselt. Mitu kommi saab igaüks?', 5],
  ['Mõtlen ühe arvu. Kui korrutan selle kolmega, saan 18. Mis arvu ma mõtlen?', 6],
  ['Mitu nurka on kolmnurgal ja ruudul kokku?', 7],
  ['Mis arv tuleb järgmisena: 2, 4, 6, …?', 8],
  ['Mis on pool kümnest pluss neli?', 9]
];
const START_TEXT = 'Tere, uurija! 🕵️\n\nLukustasin su oma töötuppa. Uks avaneb ainult 4-numbrilise koodiga ja kõik numbrid on peidetud siia tuppa.\n\nUkse kohal olev silt ütleb, mis järjekorras numbrid tulevad.\n\nKui jääd hätta, helista laual olevat kella 🔔\n\n– Professor Nupp';
let P = {}, st = {}, mode = 'menu', elapsed = 0, entry = '', kpMsg = null;

function setupPuzzle(){
  const lock = shuffle([0, 1, 2, 3, 4]).slice(0, 3), r = pick(RIDDLES);
  P = { lock, d1: randInt(1, 9), d2: randInt(3, 8), d3: r[1], riddle: r[0], d4: randInt(1, 9), globeK: randInt(1, 5) };
  P.code = `${P.d1}${P.d2}${P.d3}${P.d4}`;
  st = { ring: lock.map(c => (c + randInt(1, 4)) % 5), drawerOpen: false, uvTaken: false, uvReveal: 0, uvFound: false,
         globeK: P.globeK, globeBusy: false, globeOpen: false, keyTaken: false, chestOpen: false, fuseIn: false, riddleRead: false, escaped: false, hints: 0 };
}
function resetItems(){
  for (const h of allHands) h.held = null;
  for (const k in items) {
    const it = items[k];
    it.home.add(it.obj);
    it.obj.position.copy(it.pos); it.obj.position.y += it.obj.userData.ia.restOffset;
    it.obj.rotation.set(0, it.rotY, 0); it.obj.visible = k !== 'key';
  }
}
function applyPuzzle(){
  buildBooks();
  rings.forEach((m, i) => { m.material.color.set(COLS[st.ring[i]][1]); m.rotation.x = 0; });
  drawer.position.x = 0;
  drawUVMsg(); uvMsg.material.opacity = 0;
  globeSpin.rotation.y = GLOBE_BASE + st.globeK * Math.PI / 3; globeTop.position.y = 0; globeTop.rotation.z = 0;
  lid.rotation.x = 0; keyInLock.visible = false;
  insertedFuse.visible = false;
  setLights(0);
  entry = ''; kpMsg = null; setKeypadPower(false);
  doorPivot.rotation.y = 0; outLight.intensity = 0;
  const spots = shuffle(DUCK_SPOTS.slice());
  ducks.forEach((d, i) => { if (i < P.d2) { const s = spots[i]; d.position.set(s[0], s[1], s[2]); d.rotation.y = s[3] + (rand() - 0.5) * 0.6; d.visible = true; } else d.visible = false; });
  hourHand.rotation.z = -P.d4 * Math.PI / 6; minuteHand.rotation.z = 0;
  resetItems();
}
function setLights(v){
  hemi.intensity = 1.1 + 1.1 * v;
  ceilLight.intensity = 10 * v;
  bulbMat.emissive.setRGB(1, 0.95, 0.8).multiplyScalar(0.12 + 0.88 * v);
}
function setKeypadPower(on){
  ledMat.color.set(on ? 0x003300 : 0x330000); ledMat.emissive.set(on ? 0x00ff44 : 0x550000);
  kpButtons.forEach(b => b.material.color.set(on ? 0xffffff : 0x666666));
  drawKeypad();
}
function drawKeypad(){
  kpDisplay.draw((c, W, H) => {
    c.fillStyle = st.fuseIn ? '#0c2a12' : '#050505'; c.fillRect(0, 0, W, H);
    if (!st.fuseIn) return;
    c.font = `bold ${H * 0.62}px monospace`; c.textAlign = 'center'; c.textBaseline = 'middle';
    if (kpMsg) { c.fillStyle = kpMsg.col; c.fillText(kpMsg.t, W / 2, H * 0.55); }
    else { c.fillStyle = '#5dff7a'; c.fillText((entry + '____').slice(0, 4).split('').join(' '), W / 2, H * 0.55); }
  });
}

/* ---------- mõistatuste tegevused ---------- */
function clickRing(i){
  st.ring[i] = (st.ring[i] + 1) % 5;
  const m = rings[i], r0 = m.rotation.x;
  m.material.color.set(COLS[st.ring[i]][1]);
  tween(0.25, k => m.rotation.x = r0 + ease(k) * Math.PI * 2 / 5);
  tone(900, .03, 0, 'square', .06);
  if (st.ring.every((c, j) => c === P.lock[j])) {
    st.drawerOpen = true;
    delay(0.35, () => { sfx.unlock(); notify('Klõps! Sahtel avanes 🔓'); tween(0.9, k => drawer.position.x = -0.42 * ease(k)); });
  }
}
function clickGlobe(){
  if (st.globeBusy) return;
  st.globeBusy = true; st.globeK++;
  const r0 = globeSpin.rotation.y; sfx.rotate();
  tween(0.5, k => globeSpin.rotation.y = r0 + ease(k) * Math.PI / 3, () => { st.globeBusy = false; if (st.globeK % 6 === 0) openGlobe(); });
}
function openGlobe(){
  st.globeOpen = true; sfx.unlock(); notify('Maakera avanes! Sees on võti 🔑', 3);
  items.key.obj.visible = true;
  tween(1.0, k => { globeTop.position.y = 0.22 * ease(k); globeTop.rotation.z = 0.5 * ease(k); });
}
function consume(h){ const it = h.held; h.held = null; room.add(it); it.visible = false; return it; }
function useKey(h){
  consume(h); st.chestOpen = true; keyInLock.visible = true;
  sfx.unlock(); notify('Kirst avanes! 📦', 3);
  tween(1.2, k => lid.rotation.x = 1.9 * ease(k));
}
function useFuse(h){
  consume(h); insertedFuse.visible = true; st.fuseIn = true;
  sfx.spark(); notify('Valgus tuli tagasi! ⚡ Klahvistik töötab.', 3.5);
  tween(1.6, k => setLights(k < 0.6 ? (Math.sin(k * 70) > 0.2 ? 0.7 : 0.05) : (k - 0.6) / 0.4), () => setLights(1));
  setKeypadPower(true);
}
function keyPress(k){
  if (!st.fuseIn) { sfx.bad(); notify('Klahvistikul pole voolu ⚡'); return; }
  if (st.escaped || kpMsg) return;
  sfx.beep();
  if (k === 'C') entry = '';
  else if (k === 'OK') {
    if (entry.length < 4) notify('Kood on 4 numbrit');
    else if (entry === P.code) { codeOK(); return; }
    else { kpMsg = { t: 'VALE', col: '#ff4d4d' }; entry = ''; sfx.bad(); delay(1.0, () => { kpMsg = null; drawKeypad(); }); }
  }
  else if (entry.length < 4) entry += k;
  drawKeypad();
}
function codeOK(){
  st.escaped = true; kpMsg = { t: 'AVATUD', col: '#5dff7a' }; drawKeypad(); sfx.unlock();
  delay(0.6, () => {
    sfx.creak();
    tween(2.2, k => { doorPivot.rotation.y = 1.75 * ease(k); outLight.intensity = 6 * k; });
  });
  delay(2.8, win);
}
function win(){
  mode = 'win'; sfx.win(); closeReader();
  const stars = st.hints === 0 && elapsed < 900 ? 3 : st.hints <= 2 ? 2 : 1;
  const msg = stars === 3 ? 'Suurepärane! Tõeline detektiiv!' : stars === 2 ? 'Väga tubli!' : 'Pääsesid välja – hästi tehtud!';
  winPanel.draw((c, W, H) => {
    panelBg(c, W, H); c.textAlign = 'center'; c.textBaseline = 'middle';
    c.fillStyle = '#ffd54f'; c.font = `bold ${H * 0.13}px ${FONT}`; c.fillText('🎉 Pääsesid välja!', W / 2, H * 0.16);
    c.font = `${H * 0.17}px ${FONT}`; c.fillText('★'.repeat(stars) + '☆'.repeat(3 - stars), W / 2, H * 0.4);
    c.fillStyle = '#fff'; c.font = `bold ${H * 0.08}px ${FONT}`; c.fillText(`Aeg: ${fmtTime(elapsed)}   •   Vihjeid: ${st.hints}`, W / 2, H * 0.6);
    c.font = `${H * 0.07}px ${FONT}`; c.fillText(msg, W / 2, H * 0.76);
    c.fillStyle = '#9fb3bf'; c.font = `${H * 0.055}px ${FONT}`; c.fillText(`Kood oli ${P.code}`, W / 2, H * 0.9);
  });
  winBtns[1].visible = renderer.xr.isPresenting;
  placeInFront(winUI, 1.2, -0.05); winUI.visible = true;
}
function currentHint(){
  if (!st.drawerOpen) return 'Laua sahtlil on kolme värvirõngaga lukk. Vaata raamaturiiulit: kolm raamatut on teistest ettepoole lükatud. Nende värvid ülevalt alla on rõngaste õiged värvid vasakult paremale.';
  if (!st.uvTaken) return 'Sahtel on lahti! Seal on UV-lamp. Võta see kätte (päästik või külgnupp).';
  if (!st.uvFound) return 'UV-lamp näitab nähtamatut kirja. Suuna lamp seintele – proovi ukse vastas olevat seina.';
  if (!st.globeOpen) return 'Klõpsa maakeral, et seda keerata. Kui punane täht vaatab toa keskele, maakera avaneb.';
  if (!st.keyTaken && !st.chestOpen) return 'Maakera sees on võti. Võta see kätte.';
  if (!st.chestOpen) return 'Võti avab toa nurgas oleva kirstu. Hoia võtit käes ja klõpsa kirstu lukul.';
  if (!st.fuseIn) return 'Kirstus on kaitse. Pane see ukse kõrval olevasse elektrikilpi: hoia kaitset käes ja klõpsa kilbil.';
  if (!st.riddleRead) return 'Kirstus on ka paber mõistatusega. Klõpsa sellel, et lugeda.';
  return 'Ukse kohal olev silt ütleb numbrite järjekorra: 1) UV-kirja number, 2) mitu parti on toas, 3) mõistatuse vastus, 4) mitu tundi näitab kell. Pardid on peidus üle kogu toa – vaata ka laua alla, riiulitele ja nurkadesse!';
}
function showHint(){ if (mode !== 'play') return; st.hints++; openReader(`💡 Vihje (${st.hints})`, currentHint(), false); }

/* ---------- mängu olekud ---------- */
function hideMenus(){ menuUI.visible = false; pauseUI.visible = false; winUI.visible = false; closeReader(); }
function newGame(){
  tweens.length = 0; hideMenus();
  setupPuzzle(); applyPuzzle();
  elapsed = 0; mode = 'play';
  placePlayer(0, 1.3, 0);
  openReader('✉️ Kiri laual', START_TEXT);
}
function openPause(){
  mode = 'pause'; closeReader();
  pausePanel.draw((c, W, H) => { panelBg(c, W, H); c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillStyle = '#ffd54f'; c.font = `bold ${H * 0.34}px ${FONT}`; c.fillText('⏸ Paus', W / 2, H * 0.36); c.fillStyle = '#fff'; c.font = `${H * 0.2}px ${FONT}`; c.fillText(`Aeg ${fmtTime(elapsed)} • vihjeid ${st.hints}`, W / 2, H * 0.74); });
  pauseBtns[3].visible = renderer.xr.isPresenting;
  placeInFront(pauseUI, 1.1, -0.05); pauseUI.visible = true;
}
function closePause(){ if (mode !== 'pause') return; mode = 'play'; pauseUI.visible = false; }
function togglePause(){
  if (mode === 'play') { if (readerUI.visible) closeReader(); else openPause(); }
  else if (mode === 'pause') closePause();
}
function exitVR(){ const s = renderer.xr.getSession(); if (s) s.end(); }
function placePlayer(x, z, yaw = 0){
  const e = new THREE.Euler().setFromQuaternion(camera.quaternion, 'YXZ');
  const ry = renderer.xr.isPresenting ? yaw - e.y : yaw;
  rig.rotation.set(0, ry, 0);
  const off = renderer.xr.isPresenting ? new THREE.Vector3(camera.position.x, 0, camera.position.z).applyAxisAngle(UP, ry) : new THREE.Vector3();
  rig.position.set(x - off.x, 0, z - off.z);
}

/* =========================================================
   KÄED, KIIR JA ESEMETE VÕTMINE
   ========================================================= */
const raycaster = new THREE.Raycaster();
const mouseHolder = new THREE.Group(); mouseHolder.position.set(0.17, -0.15, -0.4); camera.add(mouseHolder);
const mouseHand = { kind: 'mouse', holder: mouseHolder, held: null };
const xrHands = [];
for (let i = 0; i < 2; i++) {
  const ctrl = renderer.xr.getController(i), grip = renderer.xr.getControllerGrip(i);
  rig.add(ctrl); rig.add(grip);
  const holder = new THREE.Group(); holder.position.set(0, -0.01, -0.07); ctrl.add(holder);
  const line = new THREE.Line(new THREE.BufferGeometry().setFromPoints([new THREE.Vector3(), new THREE.Vector3(0, 0, -1)]), new THREE.LineBasicMaterial({ color: 0xffffff, transparent: true, opacity: 0.6 }));
  line.visible = false; ctrl.add(line);
  const dot = new THREE.Mesh(new THREE.SphereGeometry(0.008, 8, 8), new THREE.MeshBasicMaterial({ color: 0xffe066 })); dot.visible = false; scene.add(dot);
  const body = new THREE.Mesh(new THREE.BoxGeometry(0.035, 0.03, 0.11), new THREE.MeshLambertMaterial({ color: 0x2a2a2a })); body.position.z = 0.03; grip.add(body);
  const h = { kind: 'xr', ctrl, grip, holder, line, dot, held: null, bySqueeze: false, handedness: null, connected: false };
  ctrl.addEventListener('connected', e => { h.connected = true; h.handedness = e.data.handedness; });
  ctrl.addEventListener('disconnected', () => { if (h.held) dropFall(h); h.connected = false; });
  ctrl.addEventListener('selectstart', () => doAction(h, cast(h)));
  ctrl.addEventListener('squeezestart', () => squeeze(h, true));
  ctrl.addEventListener('squeezeend', () => squeeze(h, false));
  xrHands.push(h);
}
const allHands = [mouseHand, ...xrHands];
const mouseNDC = new THREE.Vector2(0, 0);

function iaRoot(o){ while (o) { if (o.userData.ia) return o; o = o.parent; } return null; }
function isHeld(o){ for (const h of allHands) if (h.held) { let p = o; while (p) { if (p === h.held) return true; p = p.parent; } } return false; }
function iaOk(root){
  const ia = root.userData.ia;
  if (!shown(root)) return false;
  if (ia.ui) return true;
  if (mode !== 'play') return false;
  if (ia.enabled && !ia.enabled()) return false;
  return !isHeld(root);
}
function setRay(h){
  if (h.kind === 'mouse') raycaster.setFromCamera(mouseNDC, camera);
  else { h.ctrl.getWorldPosition(tmpV); h.ctrl.getWorldQuaternion(tmpQ); tmpD.set(0, 0, -1).applyQuaternion(tmpQ); raycaster.set(tmpV, tmpD); }
  raycaster.far = 12;
}
function cast(h){
  setRay(h);
  const ui = interactives.filter(o => o.userData.ia.ui && shown(o));
  if (ui.length) { const hit = raycaster.intersectObjects(ui, true).find(x => shown(x.object)); if (hit) return { hit, root: iaRoot(hit.object) }; }
  if (mode !== 'play') return null;
  const list = interactives.filter(o => !o.userData.ia.ui && iaOk(o));
  const hits = raycaster.intersectObjects(list.concat(statics), true);
  for (const hit of hits) {
    if (!shown(hit.object) || isHeld(hit.object)) continue;
    const root = iaRoot(hit.object);
    return { hit, root: root && list.includes(root) ? root : null };
  }
  return null;
}
function doAction(h, r){
  audio();
  const root = r && r.root;
  if (root) {
    const ia = root.userData.ia;
    if (ia.ui) { if (ia.onClick) { sfx.click(); ia.onClick(h); } return; }
    if (h.held) {
      const item = h.held.userData.ia.item;
      if (ia.accepts) { if (ia.accepts === item) ia.onUse(h); else { sfx.bad(); notify(ia.wrongMsg || 'See ese ei sobi siia.'); } return; }
      if (ia.onClick) { ia.onClick(h); return; }
      if (ia.grabbable) { notify('Käsi on täis – pane ese enne maha'); return; }
      return;
    }
    if (ia.grabbable) { grab(h, root); return; }
    if (ia.accepts) { ia.onEmpty && ia.onEmpty(); return; }
    if (ia.onClick) ia.onClick(h);
  } else if (r && h.held) dropAt(h, r.hit);
}
function grab(h, root, bySqueeze = false){
  for (const o of allHands) if (o.held === root) o.held = null;
  h.held = root; h.bySqueeze = bySqueeze;
  const ia = root.userData.ia;
  h.holder.add(root); root.position.copy(ia.holdPos); root.rotation.copy(ia.holdRot); root.visible = true;
  sfx.pick();
  if (ia.onGrab) ia.onGrab();
}
function surfaceBelow(p){
  tmpV2.set(p.x, p.y + 0.05, p.z); raycaster.set(tmpV2, DOWN); raycaster.far = 4;
  for (const h of raycaster.intersectObjects(statics, false)) {
    if (!shown(h.object) || !h.face) continue;
    const n = h.face.normal.clone().transformDirection(h.object.matrixWorld);
    if (n.y > 0.5) return h.point.y;
  }
  return 0;
}
function clampRoom(p){ p.x = clamp(p.x, -2.85, 2.85); p.z = clamp(p.z, -2.85, 2.85); return p; }
function placeItem(it, p, yaw){ room.add(it); it.position.set(p.x, p.y + it.userData.ia.restOffset, p.z); it.rotation.set(0, yaw, 0); it.visible = true; }
function dropAt(h, hit){
  const n = hit.face ? hit.face.normal.clone().transformDirection(hit.object.matrixWorld) : UP;
  const p = hit.point.clone();
  if (n.y < 0.7) { p.addScaledVector(n, 0.12); clampRoom(p); p.y = surfaceBelow(p); } else clampRoom(p);
  const it = h.held; h.held = null; placeItem(it, p, rand() * 6.28); sfx.drop();
}
function dropFall(h){
  const it = h.held; if (!it) return;
  it.getWorldPosition(tmpV);
  if (mode === 'play') for (const o of interactives) {   // lase lahti otse luku / kilbi juures = kasuta
    const ia = o.userData.ia;
    if (ia.accepts === it.userData.ia.item && iaOk(o)) { o.getWorldPosition(tmpV2); if (tmpV2.distanceTo(tmpV) < 0.3) { ia.onUse(h); return; } }
  }
  const p = clampRoom(tmpV.clone()); p.y = surfaceBelow(p);
  h.held = null; placeItem(it, p, rand() * 6.28); sfx.drop();
}
function squeeze(h, down){
  if (mode !== 'play') return;
  if (down) {
    if (h.held) return;
    h.grip.getWorldPosition(tmpV);
    let best = null, bd = 0.18;
    for (const k in items) { const o = items[k].obj; if (!iaOk(o)) continue; o.getWorldPosition(tmpV2); const d = tmpV2.distanceTo(tmpV); if (d < bd) { bd = d; best = o; } }
    if (!best) { const r = cast(h); if (r && r.root && r.root.userData.ia.grabbable) best = r.root; }
    if (best) grab(h, best, true);
  } else if (h.held && h.bySqueeze) dropFall(h);
}

/* ---------- esile tõstmine ---------- */
let hovered = new Set();
function setHL(root, on){
  const ia = root.userData.ia;
  if (ia.ui || ia.scaleHover) { root.scale.copy(root.userData.bs).multiplyScalar(on ? 1.08 : 1); return; }
  root.traverse(m => { if (m.userData.baseEm) m.material.emissive.copy(m.userData.baseEm).addScalar(on ? 0.14 : 0); });
}
function applyHover(set){ for (const o of hovered) if (!set.has(o)) setHL(o, false); for (const o of set) if (!hovered.has(o)) setHL(o, true); hovered = set; }
function updateHands(){
  const set = new Set();
  const consider = r => { if (r && r.root && !r.root.userData.ia.noHover) set.add(r.root); };
  if (renderer.xr.isPresenting) {
    for (const h of xrHands) {
      if (!h.connected) { h.line.visible = false; h.dot.visible = false; continue; }
      const r = cast(h); consider(r);
      h.line.visible = true; h.line.scale.z = r ? Math.max(0.02, r.hit.distance) : 2;
      h.line.material.color.set(r && r.root ? 0xffe066 : 0xffffff);
      h.dot.visible = !!r; if (r) h.dot.position.copy(r.hit.point);
    }
  } else {
    const r = cast(mouseHand); consider(r);
    renderer.domElement.style.cursor = r && r.root ? 'pointer' : 'default';
  }
  applyHover(set);
}

/* ---------- UV-lamp ---------- */
function updateUV(dt){
  const uv = uvObj, holding = allHands.some(h => h.held === uv);
  beam.visible = holding && mode === 'play';
  let on = false;
  if (beam.visible) {
    uv.getWorldPosition(tmpV); uv.getWorldQuaternion(tmpQ); tmpD.set(0, 0, -1).applyQuaternion(tmpQ);
    raycaster.set(tmpV, tmpD); raycaster.far = 6;
    const hit = raycaster.intersectObjects([uvMsg, ...statics], false).find(x => shown(x.object));
    on = !!hit && hit.object === uvMsg;
  }
  if (on) st.uvReveal = Math.min(1, st.uvReveal + dt * 1.3);
  else if (!st.uvFound) st.uvReveal = Math.max(0, st.uvReveal - dt * 0.8);
  if (st.uvReveal >= 1 && !st.uvFound) { st.uvFound = true; sfx.reveal(); notify('Salakiri! ✨ Loe seda seinalt.', 3); }
  uvMsg.material.opacity = st.uvFound ? (on ? 1 : 0.8) : st.uvReveal;
}

/* =========================================================
   SISEND: hiir, klaviatuur, Questi nupud
   ========================================================= */
let dragging = false, dragMoved = 0, lastX = 0, lastY = 0, pitch = 0, downBtn = 0;
const keys = {};
function setMouse(e){ mouseNDC.set(e.clientX / innerWidth * 2 - 1, -(e.clientY / innerHeight) * 2 + 1); }
renderer.domElement.addEventListener('contextmenu', e => e.preventDefault());
renderer.domElement.addEventListener('pointerdown', e => { audio(); dragging = true; dragMoved = 0; lastX = e.clientX; lastY = e.clientY; downBtn = e.button; setMouse(e); });
addEventListener('pointerup', e => {
  if (dragging && dragMoved < 6 && !renderer.xr.isPresenting) {
    setMouse(e); scene.updateMatrixWorld(true);
    const r = cast(mouseHand);
    if (downBtn === 2) { if (mouseHand.held && mode === 'play') { if (r && !r.root) dropAt(mouseHand, r.hit); else dropFall(mouseHand); } }
    else doAction(mouseHand, r);
  }
  dragging = false;
});
addEventListener('pointermove', e => {
  setMouse(e);
  if (renderer.xr.isPresenting || !dragging) return;
  const dx = e.clientX - lastX, dy = e.clientY - lastY; dragMoved += Math.abs(dx) + Math.abs(dy);
  rig.rotation.y -= dx * 0.004; pitch = clamp(pitch - dy * 0.004, -1.3, 1.3); camera.rotation.x = pitch;
  lastX = e.clientX; lastY = e.clientY;
});
addEventListener('keydown', e => { keys[e.code] = true; if (e.code === 'Escape') togglePause(); });
addEventListener('keyup', e => { keys[e.code] = false; });
addEventListener('resize', () => { camera.aspect = innerWidth / innerHeight; camera.updateProjectionMatrix(); renderer.setSize(innerWidth, innerHeight); });

const moveInput = new THREE.Vector2();
const prevBtn = {};
function edge(k, v, fn){ if (v && !prevBtn[k]) fn(); prevBtn[k] = !!v; }
let snapReady = true;
function rotateRigAroundHead(a){ camera.getWorldPosition(head); rig.position.sub(head); rig.position.applyAxisAngle(UP, a); rig.position.add(head); rig.rotation.y += a; }
function readXR(){
  moveInput.set(0, 0);
  const s = renderer.xr.getSession(); if (!s) return;
  for (const src of s.inputSources) {
    const gp = src.gamepad; if (!gp) continue;
    const ax = gp.axes, b = gp.buttons, hd = src.handedness;
    const sx = ax.length >= 4 ? ax[2] : (ax[0] || 0), sy = ax.length >= 4 ? ax[3] : (ax[1] || 0);
    if (hd === 'left') { moveInput.x += sx; moveInput.y += sy; }
    else if (hd === 'right') {
      if (Math.abs(sx) > 0.7 && snapReady) { snapReady = false; rotateRigAroundHead(sx > 0 ? -Math.PI / 6 : Math.PI / 6); }
      else if (Math.abs(sx) < 0.3) snapReady = true;
    }
    edge(hd + 'Y', b[5] && b[5].pressed, togglePause);
    edge(hd + 'X', b[4] && b[4].pressed, () => { const h = xrHands.find(x => x.handedness === hd); if (h && h.held && mode === 'play') dropFall(h); });
  }
}

/* ---------- liikumine + kokkupõrked ---------- */
const COLLIDERS = [[2.08, 3, -0.65, 1.05], [-3, -2.58, -1.55, 0.55], [1.62, 2.58, 2.22, 2.78], [-2.48, -1.92, 1.92, 2.48], [-2.65, -1.65, -2.65, -1.65], [2.3, 2.8, -2.8, -2.3]];
function blocked(x, z){
  const r = 0.22;
  if (x < -3 + r || x > 3 - r || z < -3 + r || z > 3 - r) return true;
  for (const c of COLLIDERS) if (x > c[0] - r && x < c[1] + r && z > c[2] - r && z < c[3] + r) return true;
  return false;
}
function move(dt){
  let mx = 0, my = 0;
  if (!renderer.xr.isPresenting) {
    if (keys.KeyW || keys.ArrowUp) my -= 1; if (keys.KeyS || keys.ArrowDown) my += 1;
    if (keys.KeyA) mx -= 1; if (keys.KeyD) mx += 1;
    if (keys.ArrowLeft) rig.rotation.y += 1.8 * dt; if (keys.ArrowRight) rig.rotation.y -= 1.8 * dt;
  } else { mx = moveInput.x; my = moveInput.y; }
  const len = Math.hypot(mx, my); if (len < 0.15) return;
  camera.getWorldDirection(fwd); fwd.y = 0; fwd.normalize();
  const sp = 1.6 * Math.min(1, len) * dt;
  const dx = (fwd.x * -my + -fwd.z * mx) / len * sp, dz = (fwd.z * -my + fwd.x * mx) / len * sp;
  camera.getWorldPosition(head);
  if (!blocked(head.x + dx, head.z + dz)) { rig.position.x += dx; rig.position.z += dz; }
  else if (!blocked(head.x + dx, head.z)) rig.position.x += dx;
  else if (!blocked(head.x, head.z + dz)) rig.position.z += dz;
}
function aimMouseHolder(){
  raycaster.setFromCamera(mouseNDC, camera);
  camera.getWorldQuaternion(tmpQ).invert();
  tmpD.copy(raycaster.ray.direction).applyQuaternion(tmpQ);
  mouseHolder.quaternion.setFromUnitVectors(NEG_Z, tmpD);
}

/* ---------- VR sessioon ---------- */
const info = document.getElementById('info');
renderer.xr.addEventListener('sessionstart', () => {
  info.style.display = 'none'; camera.rotation.set(0, 0, 0); pitch = 0;
  if (mouseHand.held) { const it = mouseHand.held; mouseHand.held = null; it.getWorldPosition(tmpV); const p = clampRoom(tmpV.clone()); p.y = surfaceBelow(p); placeItem(it, p, 0); }
  setTimeout(() => {
    if (mode === 'menu') placePlayer(0, 1.3, 0);
    if (menuUI.visible) placeInFront(menuUI, 1.3, -0.1);
    if (pauseUI.visible) placeInFront(pauseUI, 1.1, -0.05);
    if (winUI.visible) placeInFront(winUI, 1.2, -0.05);
    if (readerUI.visible) placeInFront(readerUI, 0.9, -0.08);
  }, 400);
});
renderer.xr.addEventListener('sessionend', () => {
  info.style.display = ''; camera.position.set(0, 1.6, 0);
  for (const h of xrHands) if (h.held) dropFall(h);
  pauseBtns[3].visible = false; winBtns[1].visible = false;
});

/* =========================================================
   KÄIVITUS + PÕHITSÜKKEL
   ========================================================= */
setupPuzzle(); applyPuzzle();
placePlayer(0, 1.3, 0);
placeInFront(menuUI, 1.3, -0.1);
const clock = new THREE.Clock();
renderer.setAnimationLoop(() => {
  const dt = Math.min(clock.getDelta(), 0.1);
  readXR();
  if (mode === 'play') { elapsed += dt; move(dt); }
  updateTweens(dt);
  if (!renderer.xr.isPresenting) aimMouseHolder();
  updateUV(dt);
  updateHands();
  if (noticeT > 0) { noticeT -= dt; if (noticeT <= 0) notice.mesh.visible = false; }
  renderer.render(scene, camera);
});
</script>
</body>
</html>
