[Uploading index (6).html…]()
<!doctype html>
<html lang="et">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Põgenemistuba VR</title>
<style>
  html,body{margin:0;height:100%;overflow:hidden;background:#000;font-family:system-ui,sans-serif}
  #info{position:fixed;top:10px;left:10px;color:#fff;background:rgba(0,0,0,.65);padding:10px 14px;border-radius:10px;font-size:14px;line-height:1.5;max-width:430px;z-index:5}
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
  <b>🔐 Põgenemistuba VR – Professor Nupu maja</b><br>
  Questis: vajuta all <b>ENTER VR</b>.<br>
  Arvutis: <b>klõps</b> = kasuta / võta ese, <b>parem klõps</b> = pane ese maha,
  <b>lohista</b> = vaata ringi, <b>WASD</b> = liigu, <b>F</b> = viska, <b>Esc</b> = paus.<br>
  <button id="tqBtn" style="margin-top:6px">📚 Õpetaja küsimused</button>
  <div id="tqBox" style="display:none;margin-top:6px">
    Üks küsimus rea kohta: <code>küsimus | õige vastus | vale | vale | vale</code><br>
    <textarea id="tqText" rows="7" cols="52" placeholder="Mis on Eesti pealinn? | Tallinn | Tartu | Pärnu | Narva"></textarea><br>
    <button id="tqSave">💾 Salvesta</button> <span id="tqMsg"></span><br>
    <small>Menüüs vali Õppimine → ✏️ Oma.</small>
  </div>
</div>
<script>
(() => { const b = document.getElementById('tqBtn'), box = document.getElementById('tqBox'), t = document.getElementById('tqText');
  try { t.value = localStorage.getItem('pt_questions') || ''; } catch (e) {}
  b.onclick = () => box.style.display = box.style.display === 'none' ? 'block' : 'none';
  document.getElementById('tqSave').onclick = () => { try { localStorage.setItem('pt_questions', t.value); } catch (e) {}
    const n = t.value.split('\n').filter(l => l.split('|').length >= 3).length; document.getElementById('tqMsg').textContent = `Salvestatud: ${n} küsimust ✔`; };
  ['keydown', 'keyup'].forEach(ev => t.addEventListener(ev, e => e.stopPropagation()));
})();
</script>
<script type="module">
import * as THREE from 'three';
import { VRButton } from 'three/addons/webxr/VRButton.js';
import { mergeGeometries } from 'three/addons/utils/BufferGeometryUtils.js';
import { XRHandModelFactory } from 'three/addons/webxr/XRHandModelFactory.js';

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
const camera = new THREE.PerspectiveCamera(70, innerWidth / innerHeight, 0.02, 80);
const rig = new THREE.Group();
scene.add(rig); rig.add(camera);
camera.position.set(0, 1.6, 0);

const hemi = new THREE.HemisphereLight(0xfff0dd, 0x45352a, 1.1); scene.add(hemi);
const moon = new THREE.DirectionalLight(0x9fb4ff, 0.7); moon.position.set(6, 3.5, 0.2); scene.add(moon);
const deskLight = new THREE.PointLight(0xffc27a, 0.6, 0, 2); deskLight.position.set(2.62, 1.06, -0.32); scene.add(deskLight);
const ceilLight = new THREE.PointLight(0xfff1dc, 0, 0, 2); ceilLight.position.set(0, 2.1, 0); scene.add(ceilLight);
const labLight = new THREE.PointLight(0xf2f8ff, 8, 0, 2); labLight.position.set(0, 2.2, -6); scene.add(labLight);
const outLight = new THREE.PointLight(0xfffbe8, 0, 0, 2); outLight.position.set(0, 1.6, -15.6); scene.add(outLight);

const UP = new THREE.Vector3(0, 1, 0), DOWN = new THREE.Vector3(0, -1, 0), NEG_Z = new THREE.Vector3(0, 0, -1);
const V3 = (x, y, z) => new THREE.Vector3(x, y, z);
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
const fmtTime = s => { s = Math.max(0, Math.floor(s)); return `${Math.floor(s / 60)}:${String(s % 60).padStart(2, '0')}`; };
const removeFrom = (a, x) => { const i = a.indexOf(x); if (i >= 0) a.splice(i, 1); };
const FONT = 'system-ui, "Segoe UI", Roboto, Arial, "Noto Color Emoji", "Apple Color Emoji", "Segoe UI Emoji", sans-serif';
function store(k, v){ try { if (v === undefined) return JSON.parse(localStorage.getItem(k)); localStorage.setItem(k, JSON.stringify(v)); } catch (e) { return null; } }

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
function noise(dur = 0.3, vol = 0.2, freq = 2000, type = 'bandpass'){
  const a = audio(), n = Math.floor(a.sampleRate * dur), b = a.createBuffer(1, n, a.sampleRate), d = b.getChannelData(0);
  for (let i = 0; i < n; i++) d[i] = (Math.random() * 2 - 1) * (1 - i / n);
  const s = a.createBufferSource(); s.buffer = b;
  const f = a.createBiquadFilter(); f.type = type; f.frequency.value = freq;
  const g = a.createGain(); g.gain.value = vol;
  s.connect(f).connect(g).connect(a.destination); s.start();
}
const NOTES = [523.25, 587.33, 659.25, 698.46, 783.99, 880, 987.77];
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
  pour(){ noise(.35, .18, 700, 'lowpass'); sweep(500, 900, .3, 'sine', .05); },
  gurgle(){ for (let i = 0; i < 5; i++) sweep(200 + rand() * 200, 120, .12, 'sine', .12, i * .1); },
  crash(){ noise(.6, .5, 4000, 'highpass'); [2200, 3100, 2700, 3600].forEach((f, i) => tone(f, .15, i * .05, 'triangle', .08)); },
  laser(){ sweep(1200, 300, .4, 'sawtooth', .05); },
  collect(){ [1046, 1318, 1568, 2093].forEach((f, i) => tone(f, .18, i * .07, 'sine', .14)); },
  step(){ noise(.07, .1, 250, 'lowpass'); },
  note(i){ tone(NOTES[i], .7, 0, 'sine', .22); tone(NOTES[i] * 2, .4, 0, 'sine', .05); },
  win(){ [523, 659, 784, 1046, 1318].forEach((f, i) => tone(f, .25, i * .13)); },
  lose(){ [392, 349, 311, 262].forEach((f, i) => tone(f, .35, i * .25, 'triangle', .15)); }
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
function wrapText(c, text, x, y, maxW, lh){ const lines = splitLines(c, text, maxW), y0 = y - (lines.length - 1) * lh / 2; lines.forEach((l, i) => c.fillText(l, x, y0 + i * lh)); }
function fitText(c, text, x, yTop, maxW, maxH, fs0, bold = false){
  let fs = fs0, lines;
  for (; fs > 10; fs -= 1) { c.font = `${bold ? 'bold ' : ''}${fs}px ${FONT}`; lines = splitLines(c, text, maxW); if (lines.length * fs * 1.32 <= maxH) break; }
  wrapText(c, text, x, yTop + maxH / 2, maxW, fs * 1.32);
}
function panelBg(c, W, H, fill = 'rgba(15,28,38,0.94)', r = 36){ roundRect(c, 4, 4, W - 8, H - 8, r); c.fillStyle = fill; c.fill(); c.lineWidth = 6; c.strokeStyle = 'rgba(255,255,255,0.3)'; c.stroke(); }
function parchment(c, W, H){ roundRect(c, 4, 4, W - 8, H - 8, 20); c.fillStyle = '#f1e4c3'; c.fill(); c.strokeStyle = '#b79a62'; c.lineWidth = 8; c.stroke(); }
class CanvasPlane {
  constructor(w, h, ppm = 400, ui = false, lit = false){
    this.canvas = document.createElement('canvas');
    this.canvas.width = Math.round(w * ppm); this.canvas.height = Math.round(h * ppm);
    this.ctx = this.canvas.getContext('2d');
    this.tex = new THREE.CanvasTexture(this.canvas); this.tex.colorSpace = THREE.SRGBColorSpace; this.tex.anisotropy = 4;
    const mat = lit ? new THREE.MeshLambertMaterial({ map: this.tex, transparent: true }) : new THREE.MeshBasicMaterial({ map: this.tex, transparent: true, depthTest: !ui, depthWrite: !ui });
    this.mesh = new THREE.Mesh(new THREE.PlaneGeometry(w, h), mat);
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
function litPlane(w, h, ppm, draw){ return new THREE.Mesh(new THREE.PlaneGeometry(w, h), new THREE.MeshLambertMaterial({ map: canvasTex(Math.round(w * ppm), Math.round(h * ppm), draw, false), transparent: true })); }
function speckle(c, W, H, n, a){ for (let i = 0; i < n; i++) { c.fillStyle = tr() < 0.5 ? `rgba(0,0,0,${a * tr()})` : `rgba(255,255,255,${a * tr()})`; c.fillRect(tr() * W, tr() * H, 2, 2); } }
function grain(c, x, y, w, h, n, a, vertical = false){
  c.save(); c.beginPath(); c.rect(x, y, w, h); c.clip();
  for (let g = 0; g < n; g++) {
    c.strokeStyle = `rgba(40,20,6,${a * (0.5 + tr())})`; c.lineWidth = 0.8 + tr() * 1.5; c.beginPath();
    if (vertical) { const xx = x + tr() * w; c.moveTo(xx, y); c.bezierCurveTo(xx + (tr() - 0.5) * 8, y + h * 0.3, xx + (tr() - 0.5) * 8, y + h * 0.7, xx + (tr() - 0.5) * 6, y + h); }
    else { const yy = y + tr() * h; c.moveTo(x, yy); c.bezierCurveTo(x + w * 0.3, yy + (tr() - 0.5) * 8, x + w * 0.7, yy + (tr() - 0.5) * 8, x + w, yy + (tr() - 0.5) * 6); }
    c.stroke();
  }
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
  c.fillStyle = '#4c7262'; c.fillRect(0, 0, W, H); speckle(c, W, H, 1500, 0.08);
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
const roundRugTex = canvasTex(512, 512, (c, W, H) => {
  c.fillStyle = '#23305a'; c.beginPath(); c.arc(W / 2, H / 2, W / 2, 0, 7); c.fill();
  for (const [r, col, lw] of [[W * 0.46, '#d9b26a', 10], [W * 0.38, '#8a3b4a', 18], [W * 0.2, '#d9b26a', 8]]) { c.strokeStyle = col; c.lineWidth = lw; c.beginPath(); c.arc(W / 2, H / 2, r, 0, 7); c.stroke(); }
  c.fillStyle = '#d9b26a'; for (let i = 0; i < 8; i++) { const a = i * Math.PI / 4; c.beginPath(); c.arc(W / 2 + Math.cos(a) * W * 0.29, H / 2 + Math.sin(a) * W * 0.29, 12, 0, 7); c.fill(); }
  c.font = `${W * 0.12}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText('★', W / 2, H / 2);
  speckle(c, W, H, 3000, 0.1);
}, false);
const paperTex = canvasTex(200, 270, (c, W, H) => {
  c.fillStyle = '#f2e6c8'; c.fillRect(0, 0, W, H);
  c.strokeStyle = 'rgba(60,40,20,0.55)'; c.lineWidth = 2;
  for (let y = 40; y < H - 20; y += 18) { c.beginPath(); c.moveTo(18, y); for (let x = 18; x < W - 18 - tr() * 40; x += 6) c.lineTo(x, y + (tr() - 0.5) * 3); c.stroke(); }
}, false);
function doorTexture(base, panel, label, labelCol){
  return canvasTex(256, 540, (c, W, H) => {
    c.fillStyle = base; c.fillRect(0, 0, W, H); speckle(c, W, H, 2500, 0.1);
    for (const [x, y, w, h] of [[28, 30, 200, 210], [28, 290, 200, 220]]) { c.fillStyle = panel; c.fillRect(x, y, w, h); c.strokeStyle = 'rgba(255,255,255,0.18)'; c.lineWidth = 3; c.strokeRect(x, y, w, h); }
    c.fillStyle = 'rgba(0,0,0,0.35)'; for (let y = 12; y < H; y += 44) for (const x of [10, W - 10]) { c.beginPath(); c.arc(x, y, 4, 0, 7); c.fill(); }
    c.fillStyle = labelCol; c.font = `bold 30px ${FONT}`; c.textAlign = 'center'; c.fillText(label, W / 2, 272);
  }, false);
}
const exitDoorTex = canvasTex(256, 540, (c, W, H) => {
  for (let x = 0; x < W; x += 64) { const b = 90 + tr() * 30; c.fillStyle = `rgb(${b | 0},${b * 0.62 | 0},${b * 0.36 | 0})`; c.fillRect(x, 0, 64, H); grain(c, x, 0, 64, H, 14, 0.15, true); c.fillStyle = 'rgba(0,0,0,0.5)'; c.fillRect(x, 0, 3, H); }
  for (const y of [70, 470]) { c.fillStyle = '#2b2b2b'; c.fillRect(0, y, W, 26); c.fillStyle = '#777'; for (let x = 16; x < W; x += 48) { c.beginPath(); c.arc(x, y + 13, 5, 0, 7); c.fill(); } }
  c.fillStyle = '#ffd54f'; c.font = `bold 30px ${FONT}`; c.textAlign = 'center'; c.fillText('🗝️ KULDUKS', W / 2, 300);
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
const tileTex = canvasTex(256, 256, (c, W, H) => { c.fillStyle = '#b9c7c2'; c.fillRect(0, 0, W, H); const s = W / 4; for (let i = 0; i < 4; i++) for (let j = 0; j < 4; j++) { const v = 222 + tr() * 22; c.fillStyle = `rgb(${v - 6 | 0},${v | 0},${v - 3 | 0})`; c.fillRect(i * s + 2, j * s + 2, s - 4, s - 4); } speckle(c, W, H, 500, 0.05); });
const checkerTex = canvasTex(256, 256, (c, W, H) => { const s = W / 4; for (let i = 0; i < 4; i++) for (let j = 0; j < 4; j++) { c.fillStyle = (i + j) % 2 ? '#e4e4dc' : '#2b2f33'; c.fillRect(i * s, j * s, s, s); } speckle(c, W, H, 1500, 0.08); });
const plankTex = canvasTex(256, 256, (c, W, H) => { for (let x = 0; x < W; x += 64) { const b = 70 + tr() * 25; c.fillStyle = `rgb(${b | 0},${b * 0.66 | 0},${b * 0.45 | 0})`; c.fillRect(x, 0, 64, H); grain(c, x, 0, 64, H, 12, 0.14, true); c.fillStyle = 'rgba(0,0,0,0.5)'; c.fillRect(x, 0, 3, H); } });
const starTex = canvasTex(1024, 1024, (c, W, H) => {
  const g = c.createRadialGradient(W / 2, H / 2, 0, W / 2, H / 2, W * 0.7); g.addColorStop(0, '#1b2350'); g.addColorStop(1, '#070a1c'); c.fillStyle = g; c.fillRect(0, 0, W, H);
  const sr = mulberry32(5);
  c.fillStyle = 'rgba(160,170,255,0.06)'; for (let i = 0; i < 400; i++) { c.beginPath(); c.arc(W * (0.2 + sr() * 0.6), H * (0.35 + sr() * 0.3) + (sr() - 0.5) * 200, 30, 0, 7); c.fill(); }
  for (let i = 0; i < 900; i++) { const r = sr() < 0.08 ? 2.5 : 1.2; c.fillStyle = `rgba(255,255,${200 + sr() * 55 | 0},${0.4 + sr() * 0.6})`; c.beginPath(); c.arc(sr() * W, sr() * H, r, 0, 7); c.fill(); }
  c.strokeStyle = 'rgba(200,210,255,0.35)'; c.lineWidth = 2; c.fillStyle = '#fff';
  for (let k = 0; k < 6; k++) { let x = sr() * W, y = sr() * H; c.beginPath(); c.moveTo(x, y); for (let i = 0; i < 5; i++) { x += (sr() - 0.5) * 160; y += (sr() - 0.5) * 160; c.lineTo(x, y); c.fillRect(x - 2, y - 2, 4, 4); } c.stroke(); }
}, false);
const vaseTex = canvasTex(256, 256, (c, W, H) => {
  c.fillStyle = '#f4f6fb'; c.fillRect(0, 0, W, H); c.strokeStyle = '#2346a8'; c.fillStyle = '#2346a8';
  c.lineWidth = 6; for (const y of [20, 236]) { c.beginPath(); c.moveTo(0, y); c.lineTo(W, y); c.stroke(); }
  for (let i = 0; i < 4; i++) { const x = 32 + i * 64; c.lineWidth = 4; c.beginPath(); c.arc(x, 128, 22, 0, 7); c.stroke(); for (let k = 0; k < 6; k++) { const a = k * Math.PI / 3; c.beginPath(); c.arc(x + Math.cos(a) * 32, 128 + Math.sin(a) * 32, 7, 0, 7); c.fill(); } }
});

const M = {
  floor: new THREE.MeshLambertMaterial({ map: floorTex }),
  floorDark: new THREE.MeshLambertMaterial({ map: floorTex, color: 0x9a8070 }),
  wall: new THREE.MeshLambertMaterial({ map: wallTex }),
  wains: new THREE.MeshLambertMaterial({ map: wainsTex }),
  tile: new THREE.MeshLambertMaterial({ map: tileTex }),
  labBand: new THREE.MeshLambertMaterial({ map: tileTex, color: 0x7fa3b4 }),
  checker: new THREE.MeshLambertMaterial({ map: checkerTex }),
  plank: new THREE.MeshLambertMaterial({ map: plankTex }),
  trim: new THREE.MeshLambertMaterial({ color: 0x4a311c }),
  greyTrim: new THREE.MeshLambertMaterial({ color: 0x8b979c }),
  ceil: new THREE.MeshBasicMaterial({ color: 0x3b352d }),
  labCeil: new THREE.MeshBasicMaterial({ color: 0xd5dcdf }),
  stars: new THREE.MeshBasicMaterial({ map: starTex }),
  wood: new THREE.MeshLambertMaterial({ map: woodTex }),
  woodDark: new THREE.MeshLambertMaterial({ map: woodTex, color: 0x9a7a60 }),
  inner: new THREE.MeshLambertMaterial({ color: 0x3a2618 }),
  metal: new THREE.MeshLambertMaterial({ color: 0x8a9298 }),
  darkMetal: new THREE.MeshLambertMaterial({ color: 0x2d3236 }),
  safe: new THREE.MeshLambertMaterial({ color: 0x3f5a4e }),
  brass: new THREE.MeshLambertMaterial({ color: 0xc9a13b, emissive: 0x1a1200 }),
  gold: new THREE.MeshLambertMaterial({ color: 0xffc83a, emissive: 0x5a3c00 }),
  silver: new THREE.MeshLambertMaterial({ color: 0xd4e6f0, emissive: 0x29404c }),
  black: new THREE.MeshLambertMaterial({ color: 0x111111 }),
  paper: new THREE.MeshLambertMaterial({ map: paperTex, side: THREE.DoubleSide }),
  fabric: new THREE.MeshLambertMaterial({ map: fabricTex }),
  leaf: new THREE.MeshLambertMaterial({ color: 0x3f7f35, flatShading: true }),
  pot: new THREE.MeshLambertMaterial({ color: 0xb5643c }),
  shade: new THREE.MeshLambertMaterial({ color: 0x2f5a3a, side: THREE.DoubleSide }),
  duck: new THREE.MeshLambertMaterial({ color: 0xffd23a }),
  beak: new THREE.MeshLambertMaterial({ color: 0xff8a1c }),
  labTop: new THREE.MeshLambertMaterial({ color: 0x2f3a40 }),
  labCab: new THREE.MeshLambertMaterial({ color: 0xdfe6e8 }),
  glass: new THREE.MeshLambertMaterial({ color: 0xdff3ff, transparent: true, opacity: 0.28, depthWrite: false }),
  porcelain: new THREE.MeshLambertMaterial({ map: vaseTex }),
  flame: new THREE.MeshBasicMaterial({ color: 0xffc04a }),
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
function coloredMerge(parts){
  const geos = [], col = new THREE.Color();
  for (const [g0, hex] of parts) {
    const g = g0.index ? g0.toNonIndexed() : g0; col.set(hex);
    const a = new Float32Array(g.attributes.position.count * 3);
    for (let k = 0; k < a.length; k += 3) { a[k] = col.r; a[k + 1] = col.g; a[k + 2] = col.b; }
    g.setAttribute('color', new THREE.BufferAttribute(a, 3)); if (g.attributes.uv) g.deleteAttribute('uv'); geos.push(g);
  }
  return new THREE.Mesh(mergeGeometries(geos), new THREE.MeshLambertMaterial({ vertexColors: true }));
}

/* =========================================================
   REGISTER: toad, staatilised ja klõpsatavad objektid
   ========================================================= */
const NR = 7;
const ROOMS = Array.from({ length: NR }, (_, i) => ({ z0: 3 - 6 * (i + 1), z1: 3 - 6 * i }));
const roomG = ROOMS.map(() => { const g = new THREE.Group(); scene.add(g); return g; });
let CUR = roomG[0];
const statics = [];
const interactives = [];
function S(geo, mat, x = 0, y = 0, z = 0, parent = CUR){ const m = new THREE.Mesh(geo, mat); m.position.set(x, y, z); parent.add(m); statics.push(m); return m; }
function D(geo, mat, x = 0, y = 0, z = 0, parent = CUR){ const m = new THREE.Mesh(geo, mat); m.position.set(x, y, z); parent.add(m); return m; }
function rod(a, b, r1, r2, mat, parent = CUR, solid = false){
  const d = new THREE.Vector3().subVectors(b, a), L = d.length();
  const m = (solid ? S : D)(new THREE.CylinderGeometry(r2, r1, L, 10), mat, (a.x + b.x) / 2, (a.y + b.y) / 2, (a.z + b.z) / 2, parent);
  m.quaternion.setFromUnitVectors(UP, d.normalize()); return m;
}
function makeIA(root, ia){
  root.userData.ia = ia; interactives.push(root);
  if (!ia.ui && !ia.scaleHover) root.traverse(m => { if (m.isMesh && m.material && m.material.emissive) { m.material = m.material.clone(); m.userData.baseEm = m.material.emissive.clone(); } });
  root.userData.bs = root.scale.clone();
  return root;
}
function shown(o){ let last = null; while (o) { if (!o.visible) return false; last = o; o = o.parent; } return last === scene; }
const roomIdx = z => clamp(Math.floor((3 - z) / 6), 0, NR - 1);
const roomAt = z => roomG[roomIdx(z)];

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
   TUBADE KESTAD (seinad, põrand, lagi)
   ========================================================= */
function wallZ(z, facePlusZ, doorX, mat, tile){
  const rot = facePlusZ ? 0 : Math.PI;
  const segs = doorX == null ? [[-3, 3, 0, 3]] : [[-3, doorX - 0.55, 0, 3], [doorX + 0.55, 3, 0, 3], [doorX - 0.55, doorX + 0.55, 2.15, 3]];
  for (const [x0, x1, y0, y1] of segs) if (x1 - x0 > 0.01) S(planeUV(x1 - x0, y1 - y0, tile).rotateY(rot), mat, (x0 + x1) / 2, (y0 + y1) / 2, z);
}
function shell(i, o){
  CUR = roomG[i]; const R = ROOMS[i], zc = (R.z0 + R.z1) / 2;
  S(planeUV(6, 6, o.floorTile).rotateX(-Math.PI / 2), o.floor, 0, 0, zc);
  D(planeUV(6, 6, o.ceilTile).rotateX(Math.PI / 2), o.ceil, 0, 3, zc);
  S(planeUV(6, 3, o.wallTile).rotateY(-Math.PI / 2), o.wall, 3, 1.5, zc);
  S(planeUV(6, 3, o.wallTile).rotateY(Math.PI / 2), o.wall, -3, 1.5, zc);
  wallZ(R.z1, false, o.southDoor, o.wall, o.wallTile);
  wallZ(R.z0, true, o.northDoor, o.wall, o.wallTile);
  const segs = doorX => doorX == null ? [[-3, 3]] : [[-3, doorX - 0.55], [doorX + 0.55, 3]];
  const bh = o.bandH;
  for (const [x0, x1] of segs(o.southDoor)) { S(boxUV(x1 - x0, bh, 0.02, o.bandTx, o.bandTy), o.band, (x0 + x1) / 2, bh / 2, R.z1 - 0.01); D(new THREE.BoxGeometry(x1 - x0, 0.04, 0.05), o.trim, (x0 + x1) / 2, bh + 0.02, R.z1 - 0.025); }
  for (const [x0, x1] of segs(o.northDoor)) { S(boxUV(x1 - x0, bh, 0.02, o.bandTx, o.bandTy), o.band, (x0 + x1) / 2, bh / 2, R.z0 + 0.01); D(new THREE.BoxGeometry(x1 - x0, 0.04, 0.05), o.trim, (x0 + x1) / 2, bh + 0.02, R.z0 + 0.025); }
  for (const x of [-2.99, 2.99]) { S(boxUV(0.02, bh, 6, o.bandTx, o.bandTy), o.band, x, bh / 2, zc); D(new THREE.BoxGeometry(0.05, 0.04, 6), o.trim, x * 0.995, bh + 0.02, zc); }
  for (const [w, d, x, z] of [[6, 0.05, 0, R.z1 - 0.025], [6, 0.05, 0, R.z0 + 0.025], [0.05, 6, 2.975, zc], [0.05, 6, -2.975, zc]]) D(new THREE.BoxGeometry(w, 0.08, d), o.trim, x, 2.96, z);
}
shell(0, { floor: M.floor, floorTile: 1.5, ceil: M.ceil, ceilTile: 6, wall: M.wall, wallTile: 1, band: M.wains, bandH: 0.9, bandTx: 0.6, bandTy: 0.9, trim: M.trim, southDoor: null, northDoor: 0 });
shell(1, { floor: M.checker, floorTile: 1.2, ceil: M.labCeil, ceilTile: 6, wall: M.tile, wallTile: 0.6, band: M.labBand, bandH: 1.0, bandTx: 0.6, bandTy: 0.6, trim: M.greyTrim, southDoor: 0, northDoor: 0 });
shell(2, { floor: M.floorDark, floorTile: 1.5, ceil: M.stars, ceilTile: 6, wall: M.plank, wallTile: 1, band: M.woodDark, bandH: 0.8, bandTx: 0.6, bandTy: 0.6, trim: M.trim, southDoor: 0, northDoor: 0 });

const stoneTex = canvasTex(256, 256, (c, W, H) => { c.fillStyle = '#141416'; c.fillRect(0, 0, W, H); for (let r = 0; r < 6; r++) { const off = (r % 2) * 32; for (let x = -off; x < W; x += 64) { const v = 40 + tr() * 24; c.fillStyle = `rgb(${v | 0},${v | 0},${v + 4 | 0})`; c.fillRect(x + 2, r * 42.67 + 2, 60, 38.6); } } speckle(c, W, H, 2500, 0.15); });
const glassTex = canvasTex(256, 256, (c, W, H) => { const g = c.createLinearGradient(0, 0, 0, H); g.addColorStop(0, '#d5f0e4'); g.addColorStop(1, '#a9d8c1'); c.fillStyle = g; c.fillRect(0, 0, W, H); c.strokeStyle = '#f2fbf6'; c.lineWidth = 12; c.strokeRect(0, 0, W, H); c.beginPath(); c.moveTo(W / 2, 0); c.lineTo(W / 2, H); c.stroke(); c.fillStyle = 'rgba(255,255,255,0.3)'; c.beginPath(); c.moveTo(20, H * 0.8); c.lineTo(60, H * 0.2); c.lineTo(80, H * 0.2); c.lineTo(40, H * 0.8); c.fill(); });
const terraTex = canvasTex(256, 256, (c, W, H) => { c.fillStyle = '#6d4c3d'; c.fillRect(0, 0, W, H); const s = W / 4; for (let i = 0; i < 4; i++) for (let j = 0; j < 4; j++) { c.fillStyle = `rgb(${175 + tr() * 30 | 0},${92 + tr() * 20 | 0},${62 + tr() * 15 | 0})`; c.fillRect(i * s + 3, j * s + 3, s - 6, s - 6); } speckle(c, W, H, 1500, 0.1); });
const retroTex = canvasTex(256, 256, (c, W, H) => { for (let x = 0; x < W; x += 32) { c.fillStyle = (x / 32) % 2 ? '#c8763a' : '#a85a2a'; c.fillRect(x, 0, 32, H); } c.fillStyle = 'rgba(255,230,180,0.35)'; for (let y = 16; y < H; y += 64) for (let x = 16; x < W; x += 64) { c.beginPath(); c.arc(x, y, 10, 0, 7); c.fill(); } speckle(c, W, H, 800, 0.08); });
const carpetTex = canvasTex(128, 128, (c, W, H) => { c.fillStyle = '#5b2c3c'; c.fillRect(0, 0, W, H); speckle(c, W, H, 3000, 0.2); });
const playTex = canvasTex(256, 256, (c, W, H) => { c.fillStyle = '#4a7fd0'; c.fillRect(0, 0, W, H); c.fillStyle = 'rgba(255,255,255,0.28)'; c.font = `${W * 0.2}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; ['★', '●', '▲', '■'].forEach((e, i) => c.fillText(e, (i % 2 + 0.5) * W / 2, (Math.floor(i / 2) + 0.5) * H / 2)); });
Object.assign(M, {
  stone: new THREE.MeshLambertMaterial({ map: stoneTex }), stoneFloor: new THREE.MeshLambertMaterial({ map: stoneTex, color: 0x9a9a9a }), darkCeil: new THREE.MeshBasicMaterial({ color: 0x0a0a0c }),
  glassWall: new THREE.MeshLambertMaterial({ map: glassTex }), terra: new THREE.MeshLambertMaterial({ map: terraTex }), greenCeil: new THREE.MeshBasicMaterial({ color: 0xd8efe6 }),
  retro: new THREE.MeshLambertMaterial({ map: retroTex }), carpet: new THREE.MeshLambertMaterial({ map: carpetTex }), retroCeil: new THREE.MeshBasicMaterial({ color: 0x7a5e44 }),
  play: new THREE.MeshLambertMaterial({ map: playTex }), gym: new THREE.MeshLambertMaterial({ map: floorTex, color: 0xffe0b0 }), playCeil: new THREE.MeshBasicMaterial({ color: 0xe6edf6 })
});
shell(3, { floor: M.stoneFloor, floorTile: 2, ceil: M.darkCeil, ceilTile: 6, wall: M.stone, wallTile: 1.5, band: M.stone, bandH: 0.4, bandTx: 1, bandTy: 1, trim: M.darkMetal, southDoor: 0, northDoor: 0 });
shell(4, { floor: M.terra, floorTile: 1.2, ceil: M.greenCeil, ceilTile: 6, wall: M.glassWall, wallTile: 1.5, band: M.woodDark, bandH: 0.7, bandTx: 0.6, bandTy: 0.6, trim: M.labCab, southDoor: 0, northDoor: 0 });
shell(5, { floor: M.carpet, floorTile: 1, ceil: M.retroCeil, ceilTile: 6, wall: M.retro, wallTile: 1, band: M.woodDark, bandH: 0.9, bandTx: 0.6, bandTy: 0.6, trim: M.trim, southDoor: 0, northDoor: 0 });
shell(6, { floor: M.gym, floorTile: 1.5, ceil: M.playCeil, ceilTile: 6, wall: M.play, wallTile: 1.5, band: M.labCab, bandH: 1.0, bandTx: 1, bandTy: 1, trim: M.labCab, southDoor: 0, northDoor: 0 });

/* ---------- uksed ---------- */
const DOORS = [];
function makeDoor(z, tex, onClick){
  for (const [w, h, x, y] of [[0.05, 2.15, -0.525, 1.075], [0.05, 2.15, 0.525, 1.075], [1.1, 0.05, 0, 2.125]]) D(new THREE.BoxGeometry(w, h, 0.12), M.trim, x, y, z, scene);
  const pivot = new THREE.Group(); pivot.position.set(-0.5, 0, z); scene.add(pivot);
  S(new THREE.BoxGeometry(1.0, 2.1, 0.05), new THREE.MeshLambertMaterial({ map: tex }), 0.5, 1.05, 0, pivot);
  for (const s of [1, -1]) { D(new THREE.BoxGeometry(0.12, 0.025, 0.04), M.brass, 0.86, 1.0, 0.045 * s, pivot); D(new THREE.CylinderGeometry(0.03, 0.03, 0.01, 16).rotateX(Math.PI / 2), M.brass, 0.9, 1.0, 0.028 * s, pivot); }
  makeIA(pivot, { onClick });
  const d = { z, pivot }; DOORS.push(d); return d;
}
makeDoor(-3, doorTexture('#596067', 'rgba(0,0,0,0.18)', '🔬 LABOR', '#e8c547'), () => { if (!st.door[0]) { sfx.bad(); notify('Uks on lukus 🔒 Sisesta kood klahvistikule.'); } });
makeDoor(-9, doorTexture('#e9eef0', 'rgba(0,40,60,0.08)', '🔭 TÄHETORN', '#1e6fd9'), () => { if (!st.door[1]) { sfx.bad(); notify('Laserlukk 🔴 Juhi kiir ukse kõrval olevasse vastuvõtjasse.'); } });
makeDoor(-15, exitDoorTex, h => {
  if (st.door[2]) return;
  if (h && h.held && h.held.userData.ia.item === 'goldkey') { useGoldKey(h); return; }
  sfx.bad(); notify('Suur uks on lukus. Vaja on kuldvõtit 🗝️');
});
makeDoor(-21, doorTexture('#3a3d40', 'rgba(255,255,255,0.05)', '🌱 KASVUHOONE', '#9ccc65'), () => { if (!st.door[3]) { sfx.bad(); notify('Uks on surve all lukus – keera ventiilid õigeks ⚙️'); } });
makeDoor(-27, doorTexture('#e8f5e9', 'rgba(0,60,0,0.08)', '📻 RAADIOTUBA', '#2e7d32'), h => { if (st.door[4]) return; if (h && h.held && h.held.userData.ia.item === 'greenkey') { consume(h); sfx.unlock(); requestDoor(4); return; } sfx.bad(); notify('Uks on lukus – vaja on rohelist võtit 🗝️'); });
makeDoor(-33, doorTexture('#8d5a3a', 'rgba(0,0,0,0.15)', '🎈 MÄNGUTUBA', '#ffe0b2'), () => { if (!st.door[5]) { sfx.bad(); notify('Uks avaneb raadiokoodiga – lukk on ukse kõrval 📻'); } });
makeDoor(-39, doorTexture('#4a7fd0', 'rgba(255,255,255,0.12)', '🚪 VÄLJAPÄÄS', '#ffeb3b'), () => { if (!st.door[6]) { sfx.bad(); notify('Viska 3 palli korvi – siis uks avaneb 🏀'); } });
function openDoor(i, cb){
  if (st.door[i]) return;
  st.door[i] = true; sfx.creak();
  const p = DOORS[i].pivot, last = i === settings.rooms - 1;
  tween(2.0, k => { p.rotation.y = 1.75 * ease(k); if (last) outLight.intensity = 6 * k; }, () => { if (cb) cb(); if (last) delay(0.6, win); });
}
const outsideBox = new THREE.Mesh(new THREE.BoxGeometry(1.3, 2.3, 1.2), new THREE.MeshBasicMaterial({ color: 0xfff6dc, side: THREE.BackSide })); scene.add(outsideBox);

/* =========================================================
   TUBA 1 – PROFESSORI TÖÖTUBA (z −3 … 3)
   ========================================================= */
CUR = roomG[0];
{ // aken
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
D(new THREE.CylinderGeometry(0.008, 0.008, 0.36, 6), M.black, 0, 2.82, 0);
D(new THREE.ConeGeometry(0.22, 0.18, 24, 1, true), M.shade, 0, 2.58, 0);
const bulbMat = new THREE.MeshLambertMaterial({ color: 0xfff6e0, emissive: 0x222018 });
D(new THREE.SphereGeometry(0.05, 12, 8), bulbMat, 0, 2.52, 0);
S(new THREE.PlaneGeometry(3.2, 2.2).rotateX(-Math.PI / 2), new THREE.MeshLambertMaterial({ map: rugTex }), 0, 0.004, 0.5);
for (const [x, z, w, d] of [[2.55, 0.2, 1.3, 2.0], [2.1, 2.5, 1.3, 0.9], [-2.2, 2.2, 0.8, 0.8], [-2.15, -2.15, 1.2, 1.2], [2.55, -2.55, 0.8, 0.8], [-2.8, -0.5, 0.7, 2.3]]) {
  const m = D(new THREE.PlaneGeometry(w, d).rotateX(-Math.PI / 2), M.shadow, x, 0.007, z); m.renderOrder = 1;
}
{ // maal
  const g = new THREE.Group(); g.position.set(-1.4, 1.65, 2.975); g.rotation.y = Math.PI; CUR.add(g);
  const pic = litPlane(0.8, 0.6, 320, (c, W, H) => {
    const gr = c.createLinearGradient(0, 0, 0, H); gr.addColorStop(0, '#f2b872'); gr.addColorStop(0.65, '#f6e3b8'); c.fillStyle = gr; c.fillRect(0, 0, W, H);
    c.fillStyle = '#fbf3cf'; c.beginPath(); c.arc(W * 0.72, H * 0.34, H * 0.11, 0, 7); c.fill();
    c.fillStyle = '#7c9a55'; c.beginPath(); c.moveTo(0, H * 0.7); c.quadraticCurveTo(W * 0.3, H * 0.45, W * 0.6, H * 0.68); c.quadraticCurveTo(W * 0.8, H * 0.8, W, H * 0.6); c.lineTo(W, H); c.lineTo(0, H); c.fill();
    c.fillStyle = '#4f6e38'; c.beginPath(); c.moveTo(0, H * 0.85); c.quadraticCurveTo(W * 0.5, H * 0.66, W, H * 0.88); c.lineTo(W, H); c.lineTo(0, H); c.fill();
    c.fillStyle = '#4a3020'; c.fillRect(W * 0.22, H * 0.48, 8, H * 0.2); c.fillStyle = '#3d6b33'; c.beginPath(); c.arc(W * 0.22 + 4, H * 0.46, 30, 0, 7); c.fill();
  });
  pic.position.z = 0.012; g.add(pic);
  D(new THREE.BoxGeometry(0.92, 0.72, 0.02), M.brass, 0, 0, 0, g);
  makeIA(g, { onClick: () => notify('Ilus maal… aga selle taga pole midagi.') });
}
// raamaturiiul
S(new THREE.BoxGeometry(0.02, 2.0, 2.0), M.woodDark, -2.97, 1.0, -0.5);
for (const z of [-1.5, 0.5]) S(boxUV(0.36, 2.0, 0.03, 0.6), M.wood, -2.8, 1.0, z);
for (const y of [0.06, 0.46, 0.86, 1.26, 1.66, 1.99]) S(boxUV(0.36, 0.03, 2.0, 0.6), M.wood, -2.8, y, -0.5);
S(boxUV(0.25, 0.18, 0.3, 0.6), M.woodDark, -2.8, 0.165, 0.2);
const BOOK_ROWS = [0.475, 0.875, 1.275, 1.675];
const MUTED = ['#5d4037', '#6d4c41', '#4e342e', '#3e2723', '#455a64', '#37474f', '#795548', '#8d6e63', '#4a3b2a', '#5b5b4f', '#33413a', '#6b5b45', '#7b5e57', '#546e7a', '#5a4636', '#704a3a'];
const COLS = [['PUNANE', '#d32f2f'], ['ROHELINE', '#2e9d3e'], ['SININE', '#1e6fd9'], ['KOLLANE', '#f2c21b'], ['VALGE', '#f2f2f2']];
const shelfBooks = new THREE.Group(); roomG[0].add(shelfBooks);
function buildBooks(){
  for (const ch of [...shelfBooks.children]) { shelfBooks.remove(ch); removeFrom(statics, ch); removeFrom(interactives, ch); ch.geometry.dispose(); }
  const parts = [];
  for (let row = 0; row < 4; row++) {
    const keyIdx = row === 0 ? -1 : 3 - row;
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
      } else { const c = new THREE.Color(pick(MUTED)).multiplyScalar(0.85 + rand() * 0.3); parts.push([g, c]); }
    });
  }
  const mesh = coloredMerge(parts); shelfBooks.add(mesh); statics.push(mesh);
}
// kirjutuslaud + sahtel
S(boxUV(0.83, 0.05, 1.6, 0.6), M.wood, 2.535, 0.755, 0.2);
for (const [x, z] of [[2.17, -0.55], [2.9, -0.55], [2.17, 0.95], [2.9, 0.95]]) S(boxUV(0.06, 0.73, 0.06, 0.6), M.woodDark, x, 0.365, z);
for (const [z0, z1] of [[-0.55, -0.11], [0.51, 0.95]]) S(boxUV(0.03, 0.13, z1 - z0, 0.6), M.woodDark, 2.17, 0.665, (z0 + z1) / 2);
for (const z of [-0.11, 0.51]) S(new THREE.BoxGeometry(0.78, 0.17, 0.02), M.inner, 2.55, 0.645, z);
S(new THREE.BoxGeometry(0.78, 0.02, 0.64), M.inner, 2.55, 0.555, 0.2);
S(new THREE.BoxGeometry(0.02, 0.17, 0.64), M.inner, 2.93, 0.645, 0.2);
const drawer = new THREE.Group(); CUR.add(drawer);
S(boxUV(0.03, 0.16, 0.6, 0.6), M.wood, 2.165, 0.645, 0.2, drawer);
S(new THREE.BoxGeometry(0.66, 0.012, 0.56), M.inner, 2.51, 0.58, 0.2, drawer);
for (const z of [-0.074, 0.474]) S(new THREE.BoxGeometry(0.66, 0.11, 0.012), M.inner, 2.51, 0.64, z, drawer);
S(new THREE.BoxGeometry(0.012, 0.11, 0.56), M.inner, 2.84, 0.64, 0.2, drawer);
D(new THREE.BoxGeometry(0.006, 0.055, 0.26), M.brass, 2.147, 0.672, 0.2, drawer);
{ const h = new THREE.Group(); h.position.set(2.135, 0.6, 0.2); drawer.add(h); D(new THREE.BoxGeometry(0.02, 0.02, 0.14), M.brass, 0, 0, 0, h);
  makeIA(h, { onClick: () => openSlide(h), lockedMsg: 'Sahtel on lukus – keera kolme värvirõngast 🎨', drag: { kind: 'slide', dir: () => V3(-1, 0, 0), get: () => -drawer.position.x, set: v => drawer.position.x = -v, min: 0, max: 0.42 }, dragOk: () => st.drawerOpen }); }
const rings = [];
for (let i = 0; i < 3; i++) {
  const m = new THREE.Mesh(new THREE.CylinderGeometry(0.03, 0.03, 0.026, 20).rotateZ(Math.PI / 2), new THREE.MeshLambertMaterial({ color: 0xffffff }));
  m.position.set(2.131, 0.672, 0.2 + (i - 1) * 0.08); drawer.add(m);
  D(new THREE.BoxGeometry(0.028, 0.006, 0.006), M.black, 0, 0.029, 0, m);
  makeIA(m, { onClick: () => clickRing(i), enabled: () => !st.drawerOpen });
  rings.push(m);
}
{
  const lamp = new THREE.Group(); lamp.position.set(2.75, 0.78, -0.4); CUR.add(lamp);
  D(new THREE.CylinderGeometry(0.07, 0.08, 0.025, 20), M.brass, 0, 0.012, 0, lamp);
  const arm = D(new THREE.CylinderGeometry(0.008, 0.008, 0.36, 8), M.brass, -0.05, 0.18, 0.04, lamp); arm.rotation.set(0.25, 0, 0.3);
  const sh = D(new THREE.ConeGeometry(0.08, 0.1, 20, 1, true), M.shade, -0.12, 0.33, 0.09, lamp); sh.rotation.z = 0.5;
  D(new THREE.SphereGeometry(0.025, 10, 8), new THREE.MeshBasicMaterial({ color: 0xffe2a8 }), -0.12, 0.3, 0.09, lamp);
}
const startNote = new THREE.Mesh(new THREE.PlaneGeometry(0.2, 0.27).rotateX(-Math.PI / 2), M.paper);
startNote.position.set(2.42, 0.783, 0.6); startNote.rotation.y = 0.25; CUR.add(startNote);
makeIA(startNote, { onClick: () => openReader('✉️ Kiri laual', startText()) });
{
  const bell = new THREE.Group(); bell.position.set(2.36, 0.78, 0.88); CUR.add(bell);
  D(new THREE.CylinderGeometry(0.055, 0.06, 0.015, 20), M.woodDark, 0, 0.0075, 0, bell);
  D(new THREE.SphereGeometry(0.045, 20, 10, 0, Math.PI * 2, 0, Math.PI / 2), M.brass, 0, 0.015, 0, bell);
  D(new THREE.CylinderGeometry(0.008, 0.008, 0.02, 8), M.brass, 0, 0.065, 0, bell);
  makeIA(bell, { onClick: () => { sfx.bell(); showHint(); } });
  const card = litPlane(0.12, 0.05, 1200, (c, W, H) => { c.fillStyle = '#f2e6c8'; c.fillRect(0, 0, W, H); c.fillStyle = '#5a1a1a'; c.font = `bold ${H * 0.55}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText('🔔 VIHJE', W / 2, H * 0.55); });
  card.position.set(2.3, 0.81, 0.72); card.rotation.y = -Math.PI / 2; CUR.add(card);
}
// kell
const clockG = new THREE.Group(); clockG.position.set(2.975, 1.95, -1.8); clockG.rotation.y = -Math.PI / 2; CUR.add(clockG);
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
// kirst
const chest = new THREE.Group(); chest.position.set(2.1, 0, 2.5); CUR.add(chest);
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
makeIA(lockPlate, { accepts: 'key', onUse: useKey, onClick: () => { sfx.bad(); notify('Kirst on lukus 🔒 Vaja on võtit.'); }, enabled: () => !st.chestOpen });
const riddleNote = new THREE.Mesh(new THREE.PlaneGeometry(0.2, 0.26).rotateX(-Math.PI / 2), M.paper);
riddleNote.position.set(0.18, 0.034, 0); riddleNote.rotation.y = -0.3; chest.add(riddleNote);
makeIA(riddleNote, { onClick: () => { st.riddleRead = true; openReader('📜 Mõistatus', P.riddle + '\n\n(Vastus on üks number.)'); }, enabled: () => st.chestOpen });
// gloobus
const globe = new THREE.Group(); globe.position.set(-2.2, 0, 2.2); CUR.add(globe);
S(new THREE.CylinderGeometry(0.2, 0.24, 0.05, 24), M.woodDark, 0, 0.025, 0, globe);
S(new THREE.CylinderGeometry(0.025, 0.03, 0.7, 10), M.woodDark, 0, 0.4, 0, globe);
D(new THREE.TorusGeometry(0.29, 0.012, 8, 48).rotateY(Math.PI / 2), M.brass, 0, 1.0, 0, globe);
D(new THREE.TorusGeometry(0.262, 0.01, 8, 48).rotateX(Math.PI / 2), M.brass, 0, 1.0, 0, globe);
const globeSpin = new THREE.Group(); globeSpin.position.y = 1.0; globe.add(globeSpin);
const globeTop = new THREE.Group(); globeSpin.add(globeTop);
{
  const gMat = new THREE.MeshLambertMaterial({ map: globeTex, side: THREE.DoubleSide });
  const top = new THREE.SphereGeometry(0.25, 40, 12, 0, Math.PI * 2, 0, Math.PI / 2), bot = new THREE.SphereGeometry(0.25, 40, 12, 0, Math.PI * 2, Math.PI / 2, Math.PI / 2);
  const tu = top.attributes.uv, bu = bot.attributes.uv;
  for (let i = 0; i < tu.count; i++) tu.setY(i, 0.5 + tu.getY(i) * 0.5);
  for (let i = 0; i < bu.count; i++) bu.setY(i, bu.getY(i) * 0.5);
  D(top, gMat, 0, 0, 0, globeTop); D(bot, gMat, 0, 0, 0, globeSpin);
  D(new THREE.CircleGeometry(0.248, 32).rotateX(-Math.PI / 2), M.woodDark, 0, 0, 0, globeSpin);
  D(new THREE.CircleGeometry(0.248, 32).rotateX(Math.PI / 2), M.woodDark, 0, 0, 0, globeTop);
}
const GLOBE_BASE = 3 * Math.PI / 4;
makeIA(globeSpin, { onClick: clickGlobe, enabled: () => !st.globeOpen });
// elektrikilp
const fuseBox = new THREE.Group(); fuseBox.position.set(-1.6, 1.45, -2.93); CUR.add(fuseBox);
S(new THREE.BoxGeometry(0.4, 0.5, 0.14), M.metal, 0, 0, 0, fuseBox);
{
  const face = litPlane(0.36, 0.46, 700, (c, W, H) => {
    c.fillStyle = '#8c949b'; c.fillRect(0, 0, W, H);
    c.fillStyle = '#f2c21b'; c.fillRect(0, 0, W, H * 0.07); c.fillRect(0, H * 0.93, W, H * 0.07);
    c.font = `bold ${H * 0.075}px ${FONT}`; c.fillStyle = '#111'; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText('⚡ ELEKTRIKILP', W / 2, H * 0.16);
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
makeIA(fuseBox, { accepts: 'fuse', onUse: useFuse, onClick: () => { sfx.bad(); notify('Elektrikilbis on üks kaitse puudu ⚡'); }, enabled: () => !st.fuseIn });
// koodi silt + klahvistik
{
  const sign = litPlane(1.3, 0.34, 600, (c, W, H) => {
    const g = c.createLinearGradient(0, 0, 0, H); g.addColorStop(0, '#d8b45a'); g.addColorStop(1, '#9c7a2c'); c.fillStyle = g; roundRect(c, 0, 0, W, H, 24); c.fill();
    c.strokeStyle = '#5a4214'; c.lineWidth = 8; roundRect(c, 10, 10, W - 20, H - 20, 18); c.stroke();
    c.fillStyle = '#3a2a0a'; c.textAlign = 'center'; c.textBaseline = 'middle';
    c.font = `bold ${H * 0.16}px ${FONT}`; c.fillText('KOODI JÄRJEKORD', W / 2, H * 0.2);
    [['🔦', 'UV-kiri'], ['🦆', 'parte toas'], ['📜', 'mõistatus'], ['🕐', 'kell (tunnid)']].forEach(([e, l], i) => {
      const cx = W * (0.14 + i * 0.24);
      c.fillStyle = 'rgba(60,40,10,0.18)'; roundRect(c, cx - W * 0.1, H * 0.34, W * 0.2, H * 0.56, 14); c.fill();
      c.fillStyle = '#3a2a0a'; c.font = `bold ${H * 0.13}px ${FONT}`; c.fillText(`${i + 1}.`, cx - W * 0.07, H * 0.5);
      c.font = `${H * 0.22}px ${FONT}`; c.fillText(e, cx + W * 0.02, H * 0.52);
      c.font = `bold ${H * 0.1}px ${FONT}`; c.fillText(l, cx, H * 0.8);
    });
  });
  sign.position.set(0, 2.56, -2.985); CUR.add(sign);
}
const keypad = new THREE.Group(); keypad.position.set(0.9, 1.35, -2.975); CUR.add(keypad);
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
{ // tugitool ja taim
  const ch = new THREE.Group(); ch.position.set(-2.15, 0, -2.15); ch.rotation.y = Math.PI / 4; CUR.add(ch);
  S(new THREE.BoxGeometry(0.7, 0.32, 0.65), M.fabric, 0, 0.16, 0, ch);
  S(new THREE.BoxGeometry(0.6, 0.12, 0.6), M.fabric, 0, 0.38, 0.02, ch);
  S(new THREE.BoxGeometry(0.7, 0.65, 0.15), M.fabric, 0, 0.7, -0.27, ch);
  for (const x of [-0.32, 0.32]) S(new THREE.BoxGeometry(0.1, 0.26, 0.65), M.fabric, x, 0.55, 0, ch);
  const pl = new THREE.Group(); pl.position.set(2.55, 0, -2.55); CUR.add(pl);
  S(new THREE.CylinderGeometry(0.18, 0.13, 0.35, 16), M.pot, 0, 0.175, 0, pl);
  for (let i = 0; i < 9; i++) { const a = i * 0.7, l = D(new THREE.SphereGeometry(0.1, 8, 6).scale(0.5, 2.6, 0.25), M.leaf, Math.cos(a) * 0.07, 0.55 + (i % 3) * 0.06, Math.sin(a) * 0.07, pl); l.rotation.set(Math.sin(a) * 0.5, a, Math.cos(a) * 0.5); }
}
// UV salakiri
const uvCanvas = new CanvasPlane(1.2, 0.7, 700);
const uvMsg = uvCanvas.mesh;
uvMsg.material.opacity = 0; uvMsg.material.blending = THREE.AdditiveBlending; uvMsg.material.depthWrite = false;
uvMsg.position.set(0.9, 1.55, 2.968); uvMsg.rotation.y = Math.PI; CUR.add(uvMsg);
function drawUVMsg(){
  uvCanvas.draw((c, W, H) => {
    c.textAlign = 'center'; c.textBaseline = 'middle';
    c.shadowColor = '#b388ff'; c.shadowBlur = 30; c.fillStyle = '#e6d6ff';
    c.font = `bold ${H * 0.4}px ${FONT}`; c.fillText(`🔦 = ${P.d1}`, W / 2, H * 0.35);
    c.shadowBlur = 16; c.fillStyle = '#c6ffef'; c.font = `bold ${H * 0.085}px ${FONT}`;
    wrapText(c, 'Võti peitub maakera sees.\nKeera maakera, kuni punane täht vaatab toa keskele!', W / 2, H * 0.78, W * 0.92, H * 0.11);
  });
}
// pardid
const DUCK_SPOTS = [
  [2.75, 0.78, -0.15, 2.4], [2.9, 1.30, 0.6, Math.PI], [-2.76, 0.475, 0.36, 1.57], [-2.76, 1.275, 0.38, 1.57],
  [-2.78, 2.005, -0.9, 1.4], [-2.75, 0.075, -0.6, 1.6], [2.6, 0.0, 0.25, 3.0], [-2.12, 0.44, -2.12, 0.8],
  [-2.62, 0.0, 2.62, 0.6], [-1.6, 1.70, -2.93, -Math.PI / 2], [1.45, 0.0, 2.7, 2.0], [2.78, 0.0, 2.86, 2.6],
  [2.2, 0.0, -2.65, 1.2], [-1.35, 0.006, 1.35, -0.6]
];
const ducks = [];
for (let i = 0; i < 8; i++) {
  const d = new THREE.Group(); CUR.add(d);
  D(new THREE.SphereGeometry(0.05, 14, 10).scale(1.35, 0.9, 1), M.duck, 0, 0.045, 0, d);
  D(new THREE.SphereGeometry(0.032, 12, 10), M.duck, 0.045, 0.1, 0, d);
  D(new THREE.ConeGeometry(0.012, 0.03, 8).rotateZ(-Math.PI / 2), M.beak, 0.085, 0.095, 0, d);
  for (const z of [-0.017, 0.017]) D(new THREE.SphereGeometry(0.005, 6, 6), M.black, 0.068, 0.11, z, d);
  D(new THREE.ConeGeometry(0.02, 0.04, 8).rotateZ(Math.PI / 2 + 0.5), M.duck, -0.07, 0.07, 0, d);
  makeIA(d, { onClick: () => { sfx.squeak(); const y0 = d.position.y; tween(0.3, k => d.position.y = y0 + Math.sin(k * Math.PI) * 0.06); } });
  ducks.push(d);
}

/* =========================================================
   TUBA 2 – LABOR (z −9 … −3)
   ========================================================= */
CUR = roomG[1];
for (const z of [-4.5, -7.5]) D(new THREE.PlaneGeometry(1.3, 0.32).rotateX(Math.PI / 2), new THREE.MeshBasicMaterial({ color: 0xf6fbff }), 0, 2.99, z);
for (const x of [-1.8, 1.8]) D(new THREE.PlaneGeometry(0.32, 1.3).rotateX(Math.PI / 2), new THREE.MeshBasicMaterial({ color: 0xf6fbff }), x, 2.99, -6);
// laborilaud (ida)
S(boxUV(0.67, 0.05, 2.7, 0.6), M.labTop, 2.635, 0.875, -5.3);
S(new THREE.BoxGeometry(0.6, 0.85, 2.6), M.labCab, 2.66, 0.425, -5.3);
for (let i = 0; i < 4; i++) { const z = -4.3 - i * 0.65; D(new THREE.PlaneGeometry(0.6, 0.75).rotateY(-Math.PI / 2), new THREE.MeshLambertMaterial({ color: 0xcfd8dc }), 2.355, 0.44, z); D(new THREE.BoxGeometry(0.02, 0.02, 0.12), M.darkMetal, 2.34, 0.7, z); }
// kolb
const flask = new THREE.Group(); flask.position.set(2.6, 0.9, -5.05); CUR.add(flask);
D(new THREE.CylinderGeometry(0.09, 0.1, 0.02, 20), M.darkMetal, 0, 0.01, 0, flask);
for (let i = 0; i < 3; i++) { const a = i * 2.094; rod(V3(Math.cos(a) * 0.085, 0.02, Math.sin(a) * 0.085), V3(Math.cos(a) * 0.08, 0.1, Math.sin(a) * 0.08), 0.005, 0.005, M.darkMetal, flask); }
D(new THREE.TorusGeometry(0.08, 0.007, 8, 24).rotateX(Math.PI / 2), M.darkMetal, 0, 0.1, 0, flask);
D(new THREE.SphereGeometry(0.1, 24, 16), M.glass, 0, 0.14, 0, flask);
D(new THREE.CylinderGeometry(0.026, 0.03, 0.15, 16, 1, true), M.glass, 0, 0.29, 0, flask);
const liquid = D(new THREE.SphereGeometry(0.092, 20, 14), new THREE.MeshLambertMaterial({ color: 0x8888ff, emissive: 0x111111 }), 0, 0.14, 0, flask);
makeIA(flask, { accepts: it => String(it).startsWith('potion'), noAutoUse: true, onUse: h => { addDose(h.held.userData.ia.ck); }, onClick: () => notify('Kolb 🧪 Vala sinna pudelitest retsepti järgi.') });
const valve = new THREE.Group(); valve.position.set(2.43, 0.94, -5.05); CUR.add(valve);
D(new THREE.TorusGeometry(0.03, 0.008, 8, 20).rotateY(Math.PI / 2), new THREE.MeshLambertMaterial({ color: 0xd32f2f }), 0, 0, 0, valve);
D(new THREE.BoxGeometry(0.008, 0.06, 0.006), new THREE.MeshLambertMaterial({ color: 0xd32f2f }), 0, 0, 0, valve);
D(new THREE.CylinderGeometry(0.008, 0.008, 0.16, 8).rotateZ(Math.PI / 2), M.darkMetal, 0.08, 0, 0, valve);
makeIA(valve, { onClick: drainFlask });
const flaskInfo = new CanvasPlane(0.24, 0.07, 1000); flaskInfo.mesh.position.set(2.31, 0.95, -5.35); flaskInfo.mesh.rotation.y = -Math.PI / 2; CUR.add(flaskInfo.mesh);
const recipeCard = new CanvasPlane(0.5, 0.36, 1200, false, true); recipeCard.mesh.position.set(2.965, 1.55, -5.05); recipeCard.mesh.rotation.y = -Math.PI / 2; CUR.add(recipeCard.mesh);
S(new THREE.BoxGeometry(0.01, 0.38, 0.52), M.woodDark, 2.985, 1.55, -5.05);
makeIA(recipeCard.mesh, { scaleHover: true, onClick: () => notify(st.recipeRead ? 'Retsept 🧪 – vala kolbi täpselt nii palju.' : 'Kogused on kirjutatud nii pisikeselt, et neid ei saa lugeda… 🔍') });
// vitriin (kristall)
const vit = new THREE.Group(); vit.position.set(2.62, 0.9, -6.2); CUR.add(vit);
S(new THREE.BoxGeometry(0.42, 0.02, 0.42), M.labCab, 0, 0.01, 0, vit);
S(new THREE.BoxGeometry(0.42, 0.02, 0.42), M.labCab, 0, 0.41, 0, vit);
S(new THREE.BoxGeometry(0.01, 0.38, 0.42), M.glass, 0.205, 0.21, 0, vit);
for (const z of [-0.205, 0.205]) S(new THREE.BoxGeometry(0.42, 0.38, 0.01), M.glass, 0, 0.21, z, vit);
for (const [x, z] of [[-0.205, -0.205], [-0.205, 0.205], [0.205, -0.205], [0.205, 0.205]]) D(new THREE.BoxGeometry(0.015, 0.4, 0.015), M.darkMetal, x, 0.21, z, vit);
const vitDoor = new THREE.Group(); vitDoor.position.set(-0.21, 0, -0.21); vit.add(vitDoor);
S(new THREE.BoxGeometry(0.01, 0.38, 0.42), M.glass, 0, 0.21, 0.21, vitDoor);
const vitKnob = new THREE.Group(); vitKnob.position.set(-0.015, 0.21, 0.38); vitDoor.add(vitKnob); D(new THREE.BoxGeometry(0.025, 0.06, 0.025), M.brass, 0, 0, 0, vitKnob);
makeIA(vitKnob, { onClick: () => openSlide(vitKnob), lockedMsg: 'Vitriin avaneb ainult õige võlujoogiga 🧪', drag: { kind: 'hinge', center: () => vitDoor.getWorldPosition(V3(0, 0, 0)), get: () => vitDoor.rotation.y, set: v => vitDoor.rotation.y = v, min: -1.7, max: 0, open: -1.7, closed: 0 }, dragOk: () => st.potionDone });
{ const t = litPlane(0.3, 0.06, 900, (c, W, H) => { c.fillStyle = '#263238'; c.fillRect(0, 0, W, H); c.fillStyle = '#ffcc80'; c.font = `bold ${H * 0.5}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText('🔒 avaneb võlujoogiga', W / 2, H * 0.55); }); t.position.set(-0.215, 0.45, 0); t.rotation.y = -Math.PI / 2; vit.add(t); }
// kaalulaud (lääs)
S(boxUV(0.67, 0.05, 1.5, 0.6), M.labTop, -2.635, 0.825, -6.3);
for (const [x, z] of [[-2.35, -5.6], [-2.92, -5.6], [-2.35, -7.0], [-2.92, -7.0]]) S(new THREE.BoxGeometry(0.05, 0.8, 0.05), M.darkMetal, x, 0.4, z);
for (const z of [-6.56, -6.04]) S(new THREE.BoxGeometry(0.6, 0.15, 0.02), M.labCab, -2.62, 0.725, z);
S(new THREE.BoxGeometry(0.6, 0.02, 0.54), M.labCab, -2.62, 0.645, -6.3);
const labDrawer = new THREE.Group(); CUR.add(labDrawer);
S(new THREE.BoxGeometry(0.03, 0.14, 0.5), M.labCab, -2.315, 0.73, -6.3, labDrawer);
const labHandle = new THREE.Group(); labHandle.position.set(-2.29, 0.73, -6.3); labDrawer.add(labHandle); D(new THREE.BoxGeometry(0.03, 0.025, 0.16), M.darkMetal, 0, 0, 0, labHandle);
makeIA(labHandle, { onClick: () => openSlide(labHandle), lockedMsg: 'Sahtel on lukus – kaal peab olema tasakaalus ⚖️', drag: { kind: 'slide', dir: () => V3(1, 0, 0), get: () => labDrawer.position.x, set: v => labDrawer.position.x = v, min: 0, max: 0.35 }, dragOk: () => st.scaleDone });
S(new THREE.BoxGeometry(0.55, 0.012, 0.46), M.inner, -2.6, 0.665, -6.3, labDrawer);
for (const z of [-6.53, -6.07]) S(new THREE.BoxGeometry(0.55, 0.1, 0.012), M.inner, -2.6, 0.71, z, labDrawer);
S(new THREE.BoxGeometry(0.012, 0.1, 0.46), M.inner, -2.88, 0.71, -6.3, labDrawer);
// kaal
const scaleG = new THREE.Group(); scaleG.position.set(-2.62, 0.85, -6.3); CUR.add(scaleG);
S(new THREE.CylinderGeometry(0.07, 0.09, 0.03, 20), M.brass, 0, 0.015, 0, scaleG);
S(new THREE.CylinderGeometry(0.012, 0.015, 0.44, 10), M.brass, 0, 0.24, 0, scaleG);
const beamG = new THREE.Group(); beamG.position.y = 0.46; scaleG.add(beamG);
D(new THREE.BoxGeometry(0.02, 0.02, 0.6), M.brass, 0, 0, 0, beamG);
D(new THREE.ConeGeometry(0.012, 0.08, 8), M.darkMetal, 0, 0.05, 0, beamG);
const pans = [0.27, -0.27].map(z0 => {
  const g = new THREE.Group(); g.userData.z0 = z0; scaleG.add(g);
  S(new THREE.CylinderGeometry(0.11, 0.1, 0.012, 24), M.brass, 0, 0, 0, g);
  for (let i = 0; i < 3; i++) { const a = i * 2.094 + 0.5; rod(V3(Math.cos(a) * 0.1, 0.006, Math.sin(a) * 0.1), V3(0, 0.22, 0), 0.002, 0.002, M.darkMetal, g); }
  return g;
});
const [panL, panR] = pans;
{ // salapurk vasakul kausil
  D(new THREE.CylinderGeometry(0.045, 0.045, 0.11, 18), new THREE.MeshLambertMaterial({ color: 0x4a2a6a }), 0, 0.061, 0, panL);
  D(new THREE.CylinderGeometry(0.048, 0.048, 0.02, 18), M.darkMetal, 0, 0.125, 0, panL);
  const l = litPlane(0.07, 0.06, 1600, (c, W, H) => { c.fillStyle = '#f2e6c8'; c.fillRect(0, 0, W, H); c.fillStyle = '#3b2a16'; c.font = `bold ${H * 0.6}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText('? kg', W / 2, H * 0.55); });
  l.position.set(0.047, 0.06, 0); l.rotation.y = Math.PI / 2; panL.add(l);
}
let scaleTheta = 0.2;
// laser
const laserEm = new THREE.Group(); CUR.add(laserEm);
S(new THREE.BoxGeometry(0.22, 0.14, 0.14), M.darkMetal, 0.04, 0, 0, laserEm);
D(new THREE.CylinderGeometry(0.03, 0.035, 0.08, 16).rotateZ(Math.PI / 2), M.metal, 0.18, 0, 0, laserEm);
D(new THREE.CircleGeometry(0.022, 16).rotateY(Math.PI / 2), new THREE.MeshBasicMaterial({ color: 0x330000 }), 0.221, 0, 0, laserEm);
D(new THREE.BoxGeometry(0.06, 0.02, 0.06), M.black, 0.03, 0.075, 0, laserEm);
const emCrystal = D(new THREE.OctahedronGeometry(0.035), new THREE.MeshLambertMaterial({ color: 0xff3d6e, emissive: 0x99102e }), 0.03, 0.11, 0, laserEm); emCrystal.visible = false;
{ const t = litPlane(0.2, 0.05, 900, (c, W, H) => { c.fillStyle = '#ffd54f'; c.fillRect(0, 0, W, H); c.fillStyle = '#111'; c.font = `bold ${H * 0.55}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText('⚠ LASER', W / 2, H * 0.55); }); t.position.set(0.04, -0.1, 0.071); laserEm.add(t); }
makeIA(laserEm, { accepts: 'crystal', onUse: useCrystal, onClick: () => { if (!st.crystalIn) { sfx.bad(); notify('Laserist on kristall puudu 💎'); } }, enabled: () => !st.crystalIn });
const mirrors = [0, 1, 2].map(i => {
  const g = new THREE.Group(); CUR.add(g);
  S(new THREE.CylinderGeometry(0.05, 0.07, 1.0, 12), M.metal, 0, 0.5, 0, g);
  S(new THREE.CylinderGeometry(0.16, 0.18, 0.04, 20), M.darkMetal, 0, 0.02, 0, g);
  const head = new THREE.Group(); head.position.y = 1.2; g.add(head);
  D(new THREE.BoxGeometry(0.4, 0.3, 0.02), M.silver, 0, 0, 0, head);
  D(new THREE.BoxGeometry(0.42, 0.32, 0.012), M.darkMetal, 0, 0, -0.012, head);
  D(new THREE.CylinderGeometry(0.02, 0.02, 0.06, 8), M.darkMetal, 0, -0.18, 0, head);
  makeIA(head, { onClick: () => rotateMirror(i), enabled: () => !st.door[1] });
  return { g, head };
});
const receptor = new THREE.Group(); CUR.add(receptor);
S(new THREE.BoxGeometry(0.2, 0.2, 0.08), M.darkMetal, 0, 0, 0, receptor);
const recLens = D(new THREE.SphereGeometry(0.05, 16, 12), new THREE.MeshLambertMaterial({ color: 0x440000, emissive: 0x330000 }), 0, 0, 0.04, receptor);
const beamMat = new THREE.MeshBasicMaterial({ color: 0xff2a2a }), glowMat = new THREE.MeshBasicMaterial({ color: 0xff3030, transparent: true, opacity: 0.35, blending: THREE.AdditiveBlending, depthWrite: false });
const beamSegs = [];
for (let i = 0; i < 12; i++) {
  const g = new THREE.Group(); g.visible = false; CUR.add(g);
  D(new THREE.BoxGeometry(1, 0.012, 0.012), beamMat, 0, 0, 0, g); D(new THREE.BoxGeometry(1, 0.05, 0.05), glowMat, 0, 0, 0, g);
  beamSegs.push(g);
}
const beamDot = D(new THREE.SphereGeometry(0.03, 10, 8), glowMat, 0, 0, 0); beamDot.visible = false;
// riiulid ja purgid (lõunasein)
{
  const parts = [];
  for (const [x0, x1] of [[-2.8, -1.0], [1.0, 2.8]]) for (const y of [1.3, 1.7]) {
    S(new THREE.BoxGeometry(x1 - x0, 0.03, 0.26), M.labCab, (x0 + x1) / 2, y, -3.15);
    for (let x = x0 + 0.08; x < x1 - 0.06; x += 0.1 + rand() * 0.06) { const h = 0.08 + rand() * 0.12, r = 0.025 + rand() * 0.02; parts.push([new THREE.CylinderGeometry(r, r, h, 10).translate(x, y + 0.015 + h / 2, -3.17 + (rand() - 0.5) * 0.06), pick(['#7cb342', '#e53935', '#8e24aa', '#29b6f6', '#ffb300', '#f5f5f5', '#6d4c41', '#26a69a'])]); }
  }
  CUR.add(coloredMerge(parts));
}
{ // plakatid
  const p = litPlane(1.0, 0.7, 500, (c, W, H) => {
    c.fillStyle = '#f5f5f0'; c.fillRect(0, 0, W, H); c.fillStyle = '#263238'; c.font = `bold ${H * 0.07}px ${FONT}`; c.textAlign = 'center'; c.fillText('PERIOODILISUSTABEL', W / 2, H * 0.1);
    const els = 'H He Li Be B C N O F Ne Na Mg Al Si P S Cl Ar K Ca Sc Ti V Cr Mn Fe Co Ni Cu Zn Ga Ge As Se Br Kr Rb Sr Y Zr Nb Mo Tc Ru Rh Pd Ag Cd In Sn Sb Te I Xe'.split(' ');
    const cols = ['#ef9a9a', '#ffcc80', '#fff59d', '#a5d6a7', '#80deea', '#90caf9', '#ce93d8'];
    els.forEach((e, i) => { const cx = i % 18, cy = Math.floor(i / 18); const x = W * 0.04 + cx * W * 0.051, y = H * 0.2 + cy * H * 0.2; c.fillStyle = cols[(cx + cy) % 7]; c.fillRect(x, y, W * 0.048, H * 0.18); c.fillStyle = '#263238'; c.font = `bold ${H * 0.06}px ${FONT}`; c.fillText(e, x + W * 0.024, y + H * 0.11); });
  });
  p.position.set(-2.975, 1.75, -8.1); p.rotation.y = Math.PI / 2; CUR.add(p);
  const s = litPlane(0.5, 0.35, 600, (c, W, H) => { c.fillStyle = '#ffd54f'; c.fillRect(0, 0, W, H); c.strokeStyle = '#111'; c.lineWidth = 10; c.strokeRect(8, 8, W - 16, H - 16); c.fillStyle = '#111'; c.textAlign = 'center'; c.font = `${H * 0.35}px ${FONT}`; c.fillText('⚠️', W / 2, H * 0.42); c.font = `bold ${H * 0.13}px ${FONT}`; c.fillText('ÄRA VAATA LASERISSE', W / 2, H * 0.78); });
  s.position.set(1.9, 2.0, -8.985); CUR.add(s);
}

/* =========================================================
   TUBA 3 – TÄHETORN (z −15 … −9)
   ========================================================= */
CUR = roomG[2];
S(new THREE.CircleGeometry(1.5, 48).rotateX(-Math.PI / 2), new THREE.MeshLambertMaterial({ map: roundRugTex }), 0, 0.004, -12);
{ // latern laes
  D(new THREE.CylinderGeometry(0.006, 0.006, 0.55, 6), M.black, 0, 2.72, -12);
  D(new THREE.CylinderGeometry(0.09, 0.12, 0.2, 6), new THREE.MeshLambertMaterial({ color: 0xffe0a0, emissive: 0xffb050 }), 0, 2.35, -12);
  D(new THREE.ConeGeometry(0.14, 0.1, 6), M.darkMetal, 0, 2.5, -12);
}
// planeedimobiil
const mobile = new THREE.Group(); mobile.position.set(-0.9, 2.25, -11.3); CUR.add(mobile);
D(new THREE.CylinderGeometry(0.004, 0.004, 0.75, 6), M.black, 0, 0.375, 0, mobile);
D(new THREE.SphereGeometry(0.09, 16, 12), new THREE.MeshBasicMaterial({ color: 0xffd25a }), 0, 0, 0, mobile);
[[0.25, 0.03, '#b0bec5'], [0.38, 0.045, '#ffab91'], [0.52, 0.05, '#42a5f5'], [0.66, 0.04, '#ef5350'], [0.85, 0.08, '#ffcc80']].forEach(([r, s, c], i) => {
  const a = i * 1.3; D(new THREE.SphereGeometry(s, 12, 10), new THREE.MeshLambertMaterial({ color: c }), Math.cos(a) * r, -0.05 - i * 0.04, Math.sin(a) * r, mobile);
  D(new THREE.BoxGeometry(r, 0.004, 0.004), M.black, Math.cos(a) * r / 2, -0.05 - i * 0.04, Math.sin(a) * r / 2, mobile).rotation.y = -a;
});
// katuseaken teleskoobi kohal
D(new THREE.CircleGeometry(0.45, 32).rotateX(Math.PI / 2), new THREE.MeshBasicMaterial({ map: canvasTex(256, 256, (c, W, H) => { c.fillStyle = '#0a1240'; c.fillRect(0, 0, W, H); for (let i = 0; i < 90; i++) { c.fillStyle = `rgba(255,255,255,${0.4 + tr() * 0.6})`; c.beginPath(); c.arc(tr() * W, tr() * H, tr() * 2 + 0.5, 0, 7); c.fill(); } }, false) }), 2.55, 2.995, -12.5);
D(new THREE.TorusGeometry(0.46, 0.03, 8, 32).rotateX(Math.PI / 2), M.brass, 2.55, 2.98, -12.5);
// teleskoop
const tele = new THREE.Group(); tele.position.set(2.2, 0, -12.2); CUR.add(tele);
for (let i = 0; i < 3; i++) { const a = i * 2.094 + 0.5; rod(V3(Math.cos(a) * 0.35, 0, Math.sin(a) * 0.35), V3(0, 1.0, 0), 0.015, 0.015, M.woodDark, tele); }
D(new THREE.SphereGeometry(0.045, 12, 8), M.brass, 0, 1.0, 0, tele);
rod(V3(0, 1.0, 0), V3(-0.05, 1.25, 0.05), 0.015, 0.015, M.brass, tele);
rod(V3(-0.42, 1.15, 0.22), V3(0.3, 1.75, -0.28), 0.045, 0.075, M.brass, tele);
rod(V3(-0.47, 1.11, 0.25), V3(-0.4, 1.165, 0.21), 0.02, 0.025, M.darkMetal, tele);
const teleLens = D(new THREE.CylinderGeometry(0.03, 0.03, 0.012, 16), new THREE.MeshLambertMaterial({ color: 0x9fd8ff, emissive: 0x204060 }), -0.475, 1.105, 0.255, tele); teleLens.visible = false;
makeIA(tele, { accepts: 'lens', onUse: useLens, onClick: () => { if (!st.lensIn) { sfx.bad(); notify('Teleskoobi okulaarist on lääts puudu 🔍'); } else viewTelescope(); } });
// vaas sambal
S(new THREE.CylinderGeometry(0.16, 0.18, 0.9, 16), M.labCab, -2.5, 0.45, -9.9);
S(new THREE.BoxGeometry(0.4, 0.04, 0.4), M.labCab, -2.5, 0.92, -9.9);
const shards = [];
// varjumõistatus
S(new THREE.CylinderGeometry(0.4, 0.4, 0.04, 24), M.woodDark, -2.35, 0.76, -12.1);
S(new THREE.CylinderGeometry(0.04, 0.06, 0.74, 10), M.woodDark, -2.35, 0.37, -12.1);
const turntable = new THREE.Group(); turntable.position.set(-2.5, 0.78, -12.1); CUR.add(turntable);
D(new THREE.CylinderGeometry(0.12, 0.13, 0.03, 24), M.brass, 0, 0.015, 0, turntable);
{
  const parts = []; const r = mulberry32(99);
  for (let i = 0; i < 9; i++) { const a = V3((r() - 0.5) * 0.18, 0.05 + r() * 0.3, (r() - 0.5) * 0.18), b = V3((r() - 0.5) * 0.18, 0.05 + r() * 0.3, (r() - 0.5) * 0.18); const d = b.clone().sub(a), L = d.length(); const g = new THREE.CylinderGeometry(0.006, 0.006, L, 6); g.applyQuaternion(new THREE.Quaternion().setFromUnitVectors(UP, d.normalize())); g.translate((a.x + b.x) / 2, (a.y + b.y) / 2, (a.z + b.z) / 2); parts.push([g, '#222']); }
  parts.push([new THREE.CylinderGeometry(0.008, 0.008, 0.08, 6).translate(0, 0.07, 0), '#222']);
  turntable.add(coloredMerge(parts));
}
makeIA(turntable, { onClick: rotateTurntable });
{
  const lampG = new THREE.Group(); lampG.position.set(-1.98, 0.78, -12.1); CUR.add(lampG);
  D(new THREE.CylinderGeometry(0.06, 0.07, 0.02, 16), M.darkMetal, 0, 0.01, 0, lampG);
  D(new THREE.CylinderGeometry(0.008, 0.008, 0.25, 8), M.darkMetal, 0, 0.13, 0, lampG);
  const sh = D(new THREE.ConeGeometry(0.07, 0.12, 16, 1, true), M.darkMetal, -0.03, 0.27, 0, lampG); sh.rotation.z = -Math.PI / 2;
  D(new THREE.SphereGeometry(0.03, 10, 8), new THREE.MeshBasicMaterial({ color: 0xfff2c0 }), -0.02, 0.27, 0, lampG);
}
const shadowCanvas = new CanvasPlane(0.9, 0.9, 500);
shadowCanvas.mesh.position.set(-2.965, 1.3, -12.1); shadowCanvas.mesh.rotation.y = Math.PI / 2; CUR.add(shadowCanvas.mesh);
// muusikakarp
S(new THREE.CylinderGeometry(0.3, 0.3, 0.03, 24), M.woodDark, 2.3, 0.75, -9.85);
S(new THREE.CylinderGeometry(0.03, 0.05, 0.74, 10), M.woodDark, 2.3, 0.37, -9.85);
const musicBox = new THREE.Group(); musicBox.position.set(2.3, 0.765, -9.85); musicBox.rotation.y = -Math.PI / 2; CUR.add(musicBox);
D(boxUV(0.24, 0.12, 0.16, 0.3), M.wood, 0, 0.06, 0, musicBox);
const mbLid = new THREE.Group(); mbLid.position.set(0, 0.12, -0.08); musicBox.add(mbLid);
D(boxUV(0.25, 0.02, 0.17, 0.3), M.wood, 0, 0.01, 0.085, mbLid);
const mbGem = D(new THREE.SphereGeometry(0.022, 14, 10), new THREE.MeshBasicMaterial({ color: 0x333333 }), 0, 0.06, 0.085, musicBox);
D(new THREE.CylinderGeometry(0.005, 0.005, 0.05, 6).rotateZ(Math.PI / 2), M.brass, 0.14, 0.06, 0, musicBox);
D(new THREE.BoxGeometry(0.005, 0.04, 0.012), M.brass, 0.165, 0.06, 0, musicBox);
makeIA(musicBox, { onClick: playMelody });
// ksülofon
S(boxUV(1.3, 0.04, 0.8, 0.6), M.wood, -2.2, 0.73, -14.2);
for (const [x, z] of [[-2.8, -14.55], [-1.6, -14.55], [-2.8, -13.85], [-1.6, -13.85]]) S(new THREE.BoxGeometry(0.05, 0.71, 0.05), M.woodDark, x, 0.355, z);
const XY_COLS = ['#e53935', '#fb8c00', '#fdd835', '#43a047', '#29b6f6', '#1e3fd8', '#8e24aa'];
for (const z of [-14.34, -14.16]) D(new THREE.BoxGeometry(1.2, 0.025, 0.025), M.woodDark, -2.2, 0.762, z);
const xyBars = XY_COLS.map((col, i) => {
  const L = 0.34 - i * 0.025, m = new THREE.Mesh(new THREE.BoxGeometry(0.12, 0.02, L), new THREE.MeshLambertMaterial({ color: col }));
  m.position.set(-2.68 + i * 0.16, 0.785, -14.25); CUR.add(m);
  makeIA(m, { poke: true, onClick: () => hitBar(i) });
  return m;
});
const xyBox = new THREE.Group(); xyBox.position.set(-2.2, 0.75, -13.92); CUR.add(xyBox);
S(new THREE.BoxGeometry(0.3, 0.012, 0.16), M.inner, 0, 0.006, 0, xyBox);
for (const x of [-0.15, 0.15]) S(new THREE.BoxGeometry(0.012, 0.09, 0.16), M.wood, x, 0.045, 0, xyBox);
S(new THREE.BoxGeometry(0.3, 0.09, 0.012), M.wood, 0, 0.045, -0.08, xyBox);
S(new THREE.BoxGeometry(0.31, 0.012, 0.17), M.wood, 0, 0.096, 0, xyBox);
const xyDrawer = new THREE.Group(); xyBox.add(xyDrawer);
S(new THREE.BoxGeometry(0.29, 0.08, 0.012), M.wood, 0, 0.045, 0.08, xyDrawer);
const xyKnob = new THREE.Group(); xyKnob.position.set(0, 0.045, 0.095); xyDrawer.add(xyKnob); D(new THREE.SphereGeometry(0.014, 8, 6), M.brass, 0, 0, 0, xyKnob);
makeIA(xyKnob, { onClick: () => openSlide(xyKnob), lockedMsg: 'Karbike on lukus – mängi õige meloodia 🎶', drag: { kind: 'slide', dir: () => V3(0, 0, 1), get: () => xyDrawer.position.z, set: v => xyDrawer.position.z = v, min: 0, max: 0.11 }, dragOk: () => st.melodyDone });
const xyCard = new THREE.Mesh(new THREE.PlaneGeometry(0.12, 0.08).rotateX(-Math.PI / 2), M.paper); xyCard.position.set(0, 0.014, 0.0); xyDrawer.add(xyCard);
makeIA(xyCard, { onClick: () => { st.pianoRead = true; openReader('🎹 Kaart', `🎹 = ${P.dPiano}\n\n(Seifi teine number)`); }, enabled: () => st.melodyDone });
// seif
const safeG = new THREE.Group(); safeG.position.set(2.25, 0, -14.45); CUR.add(safeG);
S(new THREE.BoxGeometry(0.6, 0.04, 0.5), M.safe, 0, 0.02, 0, safeG);
S(new THREE.BoxGeometry(0.6, 0.04, 0.5), M.safe, 0, 0.73, 0, safeG);
for (const x of [-0.28, 0.28]) S(new THREE.BoxGeometry(0.04, 0.75, 0.5), M.safe, x, 0.375, 0, safeG);
S(new THREE.BoxGeometry(0.6, 0.75, 0.04), M.safe, 0, 0.375, -0.23, safeG);
D(new THREE.BoxGeometry(0.52, 0.02, 0.42), M.inner, 0, 0.05, 0, safeG);
const safeDoor = new THREE.Group(); safeDoor.position.set(-0.3, 0, 0.25); safeG.add(safeDoor);
S(new THREE.BoxGeometry(0.6, 0.75, 0.04), M.safe, 0.3, 0.375, 0.02, safeDoor);
{ const t = litPlane(0.4, 0.09, 900, (c, W, H) => { c.fillStyle = '#d8c48a'; c.fillRect(0, 0, W, H); c.fillStyle = '#2a2a2a'; c.textAlign = 'center'; c.textBaseline = 'middle'; c.font = `${H * 0.6}px ${FONT}`; ['🔭', '🎹', '🕯️'].forEach((e, i) => c.fillText(e, W * (0.2 + i * 0.3), H * 0.55)); }); t.position.set(0.3, 0.6, 0.041); safeDoor.add(t); }
const safeWheels = [0, 1, 2].map(i => {
  const cp = new CanvasPlane(0.09, 0.12, 900); cp.mesh.position.set(0.3 + (i - 1) * 0.12, 0.46, 0.042); safeDoor.add(cp.mesh);
  makeIA(cp.mesh, { scaleHover: true, onClick: () => { st.safeWheel[i] = (st.safeWheel[i] + 1) % 10; drawWheel(i); tone(1400, .03, 0, 'square', .06); }, enabled: () => !st.safeOpen });
  return cp;
});
const safeHandle = new THREE.Group(); safeHandle.position.set(0.3, 0.25, 0.05); safeDoor.add(safeHandle);
D(new THREE.CylinderGeometry(0.05, 0.05, 0.02, 20).rotateX(Math.PI / 2), M.brass, 0, 0, 0, safeHandle);
for (const a of [0, Math.PI / 2]) D(new THREE.BoxGeometry(0.16, 0.018, 0.018), M.brass, 0, 0, 0.02, safeHandle).rotation.z = a;
makeIA(safeHandle, { onClick: () => { if (!st.safeOpen) trySafe(); else openSlide(safeHandle); }, drag: { kind: 'hinge', center: () => safeDoor.getWorldPosition(V3(0, 0, 0)), get: () => safeDoor.rotation.y, set: v => safeDoor.rotation.y = v, min: -1.9, max: 0, open: -1.9, closed: 0 }, dragOk: () => st.safeOpen });
// väljapääsu lukk ja silt
const exitLock = new THREE.Group(); exitLock.position.set(0.86, 1.25, 0.03); DOORS[2].pivot.add(exitLock);
D(new THREE.BoxGeometry(0.1, 0.14, 0.012), M.gold, 0, 0, 0, exitLock);
D(new THREE.CircleGeometry(0.012, 12), M.black, 0, 0.015, 0.007, exitLock);
D(new THREE.PlaneGeometry(0.008, 0.026), M.black, 0, -0.005, 0.007, exitLock);
const goldInLock = D(new THREE.TorusGeometry(0.022, 0.006, 8, 16), M.gold, 0, 0.015, 0.04, exitLock); goldInLock.visible = false;
makeIA(exitLock, { accepts: 'goldkey', onUse: useGoldKey, onClick: () => { sfx.bad(); notify('Suur uks on lukus. Vaja on kuldvõtit 🗝️'); }, enabled: () => !st.door[2] });
{ // küünlad ja kriiditahvel
  for (const [x, y, z] of [[-1.7, 0.75, -14.5], [2.05, 0.765, -9.65], [-2.1, 0.78, -12.4]]) { D(new THREE.CylinderGeometry(0.02, 0.022, 0.12, 10), new THREE.MeshLambertMaterial({ color: 0xf5efe0 }), x, y + 0.06, z); D(new THREE.ConeGeometry(0.01, 0.03, 8), M.flame, x, y + 0.135, z); }
  const b = litPlane(1.4, 0.9, 400, (c, W, H) => {
    c.fillStyle = '#2b3a2f'; c.fillRect(0, 0, W, H); c.strokeStyle = '#6b4a2e'; c.lineWidth = 24; c.strokeRect(0, 0, W, H);
    c.strokeStyle = 'rgba(255,255,255,0.75)'; c.fillStyle = 'rgba(255,255,255,0.8)'; c.lineWidth = 3; c.font = `${H * 0.07}px Georgia, serif`;
    c.fillText('Tähtkujud 🌌', W * 0.08, H * 0.15); c.fillText('E = mc²', W * 0.62, H * 0.2); c.fillText('🔭 + 🎹 + 🕯️ = 🗝️', W * 0.08, H * 0.88);
    for (let k = 0; k < 3; k++) { c.beginPath(); let x = W * (0.15 + k * 0.28), y = H * 0.45; c.moveTo(x, y); for (let i = 0; i < 4; i++) { x += (tr() - 0.3) * 80; y += (tr() - 0.5) * 70; c.lineTo(x, y); c.fillRect(x - 3, y - 3, 6, 6); } c.stroke(); }
  });
  b.position.set(2.975, 1.65, -11.0); b.rotation.y = -Math.PI / 2; CUR.add(b);
}

/* =========================================================
   TUBA 4 – KELDER (z −21 … −15)
   ========================================================= */
CUR = roomG[3];
for (const y of [2.4, 2.56]) D(new THREE.CylinderGeometry(0.06, 0.06, 6, 10).rotateX(Math.PI / 2), M.darkMetal, 2.85, y, -18);
D(new THREE.CylinderGeometry(0.05, 0.05, 6, 10).rotateX(Math.PI / 2), M.metal, -2.88, 2.5, -18);
for (const [x, z] of [[1.6, -15.7], [2.3, -15.9], [-2.3, -20.3], [-1.6, -20.5]]) {
  S(new THREE.CylinderGeometry(0.3, 0.3, 0.9, 16), M.woodDark, x, 0.45, z);
  for (const y of [0.2, 0.7]) D(new THREE.TorusGeometry(0.302, 0.015, 6, 24).rotateX(Math.PI / 2), M.darkMetal, x, y, z);
}
const VALVE_COLS = ['#e53935', '#43a047', '#1e88e5'];
const gauges = [], valves = [0, 1, 2].map(i => {
  const z = -17 - i;
  rod(V3(2.97, 1.25, z), V3(2.62, 1.25, z), 0.035, 0.035, M.darkMetal);
  rod(V3(2.97, 1.7, z), V3(2.9, 1.7, z), 0.02, 0.02, M.darkMetal);
  const g = new THREE.Group(); g.position.set(2.6, 1.25, z); CUR.add(g);
  D(new THREE.TorusGeometry(0.12, 0.018, 8, 24).rotateY(Math.PI / 2), new THREE.MeshLambertMaterial({ color: VALVE_COLS[i] }), 0, 0, 0, g);
  for (const a of [0, Math.PI / 3, 2 * Math.PI / 3]) D(new THREE.BoxGeometry(0.012, 0.24, 0.012), M.darkMetal, 0, 0, 0, g).rotation.x = a;
  makeIA(g, { onClick: () => turnValve(i) });
  const gp = new CanvasPlane(0.22, 0.22, 800, false, true); gp.mesh.position.set(2.89, 1.7, z); gp.mesh.rotation.y = -Math.PI / 2; CUR.add(gp.mesh); gauges.push(gp);
  return g;
});
const chalk = new CanvasPlane(1.3, 0.7, 500, false, true); chalk.mesh.position.set(-2.965, 1.55, -18); chalk.mesh.rotation.y = Math.PI / 2; CUR.add(chalk.mesh);
function drawGauge(i){
  gauges[i].draw((c, W, H) => {
    c.fillStyle = '#f2ead8'; c.beginPath(); c.arc(W / 2, H / 2, W / 2 - 4, 0, 7); c.fill(); c.lineWidth = 10; c.strokeStyle = VALVE_COLS[i]; c.stroke();
    c.fillStyle = '#222'; c.font = `bold ${H * 0.13}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle';
    for (let n = 0; n <= 5; n++) { const a = Math.PI * 0.75 + n * Math.PI * 1.5 / 5; c.fillText(String(n), W / 2 + Math.cos(a) * W * 0.33, H / 2 + Math.sin(a) * H * 0.33); }
    const a = Math.PI * 0.75 + st.valves[i] * Math.PI * 1.5 / 5; c.strokeStyle = '#c62828'; c.lineWidth = 6; c.beginPath(); c.moveTo(W / 2, H / 2); c.lineTo(W / 2 + Math.cos(a) * W * 0.27, H / 2 + Math.sin(a) * H * 0.27); c.stroke();
    c.fillStyle = '#333'; c.beginPath(); c.arc(W / 2, H / 2, 8, 0, 7); c.fill();
  });
}
function drawChalk(){
  chalk.draw((c, W, H) => {
    c.fillStyle = 'rgba(255,255,255,0.85)'; c.font = `bold ${H * 0.13}px Georgia, serif`; c.textAlign = 'center'; c.textBaseline = 'middle';
    c.fillText('Õige surve:', W / 2, H * 0.2);
    P.valves.forEach((v, i) => { const x = W * (0.25 + i * 0.25); c.fillStyle = VALVE_COLS[i]; c.beginPath(); c.arc(x, H * 0.55, H * 0.1, 0, 7); c.fill(); c.fillStyle = 'rgba(255,255,255,0.9)'; c.font = `bold ${H * 0.22}px Georgia, serif`; c.fillText(String(v), x, H * 0.82); });
  });
}
function turnValve(i){
  if (st.door[3]) return;
  st.valves[i] = (st.valves[i] + 1) % 6; noise(0.3, 0.15, 1500); drawGauge(i);
  const g = valves[i], r0 = g.rotation.x; tween(0.3, k => g.rotation.x = r0 + ease(k) * Math.PI / 3);
  if (st.valves.every((v, j) => v === P.valves[j])) { delay(0.4, () => { sfx.unlock(); notify('Surve on õige! ⚙️', 3); requestDoor(3); }); }
}

/* =========================================================
   TUBA 5 – KASVUHOONE (z −27 … −21)
   ========================================================= */
CUR = roomG[4];
S(boxUV(0.67, 0.05, 1.8, 0.6), M.wood, -2.635, 0.825, -23.5);
for (const [x, z] of [[-2.35, -22.65], [-2.92, -22.65], [-2.35, -24.35], [-2.92, -24.35]]) S(new THREE.BoxGeometry(0.05, 0.8, 0.05), M.woodDark, x, 0.4, z);
for (const z of [-23.76, -23.24]) S(new THREE.BoxGeometry(0.6, 0.15, 0.02), M.woodDark, -2.62, 0.725, z);
S(new THREE.BoxGeometry(0.6, 0.02, 0.54), M.woodDark, -2.62, 0.645, -23.5);
const potDrawer = new THREE.Group(); CUR.add(potDrawer);
S(new THREE.BoxGeometry(0.03, 0.14, 0.5), M.wood, -2.315, 0.73, -23.5, potDrawer);
S(new THREE.BoxGeometry(0.55, 0.012, 0.46), M.inner, -2.6, 0.665, -23.5, potDrawer);
for (const z of [-23.73, -23.27]) S(new THREE.BoxGeometry(0.55, 0.1, 0.012), M.inner, -2.6, 0.71, z, potDrawer);
S(new THREE.BoxGeometry(0.012, 0.1, 0.46), M.inner, -2.88, 0.71, -23.5, potDrawer);
const potHandle = new THREE.Group(); potHandle.position.set(-2.29, 0.73, -23.5); potDrawer.add(potHandle);
D(new THREE.BoxGeometry(0.03, 0.025, 0.16), M.brass, 0, 0, 0, potHandle);
makeIA(potHandle, { onClick: () => openSlide(potHandle), drag: { kind: 'slide', dir: () => V3(1, 0, 0), get: () => potDrawer.position.x, set: v => potDrawer.position.x = v, min: 0, max: 0.35 }, dragOk: () => true });
for (let i = 0; i < 5; i++) { const x = -2.85 + i * 0.13; D(new THREE.CylinderGeometry(0.045, 0.035, 0.08, 12), M.pot, x, 0.89, -22.8); D(new THREE.SphereGeometry(0.05, 8, 6), M.leaf, x, 0.96, -22.8); }
const soilMat = new THREE.MeshLambertMaterial({ color: 0x3e2a1c });
const pot = new THREE.Group(); pot.position.set(1.2, 0, -24.8); CUR.add(pot);
S(new THREE.CylinderGeometry(0.32, 0.24, 0.5, 20), M.pot, 0, 0.25, 0, pot);
D(new THREE.CircleGeometry(0.3, 20).rotateX(-Math.PI / 2), soilMat, 0, 0.47, 0, pot);
const plant = new THREE.Group(); plant.position.y = 0.47; pot.add(plant);
D(new THREE.CylinderGeometry(0.018, 0.03, 1.2, 8).translate(0, 0.6, 0), M.leaf, 0, 0, 0, plant);
for (let i = 0; i < 8; i++) { const l = D(new THREE.SphereGeometry(0.09, 8, 6).scale(1.6, 0.25, 0.8), M.leaf, Math.cos(i * 2.4) * 0.08, 0.2 + i * 0.12, Math.sin(i * 2.4) * 0.08, plant); l.rotation.y = -i * 2.4; l.rotation.z = 0.3; }
const flower = new THREE.Group(); flower.position.y = 1.25; plant.add(flower);
const petals = [];
for (let i = 0; i < 7; i++) { const a = i * Math.PI * 2 / 7; const pg = new THREE.Group(); pg.rotation.y = a; flower.add(pg); D(new THREE.SphereGeometry(0.07, 10, 8).scale(1.4, 0.3, 0.7), new THREE.MeshLambertMaterial({ color: 0xff6fa8 }), 0.09, 0, 0, pg); petals.push(pg); }
D(new THREE.SphereGeometry(0.04, 10, 8), new THREE.MeshLambertMaterial({ color: 0xffd23a }), 0, 0, 0, flower);
makeIA(pot, { accepts: it => it === 'seed' || it === 'can', noAutoUse: true, onUse: potUse, onClick: () => notify(st.seeded ? (st.grown ? 'Imeline lill! 🌸' : 'Taim tahab vett 💧') : 'Suur pott – siia võiks midagi istutada 🌱') });
S(new THREE.BoxGeometry(0.8, 0.04, 0.8), M.wood, 2.4, 0.73, -22.4);
for (const [x, z] of [[2.05, -22.05], [2.75, -22.05], [2.05, -22.75], [2.75, -22.75]]) S(new THREE.BoxGeometry(0.04, 0.71, 0.04), M.woodDark, x, 0.355, z);
for (let i = 0; i < 4; i++) { const z = -23.6 - i * 0.9; S(new THREE.CylinderGeometry(0.25, 0.2, 0.45, 14), M.pot, 2.65, 0.225, z); for (let k = 0; k < 6; k++) D(new THREE.SphereGeometry(0.14, 8, 6), M.leaf, 2.65 + Math.cos(k) * 0.12, 0.6 + (k % 3) * 0.1, z + Math.sin(k) * 0.12); }
function potUse(h){
  const it = h.held.userData.ia.item;
  if (it === 'seed') { consume(h); st.seeded = true; plant.visible = true; plant.scale.setScalar(0.08); sfx.ok(); notify('Seeme on mullas 🌱 Nüüd kasta!', 3); }
  else if (it === 'can') { if (!st.seeded) { sfx.bad(); notify('Enne istuta seeme 🌱'); return; } waterPlant(); }
}
function waterPlant(){
  if (st.grown) return;
  st.water++; noise(0.6, 0.15, 900, 'lowpass');
  const s0 = plant.scale.x, s1 = [0.08, 0.4, 0.7, 1.0][Math.min(3, st.water)];
  tween(1.2, k => plant.scale.setScalar(s0 + (s1 - s0) * ease(k)));
  if (st.water >= 3) { st.grown = true; delay(1.3, () => { sfx.reveal(); notify('Lill puhkes õide! 🌸 Seal sees on võti.', 3.5); items.greenkey.obj.visible = true; tween(1, k => petals.forEach(p => p.rotation.z = -0.9 * ease(k))); }); }
  else notify(`Kastetud 💧 (${st.water}/3)`);
}

/* =========================================================
   TUBA 6 – RAADIOTUBA (z −33 … −27)
   ========================================================= */
CUR = roomG[5];
S(new THREE.BoxGeometry(0.8, 0.04, 1.2), M.wood, 2.55, 0.78, -30);
for (const [x, z] of [[2.2, -29.45], [2.9, -29.45], [2.2, -30.55], [2.9, -30.55]]) S(new THREE.BoxGeometry(0.04, 0.76, 0.04), M.woodDark, x, 0.38, z);
const radio = new THREE.Group(); radio.position.set(2.62, 0.8, -30); radio.rotation.y = -Math.PI / 2; CUR.add(radio);
S(boxUV(0.56, 0.34, 0.24, 0.3), M.wood, 0, 0.17, 0, radio);
const grill = litPlane(0.22, 0.24, 600, (c, W, H) => { c.fillStyle = '#2b1e14'; c.fillRect(0, 0, W, H); c.strokeStyle = '#a07a4a'; c.lineWidth = 6; for (let y = 10; y < H; y += 18) { c.beginPath(); c.moveTo(6, y); c.lineTo(W - 6, y); c.stroke(); } });
grill.position.set(-0.14, 0.17, 0.121); radio.add(grill);
const dial = new CanvasPlane(0.26, 0.1, 1000, false, true); dial.mesh.position.set(0.13, 0.24, 0.121); radio.add(dial.mesh);
const radioLamp = D(new THREE.SphereGeometry(0.025, 12, 10), new THREE.MeshBasicMaterial({ color: 0x331100 }), 0.13, 0.36, 0, radio);
const knobs = [-1, 1].map((dir, k) => {
  const g = new THREE.Group(); g.position.set(0.07 + k * 0.12, 0.1, 0.125); radio.add(g);
  D(new THREE.CylinderGeometry(0.035, 0.035, 0.03, 16).rotateX(Math.PI / 2), M.darkMetal, 0, 0, 0, g);
  const lbl = litPlane(0.05, 0.03, 1200, (c, W, H) => { c.fillStyle = '#ffe9b0'; c.font = `bold ${H * 0.8}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText(dir < 0 ? '◀' : '▶', W / 2, H * 0.55); });
  lbl.position.set(0, 0, 0.017); g.add(lbl);
  makeIA(g, { onClick: () => tune(dir) });
  return g;
});
function drawDial(){
  dial.draw((c, W, H) => {
    c.fillStyle = '#f3e3b5'; c.fillRect(0, 0, W, H); c.strokeStyle = '#5a3d1a'; c.lineWidth = 3; c.fillStyle = '#5a3d1a'; c.font = `${H * 0.22}px ${FONT}`; c.textAlign = 'center';
    for (let f = 88; f <= 108; f += 2) { const x = W * (0.06 + (f - 88) / 20 * 0.88); c.beginPath(); c.moveTo(x, H * 0.55); c.lineTo(x, H * (f % 4 === 0 ? 0.3 : 0.42)); c.stroke(); if (f % 4 === 0) c.fillText(String(f), x, H * 0.85); }
    const x = W * (0.06 + (st.freq - 88) / 20 * 0.88); c.strokeStyle = '#d32f2f'; c.lineWidth = 5; c.beginPath(); c.moveTo(x, 4); c.lineTo(x, H - 4); c.stroke();
    c.fillStyle = '#d32f2f'; c.font = `bold ${H * 0.24}px ${FONT}`; c.textAlign = 'right'; c.fillText(`${st.freq} MHz`, W - 6, H * 0.22);
  });
}
const MORSE = ['-----', '.----', '..---', '...--', '....-', '.....', '-....', '--...', '---..', '----.'];
function tune(dir){
  st.freq = clamp(st.freq + dir, 88, 108); drawDial(); noise(0.12, 0.08, 2500);
  st.morseTok++;
  if (st.freq === P.freq) { notify('📻 Piiks-piiks… raadio saadab morset!', 3); playMorse(st.morseTok); } else radioLamp.material.color.set(0x331100);
}
function playMorse(tok){
  if (tok !== st.morseTok || mode !== 'play') return;
  let t = 0.4;
  for (const d of P.radioCode) {
    for (const s of MORSE[d]) {
      const len = s === '.' ? 0.15 : 0.45;
      delay(t, () => { if (tok !== st.morseTok) return; tone(720, len, 0, 'sine', 0.15); radioLamp.material.color.set(0xffd54f); });
      delay(t + len, () => { if (tok === st.morseTok) radioLamp.material.color.set(0x331100); });
      t += len + 0.15;
    }
    t += 0.7;
  }
  delay(t + 1.8, () => playMorse(tok));
}
const mirrorText = new CanvasPlane(1.1, 0.4, 600, false, true); mirrorText.mesh.position.set(-1.2, 1.7, -27.03); mirrorText.mesh.rotation.y = Math.PI; CUR.add(mirrorText.mesh);
function drawMirrorText(){
  mirrorText.draw((c, W, H) => {
    c.fillStyle = '#f5ead0'; c.fillRect(0, 0, W, H); c.strokeStyle = '#6b4a2e'; c.lineWidth = 10; c.strokeRect(0, 0, W, H);
    c.save(); if (!st.mirrorRead) { c.translate(W, 0); c.scale(-1, 1); }
    c.fillStyle = '#3b2a16'; c.font = `bold ${H * 0.2}px Georgia, serif`; c.textAlign = 'center'; c.textBaseline = 'middle';
    c.fillText('Professori jaam:', W / 2, H * 0.33); c.font = `bold ${H * 0.3}px Georgia, serif`; c.fillText(`${P.freq} MHz`, W / 2, H * 0.7);
    c.restore();
  });
}
{ const mt = litPlane(0.7, 0.9, 500, (c, W, H) => {
    c.fillStyle = '#f5f0e0'; c.fillRect(0, 0, W, H); c.fillStyle = '#222'; c.font = `bold ${H * 0.065}px ${FONT}`; c.textAlign = 'center'; c.fillText('MORSE: NUMBRID', W / 2, H * 0.08);
    MORSE.forEach((m, d) => { const y = H * (0.17 + d * 0.081); c.textAlign = 'left'; c.font = `bold ${H * 0.06}px ${FONT}`; c.fillText(String(d), W * 0.12, y); for (let k = 0; k < 5; k++) { const x = W * (0.32 + k * 0.12); if (m[k] === '.') { c.beginPath(); c.arc(x + W * 0.03, y - H * 0.015, H * 0.012, 0, 7); c.fill(); } else c.fillRect(x, y - H * 0.022, W * 0.09, H * 0.014); } });
    c.font = `${H * 0.035}px ${FONT}`; c.textAlign = 'center'; c.fillText('• = lühike piiks   ▬ = pikk piiks', W / 2, H * 0.97);
  });
  mt.position.set(-2.965, 1.6, -30.6); mt.rotation.y = Math.PI / 2; CUR.add(mt); }
S(new THREE.BoxGeometry(0.3, 0.03, 1.2), M.wood, -2.82, 1.2, -28.9);
for (let i = 0; i < 6; i++) D(new THREE.CylinderGeometry(0.12, 0.12, 0.01, 20).rotateZ(Math.PI / 2), new THREE.MeshLambertMaterial({ color: [0x111111, 0x8e24aa, 0x111111, 0xe53935, 0x111111, 0x1e88e5][i] }), -2.85, 1.335, -29.3 + i * 0.03);
const codeLock = new THREE.Group(); codeLock.position.set(1.05, 1.35, -32.97); CUR.add(codeLock);
S(new THREE.BoxGeometry(0.46, 0.34, 0.04), M.darkMetal, 0, 0, 0, codeLock);
{ const t = litPlane(0.4, 0.06, 900, (c, W, H) => { c.fillStyle = '#ffd54f'; c.font = `bold ${H * 0.6}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText('📻 RAADIOKOOD', W / 2, H * 0.55); }); t.position.set(0, 0.13, 0.021); codeLock.add(t); }
const codeWheels = [0, 1, 2].map(i => {
  const cp = new CanvasPlane(0.09, 0.12, 900); cp.mesh.position.set((i - 1) * 0.11 - 0.05, -0.02, 0.022); codeLock.add(cp.mesh);
  makeIA(cp.mesh, { scaleHover: true, onClick: () => { st.codeWheel[i] = (st.codeWheel[i] + 1) % 10; drawCodeWheel(i); tone(1400, .03, 0, 'square', .06); }, enabled: () => !st.door[5] });
  return cp;
});
function drawCodeWheel(i){ codeWheels[i].draw((c, W, H) => { c.fillStyle = '#1a1a1a'; roundRect(c, 3, 3, W - 6, H - 6, 10); c.fill(); c.strokeStyle = '#c9a13b'; c.lineWidth = 5; c.stroke(); c.fillStyle = '#f5f5f5'; c.font = `bold ${H * 0.6}px monospace`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText(String(st.codeWheel[i]), W / 2, H * 0.55); }); }
const codeOk = new CanvasPlane(0.09, 0.09, 900); codeOk.mesh.position.set(0.17, -0.02, 0.022); codeLock.add(codeOk.mesh);
codeOk.draw((c, W, H) => { c.fillStyle = '#2a7a35'; roundRect(c, 3, 3, W - 6, H - 6, 12); c.fill(); c.fillStyle = '#fff'; c.font = `bold ${H * 0.42}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText('OK', W / 2, H * 0.55); });
makeIA(codeOk.mesh, { scaleHover: true, onClick: () => { if (st.codeWheel.every((v, i) => v === P.radioCode[i])) { sfx.unlock(); notify('Kood õige! 📻', 3); requestDoor(5); } else { sfx.bad(); notify('Vale kood – kuula morset uuesti 🎧'); } }, enabled: () => !st.door[5] });
{ const ch = new THREE.Group(); ch.position.set(-2.1, 0, -31.6); ch.rotation.y = Math.PI / 3; CUR.add(ch);
  S(new THREE.BoxGeometry(0.7, 0.32, 0.65), M.fabric, 0, 0.16, 0, ch); S(new THREE.BoxGeometry(0.7, 0.65, 0.15), M.fabric, 0, 0.6, -0.27, ch); }

/* =========================================================
   TUBA 7 – MÄNGUTUBA (z −39 … −33)
   ========================================================= */
CUR = roomG[6];
const levers = [0, 1, 2, 3].map(i => {
  const z = -34.6 - i * 0.38;
  S(new THREE.BoxGeometry(0.04, 0.24, 0.14), M.darkMetal, 2.96, 1.2, z);
  const g = new THREE.Group(); g.position.set(2.93, 1.2, z); CUR.add(g);
  D(new THREE.CylinderGeometry(0.012, 0.012, 0.22, 8).translate(0, 0.11, 0), M.metal, 0, 0, 0, g);
  D(new THREE.SphereGeometry(0.035, 12, 10), new THREE.MeshLambertMaterial({ color: 0xe53935 }), 0, 0.22, 0, g);
  makeIA(g, { onClick: () => flipLever(i) });
  const n = litPlane(0.06, 0.06, 1200, (c, W, H) => { c.fillStyle = '#fff'; c.beginPath(); c.arc(W / 2, H / 2, W / 2 - 2, 0, 7); c.fill(); c.fillStyle = '#111'; c.font = `bold ${H * 0.6}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText(String(i + 1), W / 2, H * 0.55); });
  n.position.set(2.975, 1.42, z); n.rotation.y = -Math.PI / 2; CUR.add(n);
  return g;
});
const leverPic = new CanvasPlane(1.3, 0.55, 500, false, true); leverPic.mesh.position.set(-2.965, 1.6, -35.1); leverPic.mesh.rotation.y = Math.PI / 2; CUR.add(leverPic.mesh);
function drawLeverPic(){
  leverPic.draw((c, W, H) => {
    c.fillStyle = '#c99a4a'; c.fillRect(0, 0, W, H); c.fillStyle = '#bfe3ff'; c.fillRect(16, 16, W - 32, (H - 32) / 2); c.fillStyle = '#2f7fc1'; c.fillRect(16, H / 2, W - 32, (H - 32) / 2);
    c.textAlign = 'center'; c.textBaseline = 'middle';
    P.levers.forEach((up, i) => { const x = W * (0.16 + i * 0.227); c.font = `${H * 0.26}px ${FONT}`; c.fillText(up ? '🐦' : '🐟', x, up ? H * 0.3 : H * 0.72); c.fillStyle = '#fff'; c.font = `bold ${H * 0.1}px ${FONT}`; c.fillText(String(i + 1), x, H * 0.52); c.fillStyle = '#000'; });
  });
}
function flipLever(i){
  if (st.toyOpen) return;
  st.levers[i] = !st.levers[i]; tone(300, .08, 0, 'square', .1);
  const g = levers[i], r0 = g.rotation.z, r1 = st.levers[i] ? 0.6 : -0.6 + Math.PI;
  tween(0.25, k => g.rotation.z = r0 + (r1 - r0) * ease(k));
  if (st.levers.every((v, j) => v === P.levers[j])) delay(0.4, () => { st.toyOpen = true; sfx.unlock(); notify('Mänguasjakast avanes! 🎁', 3); tween(1, k => toyLid.rotation.x = 1.9 * ease(k)); });
}
const toyBox = new THREE.Group(); toyBox.position.set(2.2, 0, -38.3); CUR.add(toyBox);
S(new THREE.BoxGeometry(0.8, 0.03, 0.5), M.labCab, 0, 0.015, 0, toyBox);
for (const z of [-0.235, 0.235]) S(new THREE.BoxGeometry(0.8, 0.4, 0.03), new THREE.MeshLambertMaterial({ color: 0xffb300 }), 0, 0.2, z, toyBox);
for (const x of [-0.385, 0.385]) S(new THREE.BoxGeometry(0.03, 0.4, 0.44), new THREE.MeshLambertMaterial({ color: 0xffb300 }), x, 0.2, 0, toyBox);
const toyLid = new THREE.Group(); toyLid.position.set(0, 0.4, -0.25); toyBox.add(toyLid);
S(new THREE.BoxGeometry(0.84, 0.05, 0.52), new THREE.MeshLambertMaterial({ color: 0xe53935 }), 0, 0.025, 0.26, toyLid);
const BASKET = { x: -1.5, z: -38.2, r: 0.24, top: 0.55 }, LINE_Z = -35.8;
D(new THREE.CylinderGeometry(BASKET.r, 0.2, BASKET.top, 20, 1, true), new THREE.MeshLambertMaterial({ color: 0x8d6e63, side: THREE.DoubleSide }), BASKET.x, BASKET.top / 2, BASKET.z);
D(new THREE.TorusGeometry(BASKET.r, 0.015, 8, 24).rotateX(Math.PI / 2), new THREE.MeshLambertMaterial({ color: 0xff7043 }), BASKET.x, BASKET.top, BASKET.z);
D(new THREE.PlaneGeometry(4, 0.08).rotateX(-Math.PI / 2), new THREE.MeshBasicMaterial({ color: 0xffeb3b }), -0.6, 0.006, LINE_Z);
{ const t = litPlane(0.6, 0.12, 600, (c, W, H) => { c.fillStyle = '#ffeb3b'; c.font = `bold ${H * 0.7}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText('VISKEJOON', W / 2, H * 0.55); }); t.geometry.rotateX(-Math.PI / 2); t.position.set(-0.6, 0.007, LINE_Z + 0.12); CUR.add(t); }
const basketLamps = [0, 1, 2].map(i => D(new THREE.SphereGeometry(0.05, 12, 10), new THREE.MeshBasicMaterial({ color: 0x333333 }), BASKET.x - 0.15 + i * 0.15, 1.6, -38.96));
for (const [c, x, z] of [[0xef5350, -2.4, -33.6], [0x42a5f5, 1.6, -33.5], [0x66bb6a, -2.5, -37.0]]) { D(new THREE.SphereGeometry(0.13, 14, 10), new THREE.MeshLambertMaterial({ color: c }), x, 2.3, z); D(new THREE.CylinderGeometry(0.003, 0.003, 0.7, 4), M.black, x, 2.65, z); }

/* ---------- väljapääsu silt, mis liigub viimasesse tuppa ---------- */
const exitSign = litPlane(1.0, 0.24, 600, (c, W, H) => { c.fillStyle = '#2e7d32'; roundRect(c, 0, 0, W, H, 20); c.fill(); c.fillStyle = '#fff'; c.font = `bold ${H * 0.55}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText('🚪 VÄLJAPÄÄS', W / 2, H * 0.55); });
scene.add(exitSign);

/* =========================================================
   KULDSED HAMMASRATTAD (boonus)
   ========================================================= */
const GEAR_SPOTS = [
  [-2.45, 0, -1.6], [2.9, 1.30, -0.25], [-2.8, 2.005, 0.25], [2.8, 0, -2.85],
  [1.3, 1.715, -3.06], [-1.5, 1.315, -3.06], [2.62, 1.32, -6.2], [-2.85, 0.85, -7.0],
  [2.4, 0.765, -9.62], [-2.0, 0, -14.8], [2.4, 0.75, -14.45], [-2.85, 0, -9.4],
  [-2.3, 0.9, -20.3], [2.75, 0, -20.8], [2.5, 0.75, -22.2], [-2.8, 0.85, -24.2], [-2.6, 0, -27.5], [2.4, 0.8, -29.5], [-2.7, 0, -33.4], [2.6, 0, -33.6]
];
const gearGeo = (() => { const parts = [new THREE.CylinderGeometry(0.032, 0.032, 0.012, 20).rotateX(Math.PI / 2)]; for (let i = 0; i < 10; i++) { const a = i * Math.PI / 5; parts.push(new THREE.BoxGeometry(0.014, 0.014, 0.012).translate(Math.cos(a) * 0.036, Math.sin(a) * 0.036, 0)); } return mergeGeometries(parts.map(p => p.index ? p.toNonIndexed() : p)); })();
const gears = [0, 1, 2].map(() => { const g = new THREE.Group(); const m = D(gearGeo, M.gold, 0, 0.045, 0, g); g.userData.m = m; makeIA(g, { onClick: () => collectGear(g) }); return g; });

/* =========================================================
   VÕETAVAD ESEMED
   ========================================================= */
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
finishItem(uvObj, { item: 'uv', enabled: () => st.drawerOpen, onGrab: () => { if (!st.uvTaken) { st.uvTaken = true; notify('UV-lamp! 🔦 Suuna see seintele.', 3); } } });
const beam = new THREE.Mesh(new THREE.ConeGeometry(0.35, 2.2, 20, 1, true).translate(0, -1.1, 0).rotateX(Math.PI / 2),
  new THREE.MeshBasicMaterial({ color: 0x9b5cff, transparent: true, opacity: 0.13, blending: THREE.AdditiveBlending, depthWrite: false, side: THREE.DoubleSide }));
beam.position.z = -0.112; beam.visible = false; uvObj.add(beam);
function buildKey(mat, s = 1){
  const k = new THREE.Group();
  D(new THREE.TorusGeometry(0.022 * s, 0.006 * s, 8, 20).rotateX(Math.PI / 2), mat, 0, 0, 0.055 * s, k);
  D(new THREE.CylinderGeometry(0.005 * s, 0.005 * s, 0.08 * s, 8).rotateX(Math.PI / 2), mat, 0, 0, 0, k);
  D(new THREE.BoxGeometry(0.016 * s, 0.006 * s, 0.01 * s), mat, 0.01 * s, 0, -0.035 * s, k);
  D(new THREE.BoxGeometry(0.012 * s, 0.006 * s, 0.008 * s), mat, 0.008 * s, 0, -0.018 * s, k);
  return k;
}
const keyObj = buildKey(M.brass);
finishItem(keyObj, { item: 'key', enabled: () => st.globeOpen, onGrab: () => { st.keyTaken = true; } });
const fuseObj = buildFuse();
finishItem(fuseObj, { item: 'fuse', enabled: () => st.chestOpen });
const weightObjs = [1, 2, 3, 4, 5].map(kg => {
  const s = 0.045 + kg * 0.008;
  const tex = canvasTex(128, 128, (c, W, H) => { const g = c.createLinearGradient(0, 0, W, H); g.addColorStop(0, '#9aa3a8'); g.addColorStop(1, '#5d666b'); c.fillStyle = g; c.fillRect(0, 0, W, H); c.strokeStyle = '#3a4146'; c.lineWidth = 8; c.strokeRect(4, 4, W - 8, H - 8); c.fillStyle = '#fff'; c.font = `bold ${H * 0.36}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText(`${kg} kg`, W / 2, H * 0.53); }, false);
  const g = new THREE.Group(); D(new THREE.BoxGeometry(s, s, s), new THREE.MeshLambertMaterial({ map: tex }), 0, 0, 0, g);
  return finishItem(g, { item: 'w', kg });
});
const luupObj = new THREE.Group();
D(new THREE.TorusGeometry(0.04, 0.007, 10, 28), M.brass, 0, 0, -0.05, luupObj);
D(new THREE.CircleGeometry(0.04, 24), new THREE.MeshLambertMaterial({ color: 0xcfefff, transparent: true, opacity: 0.45, side: THREE.DoubleSide }), 0, 0, -0.05, luupObj);
D(new THREE.CylinderGeometry(0.009, 0.011, 0.1, 10), M.woodDark, 0, -0.095, -0.05, luupObj);
finishItem(luupObj, { item: 'luup', enabled: () => st.scaleDone, onGrab: () => { if (!st.luupTaken) { st.luupTaken = true; notify('Luup! 🔍 Suuna see pisikesele kirjale.', 3); } } });
const POT = { r: ['punane', '#e53935'], b: ['sinine', '#1e88e5'], y: ['kollane', '#fdd835'] };
const potionObjs = ['r', 'b', 'y'].map(ck => {
  const g = new THREE.Group();
  D(new THREE.CylinderGeometry(0.036, 0.036, 0.13, 16), new THREE.MeshLambertMaterial({ color: 0xe8f6ff, transparent: true, opacity: 0.35, depthWrite: false }), 0, 0.065, 0, g);
  D(new THREE.CylinderGeometry(0.032, 0.032, 0.1, 16), new THREE.MeshLambertMaterial({ color: POT[ck][1], emissive: new THREE.Color(POT[ck][1]).multiplyScalar(0.25) }), 0, 0.055, 0, g);
  D(new THREE.CylinderGeometry(0.013, 0.02, 0.06, 12), new THREE.MeshLambertMaterial({ color: 0xe8f6ff, transparent: true, opacity: 0.45 }), 0, 0.16, 0, g);
  D(new THREE.CylinderGeometry(0.012, 0.012, 0.02, 10), new THREE.MeshLambertMaterial({ color: 0x8d6e63 }), 0, 0.195, 0, g);
  const l = litPlane(0.05, 0.035, 1600, (c, W, H) => { c.fillStyle = '#f2e6c8'; c.fillRect(0, 0, W, H); c.fillStyle = POT[ck][1]; c.beginPath(); c.arc(W / 2, H / 2, H * 0.32, 0, 7); c.fill(); });
  l.position.set(0, 0.07, 0.037); g.add(l);
  return finishItem(g, { item: 'potion' + ck, ck, color: POT[ck][1], pour: { neck: V3(0, 0.2, 0), tilt: -0.2, r: 0.15, color: POT[ck][1], target: () => flask.getWorldPosition(V3(0, 0, 0)).add(V3(0, 0.37, 0)), dose: () => addDose(ck) } });
});
const crystalObj = new THREE.Group();
D(new THREE.OctahedronGeometry(0.045), new THREE.MeshLambertMaterial({ color: 0xff3d6e, emissive: 0x99102e, flatShading: true }), 0, 0, 0, crystalObj).scale.set(0.8, 1.3, 0.8);
finishItem(crystalObj, { item: 'crystal', enabled: () => st.potionDone, onGrab: () => { st.crystalTaken = true; } });
const vaseObj = new THREE.Group();
{
  const pts = [[0.0, 0], [0.06, 0], [0.075, 0.03], [0.09, 0.09], [0.085, 0.15], [0.05, 0.22], [0.035, 0.27], [0.045, 0.3]].map(([x, y]) => new THREE.Vector2(x, y));
  D(new THREE.LatheGeometry(pts, 24), M.porcelain, 0, 0, 0, vaseObj);
}
finishItem(vaseObj, { item: 'vase', fragile: true, holdPos: V3(0, -0.12, -0.05), onGrab: () => { if (!st.vaseShaken) { st.vaseShaken = true; noise(.25, .12, 3000); notify('Midagi kõliseb vaasi sees… 🏺', 3); } } });
const lensObj = new THREE.Group();
D(new THREE.CylinderGeometry(0.03, 0.03, 0.012, 20), new THREE.MeshLambertMaterial({ color: 0x9fd8ff, emissive: 0x204060 }), 0, 0, 0, lensObj);
D(new THREE.TorusGeometry(0.031, 0.004, 8, 24).rotateX(Math.PI / 2), M.brass, 0, 0, 0, lensObj);
finishItem(lensObj, { item: 'lens', enabled: () => st.vaseBroken, onGrab: () => { st.lensTaken = true; } });
const goldKeyObj = buildKey(M.gold, 1.6);
finishItem(goldKeyObj, { item: 'goldkey', enabled: () => st.safeOpen, onGrab: () => { st.goldTaken = true; } });

const items = {
  uv: { obj: uvObj, home: drawer, pos: V3(2.45, 0.586, 0.2), rotY: 0 },
  key: { obj: keyObj, home: globe, pos: V3(0, 1.0, 0), rotY: 0.6, hidden: true },
  fuse: { obj: fuseObj, home: chest, pos: V3(-0.2, 0.031, -0.05), rotY: 0.4 },
  luup: { obj: luupObj, home: labDrawer, pos: V3(-2.6, 0.672, -6.3), rotY: 0.3 },
  crystal: { obj: crystalObj, home: vit, pos: V3(0, 0.02, 0), rotY: 0 },
  vase: { obj: vaseObj, home: roomG[2], pos: V3(-2.5, 0.94, -9.9), rotY: 0 },
  lens: { obj: lensObj, home: roomG[2], pos: V3(-2.5, 0.94, -9.9), rotY: 0, hidden: true },
  goldkey: { obj: goldKeyObj, home: safeG, pos: V3(0, 0.06, 0), rotY: 0.4 }
};
weightObjs.forEach((o, i) => items['w' + (i + 1)] = { obj: o, home: roomG[1], pos: V3(0, 0, 0), rotY: 0 });
potionObjs.forEach((o, i) => items['p' + i] = { obj: o, home: roomG[1], pos: V3(0, 0, 0), rotY: 0 });
const WEIGHT_SPOTS = [[-2.45, -5.68], [-2.45, -5.85], [-2.78, -5.72], [-2.45, -6.85], [-2.65, -6.92]];
const POTION_SPOTS = [[2.72, -4.3], [2.72, -4.48], [2.72, -4.66]];
const zones = [
  { obj: panR, r: 0.13, top: 0.006, layout: () => layoutPan(panR) },
  { obj: panL, r: 0.13, top: 0.006, reject: true }
];
function layoutPan(pan){
  const its = pan.children.filter(c => c.userData.ia && c.userData.ia.item === 'w');
  const slots = [[-0.04, -0.04], [0.04, 0.04], [0.04, -0.04], [-0.04, 0.04], [0, 0]];
  its.forEach((it, i) => { const s = slots[i % 5], layer = Math.floor(i / 5); it.position.set(s[0], 0.006 + it.userData.ia.restOffset + layer * 0.09, s[1]); });
}

// tubade 4–7 esemed
const torchObj = new THREE.Group();
D(new THREE.CylinderGeometry(0.02, 0.02, 0.18, 14).rotateX(Math.PI / 2), new THREE.MeshLambertMaterial({ color: 0xf9a825 }), 0, 0, 0.02, torchObj);
D(new THREE.CylinderGeometry(0.033, 0.022, 0.05, 16).rotateX(Math.PI / 2), M.darkMetal, 0, 0, -0.09, torchObj);
D(new THREE.CircleGeometry(0.03, 16).rotateY(Math.PI), new THREE.MeshBasicMaterial({ color: 0xfff6c0 }), 0, 0, -0.116, torchObj);
finishItem(torchObj, { item: 'torch', onGrab: () => { if (!st.torchTaken) { st.torchTaken = true; notify('Taskulamp! 🔦 Valgusta pimedust.', 3); } } });
const torchSpot = new THREE.SpotLight(0xfff2d0, 0, 14, 0.42, 0.45, 1.2); scene.add(torchSpot); scene.add(torchSpot.target);
const seedObj = new THREE.Group();
{ const l = litPlane(0.09, 0.12, 1200, (c, W, H) => { c.fillStyle = '#7cb342'; c.fillRect(0, 0, W, H); c.fillStyle = '#fff'; c.font = `${H * 0.4}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText('🌻', W / 2, H * 0.42); c.font = `bold ${H * 0.14}px ${FONT}`; c.fillText('SEEMNED', W / 2, H * 0.82); }); l.geometry.rotateX(-Math.PI / 2); l.position.y = 0.006; seedObj.add(l);
  D(new THREE.BoxGeometry(0.09, 0.01, 0.12), new THREE.MeshLambertMaterial({ color: 0x558b2f }), 0, 0, 0, seedObj); }
finishItem(seedObj, { item: 'seed' });
const canObj = new THREE.Group();
D(new THREE.CylinderGeometry(0.08, 0.09, 0.16, 18), new THREE.MeshLambertMaterial({ color: 0x43a047 }), 0, 0.08, 0, canObj);
rod(V3(0.07, 0.07, 0), V3(0.2, 0.18, 0), 0.012, 0.02, new THREE.MeshLambertMaterial({ color: 0x43a047 }), canObj);
D(new THREE.TorusGeometry(0.06, 0.01, 8, 16, Math.PI).rotateZ(Math.PI / 2), new THREE.MeshLambertMaterial({ color: 0x2e7d32 }), -0.06, 0.16, 0, canObj);
finishItem(canObj, { item: 'can', pour: { neck: V3(0.2, 0.18, 0), tilt: 0.55, r: 0.32, color: '#7fc8ff', target: () => st.seeded && !st.grown ? pot.getWorldPosition(V3(0, 0, 0)).add(V3(0, 0.5, 0)) : null, dose: () => waterPlant() } });
const handMirrorObj = new THREE.Group();
D(new THREE.CylinderGeometry(0.06, 0.06, 0.01, 24).rotateX(Math.PI / 2), M.silver, 0, 0, -0.03, handMirrorObj);
D(new THREE.TorusGeometry(0.06, 0.008, 8, 24), M.brass, 0, 0, -0.03, handMirrorObj);
D(new THREE.CylinderGeometry(0.01, 0.012, 0.1, 10), M.brass, 0, -0.11, -0.03, handMirrorObj);
finishItem(handMirrorObj, { item: 'mirror', onGrab: () => { if (!st.mirrorTaken) { st.mirrorTaken = true; notify('Käsipeegel! 🪞 Suuna see peegelkirjale.', 3); } } });
const greenKeyObj = buildKey(new THREE.MeshLambertMaterial({ color: 0x66bb6a, emissive: 0x1b5e20 }), 1.3);
finishItem(greenKeyObj, { item: 'greenkey', enabled: () => st.grown });
const ballObjs = [0xef5350, 0x42a5f5, 0xffca28].map(col => {
  const g = new THREE.Group(); D(new THREE.SphereGeometry(0.07, 18, 14), new THREE.MeshLambertMaterial({ color: col }), 0, 0, 0, g);
  const o = finishItem(g, { item: 'ball', enabled: () => st.toyOpen && !g.userData.ia.scored });
  return o;
});
Object.assign(items, {
  torch: { obj: torchObj, home: roomG[3], pos: V3(1.6, 0.9, -15.7), rotY: 0.5 },
  seed: { obj: seedObj, home: potDrawer, pos: V3(-2.58, 0.672, -23.5), rotY: 0 },
  can: { obj: canObj, home: roomG[4], pos: V3(2.4, 0.75, -22.4), rotY: Math.PI },
  mirror: { obj: handMirrorObj, home: roomG[5], pos: V3(-2.8, 1.215, -28.6), rotY: Math.PI / 2 },
  greenkey: { obj: greenKeyObj, home: flower, pos: V3(0, 0.02, 0), rotY: 0, hidden: true }
});
ballObjs.forEach((o, i) => items['ball' + i] = { obj: o, home: toyBox, pos: V3(-0.2 + i * 0.2, 0.03, 0), rotY: 0 });
// rohelise võtme lukk kasvuhoone uksel
const greenLock = new THREE.Group(); greenLock.position.set(0.86, 1.25, 0.03); DOORS[4].pivot.add(greenLock);
D(new THREE.BoxGeometry(0.1, 0.14, 0.012), new THREE.MeshLambertMaterial({ color: 0x66bb6a, emissive: 0x0b3d0b }), 0, 0, 0, greenLock);
D(new THREE.CircleGeometry(0.012, 12), M.black, 0, 0.015, 0.007, greenLock);
makeIA(greenLock, { accepts: 'greenkey', onUse: h => { consume(h); sfx.unlock(); notify('Roheline võti sobib! 🌱', 3); requestDoor(4); }, onClick: () => { sfx.bad(); notify('Lukk on roheline – vaja on rohelist võtit 🗝️'); }, enabled: () => !st.door[4] });

/* =========================================================
   ÕPPIMISE REŽIIM – teadmiste lukud uste kõrval
   ========================================================= */
const EST_Q = [
  ['Mis on sõna „suur“ vastand?', 'väike', 'pikk', 'lai', 'kõrge'],
  ['Mis on sõna „soe“ vastand?', 'külm', 'märg', 'kuiv', 'kerge'],
  ['Mis on sõna „vana“ vastand?', 'uus', 'suur', 'hea', 'kole'],
  ['Mis on sõna „kiire“ vastand?', 'aeglane', 'kerge', 'lühike', 'tugev'],
  ['Milline sõna on õigesti kirjutatud?', 'jäätis', 'jäädis', 'jäätiss', 'jätis'],
  ['Milline sõna on õigesti kirjutatud?', 'kartul', 'kartull', 'gartul', 'kardul'],
  ['Mitu silpi on sõnas „kassipoeg“?', '3', '2', '4', '5'],
  ['Mitu tähte on sõnas „päike“?', '5', '4', '6', '3'],
  ['Mis on sõna „koer“ mitmus?', 'koerad', 'koerid', 'koeraid', 'koerud'],
  ['Mis on sõna „lind“ mitmus?', 'linnud', 'lindud', 'linnad', 'lindad'],
  ['Mis on sõna „laps“ mitmus?', 'lapsed', 'lapsid', 'lapsad', 'lapsud'],
  ['Milline täht on täishäälik?', 'Õ', 'K', 'S', 'T'],
  ['Milline sõna on tegusõna?', 'jooksma', 'kiire', 'laud', 'roheline'],
  ['Milline sõna on omadussõna?', 'ilus', 'laulma', 'maja', 'kiiresti'],
  ['Milline sõna on nimisõna?', 'kass', 'sööma', 'punane', 'kiiresti'],
  ['Milline sõna on liitsõna?', 'koolimaja', 'kool', 'majake', 'koolis'],
  ['Mis on sõna „rõõmus“ sünonüüm?', 'õnnelik', 'kurb', 'vihane', 'väsinud'],
  ['Milline sõna algab täishäälikuga?', 'õun', 'kala', 'puu', 'tuba'],
  ['Mis on Eesti pealinn?', 'Tallinn', 'Tartu', 'Pärnu', 'Narva']
];
const ENG_W = [['koer', 'dog'], ['kass', 'cat'], ['hobune', 'horse'], ['lind', 'bird'], ['kala', 'fish'], ['maja', 'house'], ['puu', 'tree'], ['päike', 'sun'], ['kuu', 'moon'], ['täht', 'star'], ['õun', 'apple'], ['raamat', 'book'], ['laud', 'table'], ['tool', 'chair'], ['aken', 'window'], ['uks', 'door'], ['vesi', 'water'], ['piim', 'milk'], ['leib', 'bread'], ['punane', 'red'], ['sinine', 'blue'], ['roheline', 'green'], ['suur', 'big'], ['väike', 'small'], ['kool', 'school'], ['sõber', 'friend'], ['ema', 'mother'], ['isa', 'father'], ['jooksma', 'run'], ['lugema', 'read']];
function optsQ(q, right, wrongs){ const opts = shuffle([right, ...shuffle(wrongs.filter(w => w !== right)).slice(0, 3)]); return { q, a: opts, c: opts.indexOf(right) }; }
function numOpts(q, ans){ const s = new Set([ans]); for (const d of shuffle([1, -1, 2, -2, 10, -10, 3, 5])) { if (s.size >= 4) break; if (ans + d >= 0) s.add(ans + d); } return optsQ(q, String(ans), [...s].filter(v => v !== ans).map(String)); }
function customQuestions(){ return (store('pt_questions') || '').split('\n').map(l => l.split('|').map(s => s.trim())).filter(p => p.length >= 3 && p[0] && p[1]); }
function genQuestion(){
  const kind = settings.learn, d = settings.diff;
  if (kind === 'own') { const cq = customQuestions(); if (cq.length) { const p = pick(cq); return optsQ(p[0], p[1], p.slice(2)); } }
  if (kind === 'est') { const p = pick(EST_Q); return optsQ(p[0], p[1], p.slice(2)); }
  if (kind === 'eng') { const w = pick(ENG_W); return rand() < 0.5 ? optsQ(`Mis on inglise keeles „${w[0]}“?`, w[1], ENG_W.map(x => x[1])) : optsQ(`Mida tähendab „${w[1]}“?`, w[0], ENG_W.map(x => x[0])); }
  let a, b, c;
  if (d === 'kerge') { a = randInt(1, 10); b = randInt(1, 10); if (rand() < 0.5) return numOpts(`${a} + ${b} = ?`, a + b); if (a < b) [a, b] = [b, a]; return numOpts(`${a} − ${b} = ?`, a - b); }
  if (d === 'tavaline') { const r = rand(); a = randInt(2, 10); b = randInt(2, 10); if (r < 0.4) return numOpts(`${a} · ${b} = ?`, a * b); if (r < 0.6) return numOpts(`${a * b} : ${a} = ?`, b); a = randInt(10, 60); b = randInt(5, 39); return rand() < 0.5 ? numOpts(`${a} + ${b} = ?`, a + b) : numOpts(`${a + b} − ${b} = ?`, a); }
  a = randInt(2, 12); b = randInt(2, 12); c = randInt(2, 20); const r = rand();
  if (r < 0.35) return numOpts(`${a} · ${b} + ${c} = ?`, a * b + c);
  if (r < 0.7) return numOpts(`(${a} + ${c}) · ${b} = ?`, (a + c) * b);
  return numOpts(`${a * b * 2} : ${b} − ${a} = ?`, a);
}
const quizPanels = DOORS.map((d, i) => {
  const g = new THREE.Group(); g.position.set(-1.0, 1.45, d.z + 0.03); roomG[i].add(g);
  S(new THREE.BoxGeometry(0.66, 0.42, 0.02), M.darkMetal, 0, 0, -0.012, g);
  const qp = new CanvasPlane(0.62, 0.2, 900); qp.mesh.position.set(0, 0.1, 0.001); g.add(qp.mesh);
  const btns = [0, 1, 2, 3].map(k => {
    const cp = new CanvasPlane(0.3, 0.07, 900); cp.mesh.position.set(k % 2 ? 0.155 : -0.155, -0.04 - Math.floor(k / 2) * 0.085, 0.002); g.add(cp.mesh);
    makeIA(cp.mesh, { scaleHover: true, onClick: () => answerQuiz(i, k), enabled: () => !st.quiz[i].done });
    return cp;
  });
  return { g, qp, btns };
});
function drawQuiz(i){
  const qs = st.quiz[i], p = quizPanels[i];
  p.qp.draw((c, W, H) => {
    c.fillStyle = qs.done ? '#1b5e20' : '#0d2533'; roundRect(c, 2, 2, W - 4, H - 4, 18); c.fill();
    c.fillStyle = '#ffd54f'; c.font = `bold ${H * 0.15}px ${FONT}`; c.textAlign = 'left'; c.textBaseline = 'middle';
    c.fillText('📚 Teadmiste lukk', W * 0.04, H * 0.16);
    c.textAlign = 'right'; c.fillText(qs.done ? '✅' : '✔'.repeat(qs.got) + '◻'.repeat(Math.max(0, QUIZ_NEED - qs.got)), W * 0.96, H * 0.16);
    c.fillStyle = '#fff'; c.textAlign = 'center';
    if (qs.done) { c.font = `bold ${H * 0.22}px ${FONT}`; c.fillText('Lukk avatud! 🎉', W / 2, H * 0.6); }
    else fitText(c, qs.q.q, W / 2, H * 0.3, W * 0.92, H * 0.66, Math.round(H * 0.2), true);
  });
  p.btns.forEach((b, k) => { b.mesh.visible = !qs.done; if (qs.done) return; b.draw((c, W, H) => { c.fillStyle = '#37474f'; roundRect(c, 3, 3, W - 6, H - 6, 14); c.fill(); c.strokeStyle = '#90a4ae'; c.lineWidth = 4; c.stroke(); c.fillStyle = '#fff'; c.textAlign = 'center'; c.textBaseline = 'middle'; let f = H * 0.5; c.font = `bold ${f}px ${FONT}`; const t = qs.q.a[k] ?? ''; while (c.measureText(t).width > W * 0.9 && f > 10) { f -= 1; c.font = `bold ${f}px ${FONT}`; } c.fillText(t, W / 2, H * 0.55); }); });
}
const QUIZ_NEED = 2;
function answerQuiz(i, k){
  const qs = st.quiz[i]; if (qs.done || k >= qs.q.a.length) return;
  if (k === qs.q.c) { qs.got++; sfx.ok(); if (qs.got >= QUIZ_NEED) { qs.done = true; notify('Teadmiste lukk avatud! 📚✅', 3); if (st.pending[i]) delay(0.6, () => openDoor(i)); } else notify('Õige! ✔ Veel üks küsimus.'); }
  else { sfx.bad(); notify(`Vale vastus. Õige oli: ${qs.q.a[qs.q.c]}`, 3); }
  if (!qs.done) qs.q = genQuestion();
  drawQuiz(i);
}
function requestDoor(i){
  if (st.door[i]) return;
  if (settings.learn !== 'off' && !st.quiz[i].done) { st.pending[i] = true; notify('Lukk avanes! Vasta veel teadmiste luku küsimustele 📚', 4); return; }
  openDoor(i);
}

/* =========================================================
   KASUTAJALIIDES
   ========================================================= */
function makeButton(label, w, h, onClick, { color = '#2e7d32', fs = 0.4 } = {}){
  const p = new CanvasPlane(w, h, 600, true);
  const paint = (col = color, lab = label, ring = false) => p.draw((c, W, H) => {
    roundRect(c, 5, 5, W - 10, H - 10, Math.min(W, H) * 0.25); c.fillStyle = col; c.fill(); c.lineWidth = ring ? 14 : 6; c.strokeStyle = ring ? '#ffd54f' : 'rgba(255,255,255,.85)'; c.stroke();
    c.fillStyle = '#fff'; c.textAlign = 'center'; c.textBaseline = 'middle'; let f = H * fs; c.font = `bold ${f}px ${FONT}`;
    while (c.measureText(lab).width > W * 0.88 && f > 8) { f -= 1; c.font = `bold ${f}px ${FONT}`; }
    c.fillText(lab, W / 2, H * 0.54);
  });
  paint(); p.mesh.renderOrder = 910; p.mesh.userData.paint = paint;
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
const hud = new CanvasPlane(0.2, 0.06, 700, true); hud.mesh.renderOrder = 998; hud.mesh.position.set(0, 0.3, -1); hud.mesh.visible = false; camera.add(hud.mesh);
let hudLast = '';
function drawHud(text, warn){ if (text === hudLast) return; hudLast = text; hud.draw((c, W, H) => { panelBg(c, W, H, warn ? 'rgba(140,0,0,0.85)' : 'rgba(0,0,0,0.65)', 24); c.fillStyle = '#fff'; c.font = `bold ${H * 0.55}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText(text, W / 2, H * 0.55); }); }
const readerUI = new THREE.Group(); readerUI.visible = false; scene.add(readerUI);
const readerPanel = new CanvasPlane(1.0, 0.72, 900, true); readerUI.add(readerPanel.mesh);
makeIA(readerPanel.mesh, { ui: true, noHover: true, onClick: () => closeReader() });
{ const b = makeButton('✖ Sulge', 0.3, 0.09, () => closeReader(), { color: '#455a64', fs: 0.45 }); b.position.set(0, -0.43, 0.01); readerUI.add(b); }
function openReader(title, text, paper = true){
  readerPanel.draw((c, W, H) => {
    if (paper) { parchment(c, W, H); c.fillStyle = '#3b2a16'; } else { panelBg(c, W, H); c.fillStyle = '#ffffff'; }
    c.textAlign = 'center'; c.textBaseline = 'middle';
    c.font = `bold ${H * 0.085}px ${FONT}`; c.fillText(title, W / 2, H * 0.11);
    fitText(c, text, W / 2, H * 0.2, W * 0.86, H * 0.76, Math.round(H * 0.058));
  });
  placeInFront(readerUI, 0.9, -0.08); readerUI.visible = true;
}
function openView(drawFn){ readerPanel.draw(drawFn); placeInFront(readerUI, 0.9, -0.08); readerUI.visible = true; }
function closeReader(){ readerUI.visible = false; }

/* ---------- seaded + peamenüü ---------- */
const DIFF = {
  kerge: { label: 'Kerge', hints: Infinity, melody: 4, colors: 2, maxCount: 2 },
  tavaline: { label: 'Tavaline', hints: 5, melody: 5, colors: 2, maxCount: 3 },
  raske: { label: 'Raske', hints: 2, melody: 6, colors: 3, maxCount: 3 }
};
const settings = Object.assign({ diff: 'tavaline', timer: 0, music: true, rooms: 3, learn: 'off' }, store('pt_settings') || {});
if (!DIFF[settings.diff]) settings.diff = 'tavaline';
settings.rooms = clamp(settings.rooms | 0 || 3, 1, 7);
const best = store('pt_best') || {};
const bestKey = () => `${settings.diff}_${settings.rooms}`;
const LEARN = [['off', 'Väljas'], ['math', '➗ Matem.'], ['est', '📖 Eesti k.'], ['eng', '🇬🇧 Inglise'], ['own', '✏️ Oma']];
const menuUI = new THREE.Group(); scene.add(menuUI);
const MPW = 1.7, MPH = 1.42;
const menuPanel = new CanvasPlane(MPW, MPH, 650, true); menuUI.add(menuPanel.mesh); makeIA(menuPanel.mesh, { ui: true, noHover: true });
const mY = y => (MPH / 2 - y) / MPH;
const ROW = { diff: 0.42, timer: 0.3, rooms: 0.18, learn: 0.06, music: -0.06 };
function drawMenu(){
  menuPanel.draw((c, W, H) => {
    panelBg(c, W, H); c.textAlign = 'center'; c.textBaseline = 'middle';
    c.fillStyle = '#ffd54f'; c.font = `bold ${H * 0.065}px ${FONT}`; c.fillText('🔐 Põgenemistuba', W / 2, H * mY(0.62));
    c.fillStyle = '#cfd8dc'; c.font = `${H * 0.03}px ${FONT}`; c.fillText('Professor Nupu maja • kuni 7 tuba • pultidega või paljaste kätega', W / 2, H * mY(0.545));
    c.fillStyle = '#fff'; c.textAlign = 'left'; c.font = `bold ${H * 0.033}px ${FONT}`;
    c.fillText('Raskusaste', W * 0.03, H * mY(ROW.diff)); c.fillText('Taimer', W * 0.03, H * mY(ROW.timer)); c.fillText('Tubade arv', W * 0.03, H * mY(ROW.rooms)); c.fillText('Õppimine', W * 0.03, H * mY(ROW.learn)); c.fillText('Muusika', W * 0.03, H * mY(ROW.music));
    c.textAlign = 'center'; c.fillStyle = '#b0bec5'; c.font = `${H * 0.028}px ${FONT}`;
    const b = best[bestKey()];
    c.fillText(b ? `🏆 Parim aeg (${DIFF[settings.diff].label}, ${settings.rooms} tuba): ${fmtTime(b.t)} • ${'★'.repeat(b.s)}` : `🏆 Parimat aega (${DIFF[settings.diff].label}, ${settings.rooms} tuba) veel pole`, W / 2, H * mY(-0.17));
    c.fillStyle = '#90a4ae'; c.font = `${H * 0.024}px ${FONT}`;
    c.fillText('PULDID: päästik = kasuta/võta • külgnupp = haara • vasak kang = liigu • parem kang = pööra • Y/B = paus', W / 2, H * mY(-0.5));
    c.fillText('KÄED: näpista = võta/kasuta • näpista põrandale = teleport • vajuta nuppe sõrmega • viska käega • vasaku randme ⏸ nupp = paus', W / 2, H * mY(-0.55));
    c.fillText('Õpetaja küsimused saab sisestada arvutis (nupp "📚 Õpetaja küsimused" lehe ülaosas).', W / 2, H * mY(-0.6));
  });
}
const optBtns = { diff: [], timer: [], rooms: [], learn: [], music: [] };
function paintOpts(){
  optBtns.diff.forEach(b => b.userData.paint(b.userData.val === settings.diff ? '#2e7d32' : '#37474f', undefined, b.userData.val === settings.diff));
  optBtns.timer.forEach(b => b.userData.paint(b.userData.val === settings.timer ? '#1565c0' : '#37474f', undefined, b.userData.val === settings.timer));
  optBtns.rooms.forEach(b => b.userData.paint(b.userData.val === settings.rooms ? '#e65100' : '#37474f', undefined, b.userData.val === settings.rooms));
  optBtns.learn.forEach(b => b.userData.paint(b.userData.val === settings.learn ? '#00838f' : '#37474f', undefined, b.userData.val === settings.learn));
  optBtns.music[0].userData.paint(settings.music ? '#6a1b9a' : '#37474f', settings.music ? '🎵 Sees' : '🔇 Väljas', settings.music);
  drawMenu(); store('pt_settings', settings);
}
const addOpt = (key, val, label, x, y, w) => { const b = makeButton(label, w, 0.085, () => { settings[key] = val; paintOpts(); }, { fs: 0.42 }); b.userData.val = val; b.position.set(x, y, 0.01); menuUI.add(b); optBtns[key].push(b); };
[['kerge', '🙂 Kerge'], ['tavaline', '😐 Tavaline'], ['raske', '😈 Raske']].forEach(([v, l], i) => addOpt('diff', v, l, -0.25 + i * 0.36, ROW.diff, 0.33));
[[0, '⏳ Väljas'], [30, '30 min'], [45, '45 min']].forEach(([v, l], i) => addOpt('timer', v, l, -0.25 + i * 0.36, ROW.timer, 0.33));
for (let n = 1; n <= 7; n++) addOpt('rooms', n, String(n), -0.36 + (n - 1) * 0.15, ROW.rooms, 0.13);
LEARN.forEach(([v, l], i) => addOpt('learn', v, l, -0.36 + i * 0.235, ROW.learn, 0.22));
{ const b = makeButton('🎵 Sees', 0.33, 0.085, () => { settings.music = !settings.music; paintOpts(); }, { fs: 0.42 }); b.position.set(-0.25, ROW.music, 0.01); menuUI.add(b); optBtns.music.push(b); }
{ const b = makeButton('▶ Alusta', 0.55, 0.13, () => newGame(), { color: '#e65100', fs: 0.45 }); b.position.set(0, -0.33, 0.01); menuUI.add(b); }
paintOpts();
const pauseUI = new THREE.Group(); pauseUI.visible = false; scene.add(pauseUI);
const pausePanel = new CanvasPlane(0.9, 0.36, 700, true); pausePanel.mesh.position.y = 0.4; pauseUI.add(pausePanel.mesh); makeIA(pausePanel.mesh, { ui: true, noHover: true });
const pauseBtns = [
  ['▶ Jätka', '#2e7d32', () => closePause()],
  ['💡 Vihje', '#f57f17', () => { closePause(); showHint(); }],
  ['🔁 Uus mäng', '#1565c0', () => newGame()],
  ['🏠 Peamenüü', '#455a64', () => goMenu()],
  ['🚪 Välju VR-ist', '#b71c1c', () => exitVR()]
].map(([l, col, fn], i) => { const b = makeButton(l, 0.56, 0.11, fn, { color: col, fs: 0.42 }); b.position.set(0, 0.15 - i * 0.13, 0.01); pauseUI.add(b); return b; });
const endUI = new THREE.Group(); endUI.visible = false; scene.add(endUI);
const endPanel = new CanvasPlane(1.1, 0.72, 800, true); endPanel.mesh.position.y = 0.12; endUI.add(endPanel.mesh); makeIA(endPanel.mesh, { ui: true, noHover: true });
const endBtns = [0, 1, 2].map(i => { const b = makeButton('-', 0.34, 0.11, () => {}, { fs: 0.4 }); b.position.set(-0.36 + i * 0.36, -0.33, 0.01); endUI.add(b); return b; });
function setEndButtons(list){ endBtns.forEach((b, i) => { const it = list[i]; b.visible = !!it; if (it) { b.userData.ia.onClick = it[2]; b.userData.paint(it[1], it[0]); } }); }

/* =========================================================
   MÕISTATUSTE ANDMED
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
function startText(){ const n = settings.rooms; return `Tere, uurija! 🕵️\n\nLukustasin su oma majja. ${n > 1 ? `Sul tuleb läbida ${n} tuba – iga uks avaneb isemoodi.` : 'Pääse sellest toast välja!'}\n\nSelle toa uks avaneb 4-numbrilise koodiga. Ukse kohal olev silt ütleb, mis järjekorras numbrid tulevad.\n\nKui jääd hätta, helista laual olevat kella 🔔\n\n– Professor Nupp`; }
const LAB_TEXT = 'Tubli, pääsesid laborisse! 🧪\n\nJärgmise ukse avab laserkiir – aga laserist on kristall puudu. Kristall on vitriinis, mis avaneb ainult õige võlujoogiga.\n\nRetsept on seinal, kuid kogused kirjutasin pisikeselt. Luubi panin kaalulaua sahtlisse – see avaneb, kui kaal on tasakaalus.\n\n– Professor Nupp';
const OBS_TEXT = 'Viimane tuba – minu tähetorn! 🔭\n\nVäljapääsu kuldvõti on seifis. Seifi kood koosneb kolmest numbrist: üks peitub tähtede seas, teine muusikas ja kolmas varjus.\n\nPalju õnne!\n– Professor Nupp';
const ROOM_TEXTS = [null, ['🧪 Labor', LAB_TEXT], ['🔭 Tähetorn', OBS_TEXT],
  ['🔦 Kelder', 'Siin on pime nagu kotis! 🔦\n\nUkse avab surve: keera kolm ventiili õigeks. Õiged numbrid kirjutasin kriidiga seinale – aga ilma valguseta ei näe sa midagi. Taskulamp on ukse kõrval tünni peal.\n\n– Professor Nupp'],
  ['🌱 Kasvuhoone', 'Tere tulemast kasvuhoonesse! 🌱\n\nVõti kasvab siin lille sees. Leia seemned, istuta need suurde potti ja kasta taime.\n\n– Professor Nupp'],
  ['📻 Raadiotuba', 'Raadiotuba 📻\n\nMinu salajane raadiojaam saadab koodi morsetähestikus. Jaama sageduse kirjutasin peegelkirjas seinale – riiulil on käsipeegel.\n\n– Professor Nupp'],
  ['🎈 Mängutuba', 'Mängutuba! 🎈\n\nAva mänguasjakast õigete hoobadega (vaata maali) ja viska kolm palli korvi – aga ainult viskejoone tagant!\n\n– Professor Nupp']];

let P = {}, st = {}, mode = 'menu', elapsed = 0, entry = '', kpMsg = null;
function setupPuzzle(){
  const dc = DIFF[settings.diff];
  const lock = shuffle([0, 1, 2, 3, 4]).slice(0, 3), r = pick(RIDDLES);
  const cols = shuffle(['r', 'b', 'y']).slice(0, dc.colors), recipe = { r: 0, b: 0, y: 0 };
  cols.forEach(c => recipe[c] = randInt(1, dc.maxCount));
  P = {
    lock, d1: randInt(1, 9), d2: randInt(3, 8), d3: r[1], riddle: r[0], d4: randInt(1, 9), globeK: randInt(1, 5),
    W: randInt(6, 12), recipe, flip: rand() < 0.5,
    dTele: randInt(1, 9), dPiano: randInt(1, 9), dShadow: randInt(1, 9), shadowK: randInt(0, 7),
    melody: Array.from({ length: dc.melody }, () => randInt(0, 6)),
    gearSpots: shuffle(GEAR_SPOTS.filter(g => roomIdx(g[2]) < settings.rooms)).slice(0, 3),
    valves: [randInt(0, 5), randInt(0, 5), randInt(0, 5)], freq: randInt(89, 107), radioCode: [randInt(0, 9), randInt(0, 9), randInt(0, 9)],
    levers: [0, 1, 2, 3].map(() => rand() < 0.5)
  };
  P.code = `${P.d1}${P.d2}${P.d3}${P.d4}`;
  P.safeCode = [P.dTele, P.dPiano, P.dShadow];
  st = {
    ring: lock.map(c => (c + randInt(1, 4)) % 5), drawerOpen: false, uvTaken: false, uvReveal: 0, uvFound: false,
    globeK: P.globeK, globeBusy: false, globeOpen: false, keyTaken: false, chestOpen: false, fuseIn: false, riddleRead: false,
    door: ROOMS.map(() => false), visited: ROOMS.map((_, i) => i === 0), pending: ROOMS.map(() => false),
    scaleDone: false, luupTaken: false, recipeRead: false, mix: { r: 0, b: 0, y: 0 }, potionBad: false, potionDone: false,
    crystalTaken: false, crystalIn: false, mirrorK: [0, 0, 0],
    vaseShaken: false, vaseBroken: false, lensTaken: false, lensIn: false, teleSeen: false,
    shadowK: (P.shadowK + randInt(2, 6)) % 8, shadowSeen: false, melPos: 0, melodyDone: false, pianoRead: false, playing: false, boxOpen: false,
    safeWheel: [0, 0, 0], safeOpen: false, goldTaken: false, gears: 0, hints: 0, timerOff: false, warned: {},
    torchTaken: false, valves: [0, 0, 0], seeded: false, water: 0, grown: false, mirrorTaken: false, mirrorRead: false, freq: 88, morseTok: 0, codeWheel: [0, 0, 0],
    levers: [false, false, false, false], toyOpen: false, balls: 0, quiz: ROOMS.map(() => ({ got: 0, done: false, q: null }))
  };
  do { st.valves = [randInt(0, 5), randInt(0, 5), randInt(0, 5)]; } while (st.valves.every((v, i) => v === P.valves[i]));
  do { st.levers = [0, 1, 2, 3].map(() => rand() < 0.5); } while (st.levers.every((v, i) => v === P.levers[i]));
  st.freq = P.freq > 98 ? randInt(88, 92) : randInt(104, 108);
  st.quiz.forEach(q => q.q = genQuestion());
  do { st.mirrorK = [randInt(0, 3), randInt(0, 3), randInt(0, 3)]; } while (traceLaser(true).hit);
}
function resetItems(){
  for (const h of allHands) h.held = null;
  for (const k in items) {
    const it = items[k], ia = it.obj.userData.ia;
    it.home.add(it.obj); it.obj.position.copy(it.pos); it.obj.position.y += ia.restOffset;
    it.obj.rotation.set(0, it.rotY, 0); it.obj.visible = !it.hidden;
  }
  const ws = shuffle(WEIGHT_SPOTS.slice());
  weightObjs.forEach((o, i) => { roomG[1].add(o); o.position.set(ws[i][0], 0.85 + o.userData.ia.restOffset, ws[i][1]); o.rotation.set(0, rand() * 0.6 - 0.3, 0); });
  const ps = shuffle(POTION_SPOTS.slice());
  potionObjs.forEach((o, i) => { roomG[1].add(o); o.position.set(ps[i][0], 0.9 + o.userData.ia.restOffset, ps[i][1]); o.rotation.set(0, -Math.PI / 2, 0); });
}
function applyPuzzle(){
  buildBooks();
  rings.forEach((m, i) => { m.material.color.set(COLS[st.ring[i]][1]); m.rotation.x = 0; });
  drawer.position.x = 0;
  drawUVMsg(); uvMsg.material.opacity = 0;
  globeSpin.rotation.y = GLOBE_BASE + st.globeK * Math.PI / 3; globeTop.position.y = 0; globeTop.rotation.z = 0;
  lid.rotation.x = 0; keyInLock.visible = false; insertedFuse.visible = false;
  setLights(0); entry = ''; kpMsg = null; setKeypadPower(false);
  DOORS.forEach(d => d.pivot.rotation.y = 0); outLight.intensity = 0;
  const spots = shuffle(DUCK_SPOTS.slice());
  ducks.forEach((d, i) => { if (i < P.d2) { const s = spots[i]; d.position.set(s[0], s[1], s[2]); d.rotation.y = s[3] + (rand() - 0.5) * 0.6; d.visible = true; } else d.visible = false; });
  hourHand.rotation.z = -P.d4 * Math.PI / 6; minuteHand.rotation.z = 0;
  labDrawer.position.x = 0; vitDoor.rotation.y = 0; scaleTheta = 0.2;
  drawRecipe(); updateLiquid(); drawFlaskInfo();
  const sx = P.flip ? 1 : -1;
  laserEm.position.set(sx * 2.85, 1.2, -4.5); laserEm.rotation.y = P.flip ? Math.PI : 0; emCrystal.visible = false;
  const mpos = [[-sx * 1.6, -4.5], [-sx * 1.6, -7.6], [sx * 1.4, -7.6]];
  mirrors.forEach((m, i) => { m.g.position.set(mpos[i][0], 0, mpos[i][1]); m.head.rotation.y = Math.PI / 2 - st.mirrorK[i] * Math.PI / 4; });
  receptor.position.set(sx * 1.4, 1.2, -8.96); recLens.material.color.set(0x440000); recLens.material.emissive.set(0x330000);
  beamSegs.forEach(b => b.visible = false); beamDot.visible = false;
  for (const s of shards) { s.parent.remove(s); s.geometry.dispose(); } shards.length = 0;
  teleLens.visible = false;
  turntable.rotation.y = st.shadowK * Math.PI / 4; drawShadow();
  mbLid.rotation.x = 0; mbGem.material.color.set(0x333333);
  xyDrawer.position.z = 0;
  safeDoor.rotation.y = 0; safeWheels.forEach((w, i) => drawWheel(i));
  goldInLock.visible = false;
  gears.forEach((g, i) => { const s = P.gearSpots[i]; roomAt(s[2]).add(g); g.position.set(s[0], s[1], s[2]); g.visible = true; });
  flaskInfo.mesh.visible = settings.diff !== 'raske';
  // toad 4–7
  const n = settings.rooms;
  outsideBox.position.set(0, 1.15, ROOMS[n - 1].z0 - 0.65); outLight.position.set(0, 1.6, ROOMS[n - 1].z0 - 0.6); exitSign.position.set(0, 2.5, ROOMS[n - 1].z0 + 0.016);
  DOORS.forEach((d, i) => d.pivot.visible = i < n);
  valves.forEach((v, i) => { v.rotation.x = 0; drawGauge(i); }); drawChalk();
  potDrawer.position.x = 0; plant.visible = false; plant.scale.setScalar(0.08); petals.forEach(p => p.rotation.z = 0);
  drawDial(); radioLamp.material.color.set(0x331100); drawMirrorText(); [0, 1, 2].forEach(drawCodeWheel);
  levers.forEach((g, i) => g.rotation.z = st.levers[i] ? 0.6 : Math.PI - 0.6); drawLeverPic(); toyLid.rotation.x = 0;
  basketLamps.forEach(l => l.material.color.set(0x333333)); ballObjs.forEach(b => b.userData.ia.scored = false);
  for (const f of flying) f.it.userData.ia.flying = false; flying.length = 0;
  quizPanels.forEach((p, i) => { p.g.visible = settings.learn !== 'off' && i < n; drawQuiz(i); });
  resetItems();
  rebuildColliders();
}

/* ---------- tuba 1 tegevused ---------- */
let lightV = 0;
const ROOM_LIGHT = [[0, 2.1, 0, 0xfff1dc, () => 9 * lightV], [0, 2.2, -6, 0xf2f8ff, () => 8], [0, 2.25, -12, 0xffc98a, () => 5], [0, 2.2, -18, 0x8899aa, () => 0], [0, 2.3, -24, 0xf4ffe8, () => 9], [0, 2.2, -30, 0xffd9a0, () => 5], [0, 2.2, -36, 0xffffff, () => 8]];
function setRoomLight(L, i){ if (i < 0) { L.intensity = 0; return; } const c = ROOM_LIGHT[i]; L.position.set(c[0], c[1], c[2]); L.color.set(c[3]); L.intensity = c[4](); }
function setLights(v){ lightV = v; M.ceil.color.set(0x3b352d).lerp(new THREE.Color(0xbdb5a3), clamp(v, 0, 1)); bulbMat.emissive.setRGB(1, 0.95, 0.8).multiplyScalar(0.12 + 0.88 * v); }
function setKeypadPower(on){
  ledMat.color.set(on ? 0x003300 : 0x330000); ledMat.emissive.set(on ? 0x00ff44 : 0x550000);
  kpButtons.forEach(b => b.material.color.set(on ? 0xffffff : 0x666666)); drawKeypad();
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
function clickRing(i){
  st.ring[i] = (st.ring[i] + 1) % 5;
  const m = rings[i], r0 = m.rotation.x;
  m.material.color.set(COLS[st.ring[i]][1]);
  tween(0.25, k => m.rotation.x = r0 + ease(k) * Math.PI * 2 / 5);
  tone(900, .03, 0, 'square', .06);
  if (st.ring.every((c, j) => c === P.lock[j])) { st.drawerOpen = true; delay(0.35, () => { sfx.unlock(); const amt = renderer.xr.isPresenting ? 0.07 : 0.42; notify(renderer.xr.isPresenting ? 'Klõps! 🔓 Tõmba sahtel käepidemest lahti.' : 'Klõps! Sahtel avanes 🔓'); tween(0.9, k => drawer.position.x = -amt * ease(k)); }); }
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
function consume(h){ const it = h.held; h.held = null; roomG[2].add(it); it.visible = false; return it; }
function useKey(h){ consume(h); st.chestOpen = true; keyInLock.visible = true; sfx.unlock(); notify('Kirst avanes! 📦', 3); tween(1.2, k => lid.rotation.x = 1.9 * ease(k)); }
function useFuse(h){
  consume(h); insertedFuse.visible = true; st.fuseIn = true;
  sfx.spark(); notify('Valgus tuli tagasi! ⚡ Klahvistik töötab.', 3.5);
  tween(1.6, k => setLights(k < 0.6 ? (Math.sin(k * 70) > 0.2 ? 0.7 : 0.05) : (k - 0.6) / 0.4), () => setLights(1));
  setKeypadPower(true);
}
function keyPress(k){
  if (!st.fuseIn) { sfx.bad(); notify('Klahvistikul pole voolu ⚡'); return; }
  if (st.door[0] || kpMsg) return;
  sfx.beep();
  if (k === 'C') entry = '';
  else if (k === 'OK') {
    if (entry.length < 4) notify('Kood on 4 numbrit');
    else if (entry === P.code) { kpMsg = { t: 'AVATUD', col: '#5dff7a' }; drawKeypad(); sfx.unlock(); notify('Kood õige! 🔓', 3); delay(0.5, () => requestDoor(0)); return; }
    else { kpMsg = { t: 'VALE', col: '#ff4d4d' }; entry = ''; sfx.bad(); delay(1.0, () => { kpMsg = null; drawKeypad(); }); }
  }
  else if (entry.length < 4) entry += k;
  drawKeypad();
}

/* ---------- tuba 2 tegevused ---------- */
function ryb(r, y, b){
  const C = [[1, 1, 1], [1, 0, 0], [1, 1, 0], [1, 0.5, 0], [0.163, 0.373, 0.6], [0.5, 0, 0.5], [0, 0.66, 0.2], [0.2, 0.094, 0.0]];
  const L = (a, b2, t) => a.map((v, i) => v + (b2[i] - v) * t);
  const x0 = L(L(C[0], C[1], r), L(C[2], C[3], r), y), x1 = L(L(C[4], C[5], r), L(C[6], C[7], r), y);
  return L(x0, x1, b);
}
function mixColor(m){ const mx = Math.max(m.r, m.y, m.b, 1), c = ryb(m.r / mx, m.y / mx, m.b / mx); return new THREE.Color(c[0], c[1], c[2]); }
function updateLiquid(){
  const tot = st.mix.r + st.mix.b + st.mix.y;
  liquid.visible = tot > 0;
  const s = Math.min(1, 0.18 + tot * 0.13);
  liquid.scale.set(1, s, 1); liquid.position.y = 0.14 - 0.092 * (1 - s);
  const col = st.potionBad ? new THREE.Color(0x5b4a2e) : mixColor(st.mix);
  liquid.material.color.copy(col);
  liquid.material.emissive.copy(col).multiplyScalar(st.potionDone ? 0.6 : 0.15);
  liquid.userData.baseEm = liquid.material.emissive.clone();
}
function drawFlaskInfo(){
  flaskInfo.draw((c, W, H) => {
    c.fillStyle = '#263238'; roundRect(c, 2, 2, W - 4, H - 4, 12); c.fill();
    c.fillStyle = '#fff'; c.font = `bold ${H * 0.4}px ${FONT}`; c.textAlign = 'left'; c.textBaseline = 'middle'; c.fillText('Kolvis:', W * 0.04, H * 0.52);
    ['r', 'b', 'y'].forEach((k, i) => { const x = W * (0.42 + i * 0.2); c.fillStyle = POT[k][1]; c.beginPath(); c.arc(x, H * 0.5, H * 0.22, 0, 7); c.fill(); c.fillStyle = '#fff'; c.fillText(String(st.mix[k]), x + H * 0.3, H * 0.52); });
  });
}
function drawRecipe(){
  recipeCard.draw((c, W, H) => {
    parchment(c, W, H); c.fillStyle = '#3b2a16'; c.textAlign = 'center'; c.textBaseline = 'middle';
    c.font = `bold ${H * 0.12}px ${FONT}`; c.fillText('🧪 Kristalli võlujook', W / 2, H * 0.14);
    c.font = `${H * 0.06}px ${FONT}`; c.fillText('Vala kolbi täpselt:', W / 2, H * 0.27);
    const ks = ['r', 'b', 'y'].filter(k => P.recipe[k] > 0);
    ks.forEach((k, i) => {
      const x = W * ((i + 0.5) / ks.length);
      c.fillStyle = POT[k][1]; c.beginPath(); c.arc(x, H * 0.5, H * 0.11, 0, 7); c.fill(); c.strokeStyle = '#3b2a16'; c.lineWidth = 4; c.stroke();
      c.fillStyle = '#3b2a16';
      if (st.recipeRead) { c.font = `bold ${H * 0.13}px ${FONT}`; c.fillText(`× ${P.recipe[k]}`, x, H * 0.74); }
      else { c.font = `${Math.max(5, H * 0.018)}px ${FONT}`; c.fillText(`× ${P.recipe[k]} tilka`, x, H * 0.68); }
    });
    c.font = `${H * 0.05}px ${FONT}`; c.fillStyle = '#6b4f2a'; c.fillText('Kui läheb valesti – keera punast kraani.', W / 2, H * 0.9);
  });
}
function addDose(ck){
  if (st.potionDone) return;
  if (st.potionBad) { sfx.bad(); notify('Kolb on vale seguga täis – keera punast kraani 🔴'); return; }
  st.mix[ck]++; sfx.pour(); updateLiquid(); drawFlaskInfo();
  const R = P.recipe;
  if (st.mix.r > R.r || st.mix.b > R.b || st.mix.y > R.y) { st.potionBad = true; updateLiquid(); delay(0.4, () => { sfx.gurgle(); notify('Pahh! Vale segu 🤢 Tühjenda kolb punasest kraanist.', 3.5); }); return; }
  if (st.mix.r === R.r && st.mix.b === R.b && st.mix.y === R.y) {
    st.potionDone = true; updateLiquid();
    delay(0.4, () => { sfx.reveal(); notify('Võlujook õnnestus! ✨ Vitriin avanes.', 3.5); const amt = renderer.xr.isPresenting ? 0.3 : 1.7; tween(1.0, k => vitDoor.rotation.y = -amt * ease(k)); });
  }
}
function drainFlask(){
  if (st.potionDone) { notify('Võlujook on valmis – ära seda raiska! ✨'); return; }
  st.mix = { r: 0, b: 0, y: 0 }; st.potionBad = false; sfx.gurgle(); updateLiquid(); drawFlaskInfo(); notify('Kolb tühjendatud 🔄');
}
function useCrystal(h){ consume(h); st.crystalIn = true; emCrystal.visible = true; sfx.laser(); notify('Laser töötab! 🔴 Keera peegleid.', 3); traceLaser(); }
function rotateMirror(i){
  st.mirrorK[i] = (st.mirrorK[i] + 1) % 4; sfx.rotate();
  const h = mirrors[i].head, r0 = h.rotation.y, r1 = Math.PI / 2 - st.mirrorK[i] * Math.PI / 4;
  let d = r1 - r0; while (d > Math.PI) d -= Math.PI * 2; while (d < -Math.PI) d += Math.PI * 2;
  tween(0.35, k => h.rotation.y = r0 + d * ease(k), () => traceLaser());
}
function traceLaser(dry = false){
  const sx = P.flip ? 1 : -1;
  const mp = [[-sx * 1.6, -4.5], [-sx * 1.6, -7.6], [sx * 1.4, -7.6]];
  const rx = sx * 1.4;
  let px = sx * 2.6, pz = -4.5, dx = -sx, dz = 0, last = -1, hit = false;
  const segs = [];
  for (let b = 0; b < 10; b++) {
    let type = 'wall', bm = -1;
    const tx = dx > 0 ? (2.97 - px) / dx : dx < 0 ? (-2.97 - px) / dx : Infinity;
    const tz = dz > 0 ? (-3.03 - pz) / dz : dz < 0 ? (-8.97 - pz) / dz : Infinity;
    let best = Math.min(tx, tz);
    mp.forEach(([mx, mz], i) => {
      if (i === last) return;
      const a = st.mirrorK[i] * Math.PI / 4, Tx = -Math.sin(a), Tz = Math.cos(a);
      const det = -dx * Tz + Tx * dz; if (Math.abs(det) < 1e-6) return;
      const ex = mx - px, ez = mz - pz;
      const t = (ex * -Tz + Tx * ez) / det, s = (dx * ez - dz * ex) / det;
      if (t > 1e-4 && Math.abs(s) <= 0.2 && t < best) { best = t; type = 'mirror'; bm = i; }
    });
    const nx = px + dx * best, nz = pz + dz * best;
    segs.push([px, pz, nx, nz]);
    if (type === 'mirror') {
      const a = st.mirrorK[bm] * Math.PI / 4, n0 = Math.cos(a), n1 = Math.sin(a), dn = dx * n0 + dz * n1;
      if (Math.abs(dn) < 0.1) break;
      dx -= 2 * dn * n0; dz -= 2 * dn * n1; px = nx; pz = nz; last = bm;
      const l = Math.hypot(dx, dz); dx /= l; dz /= l;
    } else { if (dz < 0 && Math.abs(nz + 8.97) < 0.05 && Math.abs(nx - rx) < 0.16) hit = true; break; }
  }
  if (dry) return { hit, segs };
  const on = st.crystalIn;
  beamSegs.forEach((g, i) => {
    const s = segs[i]; g.visible = on && !!s; if (!g.visible) return;
    const L = Math.hypot(s[2] - s[0], s[3] - s[1]);
    g.position.set((s[0] + s[2]) / 2, 1.2, (s[1] + s[3]) / 2); g.scale.set(Math.max(0.01, L), 1, 1); g.rotation.y = Math.atan2(-(s[3] - s[1]), s[2] - s[0]);
  });
  const e = segs[segs.length - 1]; beamDot.visible = on; if (on) beamDot.position.set(e[2], 1.2, e[3]);
  if (on && hit && !st.door[1]) {
    recLens.material.color.set(0x00ff66); recLens.material.emissive.set(0x00aa44);
    sfx.unlock(); notify('Kiir tabas vastuvõtjat! 🎯 Uks avaneb.', 3.5); delay(0.6, () => requestDoor(1));
  }
  return { hit, segs };
}
function updateScale(dt){
  const sumR = panR.children.filter(c => c.userData.ia && c.userData.ia.item === 'w').reduce((a, c) => a + c.userData.ia.kg, 0);
  const target = clamp((P.W - sumR) * 0.035, -0.22, 0.22);
  scaleTheta += (target - scaleTheta) * Math.min(1, dt * 3);
  beamG.rotation.x = scaleTheta;
  for (const p of pans) { const z0 = p.userData.z0; p.position.set(0, 0.46 - z0 * Math.sin(scaleTheta) - 0.22, z0 * Math.cos(scaleTheta)); }
  if (!st.scaleDone && sumR === P.W && Math.abs(scaleTheta) < 0.02) {
    st.scaleDone = true; sfx.unlock(); notify('Kaal on tasakaalus! ⚖️ Laua sahtel avanes.', 3.5);
    const amt = renderer.xr.isPresenting ? 0.06 : 0.35; tween(0.9, k => labDrawer.position.x = amt * ease(k));
  }
}

/* ---------- tuba 3 tegevused ---------- */
function breakVase(it, p){
  st.vaseBroken = true; it.visible = false; roomG[2].add(it);
  sfx.crash(); notify('Kõlin! 💥 Kildude vahel on lääts.', 3.5);
  for (let i = 0; i < 9; i++) { const s = D(new THREE.TetrahedronGeometry(0.02 + rand() * 0.025), M.porcelain, p.x + (rand() - 0.5) * 0.35, p.y + 0.01, p.z + (rand() - 0.5) * 0.35, roomAt(p.z)); s.rotation.set(rand() * 6, rand() * 6, rand() * 6); shards.push(s); }
  const lens = items.lens.obj; roomAt(p.z).add(lens); lens.position.set(p.x, p.y + lens.userData.ia.restOffset, p.z); lens.visible = true;
}
function useLens(h){ consume(h); st.lensIn = true; teleLens.visible = true; sfx.ok(); notify('Lääts on okulaaris! 🔭 Vaata teleskoobist.', 3); }
const SEG7 = { a: [[0, 0], [1, 0]], b: [[1, 0], [1, 1]], c: [[1, 1], [1, 2]], d: [[0, 2], [1, 2]], e: [[0, 1], [0, 2]], f: [[0, 0], [0, 1]], g: [[0, 1], [1, 1]] };
const DIGIT_SEGS = ['abcdef', 'bc', 'abged', 'abgcd', 'fgbc', 'afgcd', 'afgedc', 'abc', 'abcdefg', 'abcdfg'];
function viewTelescope(){
  st.teleSeen = true; sfx.reveal();
  openView((c, W, H) => {
    c.fillStyle = '#000'; c.fillRect(0, 0, W, H);
    const cx = W / 2, cy = H * 0.5, R = H * 0.46;
    c.save(); c.beginPath(); c.arc(cx, cy, R, 0, 7); c.clip();
    const g = c.createRadialGradient(cx, cy, 0, cx, cy, R); g.addColorStop(0, '#16205a'); g.addColorStop(1, '#03051a'); c.fillStyle = g; c.fillRect(0, 0, W, H);
    const sr = mulberry32(P.dTele * 77 + 3);
    for (let i = 0; i < 220; i++) { c.fillStyle = `rgba(255,255,255,${0.2 + sr() * 0.5})`; c.beginPath(); c.arc(sr() * W, sr() * H, sr() * 1.6 + 0.4, 0, 7); c.fill(); }
    const S7 = H * 0.2, ox = cx - S7 / 2, oy = cy - S7;
    const P2 = p => [ox + p[0] * S7 + (sr() - 0.5) * 8, oy + p[1] * S7 + (sr() - 0.5) * 8];
    const pts = []; c.strokeStyle = 'rgba(160,190,255,0.55)'; c.lineWidth = 3;
    for (const sg of DIGIT_SEGS[P.dTele]) { const [a, b] = SEG7[sg]; const pa = P2(a), pb = P2(b); c.beginPath(); c.moveTo(pa[0], pa[1]); c.lineTo(pb[0], pb[1]); c.stroke(); pts.push(pa, pb); }
    c.shadowColor = '#cfe0ff'; c.shadowBlur = 18; c.fillStyle = '#ffffff';
    for (const p of pts) { c.beginPath(); c.arc(p[0], p[1], 6, 0, 7); c.fill(); }
    c.restore();
    c.strokeStyle = '#555'; c.lineWidth = 14; c.beginPath(); c.arc(cx, cy, R, 0, 7); c.stroke();
    c.fillStyle = '#ddd'; c.font = `bold ${H * 0.045}px ${FONT}`; c.textAlign = 'left'; c.textBaseline = 'middle'; c.fillText('🔭 Läbi teleskoobi', W * 0.03, H * 0.06);
  });
}
function rotateTurntable(){
  st.shadowK = (st.shadowK + 1) % 8; sfx.rotate();
  const r0 = turntable.rotation.y;
  tween(0.4, k => turntable.rotation.y = r0 + ease(k) * Math.PI / 4, () => { drawShadow(); if (st.shadowK === P.shadowK) { st.shadowSeen = true; if (settings.diff === 'kerge') notify('Vaata seinale – vari näeb välja nagu number! 🕯️', 3); } });
}
function drawShadow(){
  shadowCanvas.draw((c, W, H) => {
    const g = c.createRadialGradient(W / 2, H / 2, 0, W / 2, H / 2, W / 2); g.addColorStop(0, 'rgba(255,220,150,0.5)'); g.addColorStop(1, 'rgba(255,220,150,0)'); c.fillStyle = g; c.fillRect(0, 0, W, H);
    const d = ((st.shadowK - P.shadowK) % 8 + 8) % 8, ang = d * Math.PI / 4, cs = Math.cos(ang);
    const sx = (cs < 0 ? -1 : 1) * Math.max(0.1, Math.abs(cs)), sk = Math.sin(ang) * 0.45;
    c.save(); c.translate(W / 2, H * 0.53); c.transform(sx, 0, sk, 1, 0, 0);
    c.filter = d === 0 ? 'blur(3px)' : 'blur(6px)'; c.fillStyle = 'rgba(15,8,0,0.85)';
    c.font = `bold ${H * 0.6}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText(String(P.dShadow), 0, 0);
    if (d !== 0) { const r = mulberry32(d * 13 + P.dShadow); for (let i = 0; i < 4; i++) { c.lineWidth = 10 + r() * 14; c.strokeStyle = 'rgba(15,8,0,0.8)'; c.beginPath(); c.moveTo((r() - 0.5) * W * 0.5, (r() - 0.5) * H * 0.6); c.quadraticCurveTo((r() - 0.5) * W * 0.6, (r() - 0.5) * H * 0.6, (r() - 0.5) * W * 0.5, (r() - 0.5) * H * 0.6); c.stroke(); } }
    c.restore(); c.filter = 'none';
  });
}
function playMelody(){
  if (st.playing) return;
  st.playing = true; st.melPos = 0;
  if (!st.boxOpen) { st.boxOpen = true; tween(0.6, k => mbLid.rotation.x = -1.6 * ease(k)); }
  P.melody.forEach((n, i) => delay(0.7 + i * 0.65, () => { sfx.note(n); mbGem.material.color.set(XY_COLS[n]); delay(0.4, () => mbGem.material.color.set(0x333333)); }));
  delay(0.8 + P.melody.length * 0.65, () => { st.playing = false; if (!st.melodyDone) notify('Mängi sama meloodia ksülofonil 🎶', 3); });
}
function hitBar(i){
  sfx.note(i);
  const m = xyBars[i]; tween(0.2, k => m.position.y = 0.785 - Math.sin(k * Math.PI) * 0.008);
  if (st.melodyDone || st.playing || !st.boxOpen) return;
  if (i === P.melody[st.melPos]) {
    st.melPos++;
    if (st.melPos === P.melody.length) { st.melodyDone = true; delay(0.5, () => { sfx.reveal(); notify('Õige meloodia! 🎶 Karbike avanes.', 3); tween(0.6, k => xyDrawer.position.z = 0.11 * ease(k)); }); }
  } else { st.melPos = 0; delay(0.3, () => { sfx.bad(); notify('Vale noot – alusta meloodiat algusest 🎵'); }); }
}
function drawWheel(i){
  safeWheels[i].draw((c, W, H) => {
    const g = c.createLinearGradient(0, 0, 0, H); g.addColorStop(0, '#1a1a1a'); g.addColorStop(0.5, '#3a3a3a'); g.addColorStop(1, '#1a1a1a'); c.fillStyle = g; roundRect(c, 3, 3, W - 6, H - 6, 10); c.fill();
    c.strokeStyle = '#c9a13b'; c.lineWidth = 5; c.stroke();
    c.fillStyle = '#f5f5f5'; c.font = `bold ${H * 0.6}px monospace`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText(String(st.safeWheel[i]), W / 2, H * 0.55);
  });
}
function trySafe(){
  if (st.safeWheel.every((v, i) => v === P.safeCode[i])) {
    st.safeOpen = true; sfx.unlock(); notify('Seif avanes! 🗝️ Seal on kuldvõti.', 3.5);
    const amt = renderer.xr.isPresenting ? 0.3 : 1.9; tween(1.2, k => safeDoor.rotation.y = -amt * ease(k));
  } else { sfx.bad(); notify('Seif ei avane – vale kood 🔒'); }
}
function useGoldKey(h){
  consume(h); goldInLock.visible = true; sfx.unlock(); notify('Kuldvõti sobib! 🎉', 3);
  delay(0.6, () => requestDoor(2));
}
function collectGear(g){ g.visible = false; st.gears++; sfx.collect(); notify(`Kuldne hammasratas leitud! ⚙️ ${st.gears}/3`, 3); }

/* ---------- vihjed ---------- */
function currentHint(){
  const cur = st.door.findIndex((d, i) => !d && i < settings.rooms);
  if (cur < 0) return 'Uks on lahti – mine välja! 🎉';
  if (st.pending[cur] && !st.quiz[cur].done) return 'Mõistatus on lahendatud! Nüüd vasta ukse kõrval oleva teadmiste luku küsimustele 📚 (2 õiget vastust).';
  if (cur === 0) {
    if (!st.drawerOpen) return 'Laua sahtlil on kolme värvirõngaga lukk. Vaata raamaturiiulit: kolm raamatut on teistest ettepoole lükatud. Nende värvid ülevalt alla on rõngaste õiged värvid vasakult paremale.';
    if (!st.uvTaken) return 'Sahtel on lahti! Seal on UV-lamp. Võta see kätte.';
    if (!st.uvFound) return 'UV-lamp näitab nähtamatut kirja. Suuna lamp seintele – proovi ukse vastas olevat seina.';
    if (!st.globeOpen) return 'Klõpsa maakeral, et seda keerata. Kui punane täht vaatab toa keskele, maakera avaneb.';
    if (!st.keyTaken && !st.chestOpen) return 'Maakera sees on võti. Võta see kätte.';
    if (!st.chestOpen) return 'Võti avab toa nurgas oleva kirstu. Hoia võtit käes ja klõpsa kirstu lukul.';
    if (!st.fuseIn) return 'Kirstus on kaitse. Pane see ukse kõrval olevasse elektrikilpi.';
    if (!st.riddleRead) return 'Kirstus on ka paber mõistatusega. Loe seda.';
    return 'Ukse kohal olev silt ütleb numbrite järjekorra: 1) UV-kirja number, 2) mitu parti on toas, 3) mõistatuse vastus, 4) mitu tundi näitab kell. Pardid on peidus üle kogu toa!';
  }
  if (cur === 1) {
    if (!st.scaleDone) return 'Vasakul laual on kaal. Pane paremale kausile raskusi, kuni kaal on tasakaalus. Kaalu kallutus näitab, kumb pool on raskem.';
    if (!st.luupTaken) return 'Kaalulaua sahtel avanes – tõmba see lahti ja võta luup.';
    if (!st.recipeRead) return 'Hoia luupi käes ja suuna see seinal olevale retseptile.';
    if (!st.potionDone) return 'Vala kolbi pudelitest täpselt retsepti järgi: hoia pudelit käes ja klõpsa kolvil (VR-is kalluta pudelit kolvi kohal). Vale segu korral keera punast kraani.';
    if (!st.crystalTaken) return 'Vitriin on lahti – tõmba ust ja võta kristall 💎';
    if (!st.crystalIn) return 'Pane kristall seinal olevasse laserisse.';
    return 'Keera peegleid (klõpsa neil), et laserkiir jõuaks ukse kõrval olevasse vastuvõtjasse. Iga klõps keerab peeglit 45°.';
  }
  if (cur === 2) {
    if (!st.lensIn) { if (!st.vaseBroken) return 'Vaasis kõliseb midagi… Võta vaas ja lase sel põrandale kukkuda (või viska)!'; if (!st.lensTaken) return 'Kildude vahel on lääts. Võta see.'; return 'Pane lääts teleskoobi okulaari: hoia läätse käes ja klõpsa teleskoobil.'; }
    if (!st.teleSeen) return 'Vaata teleskoobist – tähed moodustavad numbri.';
    if (!st.melodyDone) return st.boxOpen ? 'Mängi muusikakarbi meloodia ksülofonil järele. Kalliskivi värv näitab, millist plaati lüüa. Kätega saad plaate sõrmega lüüa!' : 'Klõpsa muusikakarbil ja jäta värvide järjekord meelde.';
    if (!st.pianoRead) return 'Ksülofoni ees olev karbike avanes – tõmba see lahti ja loe kaarti.';
    if (!st.shadowSeen) return 'Keera laual olevat kuju. Ühes asendis näitab vari seinal numbrit.';
    if (!st.safeOpen) return 'Seifi kood: 🔭 teleskoobi number, 🎹 ksülofoni kaardi number, 🕯️ varju number. Muuda numbreid ja vajuta käepidet.';
    if (!st.goldTaken) return 'Tõmba seifi uks lahti ja võta kuldvõti 🗝️';
    return 'Kuldvõti avab suure ukse. Hoia võtit käes ja klõpsa ukse lukul.';
  }
  if (cur === 3) { if (!st.torchTaken) return 'Keldris on pime. Ukse kõrval tünni peal on taskulamp 🔦'; return 'Valgusta taskulambiga läänepoolset seina – seal on kriidiga kirjutatud õige surve. Keera ventiile (punane, roheline, sinine), kuni manomeetrid näitavad sama.'; }
  if (cur === 4) {
    if (!st.seeded) return 'Istutuslaua sahtlis on seemned – tõmba sahtel lahti, võta seemned ja pane need suurde potti.';
    if (!st.grown) return `Kasta taime kastekannuga (laual). Hoia kannu poti kohal ja kalluta – või klõpsa potil. Kastetud ${st.water}/3.`;
    return 'Lille sees on roheline võti – võta see ja pane ukse lukku.';
  }
  if (cur === 5) {
    if (st.freq !== P.freq) return 'Seinal on peegelkiri – loe seda riiulil oleva käsipeegliga. Seal on raadiojaama sagedus. Keera raadiot nuppudega ◀ ▶.';
    return 'Raadio piiksub morset. Tõlgi numbrid seinal oleva morsetabeli abil ja sisesta need ukse kõrval olevale lukule.';
  }
  if (!st.toyOpen) return 'Maal näitab, millised hoovad peavad olema üleval (🐦) ja millised all (🐟). Hoobadel on numbrid 1–4.';
  return `Viska pallid korvi viskejoone tagant! VR-is viska käega, arvutis vajuta F (sihi hiirega). Korvis ${st.balls}/3.`;
}
function showHint(){
  if (mode !== 'play') return;
  const max = DIFF[settings.diff].hints;
  if (st.hints >= max) { openReader('💡 Vihjed', `Vihjed on otsas (${DIFF[settings.diff].label}: ${max}). Sa saad hakkama! 💪`, false); return; }
  st.hints++; openReader(`💡 Vihje ${st.hints}${max < Infinity ? '/' + max : ''}`, currentHint(), false);
}

/* ---------- mängu olekud ---------- */
function hideMenus(){ menuUI.visible = false; pauseUI.visible = false; endUI.visible = false; closeReader(); }
function newGame(){
  tweens.length = 0; hideMenus();
  setupPuzzle(); applyPuzzle();
  elapsed = 0; mode = 'play';
  placePlayer(0, 1.3, 0);
  openReader('✉️ Kiri laual', startText());
}
function goMenu(){ tweens.length = 0; hideMenus(); mode = 'menu'; setupPuzzle(); applyPuzzle(); placePlayer(0, 1.3, 0); paintOpts(); placeInFront(menuUI, 1.4, -0.05); menuUI.visible = true; }
function openPause(){
  mode = 'pause'; closeReader();
  const tm = timerLeft();
  pausePanel.draw((c, W, H) => {
    panelBg(c, W, H); c.textAlign = 'center'; c.textBaseline = 'middle';
    c.fillStyle = '#ffd54f'; c.font = `bold ${H * 0.22}px ${FONT}`; c.fillText('⏸ Paus', W / 2, H * 0.22);
    c.fillStyle = '#fff'; c.font = `${H * 0.12}px ${FONT}`;
    c.fillText(`Aeg ${fmtTime(elapsed)}${tm != null ? ` • jäänud ${fmtTime(tm)}` : ''}`, W / 2, H * 0.5);
    const max = DIFF[settings.diff].hints;
    c.fillText(`Vihjeid ${st.hints}${max < Infinity ? '/' + max : ''} • ⚙️ ${st.gears}/3 • Tuba ${roomIdx(camZ()) + 1}/${settings.rooms}`, W / 2, H * 0.74);
  });
  pauseBtns[4].visible = renderer.xr.isPresenting;
  placeInFront(pauseUI, 1.1, -0.05); pauseUI.visible = true;
}
function closePause(){ if (mode !== 'pause') return; mode = 'play'; pauseUI.visible = false; }
function togglePause(){
  if (mode === 'play') { if (readerUI.visible) closeReader(); else openPause(); }
  else if (mode === 'pause') closePause();
}
function exitVR(){ const s = renderer.xr.getSession(); if (s) s.end(); }
function camZ(){ camera.getWorldPosition(head); return head.z; }
function timerLeft(){ return settings.timer > 0 && !st.timerOff ? settings.timer * 60 - elapsed : null; }
function win(){
  mode = 'win'; sfx.win(); closeReader();
  const stars = st.hints === 0 ? 3 : st.hints <= 3 ? 2 : 1;
  const b = best[bestKey()]; let rec = false;
  if (!b || elapsed < b.t) { best[bestKey()] = { t: elapsed, s: stars }; store('pt_best', best); rec = true; }
  const msg = stars === 3 ? 'Suurepärane! Tõeline detektiiv! 🕵️' : stars === 2 ? 'Väga tubli!' : 'Pääsesid välja – hästi tehtud!';
  endPanel.draw((c, W, H) => {
    panelBg(c, W, H); c.textAlign = 'center'; c.textBaseline = 'middle';
    c.fillStyle = '#ffd54f'; c.font = `bold ${H * 0.11}px ${FONT}`; c.fillText('🎉 Pääsesid majast välja!', W / 2, H * 0.13);
    c.font = `${H * 0.15}px ${FONT}`; c.fillText('★'.repeat(stars) + '☆'.repeat(3 - stars), W / 2, H * 0.33);
    c.fillStyle = '#fff'; c.font = `bold ${H * 0.07}px ${FONT}`; c.fillText(`Aeg ${fmtTime(elapsed)} • vihjeid ${st.hints} • ⚙️ ${st.gears}/3`, W / 2, H * 0.51);
    c.font = `${H * 0.06}px ${FONT}`; c.fillText(msg, W / 2, H * 0.63);
    c.fillStyle = rec ? '#80ff9a' : '#9fb3bf'; c.fillText(rec ? `🏆 Uus rekord (${DIFF[settings.diff].label})!` : `🏆 Rekord: ${fmtTime(best[bestKey()].t)}`, W / 2, H * 0.74);
  });
  setEndButtons([['🔁 Uus mäng', '#2e7d32', () => newGame()], ['🏠 Menüü', '#455a64', () => goMenu()], renderer.xr.isPresenting ? ['🚪 Välju VR-ist', '#b71c1c', () => exitVR()] : null]);
  placeInFront(endUI, 1.2, -0.05); endUI.visible = true;
}
function lose(){
  mode = 'lost'; sfx.lose(); closeReader();
  endPanel.draw((c, W, H) => {
    panelBg(c, W, H); c.textAlign = 'center'; c.textBaseline = 'middle';
    c.fillStyle = '#ff8a80'; c.font = `bold ${H * 0.13}px ${FONT}`; c.fillText('⌛ Aeg sai otsa!', W / 2, H * 0.2);
    c.fillStyle = '#fff'; c.font = `${H * 0.07}px ${FONT}`;
    wrapText(c, 'Professor Nupp naerab oma tähetornis… 😄\nKas mängid edasi ilma taimerita või alustad uuesti?', W / 2, H * 0.52, W * 0.85, H * 0.1);
  });
  setEndButtons([['▶ Jätka ilma taimerita', '#2e7d32', () => { st.timerOff = true; endUI.visible = false; mode = 'play'; }], ['🔁 Uus mäng', '#1565c0', () => newGame()], ['🏠 Menüü', '#455a64', () => goMenu()]]);
  placeInFront(endUI, 1.2, -0.05); endUI.visible = true;
}
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
const handFactory = new XRHandModelFactory();
const wristPause = makeButton('⏸', 0.06, 0.06, () => { if (mode === 'play') openPause(); else if (mode === 'pause') closePause(); }, { color: '#455a64', fs: 0.6 });
wristPause.visible = false; scene.add(wristPause);
for (let i = 0; i < 2; i++) {
  const ctrl = renderer.xr.getController(i), grip = renderer.xr.getControllerGrip(i), hand = renderer.xr.getHand(i);
  rig.add(ctrl); rig.add(grip); rig.add(hand);
  hand.add(handFactory.createHandModel(hand, 'spheres'));
  const holder = new THREE.Group(); holder.position.set(0, -0.01, -0.07); ctrl.add(holder);
  const line = new THREE.Line(new THREE.BufferGeometry().setFromPoints([new THREE.Vector3(), new THREE.Vector3(0, 0, -1)]), new THREE.LineBasicMaterial({ color: 0xffffff, transparent: true, opacity: 0.6 }));
  line.visible = false; ctrl.add(line);
  const dot = new THREE.Mesh(new THREE.SphereGeometry(0.008, 8, 8), new THREE.MeshBasicMaterial({ color: 0xffe066 })); dot.visible = false; scene.add(dot);
  const body = new THREE.Mesh(new THREE.BoxGeometry(0.035, 0.03, 0.11), new THREE.MeshLambertMaterial({ color: 0x2a2a2a })); body.position.z = 0.03; grip.add(body);
  const stream = new THREE.Mesh(new THREE.CylinderGeometry(0.005, 0.005, 1, 6).translate(0, -0.5, 0), new THREE.MeshBasicMaterial({ color: 0xffffff })); stream.visible = false; scene.add(stream);
  const h = { kind: 'xr', ctrl, grip, hand, holder, line, dot, body, stream, pourT: 0, held: null, bySqueeze: false, handedness: null, connected: false, isHand: false, drag: null, vel: new THREE.Vector3(), lastP: null, poking: null };
  ctrl.addEventListener('connected', e => { h.connected = true; h.handedness = e.data.handedness; h.isHand = !!e.data.hand; body.visible = !h.isHand; });
  ctrl.addEventListener('disconnected', () => { if (h.held) dropFall(h); h.connected = false; h.drag = null; });
  ctrl.addEventListener('selectstart', () => onSelectStart(h));
  ctrl.addEventListener('selectend', () => onSelectEnd(h));
  ctrl.addEventListener('squeezestart', () => squeeze(h, true));
  ctrl.addEventListener('squeezeend', () => squeeze(h, false));
  xrHands.push(h);
}
function handPoint(h, out){
  if (h.isHand && h.hand.joints) { const a = h.hand.joints['index-finger-tip'], b = h.hand.joints['thumb-tip']; if (a && b) { a.getWorldPosition(out); b.getWorldPosition(tmpV2); return out.add(tmpV2).multiplyScalar(0.5); } }
  return h.grip.getWorldPosition(out);
}
function nearItem(h, r){
  const p = handPoint(h, V3(0, 0, 0)); let best = null, bd = r;
  for (const k in items) { const o = items[k].obj; if (!iaOk(o)) continue; o.getWorldPosition(tmpV); const d = tmpV.distanceTo(p); if (d < bd) { bd = d; best = o; } }
  return best;
}
function nearDrag(h, r){
  const p = handPoint(h, V3(0, 0, 0)); let best = null, bd = r;
  for (const o of interactives) { const ia = o.userData.ia; if (!ia.drag || !iaOk(o) || (ia.dragOk && !ia.dragOk())) continue; o.getWorldPosition(tmpV); const d = tmpV.distanceTo(p); if (d < bd) { bd = d; best = o; } }
  return best;
}
function startDrag(h, root){
  const g = root.userData.ia.drag, p = handPoint(h, V3(0, 0, 0));
  h.drag = { root, p0: p, v0: g.get(), a0: 0 };
  if (g.kind === 'hinge') { const c = g.center(); h.drag.a0 = Math.atan2(p.z - c.z, p.x - c.x); }
  tone(400, 0.05, 0, 'triangle', 0.1); pulse(h, 0.3, 40);
}
function updateDrags(){
  for (const h of xrHands) {
    if (!h.drag) continue;
    const d = h.drag, g = d.root.userData.ia.drag, p = handPoint(h, tmpV);
    if (g.kind === 'slide') g.set(clamp(d.v0 + tmpD.copy(p).sub(d.p0).dot(g.dir()), g.min, g.max));
    else { const c = g.center(); let da = Math.atan2(p.z - c.z, p.x - c.x) - d.a0; while (da > Math.PI) da -= Math.PI * 2; while (da < -Math.PI) da += Math.PI * 2; g.set(clamp(d.v0 - da, g.min, g.max)); }
  }
}
function openSlide(root){
  const ia = root.userData.ia, g = ia.drag;
  if (ia.dragOk && !ia.dragOk()) { if (ia.lockedMsg) { sfx.bad(); notify(ia.lockedMsg); } return; }
  const v0 = g.get(), tgt = g.open ?? g.max, cl = g.closed ?? g.min, v1 = Math.abs(v0 - tgt) < 0.05 ? cl : tgt;
  tone(400, 0.05, 0, 'triangle', 0.1); tween(0.6, k => g.set(v0 + (v1 - v0) * ease(k)));
}
function pulse(h, intensity = 0.5, ms = 60){
  try { const s = renderer.xr.getSession(); if (!s) return; for (const src of s.inputSources) if (src.handedness === h.handedness && src.gamepad && src.gamepad.hapticActuators && src.gamepad.hapticActuators[0]) src.gamepad.hapticActuators[0].pulse(intensity, ms); } catch (e) {}
}
function teleportTo(h, p){
  camera.getWorldPosition(head);
  if (blocked(p.x, p.z)) { sfx.bad(); return false; }
  if (Math.hypot(p.x - head.x, p.z - head.z) > 7) return false;
  rig.position.x += p.x - head.x; rig.position.z += p.z - head.z; sweep(300, 700, 0.15, 'sine', 0.08); return true;
}
function onSelectStart(h){
  if (mode === 'play' && !h.held) {
    if (h.isHand) { const it = nearItem(h, 0.11); if (it) { grab(h, it, true); pulse(h); return; } }
    const nd = nearDrag(h, h.isHand ? 0.12 : 0.1); if (nd) { startDrag(h, nd); return; }
  }
  const r = cast(h);
  if (mode === 'play' && r && r.root && !h.held) {
    const ia = r.root.userData.ia;
    if (ia.drag && (!ia.dragOk || ia.dragOk())) { startDrag(h, r.root); return; }
    if (ia.grabbable && h.isHand) { grab(h, r.root, true); pulse(h); return; }
  }
  if (mode === 'play' && r && !r.root && !h.held && r.hit.face) {
    const n = r.hit.face.normal.clone().transformDirection(r.hit.object.matrixWorld);
    if (n.y > 0.7 && r.hit.point.y < 0.1) { teleportTo(h, r.hit.point); return; }
  }
  doAction(h, r);
  if (h.held) pulse(h);
}
function onSelectEnd(h){
  if (h.drag) { h.drag = null; return; }
  if (h.isHand && h.held && h.bySqueeze) dropFall(h);
}
function updatePoke(){
  for (const h of xrHands) {
    if (!h.connected || !h.isHand || !h.hand.joints) continue;
    const tipJ = h.hand.joints['index-finger-tip']; if (!tipJ) continue;
    tipJ.getWorldPosition(tmpV);
    let touched = null;
    for (const o of interactives) {
      const ia = o.userData.ia; if (!(ia.ui || ia.scaleHover || ia.poke) || !o.isMesh || !iaOk(o)) continue;
      if (o === wristPause && h.handedness === 'left') continue;
      const gp = o.geometry.parameters; if (!gp || !gp.width) continue;
      const p = o.worldToLocal(tmpV2.copy(tmpV));
      if (Math.abs(p.x) < gp.width / 2 + 0.006 && Math.abs(p.y) < gp.height / 2 + 0.006 && Math.abs(p.z) < (gp.depth || 0) / 2 + 0.018) { touched = o; break; }
    }
    if (touched && h.poking !== touched) { h.poking = touched; doAction(h, { root: touched, hit: null }); pulse(h, 0.4, 30); }
    else if (!touched) h.poking = null;
  }
  // randme paus-nupp vasakul käel
  const L = xrHands.find(h => h.connected && h.isHand && h.handedness === 'left');
  wristPause.visible = !!L && renderer.xr.isPresenting && (mode === 'play' || mode === 'pause');
  if (wristPause.visible) { const w = L.hand.joints && L.hand.joints['wrist']; if (w) { w.getWorldPosition(tmpV); w.getWorldQuaternion(tmpQ); wristPause.position.copy(tmpV).add(tmpD.set(0, 0.03, 0.06).applyQuaternion(tmpQ)); wristPause.quaternion.copy(tmpQ); wristPause.rotateX(-Math.PI / 2); } }
}
function trackHands(dt){
  for (const h of xrHands) {
    if (!h.connected) continue;
    const p = handPoint(h, V3(0, 0, 0));
    if (h.lastP) h.vel.lerp(tmpD.copy(p).sub(h.lastP).divideScalar(Math.max(dt, 1e-3)), 0.5);
    h.lastP = p;
  }
}
/* ---------- viskamine ja füüsika ---------- */
const flying = [];
function throwItem(it, p, v){
  camera.getWorldPosition(head);
  roomAt(p.z).add(it); it.position.copy(p); it.visible = true; it.userData.ia.flying = true;
  flying.push({ it, v, t: 0, maxY: p.y, fromZ: head.z });
  sweep(700, 250, 0.2, 'sine', 0.06);
}
function desktopThrow(){
  if (!mouseHand.held || mode !== 'play') return;
  const it = mouseHand.held; it.getWorldPosition(tmpV); const p = tmpV.clone();
  const r = cast(mouseHand); let target;
  if (r) target = r.hit.point.clone(); else { raycaster.setFromCamera(mouseNDC, camera); target = raycaster.ray.at(4, V3(0, 0, 0)); }
  const T = clamp(p.distanceTo(target) / 6, 0.35, 1.0);
  const v = target.sub(p).divideScalar(T); v.y += 0.5 * 9.8 * T;
  mouseHand.held = null; throwItem(it, p, v);
}
function updatePhysics(dt){
  for (let i = flying.length - 1; i >= 0; i--) {
    const f = flying[i], it = f.it, ia = it.userData.ia;
    f.t += dt; f.v.y -= 9.8 * dt;
    const p0 = it.position.clone(), step = f.v.clone().multiplyScalar(dt), len = step.length();
    if (ia.item === 'ball') {
      const nx = p0.x + step.x, ny = p0.y + step.y, nz = p0.z + step.z;
      if (Math.hypot(nx - BASKET.x, nz - BASKET.z) < BASKET.r - 0.05 && ny < BASKET.top && ny > 0.05 && f.v.y < 0 && p0.y >= BASKET.top - 0.15) { flying.splice(i, 1); ia.flying = false; ballInBasket(it, f); continue; }
    }
    let landed = null;
    if (len > 1e-5) {
      raycaster.set(p0, tmpD.copy(step).divideScalar(len)); raycaster.far = len + 0.04;
      const hit = raycaster.intersectObjects(statics.filter(shown), false).find(x => x.face);
      if (hit) {
        const n = hit.face.normal.clone().transformDirection(hit.object.matrixWorld);
        if (n.y > 0.5 || f.t > 3) landed = hit.point;
        else { f.v.sub(n.clone().multiplyScalar(2 * f.v.dot(n))).multiplyScalar(0.45); it.position.copy(hit.point).addScaledVector(n, 0.04); tone(180, 0.05, 0, 'triangle', 0.08); continue; }
      }
    }
    if (!landed && (f.t > 5 || p0.y < -0.5)) { landed = clampRoom(p0.clone()); landed.y = surfaceBelow(landed); }
    if (landed) { flying.splice(i, 1); ia.flying = false; placeItem(it, landed, rand() * 6.28, f.maxY); sfx.drop(); continue; }
    it.position.add(step); it.rotation.x += dt * 6; it.rotation.z += dt * 3;
    f.maxY = Math.max(f.maxY, it.position.y);
  }
}
function ballInBasket(it, f){
  if (f.fromZ > LINE_Z) {
    st.balls++; it.userData.ia.scored = true;
    roomG[6].add(it); it.position.set(BASKET.x + (rand() - 0.5) * 0.12, 0.08 + st.balls * 0.05, BASKET.z + (rand() - 0.5) * 0.12); it.rotation.set(0, 0, 0);
    basketLamps[st.balls - 1].material.color.set(0x76ff03); sfx.collect(); notify(`Korvis! 🏀 ${st.balls}/3`, 2.5);
    if (st.balls >= 3) delay(0.6, () => requestDoor(6));
  } else {
    notify('Viska joone tagant! 🏀', 3); sfx.bad();
    roomG[6].add(it); it.position.set(BASKET.x + 0.45, it.userData.ia.restOffset, BASKET.z + 0.3); it.rotation.set(0, 0, 0);
  }
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
  if (ia.flying) return false;
  return !isHeld(root);
}
const acceptsItem = (ia, item) => typeof ia.accepts === 'function' ? ia.accepts(item) : ia.accepts === item;
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
  const hits = raycaster.intersectObjects(list.concat(statics.filter(shown)), true);
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
      if (ia.accepts && acceptsItem(ia, item)) { ia.onUse(h); return; }
      if (ia.onClick) { ia.onClick(h); return; }
      if (ia.grabbable) { notify('Käsi on täis – pane ese enne maha'); return; }
      return;
    }
    if (ia.grabbable) { grab(h, root); return; }
    if (ia.onClick) ia.onClick(h);
  } else if (r && h.held) dropAt(h, r.hit);
}
function grab(h, root, bySqueeze = false){
  for (const o of allHands) if (o.held === root) o.held = null;
  const parent = root.parent;
  h.held = root; h.bySqueeze = bySqueeze;
  const ia = root.userData.ia;
  h.holder.add(root); root.position.copy(ia.holdPos); root.rotation.copy(ia.holdRot); root.visible = true;
  if (parent === panR) layoutPan(panR);
  sfx.pick();
  if (ia.onGrab) ia.onGrab();
}
function surfaceBelow(p){
  tmpV2.set(p.x, p.y + 0.05, p.z); raycaster.set(tmpV2, DOWN); raycaster.far = 4;
  for (const h of raycaster.intersectObjects(statics.filter(shown), false)) {
    if (!h.face) continue;
    const n = h.face.normal.clone().transformDirection(h.object.matrixWorld);
    if (n.y > 0.5) return h.point.y;
  }
  return 0;
}
function clampRoom(p){ p.x = clamp(p.x, -2.85, 2.85); const R = ROOMS[roomIdx(p.z)]; p.z = clamp(p.z, R.z0 + 0.15, R.z1 - 0.15); return p; }
function placeItem(it, p, yaw, fromY){
  const ia = it.userData.ia;
  if (ia.fragile && fromY != null && fromY - p.y > 0.45) { breakVase(it, p); return; }
  if (ia.item === 'seed' && mode === 'play') { pot.getWorldPosition(tmpV2); if (Math.hypot(p.x - tmpV2.x, p.z - tmpV2.z) < 0.35) { roomG[4].add(it); it.visible = false; st.seeded = true; plant.visible = true; plant.scale.setScalar(0.08); sfx.ok(); notify('Seeme on mullas 🌱 Nüüd kasta!', 3); return; } }
  if (ia.item === 'ball' && Math.hypot(p.x - BASKET.x, p.z - BASKET.z) < 0.4) { notify('Viska joone tagant! 🏀', 3); p = V3(BASKET.x + 0.45, 0, BASKET.z + 0.3); }
  for (const zn of zones) {
    zn.obj.getWorldPosition(tmpV2);
    if (Math.hypot(p.x - tmpV2.x, p.z - tmpV2.z) < zn.r && Math.abs(p.y - (tmpV2.y + zn.top)) < 0.12) {
      if (zn.reject) { notify('Vasak kauss on purgi jaoks – pane raskused paremale ⚖️'); p = V3(tmpV2.x + 0.2, 0.85, tmpV2.z + 0.05); break; }
      if (ia.item !== 'w') { notify('Kaalule pane ainult raskusi ⚖️'); p = V3(tmpV2.x + 0.2, 0.85, tmpV2.z - 0.1); break; }
      zn.obj.add(it); it.rotation.set(0, yaw, 0); it.visible = true; zn.layout(); return;
    }
  }
  roomAt(p.z).add(it); it.position.set(p.x, p.y + ia.restOffset, p.z); it.rotation.set(0, yaw, 0); it.visible = true;
}
function dropAt(h, hit){
  const it = h.held; it.getWorldPosition(tmpV); const fromY = tmpV.y;
  const n = hit.face ? hit.face.normal.clone().transformDirection(hit.object.matrixWorld) : UP;
  const p = hit.point.clone();
  if (n.y < 0.7) { p.addScaledVector(n, 0.12); clampRoom(p); p.y = surfaceBelow(p); } else clampRoom(p);
  h.held = null; placeItem(it, p, rand() * 6.28, fromY); sfx.drop();
}
function dropFall(h){
  const it = h.held; if (!it) return;
  it.getWorldPosition(tmpV); const fromY = tmpV.y;
  if (h.kind === 'xr' && mode === 'play' && h.vel && h.vel.length() > 1.3) { h.held = null; throwItem(it, tmpV.clone(), h.vel.clone().multiplyScalar(1.15)); return; }
  if (mode === 'play') for (const o of interactives) {
    const ia = o.userData.ia;
    if (ia.accepts && !ia.noAutoUse && acceptsItem(ia, it.userData.ia.item) && iaOk(o)) { o.getWorldPosition(tmpV2); if (tmpV2.distanceTo(tmpV) < 0.3) { ia.onUse(h); return; } }
  }
  const p = clampRoom(tmpV.clone()); p.y = surfaceBelow(p);
  h.held = null; placeItem(it, p, rand() * 6.28, fromY); sfx.drop();
}
function squeeze(h, down){
  if (mode !== 'play') return;
  if (down) {
    if (h.held) return;
    const nd = nearDrag(h, 0.1); if (nd) { startDrag(h, nd); return; }
    let best = nearItem(h, 0.18);
    if (!best) { const r = cast(h); if (r && r.root) { const ia = r.root.userData.ia; if (ia.drag && (!ia.dragOk || ia.dragOk())) { startDrag(h, r.root); return; } if (ia.grabbable) best = r.root; } }
    if (best) { grab(h, best, true); pulse(h); }
  } else { if (h.drag) { h.drag = null; return; } if (h.held && h.bySqueeze) dropFall(h); }
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

/* ---------- UV-lamp, luup, valamine ---------- */
function heldBy(obj){ return allHands.find(h => h.held === obj); }
function lensRay(obj, target, far){
  obj.getWorldPosition(tmpV); obj.getWorldQuaternion(tmpQ); tmpD.set(0, 0, -1).applyQuaternion(tmpQ);
  raycaster.set(tmpV, tmpD); raycaster.far = far;
  const hit = raycaster.intersectObjects([target, ...statics.filter(shown)], false).find(x => shown(x.object));
  return !!hit && hit.object === target;
}
function updateTools(dt){
  const uvHeld = !!heldBy(uvObj);
  beam.visible = uvHeld && mode === 'play';
  const on = beam.visible && shown(uvMsg) && lensRay(uvObj, uvMsg, 6);
  if (on) st.uvReveal = Math.min(1, st.uvReveal + dt * 1.3); else if (!st.uvFound) st.uvReveal = Math.max(0, st.uvReveal - dt * 0.8);
  if (st.uvReveal >= 1 && !st.uvFound) { st.uvFound = true; sfx.reveal(); notify('Salakiri! ✨ Loe seda seinalt.', 3); }
  uvMsg.material.opacity = st.uvFound ? (on ? 1 : 0.8) : st.uvReveal;
  if (!st.recipeRead && heldBy(luupObj) && mode === 'play' && shown(recipeCard.mesh) && lensRay(luupObj, recipeCard.mesh, 2.2)) {
    st.recipeRead = true; drawRecipe(); sfx.reveal(); notify('🔍 Luubiga näed kogused!', 3);
  }
  if (!st.mirrorRead && heldBy(handMirrorObj) && mode === 'play' && shown(mirrorText.mesh) && lensRay(handMirrorObj, mirrorText.mesh, 3)) {
    st.mirrorRead = true; drawMirrorText(); sfx.reveal(); notify('🪞 Peeglis on kiri loetav!', 3);
  }
  const tor = heldBy(torchObj);
  if (tor && mode === 'play') { torchObj.getWorldPosition(tmpV); torchObj.getWorldQuaternion(tmpQ); torchSpot.position.copy(tmpV); torchSpot.target.position.copy(tmpV).add(tmpD.set(0, 0, -1).applyQuaternion(tmpQ)); torchSpot.intensity = 30; }
  else torchSpot.intensity = 0;
  xrHands.forEach(h => {
    h.stream.visible = false;
    const it = h.held, pr = it && it.userData.ia.pour;
    if (!pr || mode !== 'play') { h.pourT = 0; return; }
    it.getWorldQuaternion(tmpQ); tmpD.set(0, 1, 0).applyQuaternion(tmpQ);
    if (tmpD.y > pr.tilt) { h.pourT = 0; return; }
    const neck = it.localToWorld(pr.neck.clone()), tg = pr.target();
    const over = !!tg && Math.hypot(neck.x - tg.x, neck.z - tg.z) < pr.r && neck.y > tg.y - 0.03 && neck.y < tg.y + 0.7;
    const len = over ? Math.max(0.02, neck.y - tg.y + 0.05) : Math.min(0.7, Math.max(0.05, neck.y - 0.05));
    h.stream.position.copy(neck); h.stream.scale.set(1, len, 1); h.stream.material.color.set(pr.color); h.stream.visible = true;
    if (over) { h.pourT += dt; if (h.pourT > 0.8) { h.pourT = 0; pr.dose(); pulse(h, 0.3, 40); } } else h.pourT = 0;
  });
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
    if (downBtn === 1) desktopThrow();
    else if (downBtn === 2) { if (mouseHand.held && mode === 'play') { if (r && !r.root) dropAt(mouseHand, r.hit); else dropFall(mouseHand); } }
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
addEventListener('keydown', e => { keys[e.code] = true; if (e.code === 'Escape') togglePause(); if (e.code === 'KeyF') desktopThrow(); });
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
const BASE_COLLIDERS = [
  [2.08, 3, -0.65, 1.05], [-3, -2.58, -1.55, 0.55], [1.62, 2.58, 2.22, 2.78], [-2.48, -1.92, 1.92, 2.48], [-2.65, -1.65, -2.65, -1.65], [2.3, 2.8, -2.8, -2.3],
  [2.28, 3, -6.7, -3.9], [-3, -2.28, -7.1, -5.5], [1.0, 2.8, -3.3, -3], [-2.8, -1.0, -3.3, -3],
  [1.8, 2.65, -12.6, -11.8], [-2.72, -2.28, -10.12, -9.68], [-2.78, -1.92, -12.55, -11.65], [1.98, 2.62, -10.17, -9.53], [-2.88, -1.52, -14.62, -13.78], [1.93, 2.57, -14.72, -14.18],
  [1.3, 1.9, -16.0, -15.4], [2.0, 2.6, -16.2, -15.6], [-2.6, -2.0, -20.6, -20.0], [-1.9, -1.3, -20.8, -20.2], [2.48, 3, -19.3, -16.7],
  [-3, -2.28, -24.45, -22.55], [0.85, 1.55, -25.15, -24.45], [2.0, 2.8, -22.8, -22.0], [2.4, 3, -26.6, -23.3],
  [2.1, 3, -30.65, -29.35], [-2.6, -1.6, -32.1, -31.1], [-3, -2.65, -29.5, -28.3],
  [2.7, 3, -35.9, -34.4], [1.75, 2.65, -38.6, -38.0], [-1.78, -1.22, -38.48, -37.92]
];
let COLLIDERS = BASE_COLLIDERS.slice();
function rebuildColliders(){ COLLIDERS = BASE_COLLIDERS.slice(); mirrors.forEach(m => { const x = m.g.position.x, z = m.g.position.z; COLLIDERS.push([x - 0.2, x + 0.2, z - 0.2, z + 0.2]); }); }
function blocked(x, z){
  const r = 0.22;
  let ok = false;
  for (const R of ROOMS.slice(0, settings.rooms)) if (x > -3 + r && x < 3 - r && z > R.z0 + r && z < R.z1 - r) ok = true;
  DOORS.forEach((d, i) => { if (st.door && st.door[i] && i < settings.rooms - 1 && Math.abs(x) < 0.33 && Math.abs(z - d.z) < 0.6) ok = true; });
  if (!ok) return true;
  for (const c of COLLIDERS) if (x > c[0] - r && x < c[1] + r && z > c[2] - r && z < c[3] + r) return true;
  return false;
}
let stepAcc = 0;
function move(dt){
  let mx = 0, my = 0;
  if (!renderer.xr.isPresenting) {
    if (keys.KeyW || keys.ArrowUp) my -= 1; if (keys.KeyS || keys.ArrowDown) my += 1;
    if (keys.KeyA) mx -= 1; if (keys.KeyD) mx += 1;
    if (keys.ArrowLeft) rig.rotation.y += 1.8 * dt; if (keys.ArrowRight) rig.rotation.y -= 1.8 * dt;
  } else { mx = moveInput.x; my = moveInput.y; }
  const len = Math.hypot(mx, my); if (len < 0.15) return;
  camera.getWorldDirection(fwd); fwd.y = 0; fwd.normalize();
  const sp = (keys.ShiftLeft ? 2.4 : 1.6) * Math.min(1, len) * dt;
  const dx = (fwd.x * -my + -fwd.z * mx) / len * sp, dz = (fwd.z * -my + fwd.x * mx) / len * sp;
  camera.getWorldPosition(head);
  const x0 = rig.position.x, z0 = rig.position.z;
  if (!blocked(head.x + dx, head.z + dz)) { rig.position.x += dx; rig.position.z += dz; }
  else if (!blocked(head.x + dx, head.z)) rig.position.x += dx;
  else if (!blocked(head.x, head.z + dz)) rig.position.z += dz;
  stepAcc += Math.hypot(rig.position.x - x0, rig.position.z - z0);
  if (stepAcc > 0.7) { stepAcc = 0; sfx.step(); }
}
function aimMouseHolder(){
  raycaster.setFromCamera(mouseNDC, camera);
  camera.getWorldQuaternion(tmpQ).invert();
  tmpD.copy(raycaster.ray.direction).applyQuaternion(tmpQ);
  mouseHolder.quaternion.setFromUnitVectors(NEG_Z, tmpD);
}
function updateRoomVis(){
  const n = settings.rooms, r = Math.min(roomIdx(camZ()), n - 1);
  const vis = ROOMS.map(() => false); vis[r] = true;
  for (let i = r; i < n - 1 && st.door[i]; i++) vis[i + 1] = true;
  for (let i = r - 1; i >= 0 && st.door[i]; i--) vis[i] = true;
  roomG.forEach((g, i) => g.visible = vis[i]);
  setRoomLight(ceilLight, r); setRoomLight(labLight, vis[r + 1] ? r + 1 : vis[r - 1] ? r - 1 : -1);
  if (mode === 'play' && !st.visited[r] && ROOM_TEXTS[r]) { st.visited[r] = true; delay(0.4, () => openReader(ROOM_TEXTS[r][0], ROOM_TEXTS[r][1])); }
  return r;
}
/* ---------- taustamuusika ---------- */
let musicT = 1;
const PENTA = [261.63, 293.66, 329.63, 392.0, 440.0, 523.25, 587.33];
function updateMusic(dt){
  if (!settings.music || !actx || (mode !== 'play' && mode !== 'menu')) return;
  musicT -= dt; if (musicT > 0) return;
  musicT = 1.6 + rand() * 1.8;
  const f = pick(PENTA); tone(f, 2.6, 0, 'sine', 0.03); if (rand() < 0.4) tone(f * 1.5, 2.2, 0.3, 'sine', 0.018);
  if (rand() < 0.15) tone(pick([65.41, 73.42, 98.0]), 5, 0, 'sine', 0.035);
}

/* ---------- VR sessioon ---------- */
const info = document.getElementById('info');
renderer.xr.addEventListener('sessionstart', () => {
  info.style.display = 'none'; camera.rotation.set(0, 0, 0); pitch = 0;
  if (mouseHand.held) { const it = mouseHand.held; mouseHand.held = null; it.getWorldPosition(tmpV); const p = clampRoom(tmpV.clone()); p.y = surfaceBelow(p); placeItem(it, p, 0); }
  setTimeout(() => {
    if (mode === 'menu') placePlayer(0, 1.3, 0);
    if (menuUI.visible) placeInFront(menuUI, 1.4, -0.05);
    if (pauseUI.visible) placeInFront(pauseUI, 1.1, -0.05);
    if (endUI.visible) placeInFront(endUI, 1.2, -0.05);
    if (readerUI.visible) placeInFront(readerUI, 0.9, -0.08);
  }, 400);
});
renderer.xr.addEventListener('sessionend', () => {
  info.style.display = ''; camera.position.set(0, 1.6, 0);
  for (const h of xrHands) { if (h.held) dropFall(h); h.stream.visible = false; }
  pauseBtns[4].visible = false;
});

/* =========================================================
   KÄIVITUS + PÕHITSÜKKEL
   ========================================================= */
setupPuzzle(); applyPuzzle();
placePlayer(0, 1.3, 0);
placeInFront(menuUI, 1.4, -0.05);
const clock = new THREE.Clock();
let hudT = 0;
renderer.setAnimationLoop(() => {
  const dt = Math.min(clock.getDelta(), 0.1), t = clock.elapsedTime;
  readXR();
  if (mode === 'play') {
    elapsed += dt; move(dt);
    const tl = timerLeft();
    if (tl != null) {
      if (tl <= 300 && !st.warned[5]) { st.warned[5] = true; notify('⏳ Jäänud 5 minutit!', 3); }
      if (tl <= 60 && !st.warned[1]) { st.warned[1] = true; notify('⏳ Jäänud 1 minut!', 3); }
      if (tl <= 0) lose();
    }
  }
  updateTweens(dt);
  trackHands(dt); updateDrags(); updatePhysics(dt);
  const curRoom = updateRoomVis();
  const hTgt = curRoom === 3 ? 0.12 : 1.1 + 1.1 * lightV; hemi.intensity += (hTgt - hemi.intensity) * Math.min(1, dt * 6); moon.intensity = curRoom === 3 ? 0 : 0.7;
  if (roomG[1].visible) updateScale(dt);
  if (!renderer.xr.isPresenting) aimMouseHolder();
  updateTools(dt);
  updateHands();
  updatePoke();
  updateMusic(dt);
  mobile.rotation.y += dt * 0.15;
  for (const g of gears) if (g.visible) { g.userData.m.rotation.y += dt * 1.5; g.userData.m.position.y = 0.045 + Math.sin(t * 2) * 0.01; }
  if (noticeT > 0) { noticeT -= dt; if (noticeT <= 0) notice.mesh.visible = false; }
  hudT -= dt;
  if (hudT <= 0) { hudT = 0.25; const tl = timerLeft(); hud.mesh.visible = mode === 'play' && tl != null; if (hud.mesh.visible) drawHud(`⏳ ${fmtTime(tl)}`, tl < 60); }
  renderer.render(scene, camera);
});
</script>
</body>
</html>
