# 3d-hand-gesture
Real-time 3D hand gesture recognition and spatial tracking system for touchless human-computer intreaction
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Gesture-Reactive 3D Particle System</title>
  <style>
    :root {
      --bg: #05070e;
      --panel-bg: rgba(12, 16, 28, 0.72);
      --panel-border: rgba(255, 255, 255, 0.12);
      --text-primary: #f1f5f9;
      --text-muted: #94a3b8;
      --accent: #38bdf8;
      --btn-bg: rgba(255, 255, 255, 0.06);
      --btn-hover: rgba(255, 255, 255, 0.14);
      --btn-active: rgba(56, 189, 248, 0.22);
      --radius: 14px;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    html,
    body {
      width: 100%;
      height: 100%;
      overflow: hidden;
      background-color: var(--bg);
      color: var(--text-primary);
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Inter, Helvetica, Arial, sans-serif;
      -webkit-font-smoothing: antialiased;
      user-select: none;
    }

    #canvas-container {
      position: fixed;
      inset: 0;
      width: 100vw;
      height: 100vh;
      z-index: 1;
    }

    #canvas-container canvas {
      display: block;
      width: 100%;
      height: 100%;
    }

    /* Control Panel Overlay */
    .control-panel {
      position: fixed;
      top: 20px;
      right: 20px;
      width: 268px;
      background: var(--panel-bg);
      border: 1px solid var(--panel-border);
      border-radius: var(--radius);
      backdrop-filter: blur(16px);
      -webkit-backdrop-filter: blur(16px);
      box-shadow: 0 16px 40px rgba(0, 0, 0, 0.45);
      z-index: 10;
      padding: 16px;
      transition: transform 0.25s ease, opacity 0.25s ease;
    }

    .control-panel.collapsed .panel-body {
      display: none;
    }

    .panel-header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 8px;
    }

    .title-wrap {
      display: flex;
      align-items: center;
      gap: 9px;
      min-width: 0;
    }

    .status-dot {
      width: 9px;
      height: 9px;
      border-radius: 50%;
      background: #f59e0b;
      box-shadow: 0 0 10px rgba(245, 158, 11, 0.75);
      flex-shrink: 0;
      transition: background 0.3s ease, box-shadow 0.3s ease;
    }

    .status-dot.active {
      background: #22c55e;
      box-shadow: 0 0 10px rgba(34, 197, 94, 0.85);
    }

    .status-dot.error {
      background: #ef4444;
      box-shadow: 0 0 10px rgba(239, 68, 68, 0.8);
    }

    .panel-title {
      font-size: 0.92rem;
      font-weight: 600;
      letter-spacing: 0.02em;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }

    .icon-btn {
      background: transparent;
      border: 1px solid rgba(255, 255, 255, 0.1);
      color: var(--text-muted);
      width: 26px;
      height: 26px;
      border-radius: 8px;
      cursor: pointer;
      display: grid;
      place-items: center;
      font-size: 0.85rem;
      transition: background 0.2s, color 0.2s;
    }

    .icon-btn:hover {
      background: var(--btn-hover);
      color: var(--text-primary);
    }

    .panel-body {
      margin-top: 14px;
      display: flex;
      flex-direction: column;
      gap: 14px;
    }

    .section-label {
      font-size: 0.7rem;
      text-transform: uppercase;
      letter-spacing: 0.1em;
      color: var(--text-muted);
      margin-bottom: 8px;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .template-grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 7px;
    }

    .template-btn {
      appearance: none;
      background: var(--btn-bg);
      border: 1px solid rgba(255, 255, 255, 0.08);
      border-radius: 10px;
      color: var(--text-primary);
      padding: 8px 10px;
      font-size: 0.8rem;
      font-weight: 500;
      cursor: pointer;
      display: flex;
      align-items: center;
      gap: 6px;
      transition: background 0.2s, border-color 0.2s, transform 0.15s;
    }

    .template-btn:hover {
      background: var(--btn-hover);
      border-color: rgba(255, 255, 255, 0.22);
    }

    .template-btn:active {
      transform: scale(0.97);
    }

    .template-btn.active {
      background: var(--btn-active);
      border-color: var(--accent);
      box-shadow: 0 0 14px rgba(56, 189, 248, 0.22);
    }

    .template-btn.span-full {
      grid-column: span 2;
      justify-content: center;
    }

    .color-row {
      display: flex;
      align-items: center;
      justify-content: space-between;
      background: var(--btn-bg);
      border: 1px solid rgba(255, 255, 255, 0.08);
      border-radius: 10px;
      padding: 6px 10px;
    }

    .color-left {
      display: flex;
      align-items: center;
      gap: 10px;
    }

    #color-picker {
      -webkit-appearance: none;
      appearance: none;
      width: 30px;
      height: 24px;
      border: none;
      border-radius: 6px;
      background: transparent;
      cursor: pointer;
      padding: 0;
    }

    #color-picker::-webkit-color-swatch-wrapper {
      padding: 0;
    }

    #color-picker::-webkit-color-swatch {
      border: 1px solid rgba(255, 255, 255, 0.35);
      border-radius: 6px;
    }

    #color-picker::-moz-color-swatch {
      border: 1px solid rgba(255, 255, 255, 0.35);
      border-radius: 6px;
    }

    .color-hex {
      font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace;
      font-size: 0.78rem;
      color: var(--text-primary);
      letter-spacing: 0.04em;
    }

    .swatches {
      display: flex;
      gap: 5px;
    }

    .swatch {
      width: 16px;
      height: 16px;
      border-radius: 50%;
      border: 1px solid rgba(255, 255, 255, 0.35);
      cursor: pointer;
      transition: transform 0.15s;
    }

    .swatch:hover {
      transform: scale(1.18);
    }

    /* Telemetry / Hand Gesture Readout */
    .telemetry-box {
      background: rgba(0, 0, 0, 0.25);
      border: 1px solid rgba(255, 255, 255, 0.07);
      border-radius: 10px;
      padding: 10px;
      display: flex;
      flex-direction: column;
      gap: 8px;
    }

    .status-text {
      font-size: 0.75rem;
      color: var(--text-muted);
      line-height: 1.35;
    }

    .meter-row {
      display: flex;
      flex-direction: column;
      gap: 4px;
    }

    .meter-labels {
      display: flex;
      justify-content: space-between;
      font-size: 0.7rem;
      color: var(--text-muted);
    }

    .meter-track {
      width: 100%;
      height: 5px;
      background: rgba(255, 255, 255, 0.1);
      border-radius: 999px;
      overflow: hidden;
    }

    .meter-fill {
      height: 100%;
      width: 50%;
      background: linear-gradient(90deg, #38bdf8, #818cf8);
      border-radius: 999px;
      transition: width 0.08s linear;
    }

    .footer-actions {
      display: flex;
      gap: 7px;
    }

    .small-btn {
      flex: 1;
      background: var(--btn-bg);
      border: 1px solid rgba(255, 255, 255, 0.08);
      border-radius: 8px;
      color: var(--text-muted);
      padding: 6px 8px;
      font-size: 0.72rem;
      cursor: pointer;
      transition: background 0.2s, color 0.2s;
    }

    .small-btn:hover {
      background: var(--btn-hover);
      color: var(--text-primary);
    }

    /* Webcam Preview PiP */
    .camera-pip {
      position: fixed;
      bottom: 20px;
      right: 20px;
      width: 196px;
      height: 147px;
      border-radius: 12px;
      overflow: hidden;
      background: #0a0f1c;
      border: 1px solid var(--panel-border);
      box-shadow: 0 12px 30px rgba(0, 0, 0, 0.45);
      z-index: 9;
      transition: opacity 0.25s ease, transform 0.25s ease;
    }

    .camera-pip.hidden {
      opacity: 0;
      pointer-events: none;
      transform: translateY(10px);
    }

    #webcam-video {
      display: none;
    }

    #hand-overlay {
      width: 100%;
      height: 100%;
      display: block;
      object-fit: cover;
    }

    .pip-badge {
      position: absolute;
      bottom: 6px;
      left: 8px;
      font-size: 0.65rem;
      padding: 2px 7px;
      border-radius: 999px;
      background: rgba(5, 7, 14, 0.72);
      color: var(--text-primary);
      border: 1px solid rgba(255, 255, 255, 0.14);
      pointer-events: none;
    }

    /* Subtle Bottom-Left Hint */
    .hint-pill {
      position: fixed;
      bottom: 20px;
      left: 20px;
      z-index: 8;
      background: rgba(12, 16, 28, 0.55);
      border: 1px solid rgba(255, 255, 255, 0.08);
      backdrop-filter: blur(10px);
      -webkit-backdrop-filter: blur(10px);
      color: var(--text-muted);
      font-size: 0.73rem;
      padding: 8px 13px;
      border-radius: 999px;
      pointer-events: none;
    }

    @media (max-width: 600px) {
      .control-panel {
        top: 12px;
        right: 12px;
        width: calc(100vw - 24px);
        max-width: 300px;
      }
      .camera-pip {
        width: 140px;
        height: 105px;
        bottom: 12px;
        right: 12px;
      }
      .hint-pill {
        display: none;
      }
    }
  </style>
</head>
<body>
  <div id="canvas-container"></div>

  <!-- Control Panel -->
  <aside class="control-panel" id="control-panel">
    <div class="panel-header">
      <div class="title-wrap">
        <span class="status-dot" id="status-dot"></span>
        <span class="panel-title">Particle Studio</span>
      </div>
      <button class="icon-btn" id="toggle-panel-btn" title="Collapse panel">−</button>
    </div>

    <div class="panel-body">
      <div>
        <div class="section-label">
          <span>Shape Template</span>
        </div>
        <div class="template-grid" id="template-buttons">
          <button class="template-btn active" data-template="heart">♥ Hearts</button>
          <button class="template-btn" data-template="flower">✿ Flowers</button>
          <button class="template-btn" data-template="saturn">🪐 Saturn</button>
          <button class="template-btn" data-template="buddha">☸ Buddha</button>
          <button class="template-btn span-full" data-template="fireworks">✦ Fireworks</button>
        </div>
      </div>

      <div>
        <div class="section-label">
          <span>Particle Colour</span>
        </div>
        <div class="color-row">
          <div class="color-left">
            <input type="color" id="color-picker" value="#ff5c8a" aria-label="Particle colour" />
            <span class="color-hex" id="color-hex">#FF5C8A</span>
          </div>
          <div class="swatches">
            <button class="swatch" data-color="#ff5c8a" style="background:#ff5c8a" title="Rose"></button>
            <button class="swatch" data-color="#38bdf8" style="background:#38bdf8" title="Cyan"></button>
            <button class="swatch" data-color="#fbbf24" style="background:#fbbf24" title="Gold"></button>
            <button class="swatch" data-color="#a78bfa" style="background:#a78bfa" title="Violet"></button>
          </div>
        </div>
      </div>

      <div class="telemetry-box">
        <div class="status-text" id="tracking-status">Initializing webcam &amp; hand tracking…</div>
        <div class="meter-row">
          <div class="meter-labels">
            <span>Hand Tension (Closing)</span>
            <span id="tension-value">50%</span>
          </div>
          <div class="meter-track">
            <div class="meter-fill" id="tension-bar"></div>
          </div>
        </div>
        <div class="meter-row">
          <div class="meter-labels">
            <span>Scale / Expansion</span>
            <span id="scale-value">1.00×</span>
          </div>
          <div class="meter-track">
            <div class="meter-fill" id="scale-bar"></div>
          </div>
        </div>
      </div>

      <div class="footer-actions">
        <button class="small-btn" id="toggle-pip-btn">Hide Camera</button>
        <button class="small-btn" id="retry-cam-btn">Restart Cam</button>
      </div>
    </div>
  </aside>

  <!-- Mirrored Webcam & Hand Landmarks Preview -->
  <div class="camera-pip hidden" id="camera-pip">
    <video id="webcam-video" playsinline muted></video>
    <canvas id="hand-overlay" width="320" height="240"></canvas>
    <span class="pip-badge" id="pip-badge">0 hands</span>
  </div>

  <div class="hint-pill">
    Open hands to expand · Close fists to contract · Drag to orbit · Scroll to zoom
  </div>

  <!-- Three.js & OrbitControls via CDN -->
  <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
  <!-- MediaPipe Hands via CDN -->
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js"></script>

  <script>
    (() => {
      "use strict";

      /* ================================================================
         1. CONFIGURATION & STATE
         ================================================================ */
      const PARTICLE_COUNT = 10000;
      const MORPH_SPEED = 0.065;
      const GESTURE_SMOOTHING = 0.1;

      const TEMPLATE_DEFAULT_COLORS = {
        heart: "#ff5c8a",
        flower: "#f472b6",
        saturn: "#fbbf24",
        buddha: "#fcd34d",
        fireworks: "#38bdf8"
      };

      const state = {
        activeTemplate: "heart",
        userPickedColor: false,
        handsDetected: 0,
        lastHandSeenTime: 0,
        // tension: 0 = fully open / relaxed, 1 = tightly closed fist
        targetTension: 0.5,
        currentTension: 0.5,
        // openness: 1 - tension, drives scale & radial expansion
        targetGroupScale: 1.0,
        currentGroupScale: 1.0,
        targetExpansion: 1.0,
        currentExpansion: 1.0,
        // optional 2-hand spread bonus
        targetSpreadBonus: 0.0,
        currentSpreadBonus: 0.0,
        cameraActive: false,
        pipVisible: true
      };

      /* ================================================================
         2. DOM REFERENCES
         ================================================================ */
      const container = document.getElementById("canvas-container");
      const controlPanel = document.getElementById("control-panel");
      const togglePanelBtn = document.getElementById("toggle-panel-btn");
      const templateButtons = document.querySelectorAll(".template-btn");
      const colorPicker = document.getElementById("color-picker");
      const colorHex = document.getElementById("color-hex");
      const swatches = document.querySelectorAll(".swatch");
      const statusDot = document.getElementById("status-dot");
      const trackingStatus = document.getElementById("tracking-status");
      const tensionValueEl = document.getElementById("tension-value");
      const tensionBarEl = document.getElementById("tension-bar");
      const scaleValueEl = document.getElementById("scale-value");
      const scaleBarEl = document.getElementById("scale-bar");
      const cameraPip = document.getElementById("camera-pip");
      const webcamVideo = document.getElementById("webcam-video");
      const handOverlay = document.getElementById("hand-overlay");
      const overlayCtx = handOverlay.getContext("2d");
      const pipBadge = document.getElementById("pip-badge");
      const togglePipBtn = document.getElementById("toggle-pip-btn");
      const retryCamBtn = document.getElementById("retry-cam-btn");

      /* ================================================================
         3. THREE.JS SCENE SETUP
         ================================================================ */
      const scene = new THREE.Scene();
      scene.background = new THREE.Color(0x05070e);
      scene.fog = new THREE.FogExp2(0x05070e, 0.014);

      const camera = new THREE.PerspectiveCamera(
        55,
        window.innerWidth / window.innerHeight,
        0.1,
        500
      );
      camera.position.set(0, 1.2, 23);

      const renderer = new THREE.WebGLRenderer({
        antialias: true,
        powerPreference: "high-performance"
      });
      renderer.setPixelRatio(Math.min(window.devicePixelRatio || 1, 2));
      renderer.setSize(window.innerWidth, window.innerHeight);
      container.appendChild(renderer.domElement);

      let controls = null;
      if (typeof THREE.OrbitControls === "function") {
        controls = new THREE.OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.dampingFactor = 0.055;
        controls.rotateSpeed = 0.65;
        controls.minDistance = 7;
        controls.maxDistance = 60;
        controls.enablePan = false;
      }

      // Soft circular glow sprite texture generated via offscreen canvas
      function createParticleTexture() {
        const size = 64;
        const canvas = document.createElement("canvas");
        canvas.width = size;
        canvas.height = size;
        const ctx = canvas.getContext("2d");
        const grad = ctx.createRadialGradient(
          size / 2, size / 2, 0,
          size / 2, size / 2, size / 2
        );
        grad.addColorStop(0.0, "rgba(255, 255, 255, 1.0)");
        grad.addColorStop(0.25, "rgba(255, 255, 255, 0.85)");
        grad.addColorStop(0.55, "rgba(255, 255, 255, 0.25)");
        grad.addColorStop(1.0, "rgba(255, 255, 255, 0.0)");
        ctx.fillStyle = grad;
        ctx.fillRect(0, 0, size, size);
        const tex = new THREE.CanvasTexture(canvas);
        tex.needsUpdate = true;
        return tex;
      }

      // Subtle background starfield for spatial depth
      function createBackgroundStars() {
        const starCount = 700;
        const starPositions = new Float32Array(starCount * 3);
        for (let i = 0; i < starCount; i++) {
          const r = 45 + Math.random() * 110;
          const theta = Math.random() * Math.PI * 2;
          const phi = Math.acos(2 * Math.random() - 1);
          starPositions[i * 3] = r * Math.sin(phi) * Math.cos(theta);
          starPositions[i * 3 + 1] = r * Math.sin(phi) * Math.sin(theta);
          starPositions[i * 3 + 2] = r * Math.cos(phi);
        }
        const starGeo = new THREE.BufferGeometry();
        starGeo.setAttribute("position", new THREE.BufferAttribute(starPositions, 3));
        const starMat = new THREE.PointsMaterial({
          color: 0x64748b,
          size: 0.35,
          transparent: true,
          opacity: 0.45,
          depthWrite: false
        });
        return new THREE.Points(starGeo, starMat);
      }

      const bgStars = createBackgroundStars();
      scene.add(bgStars);

      /* ================================================================
         4. PARTICLE TEMPLATE GENERATORS
         ================================================================ */

      // Helper: uniform random point on or near sphere surface
      function randomSpherePoint(radius, thickness = 0.0) {
        const u = Math.random();
        const v = Math.random();
        const theta = 2 * Math.PI * u;
        const phi = Math.acos(2 * v - 1);
        const r = radius + (Math.random() - 0.5) * thickness;
        return [
          r * Math.sin(phi) * Math.cos(theta),
          r * Math.cos(phi),
          r * Math.sin(phi) * Math.sin(theta)
        ];
      }

      // 4.1 HEARTS — Sculpted 3D Taubin Heart + Ambient Sparkles
      function generateHeartTemplate(count) {
        const positions = new Float32Array(count * 3);
        const luminance = new Float32Array(count);
        const scale = 5.6;

        let idx = 0;
        const mainCount = Math.floor(count * 0.94);

        let guard = 0;
        while (idx < mainCount && guard < count * 80) {
          guard++;
          const x = (Math.random() * 2 - 1) * 1.32;
          const y = (Math.random() * 2 - 1) * 1.32 + 0.1;
          const z = (Math.random() * 2 - 1) * 0.8;

          const a = x * x + 2.25 * z * z + y * y - 1.0;
          const val = a * a * a - x * x * y * y * y - 0.1125 * z * z * y * y * y;

          if (val <= 0) {
            // Bias 78% toward the luminous outer shell for crisp 3D definition
            const isShell = val > -0.14;
            if (isShell || Math.random() < 0.22) {
              positions[idx * 3] = x * scale;
              positions[idx * 3 + 1] = (y - 0.12) * scale;
              positions[idx * 3 + 2] = z * scale;
              luminance[idx] = isShell ? 0.78 + Math.random() * 0.22 : 0.45 + Math.random() * 0.3;
              idx++;
            }
          }
        }

        // Fill remaining / orbiting sparkle dust
        while (idx < count) {
          const t = Math.random() * Math.PI * 2;
          const hx = 16 * Math.pow(Math.sin(t), 3);
          const hy = 13 * Math.cos(t) - 5 * Math.cos(2 * t) - 2 * Math.cos(3 * t) - Math.cos(4 * t);
          const spread = 0.44 + Math.random() * 0.16;
          positions[idx * 3] = hx * spread + (Math.random() - 0.5) * 1.6;
          positions[idx * 3 + 1] = hy * spread + (Math.random() - 0.5) * 1.6;
          positions[idx * 3 + 2] = (Math.random() - 0.5) * 4.5;
          luminance[idx] = 0.5 + Math.random() * 0.5;
          idx++;
        }

        return { positions, luminance };
      }

      // 4.2 FLOWERS — Layered 3D Blooming Lotus Blossom + Golden-Angle Core
      function generateFlowerTemplate(count) {
        const positions = new Float32Array(count * 3);
        const luminance = new Float32Array(count);

        const coreCount = Math.floor(count * 0.16);
        const layerConfigs = [
          { count: Math.floor(count * 0.22), petals: 6, length: 4.5, width: 1.55, tilt: 0.55, cup: 0.75, yOffset: 0.25, phase: 0.0 },
          { count: Math.floor(count * 0.28), petals: 9, length: 6.8, width: 2.05, tilt: 0.98, cup: 0.95, yOffset: -0.25, phase: 0.35 },
          { count: Math.floor(count * 0.27), petals: 12, length: 8.6, width: 2.3, tilt: 1.32, cup: 0.85, yOffset: -0.9, phase: 0.17 }
        ];

        let idx = 0;

        // Fibonacci phyllotaxis stamen center
        const goldenAngle = 2.399963229728653;
        for (let i = 0; i < coreCount; i++, idx++) {
          const frac = i / coreCount;
          const r = 1.95 * Math.sqrt(frac);
          const angle = i * goldenAngle;
          const domeY = Math.cos(frac * Math.PI * 0.5) * 1.35 + (Math.random() - 0.5) * 0.18;
          positions[idx * 3] = r * Math.cos(angle);
          positions[idx * 3 + 1] = domeY + 0.15;
          positions[idx * 3 + 2] = r * Math.sin(angle);
          luminance[idx] = 0.85 + Math.random() * 0.15;
        }

        // Layered 3D curved lotus petals
        for (const layer of layerConfigs) {
          for (let i = 0; i < layer.count && idx < count; i++, idx++) {
            const petalIndex = Math.floor(Math.random() * layer.petals);
            const baseAngle = (petalIndex / layer.petals) * Math.PI * 2 + layer.phase;

            // u along petal length [0..1], v across width [-1..1]
            const u = Math.pow(Math.random(), 0.72);
            const edgeBias = Math.random() < 0.35 ? (Math.random() < 0.5 ? -1 : 1) * (0.82 + Math.random() * 0.18) : (Math.random() * 2 - 1);
            const v = edgeBias;

            const halfWidth = Math.pow(Math.sin(u * Math.PI), 0.75) * layer.width * (1.0 - 0.12 * u);
            const localForward = 0.9 + u * layer.length;
            const localSide = v * halfWidth;

            // Graceful petal arch + lateral cupping
            const archY = Math.sin(u * Math.PI * 0.82) * layer.cup * 2.2 - Math.pow(u, 2.2) * (layer.tilt * 1.45);
            const cupLift = (v * v) * 0.55 * Math.sin(u * Math.PI);
            const y = archY + cupLift + layer.yOffset + (Math.random() - 0.5) * 0.12;

            // Rotate into petal orientation around Y axis
            const cosA = Math.cos(baseAngle);
            const sinA = Math.sin(baseAngle);
            const px = localForward * cosA - localSide * sinA;
            const pz = localForward * sinA + localSide * cosA;

            positions[idx * 3] = px;
            positions[idx * 3 + 1] = y;
            positions[idx * 3 + 2] = pz;

            // Brighter along petal tips and margins
            const highlight = 0.55 + 0.32 * u + 0.13 * Math.abs(v);
            luminance[idx] = Math.min(1.0, highlight);
          }
        }

        // Floating pollen glow particles
        while (idx < count) {
          const r = 2.0 + Math.random() * 7.5;
          const angle = Math.random() * Math.PI * 2;
          positions[idx * 3] = r * Math.cos(angle);
          positions[idx * 3 + 1] = (Math.random() - 0.3) * 6.0;
          positions[idx * 3 + 2] = r * Math.sin(angle);
          luminance[idx] = 0.5 + Math.random() * 0.5;
          idx++;
        }

        return { positions, luminance };
      }

      // 4.3 SATURN — Gas Giant Sphere + Multi-Band Tilted Ring System
      function generateSaturnTemplate(count) {
        const positions = new Float32Array(count * 3);
        const luminance = new Float32Array(count);

        const sphereCount = Math.floor(count * 0.42);
        const ringCount = Math.floor(count * 0.54);
        const tiltX = 26.7 * (Math.PI / 180);
        const tiltZ = 12.0 * (Math.PI / 180);
        const cosX = Math.cos(tiltX), sinX = Math.sin(tiltX);
        const cosZ = Math.cos(tiltZ), sinZ = Math.sin(tiltZ);

        function applyTilt(x, y, z) {
          // Rotate X then Z
          const y1 = y * cosX - z * sinX;
          const z1 = y * sinX + z * cosX;
          const x2 = x * cosZ - y1 * sinZ;
          const y2 = x * sinZ + y1 * cosZ;
          return [x2, y2, z1];
        }

        let idx = 0;

        // Oblate planet sphere
        const planetRadius = 3.85;
        for (let i = 0; i < sphereCount; i++, idx++) {
          const [sx, sy, sz] = randomSpherePoint(planetRadius, 0.28);
          const oblateY = sy * 0.91; // Saturn's polar flattening
          const [tx, ty, tz] = applyTilt(sx, oblateY, sz);
          positions[idx * 3] = tx;
          positions[idx * 3 + 1] = ty;
          positions[idx * 3 + 2] = tz;

          // Subtle atmospheric latitude bands
          const band = 0.72 + 0.25 * Math.sin((sy / planetRadius) * 14.0);
          luminance[idx] = Math.min(1.0, Math.max(0.45, band));
        }

        // Concentric rings with Cassini Division gap
        for (let i = 0; i < ringCount; i++, idx++) {
          const angle = Math.random() * Math.PI * 2;
          const pick = Math.random();
          let r, lum;

          if (pick < 0.11) {
            // Inner faint C-ring
            r = 4.9 + Math.random() * 0.95;
            lum = 0.45 + Math.random() * 0.18;
          } else if (pick < 0.62) {
            // Dense bright B-ring
            r = 5.85 + Math.random() * 2.25;
            lum = 0.78 + Math.random() * 0.22;
          } else if (pick < 0.64) {
            // Sparse Cassini division
            r = 8.1 + Math.random() * 0.55;
            lum = 0.35 + Math.random() * 0.15;
          } else {
            // Outer A-ring
            r = 8.65 + Math.random() * 2.15;
            lum = 0.65 + Math.random() * 0.25;
          }

          const rx = r * Math.cos(angle);
          const ry = (Math.random() - 0.5) * 0.16;
          const rz = r * Math.sin(angle);
          const [tx, ty, tz] = applyTilt(rx, ry, rz);

          positions[idx * 3] = tx;
          positions[idx * 3 + 1] = ty;
          positions[idx * 3 + 2] = tz;
          luminance[idx] = lum;
        }

        // Two small moons + ambient stardust
        const moon1Count = Math.floor((count - idx) * 0.45);
        const moon2Count = Math.floor((count - idx) * 0.25);

        for (let i = 0; i < moon1Count && idx < count; i++, idx++) {
          const [mx, my, mz] = randomSpherePoint(0.45, 0.1);
          const [tx, ty, tz] = applyTilt(mx + 11.8, my + 0.3, mz - 2.4);
          positions[idx * 3] = tx;
          positions[idx * 3 + 1] = ty;
          positions[idx * 3 + 2] = tz;
          luminance[idx] = 0.95;
        }

        for (let i = 0; i < moon2Count && idx < count; i++, idx++) {
          const [mx, my, mz] = randomSpherePoint(0.28, 0.08);
          const [tx, ty, tz] = applyTilt(mx - 10.6, my - 0.2, mz + 3.8);
          positions[idx * 3] = tx;
          positions[idx * 3 + 1] = ty;
          positions[idx * 3 + 2] = tz;
          luminance[idx] = 0.9;
        }

        while (idx < count) {
          const [sx, sy, sz] = randomSpherePoint(12.5, 4.5);
          positions[idx * 3] = sx;
          positions[idx * 3 + 1] = sy;
          positions[idx * 3 + 2] = sz;
          luminance[idx] = 0.4 + Math.random() * 0.35;
          idx++;
        }

        return { positions, luminance };
      }

      // 4.4 BUDDHA STATUE SILHOUETTE — Seated Meditating Buddha + Radiant Halo
      function generateBuddhaTemplate(count) {
        const positions = new Float32Array(count * 3);
        const luminance = new Float32Array(count);
        let idx = 0;

        function pushPoint(x, y, z, lum) {
          if (idx >= count) return;
          positions[idx * 3] = x;
          positions[idx * 3 + 1] = y;
          positions[idx * 3 + 2] = z;
          luminance[idx] = lum;
          idx++;
        }

        // Helper: sample ellipsoid surface (or slightly filled)
        function sampleEllipsoid(n, cx, cy, cz, rx, ry, rz, shellBias, lumBase) {
          for (let i = 0; i < n && idx < count; i++) {
            const u = Math.random();
            const v = Math.random();
            const theta = 2 * Math.PI * u;
            const phi = Math.acos(2 * v - 1);
            const r = shellBias ? (0.82 + 0.18 * Math.random()) : Math.cbrt(Math.random());
            const x = cx + rx * r * Math.sin(phi) * Math.cos(theta);
            const y = cy + ry * r * Math.cos(phi);
            const z = cz + rz * r * Math.sin(phi) * Math.sin(theta);
            pushPoint(x, y, z, lumBase + Math.random() * 0.2);
          }
        }

        // Helper: sample along a curved 3D tube (quadratic Bezier)
        function sampleCurveTube(n, p0, p1, p2, rStart, rEnd, lumBase) {
          for (let i = 0; i < n && idx < count; i++) {
            const t = Math.random();
            const mt = 1 - t;
            const bx = mt * mt * p0[0] + 2 * mt * t * p1[0] + t * t * p2[0];
            const by = mt * mt * p0[1] + 2 * mt * t * p1[1] + t * t * p2[1];
            const bz = mt * mt * p0[2] + 2 * mt * t * p1[2] + t * t * p2[2];
            const rad = (rStart + (rEnd - rStart) * t) * (0.72 + 0.28 * Math.random());
            const [ox, oy, oz] = randomSpherePoint(rad, 0.08);
            pushPoint(bx + ox, by + oy, bz + oz, lumBase + Math.random() * 0.2);
          }
        }

        // 1. Lotus Pedestal Base (14%)
        const baseCount = Math.floor(count * 0.14);
        for (let i = 0; i < baseCount; i++) {
          const angle = Math.random() * Math.PI * 2;
          const h = Math.random(); // 0 bottom -> 1 top
          const petalWave = 1.0 + 0.075 * Math.sin(angle * 16.0) * Math.sin(h * Math.PI);
          const profile = 1.0 - 0.12 * Math.abs(h - 0.5);
          const shell = Math.random() < 0.8 ? (0.9 + 0.1 * Math.random()) : Math.sqrt(Math.random());
          const rx = 5.3 * profile * petalWave * shell;
          const rz = 3.15 * profile * petalWave * shell;
          const y = -5.6 + h * 1.35;
          pushPoint(rx * Math.cos(angle), y, rz * Math.sin(angle), 0.62 + 0.25 * h);
        }

        // 2. Crossed Legs in Lotus Posture (22%)
        const legCount = Math.floor(count * 0.22);
        // Left & right knee/thigh lobes
        sampleEllipsoid(Math.floor(legCount * 0.34), -2.95, -3.55, 0.25, 2.25, 1.05, 1.55, true, 0.68);
        sampleEllipsoid(Math.floor(legCount * 0.34), 2.95, -3.55, 0.25, 2.25, 1.05, 1.55, true, 0.68);
        // Central folded shins & lap volume
        sampleEllipsoid(Math.floor(legCount * 0.32), 0, -3.45, 0.85, 2.8, 0.95, 1.45, true, 0.72);

        // 3. Seated Torso & Draped Robe (21%)
        const torsoCount = Math.floor(count * 0.21);
        for (let i = 0; i < torsoCount; i++) {
          const h = Math.random(); // 0 waist (-2.6) -> 1 shoulders/collar (+1.85)
          const y = -2.6 + h * 4.45;
          // Smooth anatomical silhouette: narrow serene waist -> broad shoulders -> tapered collar
          let rx, rz;
          if (h < 0.72) {
            const t = h / 0.72;
            rx = 1.75 + 0.92 * Math.sin(t * Math.PI * 0.55);
            rz = 1.25 + 0.28 * Math.sin(t * Math.PI * 0.65);
          } else {
            const t = (h - 0.72) / 0.28;
            rx = 2.65 * (1 - t) + 0.78 * t;
            rz = 1.42 * (1 - t) + 0.72 * t;
          }
          const angle = Math.random() * Math.PI * 2;
          const r = Math.random() < 0.82 ? (0.88 + 0.12 * Math.random()) : Math.sqrt(Math.random());
          const x = rx * r * Math.cos(angle);
          const z = rz * r * Math.sin(angle);
          // Subtle diagonal sanghati robe fold highlight
          const robeBand = Math.abs(Math.sin((x * 0.8 + y * 1.1) * 1.6)) < 0.16 ? 0.16 : 0;
          pushPoint(x, y, z, Math.min(1.0, 0.68 + robeBand + Math.random() * 0.16));
        }

        // 4. Arms & Meditating Hands (Dhyana Mudra) (14%)
        const armCount = Math.floor(count * 0.14);
        const perArm = Math.floor(armCount * 0.42);
        // Left arm: shoulder -> elbow -> lap
        sampleCurveTube(perArm, [-2.45, 0.85, 0.0], [-3.15, -1.65, 0.25], [-0.4, -2.55, 1.55], 0.68, 0.48, 0.72);
        // Right arm: shoulder -> elbow -> lap
        sampleCurveTube(perArm, [2.45, 0.85, 0.0], [3.15, -1.65, 0.25], [0.4, -2.55, 1.55], 0.68, 0.48, 0.72);
        // Resting hands / energy bowl in lap
        sampleEllipsoid(armCount - perArm * 2, 0, -2.45, 1.6, 0.95, 0.36, 0.62, true, 0.88);

        // 5. Neck, Serene Head, Elongated Ears & Ushnisha (16%)
        const headSection = Math.floor(count * 0.16);
        // Neck
        sampleEllipsoid(Math.floor(headSection * 0.1), 0, 2.05, 0.0, 0.62, 0.45, 0.6, true, 0.72);
        // Main head
        sampleEllipsoid(Math.floor(headSection * 0.56), 0, 3.35, 0.08, 1.18, 1.35, 1.15, true, 0.78);
        // Elongated earlobes
        sampleEllipsoid(Math.floor(headSection * 0.07), -1.28, 3.0, -0.05, 0.22, 0.62, 0.24, true, 0.75);
        sampleEllipsoid(Math.floor(headSection * 0.07), 1.28, 3.0, -0.05, 0.22, 0.62, 0.24, true, 0.75);
        // Ushnisha (crown topknot)
        sampleEllipsoid(Math.floor(headSection * 0.17), 0, 4.82, 0.0, 0.62, 0.56, 0.6, true, 0.85);
        // Glowing Urna (forehead jewel)
        sampleEllipsoid(Math.max(12, Math.floor(headSection * 0.03)), 0, 3.65, 1.22, 0.14, 0.14, 0.08, false, 1.0);

        // 6. Radiant Prabhamandala Halo behind head (remaining ~13%)
        const haloCenterY = 3.35;
        const haloZ = -1.15;
        while (idx < count) {
          const angle = Math.random() * Math.PI * 2;
          const pick = Math.random();
          if (pick < 0.65) {
            // Outer luminous halo ring
            const r = 2.65 + (Math.random() - 0.5) * 0.26;
            pushPoint(r * Math.cos(angle), haloCenterY + r * Math.sin(angle), haloZ + (Math.random() - 0.5) * 0.14, 0.85 + Math.random() * 0.15);
          } else if (pick < 0.85) {
            // Inner delicate halo ring
            const r = 2.15 + (Math.random() - 0.5) * 0.14;
            pushPoint(r * Math.cos(angle), haloCenterY + r * Math.sin(angle), haloZ + (Math.random() - 0.5) * 0.12, 0.72 + Math.random() * 0.2);
          } else {
            // Radiant light spokes
            const spoke = Math.floor(Math.random() * 16);
            const sa = (spoke / 16) * Math.PI * 2;
            const r = 1.45 + Math.random() * 1.95;
            pushPoint(r * Math.cos(sa), haloCenterY + r * Math.sin(sa), haloZ - 0.08, 0.6 + 0.35 * (r / 3.4));
          }
        }

        return { positions, luminance };
      }

      // 4.5 FIREWORKS — Multi-Burst Radial Streamers with Ballistic Droop
      function generateFireworksTemplate(count) {
        const positions = new Float32Array(count * 3);
        const luminance = new Float32Array(count);

        const bursts = [
          { cx: 0.0, cy: 0.8, cz: 0.0, radius: 8.6, rays: 56, share: 0.52 },
          { cx: -6.0, cy: 3.4, cz: -1.8, radius: 4.5, rays: 28, share: 0.18 },
          { cx: 5.8, cy: 2.6, cz: 1.5, radius: 4.2, rays: 26, share: 0.16 },
          { cx: 0.6, cy: -4.2, cz: -1.4, radius: 3.8, rays: 24, share: 0.14 }
        ];

        let idx = 0;

        for (let b = 0; b < bursts.length; b++) {
          const burst = bursts[b];
          const burstCount = b === bursts.length - 1 ? count - idx : Math.floor(count * burst.share);

          // Pre-generate evenly distributed Fibonacci sphere ray directions for crisp chrysanthemum look
          const dirs = [];
          const golden = Math.PI * (3 - Math.sqrt(5));
          for (let r = 0; r < burst.rays; r++) {
            const y = 1 - (r / Math.max(1, burst.rays - 1)) * 2;
            const radAtY = Math.sqrt(Math.max(0, 1 - y * y));
            const theta = golden * r + b * 1.1;
            // Small organic angular jitter
            const jx = (Math.random() - 0.5) * 0.12;
            const jy = (Math.random() - 0.5) * 0.12;
            const jz = (Math.random() - 0.5) * 0.12;
            let dx = Math.cos(theta) * radAtY + jx;
            let dy = y + jy;
            let dz = Math.sin(theta) * radAtY + jz;
            const len = Math.hypot(dx, dy, dz) || 1;
            dirs.push([dx / len, dy / len, dz / len, 0.85 + Math.random() * 0.3]);
          }

          for (let i = 0; i < burstCount && idx < count; i++, idx++) {
            const ray = dirs[i % burst.rays];
            // Bias points toward outer luminous tips while leaving glowing trails behind
            const t = Math.pow(Math.random(), 0.52);
            const dist = t * burst.radius * ray[3];
            const droop = 0.036 * dist * dist; // Realistic gravity curve on streamers
            const scatter = (1.0 - t * 0.45) * 0.16;

            positions[idx * 3] = burst.cx + ray[0] * dist + (Math.random() - 0.5) * scatter;
            positions[idx * 3 + 1] = burst.cy + ray[1] * dist - droop + (Math.random() - 0.5) * scatter;
            positions[idx * 3 + 2] = burst.cz + ray[2] * dist + (Math.random() - 0.5) * scatter;

            // Brightest spark heads at tip (t -> 1), soft fading trail near core
            luminance[idx] = t > 0.86 ? 1.0 : 0.35 + 0.6 * t;
          }
        }

        return { positions, luminance };
      }

      // Pre-compute and cache all 5 templates for instant, lag-free switching
      const templates = {
        heart: generateHeartTemplate(PARTICLE_COUNT),
        flower: generateFlowerTemplate(PARTICLE_COUNT),
        saturn: generateSaturnTemplate(PARTICLE_COUNT),
        buddha: generateBuddhaTemplate(PARTICLE_COUNT),
        fireworks: generateFireworksTemplate(PARTICLE_COUNT)
      };

      /* ================================================================
         5. PARTICLE SYSTEM BUFFERS & MATERIAL
         ================================================================ */
      const renderPositions = new Float32Array(PARTICLE_COUNT * 3);
      const basePositions = new Float32Array(PARTICLE_COUNT * 3);
      const targetPositions = new Float32Array(PARTICLE_COUNT * 3);

      const vertexColors = new Float32Array(PARTICLE_COUNT * 3);
      const currentLuminance = new Float32Array(PARTICLE_COUNT);
      const targetLuminance = new Float32Array(PARTICLE_COUNT);

      // Per-particle organic motion & radial dispersion seeds
      const particleSeeds = new Float32Array(PARTICLE_COUNT * 4);
      for (let i = 0; i < PARTICLE_COUNT; i++) {
        const i4 = i * 4;
        particleSeeds[i4] = Math.random() * Math.PI * 2;        // phase
        particleSeeds[i4 + 1] = 0.6 + Math.random() * 1.1;      // speed multiplier
        particleSeeds[i4 + 2] = 0.75 + Math.random() * 0.55;    // individual expansion sensitivity
        particleSeeds[i4 + 3] = (Math.random() - 0.5) * 2.0;    // micro-offset direction
      }

      // Initialize with the default template
      function loadTemplate(name, immediate = false) {
        const tpl = templates[name];
        if (!tpl) return;
        state.activeTemplate = name;
        targetPositions.set(tpl.positions);
        targetLuminance.set(tpl.luminance);

        if (immediate) {
          basePositions.set(tpl.positions);
          renderPositions.set(tpl.positions);
          currentLuminance.set(tpl.luminance);
          for (let i = 0; i < PARTICLE_COUNT; i++) {
            const lum = currentLuminance[i];
            vertexColors[i * 3] = lum;
            vertexColors[i * 3 + 1] = lum;
            vertexColors[i * 3 + 2] = lum;
          }
        }
      }

      loadTemplate("heart", true);

      const particleGeometry = new THREE.BufferGeometry();
      particleGeometry.setAttribute("position", new THREE.BufferAttribute(renderPositions, 3));
      particleGeometry.setAttribute("color", new THREE.BufferAttribute(vertexColors, 3));

      const particleMaterial = new THREE.PointsMaterial({
        size: 0.27,
        map: createParticleTexture(),
        color: new THREE.Color(colorPicker.value),
        vertexColors: true,
        transparent: true,
        opacity: 0.92,
        blending: THREE.AdditiveBlending,
        depthWrite: false,
        sizeAttenuation: true
      });

      const particlePoints = new THREE.Points(particleGeometry, particleMaterial);
      const particleGroup = new THREE.Group();
      particleGroup.add(particlePoints);
      scene.add(particleGroup);

      /* ================================================================
         6. UI INTERACTION HANDLERS
         ================================================================ */
      function setParticleColor(hex, fromUser = true) {
        const normalized = hex.toLowerCase();
        colorPicker.value = normalized;
        colorHex.textContent = normalized.toUpperCase();
        particleMaterial.color.set(normalized);
        if (fromUser) state.userPickedColor = true;
      }

      templateButtons.forEach((btn) => {
        btn.addEventListener("click", () => {
          const tplName = btn.dataset.template;
          templateButtons.forEach((b) => b.classList.toggle("active", b === btn));
          loadTemplate(tplName, false);
          if (!state.userPickedColor && TEMPLATE_DEFAULT_COLORS[tplName]) {
            setParticleColor(TEMPLATE_DEFAULT_COLORS[tplName], false);
          }
        });
      });

      colorPicker.addEventListener("input", (e) => {
        setParticleColor(e.target.value, true);
      });

      swatches.forEach((swatch) => {
        swatch.addEventListener("click", () => {
          setParticleColor(swatch.dataset.color, true);
        });
      });

      togglePanelBtn.addEventListener("click", () => {
        const collapsed = controlPanel.classList.toggle("collapsed");
        togglePanelBtn.textContent = collapsed ? "+" : "−";
        togglePanelBtn.title = collapsed ? "Expand panel" : "Collapse panel";
      });

      togglePipBtn.addEventListener("click", () => {
        state.pipVisible = !state.pipVisible;
        cameraPip.classList.toggle("hidden", !state.pipVisible || !state.cameraActive);
        togglePipBtn.textContent = state.pipVisible ? "Hide Camera" : "Show Camera";
      });

      retryCamBtn.addEventListener("click", () => {
        startHandTracking();
      });

      window.addEventListener("resize", () => {
        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
      });

      /* ================================================================
         7. WEBCAM & MEDIAPIPE HANDS TRACKING
         ================================================================ */
      const FINGERTIP_INDICES = [4, 8, 12, 16, 20];
      const PALM_INDICES = [0, 5, 9, 13, 17];
      const HAND_CONNECTIONS = [
        [0, 1], [1, 2], [2, 3], [3, 4],
        [0, 5], [5, 6], [6, 7], [7, 8],
        [5, 9], [9, 10], [10, 11], [11, 12],
        [9, 13], [13, 14], [14, 15], [15, 16],
        [13, 17], [17, 18], [18, 19], [19, 20],
        [0, 17]
      ];

      let mediaStream = null;
      let handsDetector = null;
      let isProcessingFrame = false;
      let trackingLoopId = null;

      function setStatus(mode, message) {
        statusDot.classList.remove("active", "error");
        if (mode === "active") statusDot.classList.add("active");
        if (mode === "error") statusDot.classList.add("error");
        trackingStatus.textContent = message;
      }

      /**
       * Compute normalized hand closing tension [0..1] for one hand.
       * 0 = hand wide open (fingertips far from palm)
       * 1 = hand tightly closed / fist (fingertips close to palm)
       */
      function computeHandTension(landmarks) {
        // Palm center: average of wrist and knuckle MCP joints
        let px = 0, py = 0, pz = 0;
        for (const idx of PALM_INDICES) {
          px += landmarks[idx].x;
          py += landmarks[idx].y;
          pz += landmarks[idx].z;
        }
        px /= PALM_INDICES.length;
        py /= PALM_INDICES.length;
        pz /= PALM_INDICES.length;

        // Reference palm length (wrist 0 to middle MCP 9) for distance-invariance
        const wrist = landmarks[0];
        const midMcp = landmarks[9];
        const palmScale = Math.hypot(
          midMcp.x - wrist.x,
          midMcp.y - wrist.y,
          (midMcp.z - wrist.z) * 0.5
        ) || 0.001;

        // Average distance from all 5 fingertips to palm center
        let totalTipDist = 0;
        for (const tipIdx of FINGERTIP_INDICES) {
          const tip = landmarks[tipIdx];
          const d = Math.hypot(
            tip.x - px,
            tip.y - py,
            (tip.z - pz) * 0.5
          );
          totalTipDist += d;
        }
        const avgRatio = (totalTipDist / FINGERTIP_INDICES.length) / palmScale;

        // Empirically: closed fist ratio ~ 0.72, wide open hand ratio ~ 1.88
        const openness = Math.min(1, Math.max(0, (avgRatio - 0.72) / (1.88 - 0.72)));
        const tension = 1.0 - openness;
        return { tension, openness, palmCenter: { x: px, y: py } };
      }

      function drawHandOverlay(results) {
        const w = handOverlay.width;
        const h = handOverlay.height;
        overlayCtx.save();
        overlayCtx.clearRect(0, 0, w, h);

        // Draw mirrored webcam feed
        overlayCtx.translate(w, 0);
        overlayCtx.scale(-1, 1);
        if (webcamVideo.readyState >= 2) {
          overlayCtx.drawImage(webcamVideo, 0, 0, w, h);
        }

        // Subtle dark tint for skeleton contrast
        overlayCtx.fillStyle = "rgba(5, 7, 14, 0.25)";
        overlayCtx.fillRect(0, 0, w, h);

        const multiLandmarks = results.multiHandLandmarks || [];
        for (const landmarks of multiLandmarks) {
          // Skeleton lines
          overlayCtx.strokeStyle = "rgba(56, 189, 248, 0.85)";
          overlayCtx.lineWidth = 2;
          overlayCtx.beginPath();
          for (const [a, b] of HAND_CONNECTIONS) {
            overlayCtx.moveTo(landmarks[a].x * w, landmarks[a].y * h);
            overlayCtx.lineTo(landmarks[b].x * w, landmarks[b].y * h);
          }
          overlayCtx.stroke();

          // Landmark dots (highlight fingertips)
          for (let i = 0; i < landmarks.length; i++) {
            const isTip = FINGERTIP_INDICES.includes(i);
            overlayCtx.fillStyle = isTip ? "#f472b6" : "#e2e8f0";
            overlayCtx.beginPath();
            overlayCtx.arc(
              landmarks[i].x * w,
              landmarks[i].y * h,
              isTip ? 3.8 : 2.1,
              0,
              Math.PI * 2
            );
            overlayCtx.fill();
          }
        }
        overlayCtx.restore();
      }

      function onHandResults(results) {
        if (state.cameraActive && state.pipVisible) {
          drawHandOverlay(results);
        }

        const handsList = results.multiHandLandmarks || [];
        state.handsDetected = handsList.length;
        pipBadge.textContent = handsList.length === 1 ? "1 hand" : `${handsList.length} hands`;

        if (handsList.length > 0) {
          state.lastHandSeenTime = performance.now();

          let sumTension = 0;
          const centers = [];
          for (const lm of handsList) {
            const metrics = computeHandTension(lm);
            sumTension += metrics.tension;
            centers.push(metrics.palmCenter);
          }

          const avgTension = sumTension / handsList.length;
          const openness = 1.0 - avgTension;

          // If two hands are detected, their separation adds extra responsive expansion
          let spreadBonus = 0;
          if (centers.length === 2) {
            const handGap = Math.hypot(
              centers[0].x - centers[1].x,
              centers[0].y - centers[1].y
            );
            spreadBonus = Math.min(0.45, Math.max(-0.2, (handGap - 0.35) * 0.9));
          }

          state.targetTension = avgTension;
          state.targetSpreadBonus = spreadBonus;

          // Map tension/openness to both overall group scale and particle expansion radius:
          // Closed fist (tension -> 1) contracts; wide open hand (tension -> 0) expands
          state.targetGroupScale = 0.52 + openness * 0.92 + spreadBonus * 0.45;
          state.targetExpansion = 0.58 + openness * 0.96 + spreadBonus * 0.55;

          const label = handsList.length === 2 ? "Tracking 2 hands live" : "Tracking 1 hand live";
          setStatus("active", label);
        } else {
          setStatus("idle", "Camera active · Show hands to control");
        }
      }

      async function processWebcamFrame() {
        if (!state.cameraActive) return;

        if (!isProcessingFrame && webcamVideo.readyState >= 2 && handsDetector) {
          isProcessingFrame = true;
          try {
            await handsDetector.send({ image: webcamVideo });
          } catch (err) {
            console.error("[HandTracking] Error processing webcam frame:", err);
          } finally {
            isProcessingFrame = false;
          }
        }

        trackingLoopId = requestAnimationFrame(processWebcamFrame);
      }

      function stopCamera() {
        state.cameraActive = false;
        if (trackingLoopId !== null) {
          cancelAnimationFrame(trackingLoopId);
          trackingLoopId = null;
        }
        if (mediaStream) {
          mediaStream.getTracks().forEach((t) => t.stop());
          mediaStream = null;
        }
        cameraPip.classList.add("hidden");
      }

      async function startHandTracking() {
        stopCamera();

        if (typeof Hands !== "function") {
          const msg = "MediaPipe Hands library failed to load from CDN. Running in idle animation mode.";
          console.error("[HandTracking]", msg);
          setStatus("error", "Hand tracking CDN unavailable · Idle mode");
          return;
        }

        if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
          const msg =
            "navigator.mediaDevices.getUserMedia is unavailable. Ensure you are running on https:// or http://localhost in a modern browser.";
          console.error("[HandTracking]", msg);
          setStatus("error", "Camera API unavailable · Idle mode");
          return;
        }

        setStatus("idle", "Requesting webcam permission…");

        try {
          mediaStream = await navigator.mediaDevices.getUserMedia({
            video: {
              width: { ideal: 640 },
              height: { ideal: 480 },
              facingMode: "user"
            },
            audio: false
          });

          webcamVideo.srcObject = mediaStream;
          await webcamVideo.play();

          if (!handsDetector) {
            handsDetector = new Hands({
              locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`
            });
            handsDetector.setOptions({
              maxNumHands: 2,
              modelComplexity: 1,
              minDetectionConfidence: 0.6,
              minTrackingConfidence: 0.55
            });
            handsDetector.onResults(onHandResults);
          }

          state.cameraActive = true;
          if (state.pipVisible) {
            cameraPip.classList.remove("hidden");
          }
          setStatus("idle", "Camera active · Show hands to control");
          processWebcamFrame();
        } catch (err) {
          console.error("[HandTracking] Failed to access webcam:", err.name, err.message, err);
          if (err.name === "NotAllowedError" || err.name === "PermissionDeniedError") {
            setStatus("error", "Camera permission denied · Idle mode");
          } else if (err.name === "NotFoundError" || err.name === "DevicesNotFoundError") {
            setStatus("error", "No webcam found · Idle mode");
          } else {
            setStatus("error", "Camera unavailable · Idle mode");
          }
          stopCamera();
        }
      }

      /* ================================================================
         8. MAIN ANIMATION & RENDER LOOP
         ================================================================ */
      const clock = new THREE.Clock();
      let lastUiUpdate = 0;

      function animate() {
        requestAnimationFrame(animate);

        const elapsed = clock.getElapsedTime();
        const now = performance.now();

        // Fallback to smooth idle breathing when no hands have been detected recently
        const handsRecentlySeen = state.handsDetected > 0 || (now - state.lastHandSeenTime < 650);
        if (!handsRecentlySeen) {
          const breath = Math.sin(elapsed * 1.15);
          state.targetTension = 0.5 - breath * 0.12;
          state.targetGroupScale = 1.0 + breath * 0.08;
          state.targetExpansion = 1.0 + breath * 0.09;
          state.targetSpreadBonus = 0;
        }

        // Smoothly interpolate gesture-driven metrics
        state.currentTension += (state.targetTension - state.currentTension) * GESTURE_SMOOTHING;
        state.currentGroupScale += (state.targetGroupScale - state.currentGroupScale) * GESTURE_SMOOTHING;
        state.currentExpansion += (state.targetExpansion - state.currentExpansion) * GESTURE_SMOOTHING;
        state.currentSpreadBonus += (state.targetSpreadBonus - state.currentSpreadBonus) * GESTURE_SMOOTHING;

        // Apply group-level scaling
        particleGroup.scale.setScalar(state.currentGroupScale);

        // Gentle continuous rotation so 3D depth shines from every angle
        particleGroup.rotation.y += 0.0028;
        bgStars.rotation.y -= 0.00025;

        // Per-particle morphing + radial expansion/contraction + organic micro-motion
        const posAttr = particleGeometry.attributes.position;
        const colAttr = particleGeometry.attributes.color;
        const expansionDelta = state.currentExpansion - 1.0;
        const isFireworks = state.activeTemplate === "fireworks";
        const driftAmp = 0.06 + Math.max(0, expansionDelta) * 0.22;

        for (let i = 0; i < PARTICLE_COUNT; i++) {
          const i3 = i * 3;
          const i4 = i * 4;

          // Smoothly morph base shape toward active template target
          const bx = basePositions[i3] + (targetPositions[i3] - basePositions[i3]) * MORPH_SPEED;
          const by = basePositions[i3 + 1] + (targetPositions[i3 + 1] - basePositions[i3 + 1]) * MORPH_SPEED;
          const bz = basePositions[i3 + 2] + (targetPositions[i3 + 2] - basePositions[i3 + 2]) * MORPH_SPEED;

          basePositions[i3] = bx;
          basePositions[i3 + 1] = by;
          basePositions[i3 + 2] = bz;

          // Smooth luminance transition
          const lum = currentLuminance[i] + (targetLuminance[i] - currentLuminance[i]) * MORPH_SPEED;
          currentLuminance[i] = lum;

          const phase = particleSeeds[i4];
          const speed = particleSeeds[i4 + 1];
          const sensitivity = particleSeeds[i4 + 2];
          const dir = particleSeeds[i4 + 3];

          // Radial expansion/contraction modulated slightly per particle for volumetric feel
          let radialFactor = 1.0 + expansionDelta * sensitivity;
          if (isFireworks) {
            radialFactor *= 1.0 + 0.045 * Math.sin(elapsed * 2.4 + phase);
          }

          // Subtle organic floating drift
          const wave = elapsed * speed + phase;
          const dx = Math.sin(wave) * driftAmp;
          const dy = Math.cos(wave * 0.85) * driftAmp;
          const dz = Math.sin(wave * 0.7 + dir) * driftAmp;

          renderPositions[i3] = bx * radialFactor + dx;
          renderPositions[i3 + 1] = by * radialFactor + dy;
          renderPositions[i3 + 2] = bz * radialFactor + dz;

          // Twinkle shimmer on fireworks or expanded state
          const twinkle = isFireworks
            ? 0.8 + 0.2 * Math.sin(elapsed * 5.0 + phase * 3.0)
            : 0.92 + 0.08 * Math.sin(elapsed * 2.0 + phase);
          const finalLum = Math.min(1.0, lum * twinkle);

          vertexColors[i3] = finalLum;
          vertexColors[i3 + 1] = finalLum;
          vertexColors[i3 + 2] = finalLum;
        }

        posAttr.needsUpdate = true;
        colAttr.needsUpdate = true;

        // Subtle particle size response to expansion
        particleMaterial.size = 0.24 + (1.0 - state.currentTension) * 0.07;

        // Update UI telemetry at ~15fps to avoid unnecessary DOM thrashing
        if (now - lastUiUpdate > 65) {
          lastUiUpdate = now;
          const tensionPct = Math.round(state.currentTension * 100);
          tensionValueEl.textContent = `${tensionPct}%`;
          tensionBarEl.style.width = `${tensionPct}%`;

          const combinedScale = state.currentGroupScale * state.currentExpansion;
          scaleValueEl.textContent = `${combinedScale.toFixed(2)}×`;
          const scalePct = Math.min(100, Math.max(5, ((combinedScale - 0.3) / 2.2) * 100));
          scaleBarEl.style.width = `${scalePct}%`;
        }

        if (controls) controls.update();
        renderer.render(scene, camera);
      }

      // Kick off render loop immediately and initialize webcam hand tracking
      animate();
      startHandTracking();
    })();
  </script>
</body>
</html>
