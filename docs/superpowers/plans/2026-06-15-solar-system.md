# 3D Solar System Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file realistic 3D solar system with full interactivity, control panel, and info modals.

**Architecture:** Single HTML file containing all HTML structure, CSS styles, and JavaScript logic. Three.js loaded via CDN importmap. Planet textures generated procedurally via Canvas2D. All DOM-based UI overlays on top of WebGL canvas.

**Tech Stack:** Three.js r170 (CDN), OrbitControls (CDN), Canvas2D for textures, CSS frosted glass UI

---

## File Structure

Single file: `solar-system.html` at project root.

Internal code sections (clearly commented):
1. HTML structure (canvas + UI overlay)
2. CSS styles
3. JS: Imports & Constants
4. JS: Planet Data Config
5. JS: Scene/Lighting/Starfield Init
6. JS: Texture Generation
7. JS: Celestial Body Creation
8. JS: Orbit Ring Creation
9. JS: Animation Loop
10. JS: Raycaster Interaction
11. JS: Control Panel Handlers

---

### Task 1: HTML Structure

**Files:**
- Create: `solar-system.html`

- [ ] **Step 1: Create the complete HTML skeleton**

Write the file with HTML structure:

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>3D Solar System</title>
<link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&display=swap" rel="stylesheet">
<style>
/* CSS goes here — see Task 2 */
</style>
</head>
<body>
<canvas id="three-canvas"></canvas>

<div id="ui-layer">
  <!-- Control Panel -->
  <div id="control-panel">
    <div class="panel-title">◈ Solar System Control ◈</div>

    <div class="control-group">
      <button id="btn-toggle-orbits" class="ctrl-btn active">
        <span class="btn-icon">◎</span> 显示轨道
      </button>
    </div>

    <div class="control-group">
      <label class="ctrl-label">🔭 视角缩放</label>
      <div class="btn-row">
        <button id="btn-zoom-out" class="ctrl-btn small">⊖ 远</button>
        <button id="btn-zoom-default" class="ctrl-btn small active">◉ 默认</button>
        <button id="btn-zoom-in" class="ctrl-btn small">⊕ 近</button>
      </div>
    </div>

    <div class="control-group">
      <button id="btn-reset-view" class="ctrl-btn">
        <span class="btn-icon">↺</span> 重置视角
      </button>
    </div>

    <div class="control-group">
      <label class="ctrl-label">
        ⏱ 公转速度 <span id="speed-value">1.0x</span>
      </label>
      <input type="range" id="speed-slider" min="0" max="3" step="0.1" value="1.0">
      <div class="speed-labels">
        <span>暂停</span><span>慢速</span><span>正常</span><span>快速</span>
      </div>
    </div>

    <div class="control-group">
      <button id="btn-toggle-twinkle" class="ctrl-btn active">
        <span class="btn-icon">✨</span> 星光闪烁: 开
      </button>
    </div>
  </div>

  <!-- Info Modal -->
  <div id="info-modal" class="hidden">
    <div class="modal-header">
      <span id="modal-name" class="modal-name"></span>
      <button id="modal-close" class="modal-close">✕</button>
    </div>
    <div class="modal-body">
      <div class="modal-stat"><span class="stat-label">直径:</span> <span id="modal-diameter"></span></div>
      <div class="modal-stat"><span class="stat-label">自转周期:</span> <span id="modal-rotation"></span></div>
      <div class="modal-stat"><span class="stat-label">公转周期:</span> <span id="modal-orbital"></span></div>
      <div class="modal-desc" id="modal-desc"></div>
    </div>
  </div>
</div>

<script type="importmap">
{
  "imports": {
    "three": "https://unpkg.com/three@0.170.0/build/three.module.js",
    "three/addons/": "https://unpkg.com/three@0.170.0/examples/jsm/"
  }
}
</script>

<script type="module">
// All JS code goes here — see Tasks 3-11
</script>
</body>
</html>
```

---

### Task 2: CSS Styles

**Files:**
- Modify: `solar-system.html` (add CSS in `<style>` block)

- [ ] **Step 1: Add complete CSS styles**

```css
*, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }

body {
  width: 100vw; height: 100vh;
  overflow: hidden;
  background: #000;
  font-family: 'Orbitron', 'Microsoft YaHei', sans-serif;
  user-select: none;
}

#three-canvas {
  position: fixed; top: 0; left: 0;
  width: 100%; height: 100%;
  display: block;
}

#ui-layer {
  position: fixed; top: 0; left: 0;
  width: 100%; height: 100%;
  pointer-events: none;
  z-index: 10;
}

#ui-layer > * { pointer-events: auto; }

/* ===== Control Panel ===== */
#control-panel {
  position: absolute;
  top: 20px; right: 20px;
  width: 280px;
  padding: 20px 18px 16px;
  background: rgba(10, 15, 30, 0.75);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border: 1px solid rgba(80, 180, 255, 0.25);
  border-radius: 16px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.5), inset 0 1px 0 rgba(255,255,255,0.05);
  color: #c8e6ff;
  font-size: 13px;
  transition: border-color 0.3s;
}

#control-panel:hover {
  border-color: rgba(80, 180, 255, 0.5);
}

.panel-title {
  text-align: center;
  font-size: 14px;
  font-weight: 700;
  letter-spacing: 2px;
  color: #5cb8ff;
  padding-bottom: 14px;
  border-bottom: 1px solid rgba(80, 180, 255, 0.2);
  margin-bottom: 14px;
}

.control-group {
  margin-bottom: 12px;
}

.ctrl-label {
  display: block;
  font-size: 11px;
  color: #8ab8dc;
  margin-bottom: 6px;
  letter-spacing: 1px;
}

.ctrl-btn {
  display: flex; align-items: center; justify-content: center; gap: 6px;
  width: 100%;
  padding: 9px 14px;
  background: rgba(30, 60, 100, 0.4);
  border: 1px solid rgba(80, 180, 255, 0.3);
  border-radius: 10px;
  color: #a0d0f0;
  font-family: inherit; font-size: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
  letter-spacing: 1px;
}

.ctrl-btn:hover {
  background: rgba(50, 100, 180, 0.5);
  border-color: rgba(100, 200, 255, 0.6);
  color: #e0f0ff;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0, 100, 200, 0.3);
}

.ctrl-btn:active {
  transform: scale(0.96);
  transition: transform 0.08s ease;
}

.ctrl-btn.active {
  background: rgba(40, 100, 180, 0.55);
  border-color: rgba(100, 200, 255, 0.6);
  color: #fff;
  box-shadow: 0 0 16px rgba(60, 140, 220, 0.35);
}

.ctrl-btn.small {
  width: auto; flex: 1;
  padding: 7px 8px;
  font-size: 11px;
}

.btn-icon { font-size: 14px; }

.btn-row {
  display: flex; gap: 6px;
}

/* Speed Slider */
#speed-slider {
  width: 100%;
  height: 6px;
  -webkit-appearance: none;
  appearance: none;
  background: linear-gradient(90deg, #1a3a5c, #3a8ad4);
  border-radius: 3px;
  outline: none;
  cursor: pointer;
  margin: 6px 0 2px;
}

#speed-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 18px; height: 18px;
  background: #5cb8ff;
  border-radius: 50%;
  border: 2px solid #fff;
  cursor: pointer;
  box-shadow: 0 0 10px rgba(80, 180, 255, 0.6);
}

.speed-labels {
  display: flex; justify-content: space-between;
  font-size: 9px; color: #5a80a0;
  margin-top: 2px;
}

/* ===== Info Modal ===== */
#info-modal {
  position: absolute;
  top: 50%; left: 50%;
  transform: translate(-50%, -50%);
  min-width: 300px; max-width: 380px;
  background: rgba(10, 18, 35, 0.8);
  backdrop-filter: blur(24px);
  -webkit-backdrop-filter: blur(24px);
  border: 1px solid rgba(80, 180, 255, 0.35);
  border-radius: 16px;
  box-shadow: 0 12px 48px rgba(0, 0, 0, 0.6), 0 0 30px rgba(60, 140, 220, 0.15);
  color: #d0e8ff;
  overflow: hidden;
  transition: opacity 0.3s ease, transform 0.3s ease;
  z-index: 100;
}

#info-modal.hidden {
  opacity: 0;
  transform: translate(-50%, -50%) scale(0.9);
  pointer-events: none;
}

.modal-header {
  display: flex; align-items: center; justify-content: space-between;
  padding: 16px 20px 12px;
  border-bottom: 1px solid rgba(80, 180, 255, 0.2);
}

.modal-name {
  font-size: 20px; font-weight: 700;
  color: #5cb8ff;
  letter-spacing: 2px;
}

.modal-close {
  width: 30px; height: 30px;
  display: flex; align-items: center; justify-content: center;
  background: rgba(255,255,255,0.05);
  border: 1px solid rgba(255,255,255,0.15);
  border-radius: 50%;
  color: #8ab8dc;
  font-size: 14px; cursor: pointer;
  transition: all 0.2s;
}

.modal-close:hover {
  background: rgba(255,80,80,0.3);
  border-color: rgba(255,120,120,0.5);
  color: #fff;
}

.modal-body {
  padding: 16px 20px 20px;
  font-size: 13px;
  line-height: 1.8;
}

.modal-stat {
  display: flex; justify-content: space-between;
  padding: 4px 0;
  border-bottom: 1px dotted rgba(255,255,255,0.08);
}

.stat-label { color: #6a9fc0; }
.modal-desc {
  margin-top: 14px;
  padding-top: 12px;
  border-top: 1px solid rgba(80, 180, 255, 0.15);
  font-size: 12px; color: #a0c8e0;
  line-height: 1.7;
}
```

---

### Task 3: JS Imports & Planet Data Configuration

**Files:**
- Modify: `solar-system.html` (add inside `<script type="module">`)

- [ ] **Step 1: Add imports, constants, and planet data**

```javascript
// ============ IMPORTS ============
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

// ============ CONSTANTS ============
const SIZE_SCALE = 1.8;       // Visual scaling for better viewing
const ORBIT_SCALE = 0.35;     // Orbit spacing scale
const DISTANCE_SCALE = 1.0;   // Distance multiplier

// ============ PLANET DATA ============
const PLANET_DATA = [
  {
    name: '太阳', nameEn: 'Sun',
    radius: 5.0 * SIZE_SCALE,
    orbitRadius: 0,
    speed: 0,
    rotationSpeed: 0.004,
    color: '#ff8c00',
    type: 'star',
    diameter: '1,392,700 km',
    rotationPeriod: '25.05 地球日',
    orbitalPeriod: '—',
    desc: '太阳是太阳系的中心恒星，占太阳系总质量的99.86%。它通过核聚变将氢转化为氦，每秒释放的能量相当于数十亿颗核弹。表面温度约5,500°C，核心温度高达1,500万°C。'
  },
  {
    name: '水星', nameEn: 'Mercury',
    radius: 0.4 * SIZE_SCALE,
    orbitRadius: 8 * ORBIT_SCALE,
    speed: 4.15,
    rotationSpeed: 0.01,
    color: '#b0ada6',
    type: 'planet',
    diameter: '4,879 km',
    rotationPeriod: '58.6 地球日',
    orbitalPeriod: '88 地球日',
    desc: '水星是太阳系中最小的行星，也是离太阳最近的行星。表面布满了撞击坑，昼夜温差极大，白天可达430°C，夜晚降至-180°C。'
  },
  {
    name: '金星', nameEn: 'Venus',
    radius: 0.9 * SIZE_SCALE,
    orbitRadius: 12 * ORBIT_SCALE,
    speed: 3.0,
    rotationSpeed: 0.008,
    color: '#e8cda0',
    type: 'planet',
    diameter: '12,104 km',
    rotationPeriod: '243 地球日',
    orbitalPeriod: '225 地球日',
    desc: '金星是太阳系中最亮的行星，大小与地球相近。浓厚的大气层主要由二氧化碳组成，温室效应使表面温度高达465°C。金星是唯一逆向自转的行星。'
  },
  {
    name: '地球', nameEn: 'Earth',
    radius: 1.0 * SIZE_SCALE,
    orbitRadius: 16 * ORBIT_SCALE,
    speed: 2.4,
    rotationSpeed: 0.02,
    color: '#4da6ff',
    type: 'planet',
    diameter: '12,742 km',
    rotationPeriod: '24 小时',
    orbitalPeriod: '365.25 地球日',
    desc: '地球是太阳系中唯一已知存在生命的天体。拥有液态水、氧气大气层和适中的温度。地球的卫星——月球，帮助稳定了地球的自转轴倾角。'
  },
  {
    name: '火星', nameEn: 'Mars',
    radius: 0.5 * SIZE_SCALE,
    orbitRadius: 20 * ORBIT_SCALE,
    speed: 1.8,
    rotationSpeed: 0.018,
    color: '#c1440e',
    type: 'planet',
    diameter: '6,779 km',
    rotationPeriod: '24.6 小时',
    orbitalPeriod: '687 地球日',
    desc: '火星因其红色外观被称为"红色星球"，拥有太阳系最高的山——奥林帕斯山(21.9公里)。科学家认为火星曾经有液态水，是人类探索和移民的首要目标。'
  },
  {
    name: '木星', nameEn: 'Jupiter',
    radius: 3.0 * SIZE_SCALE,
    orbitRadius: 28 * ORBIT_SCALE,
    speed: 1.0,
    rotationSpeed: 0.04,
    color: '#d4b896',
    type: 'planet',
    diameter: '139,820 km',
    rotationPeriod: '9.9 小时',
    orbitalPeriod: '11.86 地球年',
    desc: '木星是太阳系中最大的行星，质量是其他行星总和的2.5倍。著名的大红斑是一个持续了350多年的巨大风暴。木星拥有79颗已知卫星。'
  },
  {
    name: '土星', nameEn: 'Saturn',
    radius: 2.5 * SIZE_SCALE,
    orbitRadius: 36 * ORBIT_SCALE,
    speed: 0.7,
    rotationSpeed: 0.035,
    color: '#e8d5a3',
    type: 'planet',
    diameter: '116,460 km',
    rotationPeriod: '10.7 小时',
    orbitalPeriod: '29.46 地球年',
    desc: '土星以其壮丽的环系统著称，由数十亿冰粒和岩石碎片组成。土星密度极低，理论上可以浮在水面上。土星的卫星土卫六拥有浓厚的大气层。'
  },
  {
    name: '天王星', nameEn: 'Uranus',
    radius: 1.8 * SIZE_SCALE,
    orbitRadius: 44 * ORBIT_SCALE,
    speed: 0.5,
    rotationSpeed: 0.03,
    color: '#7ec8e3',
    type: 'planet',
    diameter: '50,724 km',
    rotationPeriod: '17.2 小时',
    orbitalPeriod: '84.01 地球年',
    desc: '天王星是太阳系中最冷的行星之一，自转轴几乎平躺在公转平面上（倾斜约98°）。其蓝绿色外观来自大气中的甲烷吸收红光。'
  },
  {
    name: '海王星', nameEn: 'Neptune',
    radius: 1.7 * SIZE_SCALE,
    orbitRadius: 50 * ORBIT_SCALE,
    speed: 0.4,
    rotationSpeed: 0.028,
    color: '#3355ff',
    type: 'planet',
    diameter: '49,244 km',
    rotationPeriod: '16.1 小时',
    orbitalPeriod: '164.8 地球年',
    desc: '海王星是太阳系最远的行星，拥有太阳系最强的风暴，风速可达2,100公里/小时。海王星通过数学预测发现，是科学计算的一大胜利。'
  },
  {
    name: '月球', nameEn: 'Moon',
    radius: 0.27 * SIZE_SCALE,
    orbitRadius: 2.5,
    speed: 5.0,
    rotationSpeed: 0.005,
    color: '#c8c8c8',
    type: 'moon',
    parentIndex: 3,  // Earth index
    diameter: '3,474 km',
    rotationPeriod: '27.3 地球日',
    orbitalPeriod: '27.3 地球日',
    desc: '月球是地球唯一的天然卫星，通过潮汐力影响地球的海洋。月球表面布满了陨石坑，没有大气层。人类于1969年首次登陆月球。'
  }
];
```

---

### Task 4: Scene, Renderer, Camera, Lighting, Starfield

**Files:**
- Modify: `solar-system.html` (continue in `<script type="module">`)

- [ ] **Step 1: Initialize Three.js scene, renderer, camera, controls**

```javascript
// ============ SCENE INITIALIZATION ============
const canvas = document.getElementById('three-canvas');
const renderer = new THREE.WebGLRenderer({ canvas, antialias: true, alpha: false });
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.2;

const scene = new THREE.Scene();

const camera = new THREE.PerspectiveCamera(50, window.innerWidth / window.innerHeight, 0.5, 500);
camera.position.set(15, 25, 40);
camera.lookAt(0, 0, 0);

const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.08;
controls.minDistance = 5;
controls.maxDistance = 150;
controls.target.set(0, 0, 0);
controls.update();

// Store default camera state
const defaultCamera = {
  pos: camera.position.clone(),
  target: controls.target.clone()
};
```

- [ ] **Step 2: Add lighting**

```javascript
// ============ LIGHTING ============
const ambientLight = new THREE.AmbientLight(0x111133, 0.6);
scene.add(ambientLight);

const sunLight = new THREE.PointLight(0xfffde6, 300, 200, 0.5);
sunLight.position.set(0, 0, 0);
sunLight.castShadow = true;
sunLight.shadow.mapSize.width = 2048;
sunLight.shadow.mapSize.height = 2048;
sunLight.shadow.camera.near = 0.5;
sunLight.shadow.camera.far = 200;
sunLight.shadow.bias = -0.0001;
scene.add(sunLight);

const sunLight2 = new THREE.PointLight(0xffaa33, 150, 120, 0.6);
sunLight2.position.set(0, 0, 0);
scene.add(sunLight2);
```

- [ ] **Step 3: Generate starfield particle system**

```javascript
// ============ STARFIELD ============
let twinkleEnabled = true;
const starLayers = [];

function createStarLayer(count, size, spread, colorVariation) {
  const geo = new THREE.BufferGeometry();
  const positions = new Float32Array(count * 3);
  const colors = new Float32Array(count * 3);
  const sizes = new Float32Array(count);

  for (let i = 0; i < count; i++) {
    const theta = Math.random() * Math.PI * 2;
    const phi = Math.acos(2 * Math.random() - 1);
    const r = spread * (0.7 + Math.random() * 0.3);
    positions[i * 3] = r * Math.sin(phi) * Math.cos(theta);
    positions[i * 3 + 1] = r * Math.sin(phi) * Math.sin(theta);
    positions[i * 3 + 2] = r * Math.cos(phi);

    const brightness = 0.5 + Math.random() * 0.5;
    const tint = colorVariation * (Math.random() - 0.5);
    colors[i * 3] = brightness + tint;
    colors[i * 3 + 1] = brightness + tint * 0.5;
    colors[i * 3 + 2] = brightness;
    sizes[i] = size * (0.3 + Math.random() * 0.7);
  }

  geo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
  geo.setAttribute('color', new THREE.BufferAttribute(colors, 3));
  geo.setAttribute('size', new THREE.BufferAttribute(sizes, 1));

  const mat = new THREE.PointsMaterial({
    size: 0.15,
    vertexColors: true,
    blending: THREE.AdditiveBlending,
    depthWrite: false,
    transparent: true,
    opacity: 0.8,
    sizeAttenuation: true,
  });

  const points = new THREE.Points(geo, mat);
  return points;
}

const starLayer1 = createStarLayer(3000, 0.15, 90, 0.3);
const starLayer2 = createStarLayer(2000, 0.20, 70, 0.2);
const starLayer3 = createStarLayer(1000, 0.25, 50, 0.15);

scene.add(starLayer1);
scene.add(starLayer2);
scene.add(starLayer3);
starLayers.push(starLayer1, starLayer2, starLayer3);
```

---

### Task 5: Procedural Texture Generation

**Files:**
- Modify: `solar-system.html` (continue in `<script type="module">`)

- [ ] **Step 1: Add Canvas texture generator functions**

```javascript
// ============ TEXTURE GENERATION ============
function createCanvasTexture(width, height, drawFn) {
  const canvas = document.createElement('canvas');
  canvas.width = width;
  canvas.height = height;
  const ctx = canvas.getContext('2d');
  drawFn(ctx, width, height);
  const tex = new THREE.CanvasTexture(canvas);
  tex.wrapS = THREE.RepeatWrapping;
  tex.wrapU = THREE.RepeatWrapping;
  tex.colorSpace = THREE.SRGBColorSpace;
  return tex;
}

function sunTexture() {
  return createCanvasTexture(512, 256, (ctx, w, h) => {
    const gradient = ctx.createRadialGradient(w/2, h/2, 0, w/2, h/2, w/2);
    gradient.addColorStop(0, '#fff7a0');
    gradient.addColorStop(0.2, '#ffdd44');
    gradient.addColorStop(0.5, '#ff8800');
    gradient.addColorStop(0.8, '#ee4400');
    gradient.addColorStop(1, '#992200');
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, w, h);
    // Bright spots / solar flares
    for (let i = 0; i < 200; i++) {
      const x = Math.random() * w;
      const y = Math.random() * h;
      const r = 1 + Math.random() * 6;
      const alpha = 0.1 + Math.random() * 0.4;
      ctx.fillStyle = `rgba(255,255,200,${alpha})`;
      ctx.beginPath();
      ctx.arc(x, y, r, 0, Math.PI * 2);
      ctx.fill();
    }
    // Darker spots (sunspots)
    for (let i = 0; i < 30; i++) {
      const x = Math.random() * w;
      const y = Math.random() * h;
      const r = 1 + Math.random() * 4;
      ctx.fillStyle = `rgba(180,80,0,${0.2 + Math.random() * 0.3})`;
      ctx.beginPath();
      ctx.arc(x, y, r, 0, Math.PI * 2);
      ctx.fill();
    }
  });
}

function rockyPlanetTexture(baseColor, craterCount, stripeCount) {
  return createCanvasTexture(256, 128, (ctx, w, h) => {
    ctx.fillStyle = baseColor;
    ctx.fillRect(0, 0, w, h);
    // Surface noise
    for (let i = 0; i < 1000; i++) {
      const x = Math.random() * w;
      const y = Math.random() * h;
      ctx.fillStyle = `rgba(${Math.random() > 0.5 ? '255,255,255' : '0,0,0'},${0.02 + Math.random() * 0.06})`;
      ctx.fillRect(x, y, 1 + Math.random() * 3, 1 + Math.random() * 3);
    }
    // Craters
    for (let i = 0; i < craterCount; i++) {
      const x = Math.random() * w;
      const y = Math.random() * h;
      const r = 2 + Math.random() * 8;
      ctx.strokeStyle = `rgba(0,0,0,0.3)`;
      ctx.lineWidth = 1;
      ctx.beginPath();
      ctx.arc(x, y, r, 0, Math.PI * 2);
      ctx.stroke();
      ctx.fillStyle = `rgba(0,0,0,0.1)`;
      ctx.fill();
    }
  });
}

function gasGiantTexture(colors) {
  return createCanvasTexture(512, 256, (ctx, w, h) => {
    for (let y = 0; y < h; y++) {
      const t = y / h;
      const idx = t * (colors.length - 1);
      const i0 = Math.floor(idx);
      const i1 = Math.min(i0 + 1, colors.length - 1);
      const frac = idx - i0;
      const c = lerpColor(colors[i0], colors[i1], frac);
      ctx.fillStyle = c;
      ctx.fillRect(0, y, w, 1);
    }
    // Turbulence bands
    for (let i = 0; i < 500; i++) {
      const x = Math.random() * w;
      const y = Math.random() * h;
      ctx.fillStyle = `rgba(${Math.random() > 0.5 ? '255,255,255' : '0,0,0'},${0.03 + Math.random() * 0.08})`;
      ctx.fillRect(x, y, Math.random() * 30, 1 + Math.random() * 3);
    }
    // Oval storm spots (like Jupiter's Great Red Spot)
    if (colors.length > 4) {
      const sx = w * 0.4 + Math.random() * w * 0.2;
      const sy = h * 0.4 + Math.random() * h * 0.2;
      ctx.fillStyle = 'rgba(200,100,60,0.5)';
      ctx.beginPath();
      ctx.ellipse(sx, sy, 20, 12, 0, 0, Math.PI * 2);
      ctx.fill();
      ctx.strokeStyle = 'rgba(255,200,150,0.4)';
      ctx.lineWidth = 2;
      ctx.stroke();
    }
  });
}

function lerpColor(c1, c2, t) {
  const r1 = parseInt(c1.slice(1,3), 16);
  const g1 = parseInt(c1.slice(3,5), 16);
  const b1 = parseInt(c1.slice(5,7), 16);
  const r2 = parseInt(c2.slice(1,3), 16);
  const g2 = parseInt(c2.slice(3,5), 16);
  const b2 = parseInt(c2.slice(5,7), 16);
  const r = Math.round(r1 + (r2 - r1) * t);
  const g = Math.round(g1 + (g2 - g1) * t);
  const b = Math.round(b1 + (b2 - b1) * t);
  return `rgb(${r},${g},${b})`;
}

function earthTexture() {
  return createCanvasTexture(512, 256, (ctx, w, h) => {
    // Ocean base
    const oceanGrad = ctx.createLinearGradient(0, 0, 0, h);
    oceanGrad.addColorStop(0, '#2255aa');
    oceanGrad.addColorStop(0.3, '#3366cc');
    oceanGrad.addColorStop(0.5, '#4477dd');
    oceanGrad.addColorStop(0.7, '#3366cc');
    oceanGrad.addColorStop(1, '#2255aa');
    ctx.fillStyle = oceanGrad;
    ctx.fillRect(0, 0, w, h);
    // Continents (simplified blobs)
    const continents = [
      {x: w*0.15, y: h*0.35, rx: 60, ry: 50},
      {x: w*0.25, y: h*0.3, rx: 50, ry: 40},
      {x: w*0.5, y: h*0.4, rx: 70, ry: 55},
      {x: w*0.55, y: h*0.25, rx: 45, ry: 35},
      {x: w*0.7, y: h*0.45, rx: 55, ry: 60},
      {x: w*0.35, y: h*0.6, rx: 65, ry: 45},
      {x: w*0.8, y: h*0.35, rx: 40, ry: 50},
    ];
    continents.forEach(({x, y, rx, ry}) => {
      ctx.fillStyle = '#4a8c3f';
      ctx.beginPath();
      ctx.ellipse(x, y, rx, ry, Math.random() * 0.5, 0, Math.PI * 2);
      ctx.fill();
      // Inner variation
      ctx.fillStyle = '#5a9c4f';
      ctx.beginPath();
      ctx.ellipse(x + Math.random()*20-10, y + Math.random()*20-10, rx*0.7, ry*0.6, Math.random() * 0.4, 0, Math.PI * 2);
      ctx.fill();
    });
    // Ice caps
    ctx.fillStyle = 'rgba(240,245,255,0.7)';
    ctx.fillRect(0, 0, w, h * 0.08);
    ctx.fillRect(0, h * 0.92, w, h * 0.08);
    // Clouds
    for (let i = 0; i < 300; i++) {
      ctx.fillStyle = `rgba(255,255,255,${0.05 + Math.random() * 0.1})`;
      ctx.beginPath();
      ctx.arc(Math.random() * w, Math.random() * h, 3 + Math.random() * 15, 0, Math.PI * 2);
      ctx.fill();
    }
  });
}

function cloudTexture() {
  return createCanvasTexture(512, 256, (ctx, w, h) => {
    ctx.clearRect(0, 0, w, h);
    for (let i = 0; i < 400; i++) {
      const alpha = 0.05 + Math.random() * 0.25;
      ctx.fillStyle = `rgba(255,255,255,${alpha})`;
      ctx.beginPath();
      ctx.arc(Math.random() * w, Math.random() * h, 5 + Math.random() * 20, 0, Math.PI * 2);
      ctx.fill();
    }
  });
}

// Generate all textures
const textures = {
  sun: sunTexture(),
  mercury: rockyPlanetTexture('#a8a49e', 80, 0),
  venus: rockyPlanetTexture('#d4c8a0', 30, 0),
  earth: earthTexture(),
  clouds: cloudTexture(),
  mars: rockyPlanetTexture('#c1704a', 70, 0),
  jupiter: gasGiantTexture(['#d4b896','#c4946e','#e8d5b7','#b8845a','#d4b896','#c4946e','#a0704a','#d4b896']),
  saturn: gasGiantTexture(['#f0e0c0','#e8d5a3','#f5e8c8','#dcc890','#f0e0c0','#e8d5a3']),
  uranus: gasGiantTexture(['#a8d8ea','#7ec8e3','#b8e8f8','#6ab8d8','#a8d8ea']),
  neptune: gasGiantTexture(['#3355cc','#2244aa','#4466dd','#2244aa','#3355cc']),
  moon: rockyPlanetTexture('#c0c0c0', 120, 0),
};
```

---

### Task 6: Celestial Body Creation

**Files:**
- Modify: `solar-system.html` (continue in `<script type="module">`)

- [ ] **Step 1: Create sun, planets, moon, and their meshes**

```javascript
// ============ CELESTIAL BODY CREATION ============
const celestialBodies = [];  // For raycasting
const orbitGroups = [];      // For orbit animation
let orbitLines = [];         // For toggle

// Create Sun
const sunGeo = new THREE.SphereGeometry(PLANET_DATA[0].radius, 64, 64);
const sunMat = new THREE.MeshStandardMaterial({
  map: textures.sun,
  emissive: new THREE.Color('#ff6600'),
  emissiveIntensity: 2.0,
  roughness: 0.5,
});
const sun = new THREE.Mesh(sunGeo, sunMat);
sun.castShadow = false;
scene.add(sun);
celestialBodies.push({ mesh: sun, data: PLANET_DATA[0], orbitGroup: null });

// Sun glow sprite
const glowTex = createCanvasTexture(128, 128, (ctx, w, h) => {
  const g = ctx.createRadialGradient(w/2, h/2, 0, w/2, h/2, w/2);
  g.addColorStop(0, 'rgba(255,200,50,1)');
  g.addColorStop(0.15, 'rgba(255,150,20,0.8)');
  g.addColorStop(0.4, 'rgba(255,80,0,0.3)');
  g.addColorStop(0.7, 'rgba(255,40,0,0.05)');
  g.addColorStop(1, 'rgba(0,0,0,0)');
  ctx.fillStyle = g;
  ctx.fillRect(0, 0, w, h);
});
const glowSprite = new THREE.Sprite(new THREE.SpriteMaterial({
  map: glowTex,
  blending: THREE.AdditiveBlending,
  depthWrite: false,
  transparent: true,
  opacity: 0.7,
}));
glowSprite.scale.set(PLANET_DATA[0].radius * 5, PLANET_DATA[0].radius * 5, 1);
sun.add(glowSprite);

// Create planets (indices 1-8, excluding moon at index 9)
const planetMeshes = [];
for (let i = 1; i < PLANET_DATA.length - 1; i++) {
  const data = PLANET_DATA[i];
  const geo = new THREE.SphereGeometry(data.radius, 48, 48);

  let material;
  if (i === 3) { // Earth - special handling for cloud layer
    material = new THREE.MeshStandardMaterial({
      map: textures.earth,
      roughness: 0.7,
      metalness: 0.1,
    });
  } else {
    material = new THREE.MeshStandardMaterial({
      map: getTextureForPlanet(i),
      roughness: 0.8,
      metalness: 0.05,
    });
  }

  const mesh = new THREE.Mesh(geo, material);
  mesh.castShadow = true;
  mesh.receiveShadow = true;

  // Orbit group
  const orbitGroup = new THREE.Group();
  mesh.position.set(data.orbitRadius, 0, 0);
  orbitGroup.add(mesh);
  scene.add(orbitGroup);

  // Earth cloud layer
  if (i === 3) {
    const cloudGeo = new THREE.SphereGeometry(data.radius * 1.02, 48, 48);
    const cloudMat = new THREE.MeshStandardMaterial({
      map: textures.clouds,
      transparent: true,
      opacity: 0.4,
      roughness: 1,
      depthWrite: false,
    });
    const cloudMesh = new THREE.Mesh(cloudGeo, cloudMat);
    mesh.add(cloudMesh);
    mesh.userData.cloudMesh = cloudMesh;
  }

  // Saturn rings
  if (i === 6) {
    const ringGeo = new THREE.RingGeometry(data.radius * 1.4, data.radius * 2.3, 64);
    const ringTex = createCanvasTexture(512, 64, (ctx, w, h) => {
      const g = ctx.createLinearGradient(0, 0, w, 0);
      g.addColorStop(0, 'rgba(200,180,140,0)');
      g.addColorStop(0.15, 'rgba(210,190,150,0.9)');
      g.addColorStop(0.25, 'rgba(180,160,120,0.3)');
      g.addColorStop(0.35, 'rgba(220,200,160,0.95)');
      g.addColorStop(0.5, 'rgba(240,220,180,0.8)');
      g.addColorStop(0.6, 'rgba(190,170,130,0.6)');
      g.addColorStop(0.75, 'rgba(200,180,140,0.85)');
      g.addColorStop(0.9, 'rgba(160,140,100,0.2)');
      g.addColorStop(1, 'rgba(100,80,50,0)');
      ctx.fillStyle = g;
      ctx.fillRect(0, 0, w, h);
    });
    ringTex.colorSpace = THREE.SRGBColorSpace;
    const pos = ringGeo.attributes.position;
    const uv = ringGeo.attributes.uv;
    for (let j = 0; j < pos.count; j++) {
      const x = pos.getX(j);
      const y = pos.getY(j);
      const dist = Math.sqrt(x*x + y*y);
      uv.setXY(j, (dist - data.radius * 1.4) / (data.radius * 0.9), 0.5);
    }
    const ringMat = new THREE.MeshStandardMaterial({
      map: ringTex,
      side: THREE.DoubleSide,
      transparent: true,
      opacity: 0.8,
      roughness: 0.7,
      depthWrite: false,
    });
    const ring = new THREE.Mesh(ringGeo, ringMat);
    ring.rotation.x = Math.PI * 0.45;
    mesh.add(ring);
    mesh.userData.ring = ring;
  }

  planetMeshes.push(mesh);
  orbitGroups.push({ group: orbitGroup, speed: data.speed, mesh: mesh, rotSpeed: data.rotationSpeed });
  celestialBodies.push({ mesh, data, orbitGroup });
}

// Create Moon (orbits Earth)
const moonData = PLANET_DATA[9];
const moonGeo = new THREE.SphereGeometry(moonData.radius, 32, 32);
const moonMat = new THREE.MeshStandardMaterial({
  map: textures.moon,
  roughness: 0.9,
  metalness: 0,
});
const moon = new THREE.Mesh(moonGeo, moonMat);
moon.castShadow = true;
moon.receiveShadow = true;
moon.position.set(moonData.orbitRadius, 0, 0);
const moonOrbitGroup = new THREE.Group();
moonOrbitGroup.add(moon);
// Attach to Earth's orbit group
const earthOrbitGroup = orbitGroups[3].group;
earthOrbitGroup.add(moonOrbitGroup);
celestialBodies.push({ mesh: moon, data: moonData, orbitGroup: moonOrbitGroup });
// Store for animation
orbitGroups.push({ group: moonOrbitGroup, speed: moonData.speed, mesh: moon, rotSpeed: moonData.rotationSpeed, isMoon: true });

function getTextureForPlanet(index) {
  const keys = [null, 'mercury', 'venus', 'earth', 'mars', 'jupiter', 'saturn', 'uranus', 'neptune'];
  return textures[keys[index]] || textures.mercury;
}
```

---

### Task 7: Orbit Rings

**Files:**
- Modify: `solar-system.html` (continue in `<script type="module">`)

- [ ] **Step 1: Create orbit ring lines**

```javascript
// ============ ORBIT RINGS ============
function createOrbitRing(radius, color = 0x3366aa) {
  const segments = 256;
  const points = [];
  for (let i = 0; i <= segments; i++) {
    const angle = (i / segments) * Math.PI * 2;
    points.push(new THREE.Vector3(Math.cos(angle) * radius, 0, Math.sin(angle) * radius));
  }
  const geo = new THREE.BufferGeometry().setFromPoints(points);
  const mat = new THREE.LineBasicMaterial({
    color: color,
    transparent: true,
    opacity: 0.25,
    depthWrite: false,
  });
  const line = new THREE.Line(geo, mat);
  return line;
}

// Create orbits for all planets (indices 1-8)
for (let i = 1; i < PLANET_DATA.length - 1; i++) {
  const data = PLANET_DATA[i];
  const ring = createOrbitRing(data.orbitRadius, 0x4488cc);
  scene.add(ring);
  orbitLines.push(ring);
}
```

---

### Task 8: Raycaster Interaction & Modal

**Files:**
- Modify: `solar-system.html` (continue in `<script type="module">`)

- [ ] **Step 1: Implement raycaster click handling and modal logic**

```javascript
// ============ RAYCASTER INTERACTION ============
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();
let selectedBody = null;
let originalEmissive = null;

// Glow ring around selected body
const highlightRingGeo = new THREE.TorusGeometry(1, 0.15, 16, 64);
const highlightRingMat = new THREE.MeshBasicMaterial({
  color: 0x44aaff,
  transparent: true,
  opacity: 0.7,
  depthTest: false,
  depthWrite: false,
});
const highlightRing = new THREE.Mesh(highlightRingGeo, highlightRingMat);
highlightRing.visible = false;
scene.add(highlightRing);

function getIntersections(event) {
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

  raycaster.setFromCamera(mouse, camera);
  const targets = celestialBodies.map(cb => cb.mesh);
  return raycaster.intersectObjects(targets, false);
}

function selectBody(bodyEntry) {
  // Clear previous
  clearSelection();

  selectedBody = bodyEntry;
  const mesh = bodyEntry.mesh;

  // Highlight
  originalEmissive = mesh.material.emissive ? mesh.material.emissive.getHex() : 0;
  if (mesh.material.emissive) {
    mesh.material.emissive = new THREE.Color(0x3388ff);
    mesh.material.emissiveIntensity = (mesh.material.emissiveIntensity || 0) + 0.6;
  }

  // Glow ring
  const worldPos = new THREE.Vector3();
  mesh.getWorldPosition(worldPos);
  highlightRing.position.copy(worldPos);
  const scale = mesh.geometry.parameters?.radius || mesh.scale.x;
  highlightRing.scale.setScalar(scale * 1.3);
  highlightRing.visible = true;

  // Show modal
  showModal(bodyEntry.data, event);
}

function clearSelection() {
  if (selectedBody && originalEmissive !== null) {
    if (selectedBody.mesh.material.emissive) {
      selectedBody.mesh.material.emissive.setHex(originalEmissive);
      selectedBody.mesh.material.emissiveIntensity = Math.max(0, (selectedBody.mesh.material.emissiveIntensity || 1) - 0.6);
    }
  }
  selectedBody = null;
  originalEmissive = null;
  highlightRing.visible = false;
  hideModal();
}

canvas.addEventListener('click', (event) => {
  // Don't process if clicking UI elements
  if (event.target !== canvas) return;

  const intersects = getIntersections(event);
  if (intersects.length > 0) {
    const clickedMesh = intersects[0].object;
    const bodyEntry = celestialBodies.find(cb => cb.mesh === clickedMesh);

    if (bodyEntry) {
      if (selectedBody === bodyEntry) {
        clearSelection();
      } else {
        selectBody(bodyEntry);
      }
      return;
    }
  }
  // Click on empty space
  clearSelection();
});

// ============ MODAL LOGIC ============
const modal = document.getElementById('info-modal');
const modalName = document.getElementById('modal-name');
const modalDiameter = document.getElementById('modal-diameter');
const modalRotation = document.getElementById('modal-rotation');
const modalOrbital = document.getElementById('modal-orbital');
const modalDesc = document.getElementById('modal-desc');
const modalClose = document.getElementById('modal-close');

function showModal(data) {
  modalName.textContent = `${data.name} ${data.nameEn}`;
  modalDiameter.textContent = data.diameter;
  modalRotation.textContent = data.rotationPeriod;
  modalOrbital.textContent = data.orbitalPeriod;
  modalDesc.textContent = data.desc;
  modal.classList.remove('hidden');
}

function hideModal() {
  modal.classList.add('hidden');
}

modalClose.addEventListener('click', (e) => {
  e.stopPropagation();
  clearSelection();
});

// Prevent modal click from propagating to canvas
modal.addEventListener('click', (e) => {
  e.stopPropagation();
});
```

---

### Task 9: Animation Loop

**Files:**
- Modify: `solar-system.html` (continue in `<script type="module">`)

- [ ] **Step 1: Implement the requestAnimationFrame loop**

```javascript
// ============ ANIMATION LOOP ============
let timeScale = 1.0;
const clock = new THREE.Clock();

function animate() {
  requestAnimationFrame(animate);

  const delta = clock.getDelta();
  const scaledDelta = delta * timeScale;

  controls.update();

  // Rotate sun
  sun.rotation.y += 0.002 * timeScale;

  // Animate all orbiting bodies
  orbitGroups.forEach(og => {
    og.group.rotation.y += og.speed * 0.15 * scaledDelta;
    og.mesh.rotation.y += og.rotSpeed * scaledDelta;
  });

  // Rotate Earth clouds faster
  if (planetMeshes[3]?.userData.cloudMesh) {
    planetMeshes[3].userData.cloudMesh.rotation.y += 0.01 * scaledDelta;
  }

  // Star twinkle
  if (twinkleEnabled) {
    const time = Date.now() * 0.001;
    starLayer1.material.opacity = 0.7 + Math.sin(time * 1.7) * 0.1 + Math.cos(time * 2.3) * 0.05;
    starLayer2.material.opacity = 0.75 + Math.cos(time * 1.3) * 0.08 + Math.sin(time * 1.9) * 0.06;
    starLayer3.material.opacity = 0.8 + Math.sin(time * 0.9) * 0.05 + Math.cos(time * 1.5) * 0.07;
  }

  // Pulse highlight ring
  if (highlightRing.visible) {
    const pulse = 1 + Math.sin(Date.now() * 0.005) * 0.05;
    const baseScale = selectedBody?.mesh.geometry.parameters?.radius || 1;
    highlightRing.scale.setScalar(baseScale * 1.3 * pulse);
    highlightRing.material.opacity = 0.5 + Math.sin(Date.now() * 0.004) * 0.2;
  }

  renderer.render(scene, camera);
}

animate();
```

---

### Task 10: Control Panel Handlers

**Files:**
- Modify: `solar-system.html` (continue in `<script type="module">`)

- [ ] **Step 1: Wire up all control panel buttons and slider**

```javascript
// ============ CONTROL PANEL HANDLERS ============

// --- Orbit Toggle ---
let orbitsVisible = true;
const btnToggleOrbits = document.getElementById('btn-toggle-orbits');

btnToggleOrbits.addEventListener('click', (e) => {
  e.stopPropagation();
  orbitsVisible = !orbitsVisible;
  orbitLines.forEach(line => line.visible = orbitsVisible);
  btnToggleOrbits.classList.toggle('active', orbitsVisible);
  btnToggleOrbits.innerHTML = orbitsVisible
    ? '<span class="btn-icon">◎</span> 显示轨道'
    : '<span class="btn-icon">◌</span> 隐藏轨道';
});

// --- Zoom Presets ---
const zoomLevels = [
  { name: 'zoom-out', distance: 70, targetY: 0 },
  { name: 'zoom-default', distance: 40, targetY: 0 },
  { name: 'zoom-in', distance: 12, targetY: 0 },
];

const btnZoomOut = document.getElementById('btn-zoom-out');
const btnZoomDefault = document.getElementById('btn-zoom-default');
const btnZoomIn = document.getElementById('btn-zoom-in');
const zoomBtns = [btnZoomOut, btnZoomDefault, btnZoomIn];

function setZoom(level) {
  const config = zoomLevels[level];
  const dir = camera.position.clone().normalize();
  const newPos = dir.multiplyScalar(config.distance);
  // Smooth animate camera position
  const startPos = camera.position.clone();
  const startTime = Date.now();
  const duration = 800;

  function animStep() {
    const elapsed = Date.now() - startTime;
    const t = Math.min(elapsed / duration, 1.0);
    // Ease in-out
    const ease = t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t;
    camera.position.lerpVectors(startPos, newPos, ease);
    if (t < 1) {
      requestAnimationFrame(animStep);
    }
  }
  animStep();

  zoomBtns.forEach((b, i) => b.classList.toggle('active', i === level));
}

btnZoomOut.addEventListener('click', (e) => { e.stopPropagation(); setZoom(0); });
btnZoomDefault.addEventListener('click', (e) => { e.stopPropagation(); setZoom(1); });
btnZoomIn.addEventListener('click', (e) => { e.stopPropagation(); setZoom(2); });

// --- Reset View ---
const btnReset = document.getElementById('btn-reset-view');
btnReset.addEventListener('click', (e) => {
  e.stopPropagation();
  const startPos = camera.position.clone();
  const startTarget = controls.target.clone();
  const endPos = defaultCamera.pos.clone();
  const endTarget = defaultCamera.target.clone();
  const startTime = Date.now();
  const duration = 800;

  function animReset() {
    const elapsed = Date.now() - startTime;
    const t = Math.min(elapsed / duration, 1.0);
    const ease = t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t;
    camera.position.lerpVectors(startPos, endPos, ease);
    controls.target.lerpVectors(startTarget, endTarget, ease);
    if (t < 1) {
      requestAnimationFrame(animReset);
    }
  }
  animReset();
  setZoom(1); // Reset zoom buttons to default
});

// --- Speed Slider ---
const speedSlider = document.getElementById('speed-slider');
const speedValue = document.getElementById('speed-value');

speedSlider.addEventListener('input', (e) => {
  e.stopPropagation();
  timeScale = parseFloat(e.target.value);
  speedValue.textContent = timeScale.toFixed(1) + 'x';
});

// --- Twinkle Toggle ---
let twinkleOn = true;
const btnTwinkle = document.getElementById('btn-toggle-twinkle');

btnTwinkle.addEventListener('click', (e) => {
  e.stopPropagation();
  twinkleEnabled = !twinkleEnabled;
  btnTwinkle.classList.toggle('active', twinkleEnabled);
  btnTwinkle.innerHTML = twinkleEnabled
    ? '<span class="btn-icon">✨</span> 星光闪烁: 开'
    : '<span class="btn-icon">✦</span> 星光闪烁: 关';
  if (!twinkleEnabled) {
    starLayer1.material.opacity = 0.8;
    starLayer2.material.opacity = 0.8;
    starLayer3.material.opacity = 0.8;
  }
});

// Prevent panel clicks from propagating to canvas (but allow interaction)
document.getElementById('control-panel').addEventListener('click', (e) => {
  e.stopPropagation();
});
```

---

### Task 11: Window Resize Handler

**Files:**
- Modify: `solar-system.html` (continue in `<script type="module">`)

- [ ] **Step 1: Add resize handler**

```javascript
// ============ RESIZE HANDLER ============
window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
```

---

### Task 12: Final Assembly & Testing

**Files:**
- Modify: `solar-system.html`

- [ ] **Step 1: Verify all code sections are ordered correctly**

Check order: Imports → Constants → Planet Data → Scene Init → Lighting → Starfield → Texture Gen → Body Creation → Orbits → Raycaster → Animation → Panel Handlers → Resize

- [ ] **Step 2: Open in browser and verify**

Open `solar-system.html` in Chrome/Edge. Check:
- [ ] Starfield renders with twinkling
- [ ] Sun with glow visible at center
- [ ] 8 planets + moon orbiting at correct distances
- [ ] Orbits visible as semi-transparent rings
- [ ] Mouse drag rotates, scroll zooms, right-drag pans
- [ ] Click on planet → highlight ring + modal with data
- [ ] Click empty space → modal closes
- [ ] Re-click same planet → modal closes
- [ ] Orbit toggle button works
- [ ] Zoom preset buttons work with smooth animation
- [ ] Reset view restores default camera
- [ ] Speed slider changes revolution speed in real-time
- [ ] Twinkle toggle enables/disables star flicker
- [ ] No console errors
- [ ] Panel does not block 3D interaction
```

---

## Self-Review

### Spec Coverage Check
- ✅ Deep space background with layered twinkling stars → Task 4
- ✅ Realistic planet sizes/proportions → Task 3 (data), Task 6 (creation)
- ✅ Procedural textures with detail → Task 5
- ✅ Earth cloud layer → Task 6
- ✅ Saturn rings → Task 6
- ✅ Sun glow/flare → Task 6
- ✅ Orbit rings → Task 7
- ✅ Mouse controls → Task 4
- ✅ Click planet → info modal → Task 8
- ✅ Frosted glass modal → Task 2 (CSS)
- ✅ Highlight selected body → Task 8
- ✅ Control panel (all 5 features) → Task 10
- ✅ Smooth animations → Task 9, 10
- ✅ No local dependencies → CDN importmap

### Placeholder Scan
- No TBD/TODO found
- All code blocks are concrete
- All functions have implementations

### Type Consistency Check
- `celestialBodies` array used in Task 6 and Task 8 → consistent
- `orbitGroups` array used in Task 6 and Task 9 → consistent
- `orbitLines` array used in Task 7 and Task 10 → consistent
- `timeScale` defined in Task 9, used in Task 10 → consistent
- `twinkleEnabled` defined in Task 4, used in Task 9, 10 → consistent
- `planetMeshes` defined in Task 6, referenced in Task 9 → consistent
