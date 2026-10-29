<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Cozy Autumn Masquerade - Interactive Scene</title>
  <!-- Google Fonts & Font Awesome -->
  <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,600;0,700;1,400&family=Plus+Jakarta+Sans:wght@400;500;600;700&display=swap">
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
  <!-- Three.js & OrbitControls -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; }
    body, html { width: 100%; height: 100%; overflow: hidden; font-family: 'Plus Jakarta Sans', sans-serif; background-color: #0c0a09; color: #FAF6F0; }
    
    #canvas-container { width: 100%; height: 100%; position: absolute; top: 0; left: 0; z-index: 1; }

    /* UI Overlay */
    .ui-layer { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 10; pointer-events: none; display: flex; flex-direction: column; justify-content: space-between; padding: 24px; }
    .interactive { pointer-events: auto; }

    /* Top Bar */
    .top-bar { display: flex; justify-content: space-between; align-items: center; background: rgba(28, 25, 23, 0.75); backdrop-filter: blur(16px); padding: 14px 24px; border-radius: 16px; border: 1px solid rgba(217, 119, 6, 0.3); box-shadow: 0 8px 32px rgba(0,0,0,0.5); }
    .title-group h1 { font-family: 'Playfair Display', serif; font-size: 22px; color: #FAF6F0; display: flex; align-items: center; gap: 10px; }
    .title-group h1 i { color: #D97706; }
    .title-group p { font-size: 12px; color: #A8A29E; text-transform: uppercase; letter-spacing: 1.5px; margin-top: 2px; }

    .nav-buttons { display: flex; gap: 8px; flex-wrap: wrap; }
    .nav-btn { background: rgba(255, 255, 255, 0.08); border: 1px solid rgba(255, 255, 255, 0.15); color: #FAF6F0; padding: 8px 14px; border-radius: 10px; font-size: 13px; font-weight: 600; cursor: pointer; transition: all 0.2s ease; display: flex; align-items: center; gap: 6px; }
    .nav-btn:hover, .nav-btn.active { background: #D97706; color: #1C1917; border-color: #D97706; box-shadow: 0 0 15px rgba(217, 119, 6, 0.4); }

    /* Controls Group */
    .header-actions { display: flex; gap: 10px; align-items: center; }
    .action-circle-btn { width: 40px; height: 40px; border-radius: 50%; background: rgba(255, 255, 255, 0.1); border: 1px solid rgba(255, 255, 255, 0.2); color: #FAF6F0; font-size: 16px; cursor: pointer; display: flex; align-items: center; justify-content: center; transition: all 0.2s; }
    .action-circle-btn:hover { background: #D97706; color: #1C1917; }
    .action-circle-btn.active { background: #10B981; color: #1C1917; border-color: #10B981; }

    /* Glow Stick Color Picker Bar */
    .glow-picker { display: flex; gap: 6px; align-items: center; background: rgba(0, 0, 0, 0.5); padding: 5px 10px; border-radius: 20px; border: 1px solid rgba(255, 255, 255, 0.2); }
    .glow-dot { width: 18px; height: 18px; border-radius: 50%; cursor: pointer; border: 2px solid transparent; transition: all 0.2s; }
    .glow-dot:hover, .glow-dot.selected { transform: scale(1.3); border-color: #FFFFFF; box-shadow: 0 0 12px currentColor; }

    /* Hotspot Badge Overlays */
    .hotspot-label { position: absolute; transform: translate(-50%, -50%); background: rgba(28, 25, 23, 0.85); backdrop-filter: blur(8px); border: 1px solid #D97706; padding: 6px 14px; border-radius: 20px; color: #FAF6F0; font-size: 12px; font-weight: 700; cursor: pointer; transition: all 0.25s ease; box-shadow: 0 4px 20px rgba(0,0,0,0.6); pointer-events: auto; display: flex; align-items: center; gap: 6px; white-space: nowrap; }
    .hotspot-label:hover { transform: translate(-50%, -50%) scale(1.1); background: #D97706; color: #1C1917; box-shadow: 0 0 20px rgba(217, 119, 6, 0.8); }
    .hotspot-label i { color: #D97706; }
    .hotspot-label:hover i { color: #1C1917; }

    /* Bottom Information & Interactive Card */
    .bottom-container { display: flex; justify-content: space-between; align-items: flex-end; gap: 20px; width: 100%; }
    .info-card { background: rgba(28, 25, 23, 0.88); backdrop-filter: blur(16px); border: 1px solid rgba(217, 119, 6, 0.35); padding: 20px 24px; border-radius: 20px; max-width: 420px; width: 100%; box-shadow: 0 12px 40px rgba(0,0,0,0.6); transition: all 0.3s ease; transform: translateY(0); }
    .info-card.hidden { transform: translateY(120%); opacity: 0; pointer-events: none; }
    .info-card-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
    .info-card-title { font-family: 'Playfair Display', serif; font-size: 22px; color: #D97706; font-weight: 700; }
    .info-card-body { font-size: 14px; color: #D6D3D1; line-height: 1.5; margin-bottom: 16px; }
    .info-card-actions { display: flex; gap: 10px; }
    .btn-primary { background: #D97706; color: #1C1917; border: none; padding: 10px 18px; border-radius: 10px; font-weight: 700; font-size: 13px; cursor: pointer; transition: all 0.2s; flex: 1; text-align: center; }
    .btn-primary:hover { background: #F59E0B; box-shadow: 0 0 15px rgba(217, 119, 6, 0.5); }

    /* Mask Customizer Modal */
    .modal-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.75); backdrop-filter: blur(12px); z-index: 100; display: flex; align-items: center; justify-content: center; opacity: 0; pointer-events: none; transition: opacity 0.3s ease; }
    .modal-overlay.active { opacity: 1; pointer-events: auto; }
    .modal-content { background: #1C1917; border: 1px solid #D97706; border-radius: 24px; width: 480px; max-width: 90%; padding: 32px; box-shadow: 0 20px 50px rgba(0,0,0,0.8); display: flex; flex-direction: column; gap: 20px; }
    .modal-header { display: flex; justify-content: space-between; align-items: center; }
    .modal-title { font-family: 'Playfair Display', serif; font-size: 24px; color: #FAF6F0; }
    .close-btn { background: none; border: none; color: #A8A29E; font-size: 20px; cursor: pointer; }
    .close-btn:hover { color: #FAF6F0; }

    /* Canvas Mask Preview */
    .mask-preview-box { width: 100%; height: 180px; background: #2A2421; border-radius: 16px; display: flex; align-items: center; justify-content: center; border: 1px dashed rgba(217, 119, 6, 0.4); position: relative; }
    #mask-canvas { width: 320px; height: 140px; }

    .color-options { display: flex; gap: 12px; align-items: center; }
    .color-dot { width: 32px; height: 32px; border-radius: 50%; cursor: pointer; border: 2px solid transparent; transition: all 0.2s; }
    .color-dot.selected { border-color: #FAF6F0; transform: scale(1.15); }

    /* Notification Toast */
    .toast { position: absolute; top: 90px; right: 24px; background: #10B981; color: #064E3B; font-weight: 700; padding: 12px 20px; border-radius: 12px; box-shadow: 0 8px 24px rgba(0,0,0,0.4); z-index: 90; transform: translateX(150%); transition: transform 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275); display: flex; align-items: center; gap: 10px; font-size: 14px; }
    .toast.active { transform: translateX(0); }

    /* Flash Effect for Photo Booth */
    #flash-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: #fff; z-index: 200; opacity: 0; pointer-events: none; transition: opacity 0.1s ease-out; }

    /* Instruction Hint */
    .hint-box { background: rgba(0,0,0,0.5); backdrop-filter: blur(8px); padding: 8px 16px; border-radius: 20px; font-size: 12px; color: #A8A29E; border: 1px solid rgba(255,255,255,0.1); display: flex; align-items: center; gap: 8px; }
  </style>
</head>
<body>

  <!-- 3D WebGL Canvas Container -->
  <div id="canvas-container"></div>

  <!-- Camera Flash Effect for Photo Booth -->
  <div id="flash-overlay"></div>

  <!-- UI Overlay -->
  <div class="ui-layer">
    <!-- Top Bar Navigation -->
    <div class="top-bar interactive">
      <div class="title-group">
        <h1><i class="fa-solid fa-masks-theater"></i> Autumn Masquerade Scene</h1>
        <p>Class of 2027 • Gymnasium Transformation</p>
      </div>

      <div class="nav-buttons">
        <button class="nav-btn active" onclick="focusZone('overview')"><i class="fa-solid fa-compass"></i> Full View</button>
        <button class="nav-btn" onclick="focusZone('entrance')"><i class="fa-solid fa-door-open"></i> Entrance</button>
        <button class="nav-btn" onclick="focusZone('dancefloor')"><i class="fa-solid fa-compact-disc"></i> Dance Floor</button>
        <button class="nav-btn" onclick="focusZone('ciderbar')"><i class="fa-solid fa-mug-hot"></i> Cider Bar</button>
        <button class="nav-btn" onclick="focusZone('photobooth')"><i class="fa-solid fa-camera"></i> Photo Lounge</button>
        <button class="nav-btn" onclick="focusZone('masktable')"><i class="fa-solid fa-palette"></i> Mask Table</button>
      </div>

      <div class="header-actions">
        <!-- Glow Stick Color Selector -->
        <div class="glow-picker" title="Select Glow Stick Color">
          <span style="font-size:10px; color:#A8A29E; font-weight:700; margin-right:2px;">GLOW:</span>
          <div class="glow-dot selected" style="background:#00ff88; color:#00ff88;" onclick="setGlowColor(0x00ff88, this)"></div>
          <div class="glow-dot" style="background:#00f3ff; color:#00f3ff;" onclick="setGlowColor(0x00f3ff, this)"></div>
          <div class="glow-dot" style="background:#ff007f; color:#ff007f;" onclick="setGlowColor(0xff007f, this)"></div>
          <div class="glow-dot" style="background:#ffaa00; color:#ffaa00;" onclick="setGlowColor(0xffaa00, this)"></div>
          <div class="glow-dot" style="background:#a855f7; color:#a855f7;" onclick="setGlowColor(0xa855f7, this)"></div>
        </div>

        <button class="action-circle-btn" id="neon-toggle-btn" title="Toggle Dark Neon Glow Party Mode" onclick="toggleNeonMode()"><i class="fa-solid fa-bolt"></i></button>
        <button class="action-circle-btn" id="audio-toggle-btn" title="Toggle Cozy Ambience Audio" onclick="toggleAudio()"><i class="fa-solid fa-volume-xmark"></i></button>
        <button class="action-circle-btn" title="Customize Your Mask" onclick="openMaskModal()"><i class="fa-solid fa-wand-magic-sparkles"></i></button>
      </div>
    </div>

    <!-- 3D Hotspot Label Overlays -->
    <div id="hotspot-entrance" class="hotspot-label" onclick="focusZone('entrance')"><i class="fa-solid fa-door-open"></i> Velvet Archway</div>
    <div id="hotspot-dancefloor" class="hotspot-label" onclick="focusZone('dancefloor')"><i class="fa-solid fa-compact-disc"></i> DJ Dance Floor</div>
    <div id="hotspot-ciderbar" class="hotspot-label" onclick="focusZone('ciderbar')"><i class="fa-solid fa-mug-hot"></i> Cider & Treats</div>
    <div id="hotspot-photobooth" class="hotspot-label" onclick="focusZone('photobooth')"><i class="fa-solid fa-camera"></i> Photo Booth</div>
    <div id="hotspot-masktable" class="hotspot-label" onclick="focusZone('masktable')"><i class="fa-solid fa-palette"></i> Mask Station</div>

    <!-- Bottom Container -->
    <div class="bottom-container">
      <div class="hint-box interactive">
        <i class="fa-solid fa-hand-pointer"></i> Drag to rotate • Double click floor to drop glow stick • Click controls to toggle Neon Mode
      </div>

      <!-- Info Card Panel -->
      <div class="info-card interactive" id="info-card">
        <div class="info-card-header">
          <div class="info-card-title" id="card-title">Welcome to the Masquerade</div>
          <span style="font-size: 12px; color: #A8A29E;" id="card-subtitle">Overview</span>
        </div>
        <div class="info-card-body" id="card-body">
          Step into a rustic autumn wonderland draped in warm fairy lights, golden maple foliage, animated dancers, and neon glow accessories!
        </div>
        <div class="info-card-actions" id="card-actions">
          <button class="btn-primary" onclick="dropRandomGlowStick()"><i class="fa-solid fa-wand-magic-sparkles"></i> Drop Glow Stick</button>
          <button class="btn-primary" style="background: rgba(255,255,255,0.1); color: #FAF6F0;" onclick="toggleNeonMode()"><i class="fa-solid fa-bolt"></i> Dark Neon Rave</button>
        </div>
      </div>
    </div>
  </div>

  <!-- Mask Customizer Modal -->
  <div class="modal-overlay" id="mask-modal">
    <div class="modal-content">
      <div class="modal-header">
        <div class="modal-title"><i class="fa-solid fa-masks-theater" style="color:#D97706;"></i> Venetian Mask Studio</div>
        <button class="close-btn" onclick="closeMaskModal()"><i class="fa-solid fa-xmark"></i></button>
      </div>

      <div class="mask-preview-box">
        <canvas id="mask-canvas" width="320" height="140"></canvas>
      </div>

      <div style="display:flex; flex-direction:column; gap:12px;">
        <label style="font-size: 13px; font-weight: 700; color: #A8A29E; text-transform: uppercase;">Primary Mask Color</label>
        <div class="color-options">
          <div class="color-dot selected" style="background: #7C2D12;" onclick="setMaskColor('#7C2D12', this)"></div>
          <div class="color-dot" style="background: #D97706;" onclick="setMaskColor('#D97706', this)"></div>
          <div class="color-dot" style="background: #1C1917;" onclick="setMaskColor('#1C1917', this)"></div>
          <div class="color-dot" style="background: #065F46;" onclick="setMaskColor('#065F46', this)"></div>
          <div class="color-dot" style="background: #4C1D95;" onclick="setMaskColor('#4C1D95', this)"></div>
        </div>
      </div>

      <div style="display:flex; flex-direction:column; gap:12px;">
        <label style="font-size: 13px; font-weight: 700; color: #A8A29E; text-transform: uppercase;">Accent Trim</label>
        <div class="color-options">
          <button class="nav-btn active" style="font-size:12px;" onclick="setMaskTrim('#F59E0B')">✨ Gold Leaf</button>
          <button class="nav-btn" style="font-size:12px;" onclick="setMaskTrim('#E2E8F0')">❄️ Silver Filigree</button>
          <button class="nav-btn" style="font-size:12px;" onclick="setMaskTrim('#1C1917')">🖤 Black Lace</button>
        </div>
      </div>

      <button class="btn-primary" style="margin-top: 10px;" onclick="wearCustomMask()"><i class="fa-solid fa-check"></i> Wear My Custom Mask</button>
    </div>
  </div>

  <!-- Notification Toast -->
  <div class="toast" id="toast">
    <i class="fa-solid fa-circle-check"></i> <span id="toast-text">Custom Mask Equipped!</span>
  </div>

  <script>
    /* ==================== THREE.JS 3D SCENE SETUP ==================== */
    let scene, camera, renderer, controls;
    let leavesParticles, sparklesParticles;
    let pointLights = [];
    let isNightMode = false;
    let isNeonMode = false;
    let currentFocusZone = 'overview';
    let ambientLight, mainSpot;

    // Glow Stick & Avatar State
    let dancers = [];
    let glowSticks = [];
    let selectedGlowColor = 0x00ff88;

    // 3D Interactive Target Mesh Position Anchors
    const zonePositions = {
      overview:   { cam: { x: 0, y: 18, z: 32 }, target: { x: 0, y: 2, z: 0 } },
      entrance:   { cam: { x: 0, y: 6, z: 22 },  target: { x: 0, y: 3, z: 14 } },
      dancefloor: { cam: { x: -2, y: 10, z: 10 }, target: { x: -2, y: 1, z: -2 } },
      ciderbar:   { cam: { x: -16, y: 6, z: 6 },  target: { x: -18, y: 2, z: 0 } },
      photobooth: { cam: { x: 16, y: 6, z: 6 },   target: { x: 18, y: 3, z: 0 } },
      masktable:  { cam: { x: 12, y: 6, z: 18 },  target: { x: 10, y: 2, z: 10 } }
    };

    function init3D() {
      const container = document.getElementById('canvas-container');
      
      // Scene
      scene = new THREE.Scene();
      scene.background = new THREE.Color(0x0e0c0a);
      scene.fog = new THREE.FogExp2(0x0e0c0a, 0.022);

      // Camera
      camera = new THREE.PerspectiveCamera(55, window.innerWidth / window.innerHeight, 0.1, 1000);
      camera.position.set(zonePositions.overview.cam.x, zonePositions.overview.cam.y, zonePositions.overview.cam.z);

      // Renderer
      renderer = new THREE.WebGLRenderer({ antialias: true });
      renderer.setPixelRatio(window.devicePixelRatio);
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.shadowMap.enabled = true;
      renderer.shadowMap.type = THREE.PCFSoftShadowMap;
      container.appendChild(renderer.domElement);

      // OrbitControls
      controls = new THREE.OrbitControls(camera, renderer.domElement);
      controls.enableDamping = true;
      controls.dampingFactor = 0.05;
      controls.maxPolarAngle = Math.PI / 2 - 0.02;
      controls.minDistance = 6;
      controls.maxDistance = 50;
      controls.target.set(0, 2, 0);

      // Lights
      ambientLight = new THREE.AmbientLight(0xfff1e0, 0.8);
      scene.add(ambientLight);

      mainSpot = new THREE.SpotLight(0xffb703, 1.5);
      mainSpot.position.set(0, 25, 0);
      mainSpot.angle = Math.PI / 3;
      mainSpot.penumbra = 0.8;
      mainSpot.castShadow = true;
      scene.add(mainSpot);

      // Warm Fairy Lights Grid
      const fairyColors = [0xd97706, 0xf59e0b, 0xef4444, 0xfbbf24];
      for(let x = -20; x <= 20; x += 10) {
        for(let z = -20; z <= 20; z += 10) {
          const color = fairyColors[Math.floor(Math.random() * fairyColors.length)];
          const pLight = new THREE.PointLight(color, 0.6, 12);
          pLight.position.set(x + (Math.random()*2-1), 12, z + (Math.random()*2-1));
          scene.add(pLight);
          pointLights.push(pLight);

          const bulbGeo = new THREE.SphereGeometry(0.12, 8, 8);
          const bulbMat = new THREE.MeshBasicMaterial({ color: color });
          const bulb = new THREE.Mesh(bulbGeo, bulbMat);
          bulb.position.copy(pLight.position);
          scene.add(bulb);
        }
      }

      // Build Venue Geometry
      buildFloor();
      buildWallsAndBeams();
      buildEntranceArch();
      buildDanceFloor();
      buildDJBooth();
      buildCiderBar();
      buildPhotoBooth();
      buildMaskTable();

      // Build Low-Poly Animated Dancers & Party Guests
      createDancers();

      // Build Particles
      createFallingLeaves();
      createSparkles();

      // Double click listener to drop glow sticks
      window.addEventListener('dblclick', onSceneDoubleClick);

      // Window Resize Listener
      window.addEventListener('resize', onWindowResize);

      // Start Render Loop
      animate();
    }

    /* ==================== 3D VENUE GEOMETRY BUILDERS ==================== */
    function buildFloor() {
      // Polished Hardwood Basketball Court Floor (matching glossy wood in photo)
      const floorGeo = new THREE.PlaneGeometry(50, 50);
      const floorMat = new THREE.MeshStandardMaterial({ 
        color: 0xdcae72, 
        roughness: 0.22, 
        metalness: 0.08 
      });
      const floor = new THREE.Mesh(floorGeo, floorMat);
      floor.rotation.x = -Math.PI / 2;
      floor.receiveShadow = true;
      scene.add(floor);

      // Black Outer Court Perimeter Border & Apron
      const borderMat = new THREE.MeshStandardMaterial({ color: 0x111111, roughness: 0.3 });
      
      const leftApron = new THREE.Mesh(new THREE.PlaneGeometry(4, 46), borderMat);
      leftApron.rotation.x = -Math.PI / 2; leftApron.position.set(-22, 0.01, 0); scene.add(leftApron);

      const rightApron = new THREE.Mesh(new THREE.PlaneGeometry(4, 46), borderMat);
      rightApron.rotation.x = -Math.PI / 2; rightApron.position.set(22, 0.01, 0); scene.add(rightApron);

      const topApron = new THREE.Mesh(new THREE.PlaneGeometry(48, 4), borderMat);
      topApron.rotation.x = -Math.PI / 2; topApron.position.set(0, 0.01, -22); scene.add(topApron);

      const botApron = new THREE.Mesh(new THREE.PlaneGeometry(48, 4), borderMat);
      botApron.rotation.x = -Math.PI / 2; botApron.position.set(0, 0.01, 22); scene.add(botApron);

      // Basketball Key / Paint Areas (Black)
      const keyMat = new THREE.MeshStandardMaterial({ color: 0x181818, roughness: 0.3 });
      const northKey = new THREE.Mesh(new THREE.PlaneGeometry(10, 14), keyMat);
      northKey.rotation.x = -Math.PI / 2; northKey.position.set(0, 0.015, -15); scene.add(northKey);

      const southKey = new THREE.Mesh(new THREE.PlaneGeometry(10, 14), keyMat);
      southKey.rotation.x = -Math.PI / 2; southKey.position.set(0, 0.015, 15); scene.add(southKey);

      // Court Lines Overlay
      const lineMat = new THREE.MeshBasicMaterial({ color: 0xffffff });
      
      // Half court line
      const halfLine = new THREE.Mesh(new THREE.PlaneGeometry(40, 0.2), lineMat);
      halfLine.rotation.x = -Math.PI / 2; halfLine.position.set(0, 0.02, 0); scene.add(halfLine);

      // Center Circle Ring
      const circleGeo = new THREE.RingGeometry(3.8, 4.0, 32);
      const centerCircle = new THREE.Mesh(circleGeo, lineMat);
      centerCircle.rotation.x = -Math.PI / 2; centerCircle.position.set(0, 0.02, 0); scene.add(centerCircle);

      // Center Court Logo (Black Circle with Gold Text Accent)
      const logoCenter = new THREE.Mesh(new THREE.CircleGeometry(3.7, 32), keyMat);
      logoCenter.rotation.x = -Math.PI / 2; logoCenter.position.set(0, 0.018, 0); scene.add(logoCenter);

      // Gold Lettering Emblem in Center Circle
      const goldMat = new THREE.MeshStandardMaterial({ color: 0xd97706, metalness: 0.6, roughness: 0.3 });
      const logoDecal = new THREE.Mesh(new THREE.BoxGeometry(3, 0.01, 1.2), goldMat);
      logoDecal.position.set(0, 0.025, 0); scene.add(logoDecal);
    }

    function buildWallsAndBeams() {
      const wallMat = new THREE.MeshStandardMaterial({ color: 0xe8e4dc, roughness: 0.8 }); // Off-white gym walls
      const beamMat = new THREE.MeshStandardMaterial({ color: 0x3d3835, roughness: 0.5 }); // Ceiling trusses

      // Back Wall
      const backWall = new THREE.Mesh(new THREE.BoxGeometry(50, 16, 1), wallMat);
      backWall.position.set(0, 8, -25);
      scene.add(backWall);

      // Left Wall
      const leftWall = new THREE.Mesh(new THREE.BoxGeometry(1, 16, 50), wallMat);
      leftWall.position.set(-25, 8, 0);
      scene.add(leftWall);

      // Right Wall with Arch Windows
      const rightWall = new THREE.Mesh(new THREE.BoxGeometry(1, 16, 50), wallMat);
      rightWall.position.set(25, 8, 0);
      scene.add(rightWall);

      // Arch Windows along Right Wall (matching screenshot)
      const windowGlassMat = new THREE.MeshStandardMaterial({ color: 0x88c0d0, roughness: 0.1, transparent: true, opacity: 0.4 });
      const windowFrameMat = new THREE.MeshStandardMaterial({ color: 0xaaaaaa, metalness: 0.5 });

      for(let z = -18; z <= 18; z += 9) {
        const winPlane = new THREE.Mesh(new THREE.PlaneGeometry(4, 7), windowGlassMat);
        winPlane.rotation.y = -Math.PI / 2;
        winPlane.position.set(24.4, 5.5, z);
        scene.add(winPlane);

        const archTop = new THREE.Mesh(new THREE.CylinderGeometry(2, 2, 0.4, 16, 1, false, 0, Math.PI), windowFrameMat);
        archTop.rotation.y = -Math.PI / 2;
        archTop.position.set(24.3, 9, z);
        scene.add(archTop);
      }

      // --- STAGE & BLACK THEATER CURTAIN (Matching image back wall) ---
      const stageFloorMat = new THREE.MeshStandardMaterial({ color: 0x261c14, roughness: 0.6 });
      const stageFloor = new THREE.Mesh(new THREE.BoxGeometry(20, 1.2, 8), stageFloorMat);
      stageFloor.position.set(0, 0.6, -21);
      scene.add(stageFloor);

      const curtainMat = new THREE.MeshStandardMaterial({ color: 0x0d0d0d, roughness: 0.9 });
      const stageCurtain = new THREE.Mesh(new THREE.BoxGeometry(20, 11, 0.4), curtainMat);
      stageCurtain.position.set(0, 6.5, -24.3);
      scene.add(stageCurtain);

      // US Flag on Back Wall
      const usFlagMat = new THREE.MeshBasicMaterial({ color: 0xb22234 });
      const usFlag = new THREE.Mesh(new THREE.BoxGeometry(3.5, 2.2, 0.1), usFlagMat);
      usFlag.position.set(0, 13.2, -24.2);
      scene.add(usFlag);

      // --- INTERNATIONAL FLAGS ROW (Ringing the upper perimeter wall like in photo) ---
      const flagColors = [0xef4444, 0x3b82f6, 0x10b981, 0xf59e0b, 0x8b5cf6, 0xec4899, 0x06b6d4, 0x84cc16, 0xd97706, 0x6366f1];
      
      // Right Wall Flags Row
      for(let z = -22, i = 0; z <= 22; z += 3.5, i++) {
        const flagMat = new THREE.MeshStandardMaterial({ color: flagColors[i % flagColors.length], roughness: 0.6 });
        const flag = new THREE.Mesh(new THREE.PlaneGeometry(2.4, 1.6), flagMat);
        flag.rotation.y = -Math.PI / 2;
        flag.position.set(24.3, 11.8, z);
        scene.add(flag);
      }

      // Back Wall Flags Row
      for(let x = -22, i = 0; x <= 22; x += 3.5, i++) {
        if (Math.abs(x) < 4) continue; // Skip center US flag
        const flagMat = new THREE.MeshStandardMaterial({ color: flagColors[(i + 3) % flagColors.length], roughness: 0.6 });
        const flag = new THREE.Mesh(new THREE.PlaneGeometry(2.4, 1.6), flagMat);
        flag.position.set(x, 11.8, -24.3);
        scene.add(flag);
      }

      // --- SCOREBOARD ---
      const sbMat = new THREE.MeshStandardMaterial({ color: 0x111111, metalness: 0.8, roughness: 0.2 });
      const scoreboard = new THREE.Mesh(new THREE.BoxGeometry(4.5, 2.5, 0.6), sbMat);
      scoreboard.position.set(-14, 10, -24.2);
      scene.add(scoreboard);

      const scoreMat = new THREE.MeshBasicMaterial({ color: 0xff0000 });
      const homeScore = new THREE.Mesh(new THREE.BoxGeometry(1.2, 0.8, 0.1), scoreMat);
      homeScore.position.set(-15, 10, -23.8);
      scene.add(homeScore);

      const guestScore = new THREE.Mesh(new THREE.BoxGeometry(1.2, 0.8, 0.1), scoreMat);
      guestScore.position.set(-13, 10, -23.8);
      scene.add(guestScore);

      // --- CEILING TRUSSES & AIR DUCTS ---
      const trussMat = new THREE.MeshStandardMaterial({ color: 0x555555, metalness: 0.7, roughness: 0.3 });
      const ductMat = new THREE.MeshStandardMaterial({ color: 0xcccccc, metalness: 0.8, roughness: 0.2 });

      // Steel Ceiling Trusses
      for (let z = -20; z <= 20; z += 8) {
        const truss = new THREE.Mesh(new THREE.BoxGeometry(49, 0.6, 0.6), trussMat);
        truss.position.set(0, 15, z);
        scene.add(truss);
      }

      // Exposed HVAC Air Ducts
      const duct1 = new THREE.Mesh(new THREE.CylinderGeometry(0.7, 0.7, 48, 16), ductMat);
      duct1.rotation.z = Math.PI / 2;
      duct1.position.set(0, 14.2, -18);
      scene.add(duct1);

      const duct2 = new THREE.Mesh(new THREE.CylinderGeometry(0.7, 0.7, 48, 16), ductMat);
      duct2.rotation.z = Math.PI / 2;
      duct2.position.set(0, 14.2, 18);
      scene.add(duct2);

      // --- BASKETBALL HOOPS & BACKBOARDS (Main court ends) ---
      buildBasketballHoop(0, -19.5, 0); // North hoop
      buildBasketballHoop(0, 19.5, Math.PI); // South hoop
    }

    function buildBasketballHoop(x, z, rotY) {
      const hoopGroup = new THREE.Group();
      
      // Glass Backboard
      const bbMat = new THREE.MeshStandardMaterial({ color: 0xffffff, transparent: true, opacity: 0.7, roughness: 0.1 });
      const bbBorderMat = new THREE.MeshStandardMaterial({ color: 0xffffff });
      const backboard = new THREE.Mesh(new THREE.BoxGeometry(3.6, 2.2, 0.1), bbMat);
      backboard.position.set(0, 6.2, 0);
      hoopGroup.add(backboard);

      // White Backboard Inner Square
      const sq = new THREE.Mesh(new THREE.BoxGeometry(1.2, 0.9, 0.12), bbBorderMat);
      sq.position.set(0, 5.8, 0);
      hoopGroup.add(sq);

      // Orange Metal Rim
      const rimMat = new THREE.MeshStandardMaterial({ color: 0xff4500, metalness: 0.8 });
      const rim = new THREE.Mesh(new THREE.TorusGeometry(0.45, 0.04, 8, 24), rimMat);
      rim.rotation.x = Math.PI / 2;
      rim.position.set(0, 5.5, 0.5);
      hoopGroup.add(rim);

      // Net
      const netMat = new THREE.MeshBasicMaterial({ color: 0xffffff, wireframe: true });
      const net = new THREE.Mesh(new THREE.CylinderGeometry(0.45, 0.25, 0.7, 12, 3, true), netMat);
      net.position.set(0, 5.15, 0.5);
      hoopGroup.add(net);

      // Ceiling Support Arm Structure
      const armMat = new THREE.MeshStandardMaterial({ color: 0x111111, metalness: 0.8 });
      const arm1 = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.08, 8.5), armMat);
      arm1.rotation.x = -Math.PI / 4;
      arm1.position.set(0, 10, -2.5);
      hoopGroup.add(arm1);

      hoopGroup.position.set(x, 0, z);
      hoopGroup.rotation.y = rotY;
      scene.add(hoopGroup);
    }

    function buildEntranceArch() {
      const archGroup = new THREE.Group();
      const woodMat = new THREE.MeshStandardMaterial({ color: 0x3d261a, roughness: 0.7 });
      const leafMat = new THREE.MeshStandardMaterial({ color: 0xd97706, roughness: 0.5 });
      const velvetMat = new THREE.MeshStandardMaterial({ color: 0x7c2d12, roughness: 0.8 });

      // Arch pillars & top
      const p1 = new THREE.Mesh(new THREE.BoxGeometry(0.8, 6, 0.8), woodMat);
      p1.position.set(-3, 3, 18);
      const p2 = new THREE.Mesh(new THREE.BoxGeometry(0.8, 6, 0.8), woodMat);
      p2.position.set(3, 3, 18);

      const top = new THREE.Mesh(new THREE.BoxGeometry(7, 0.8, 0.8), woodMat);
      top.position.set(0, 6, 18);

      // Velvet drapes
      const d1 = new THREE.Mesh(new THREE.BoxGeometry(1.2, 5.5, 0.4), velvetMat);
      d1.position.set(-2.8, 2.75, 18.1);
      const d2 = new THREE.Mesh(new THREE.BoxGeometry(1.2, 5.5, 0.4), velvetMat);
      d2.position.set(2.8, 2.75, 18.1);

      // Decorative foliage top cluster
      for (let i = 0; i < 12; i++) {
        const leaf = new THREE.Mesh(new THREE.BoxGeometry(0.6, 0.6, 0.6), leafMat);
        leaf.position.set((Math.random() - 0.5) * 6, 6.2 + (Math.random() - 0.5) * 0.4, 18 + (Math.random() - 0.5) * 0.5);
        leaf.rotation.set(Math.random(), Math.random(), Math.random());
        archGroup.add(leaf);
      }

      archGroup.add(p1, p2, top, d1, d2);
      scene.add(archGroup);
    }

    function buildDanceFloor() {
      const dfGeo = new THREE.BoxGeometry(16, 0.1, 16);
      const dfMat = new THREE.MeshStandardMaterial({ color: 0x2e1c14, roughness: 0.3, metalness: 0.1 });
      const df = new THREE.Mesh(dfGeo, dfMat);
      df.position.set(-2, 0.05, -2);
      df.receiveShadow = true;
      scene.add(df);

      const borderGeo = new THREE.BoxGeometry(16.6, 0.12, 16.6);
      const borderMat = new THREE.MeshStandardMaterial({ color: 0xd97706, metalness: 0.7, roughness: 0.3 });
      const border = new THREE.Mesh(borderGeo, borderMat);
      border.position.set(-2, 0.04, -2);
      scene.add(border);
    }

    function buildDJBooth() {
      const djGroup = new THREE.Group();
      const deskMat = new THREE.MeshStandardMaterial({ color: 0x1c1917, roughness: 0.5 });
      const metallicMat = new THREE.MeshStandardMaterial({ color: 0x9ca3af, metalness: 0.8, roughness: 0.2 });
      const speakerMat = new THREE.MeshStandardMaterial({ color: 0x111111, roughness: 0.9 });

      const desk = new THREE.Mesh(new THREE.BoxGeometry(4, 1.8, 1.5), deskMat);
      desk.position.set(-2, 0.9, -12);

      const laptop = new THREE.Mesh(new THREE.BoxGeometry(0.8, 0.1, 0.6), metallicMat);
      laptop.position.set(-2, 1.85, -12);

      const spk1 = new THREE.Mesh(new THREE.BoxGeometry(1.2, 3, 1.2), speakerMat);
      spk1.position.set(-5, 1.5, -12);
      const spk2 = new THREE.Mesh(new THREE.BoxGeometry(1.2, 3, 1.2), speakerMat);
      spk2.position.set(1, 1.5, -12);

      djGroup.add(desk, laptop, spk1, spk2);
      scene.add(djGroup);
    }

    function buildCiderBar() {
      const ciderGroup = new THREE.Group();
      const woodMat = new THREE.MeshStandardMaterial({ color: 0x452a1d, roughness: 0.7 });
      const glassMat = new THREE.MeshStandardMaterial({ color: 0xd97706, roughness: 0.1, transparent: true, opacity: 0.8 });

      const bar = new THREE.Mesh(new THREE.BoxGeometry(6, 1.8, 1.8), woodMat);
      bar.position.set(-18, 0.9, 0);

      const dispenser = new THREE.Mesh(new THREE.CylinderGeometry(0.5, 0.5, 1.2, 16), glassMat);
      dispenser.position.set(-18, 2.4, 0);

      const cupMat = new THREE.MeshStandardMaterial({ color: 0xfef3c7, roughness: 0.4 });
      for (let i = -1.5; i <= 1.5; i += 0.8) {
        const cup = new THREE.Mesh(new THREE.CylinderGeometry(0.12, 0.1, 0.25, 12), cupMat);
        cup.position.set(-18 + i, 1.92, 0.3);
        ciderGroup.add(cup);
      }

      ciderGroup.add(bar, dispenser);
      scene.add(ciderGroup);
    }

    function buildPhotoBooth() {
      const photoGroup = new THREE.Group();
      const backdropMat = new THREE.MeshStandardMaterial({ color: 0x7c2d12, roughness: 0.8 });
      const frameMat = new THREE.MeshStandardMaterial({ color: 0xf59e0b, metalness: 0.8, roughness: 0.2 });

      const backdrop = new THREE.Mesh(new THREE.BoxGeometry(6, 4.5, 0.2), backdropMat);
      backdrop.position.set(18, 2.25, 0);

      const frame = new THREE.Mesh(new THREE.BoxGeometry(4, 3, 0.1), frameMat);
      frame.position.set(18, 2.5, 0.15);

      const chairMat = new THREE.MeshStandardMaterial({ color: 0x4c1d95, roughness: 0.6 });
      const chairSeat = new THREE.Mesh(new THREE.BoxGeometry(1.6, 0.6, 1.6), chairMat);
      chairSeat.position.set(18, 0.3, 1.5);
      const chairBack = new THREE.Mesh(new THREE.BoxGeometry(1.6, 1.8, 0.3), chairMat);
      chairBack.position.set(18, 1.2, 2.15);

      photoGroup.add(backdrop, frame, chairSeat, chairBack);
      scene.add(photoGroup);
    }

    function buildMaskTable() {
      const maskGroup = new THREE.Group();
      const woodMat = new THREE.MeshStandardMaterial({ color: 0x3d261a, roughness: 0.7 });
      const maskMat = new THREE.MeshStandardMaterial({ color: 0xd97706, metalness: 0.6 });

      const table = new THREE.Mesh(new THREE.BoxGeometry(5, 1.5, 2.5), woodMat);
      table.position.set(10, 0.75, 10);

      for (let i = -1.5; i <= 1.5; i += 1.0) {
        const mask = new THREE.Mesh(new THREE.BoxGeometry(0.4, 0.05, 0.2), maskMat);
        mask.position.set(10 + i, 1.55, 10 + (Math.random() - 0.5) * 0.5);
        mask.rotation.y = (Math.random() - 0.5) * 0.8;
        maskGroup.add(mask);
      }

      maskGroup.add(table);
      scene.add(maskGroup);
    }

    function createFallingLeaves() {
      const leafCount = 120;
      const geometry = new THREE.BufferGeometry();
      const positions = new Float32Array(leafCount * 3);
      const velocities = [];

      for (let i = 0; i < leafCount; i++) {
        positions[i * 3] = (Math.random() - 0.5) * 48;
        positions[i * 3 + 1] = Math.random() * 15;
        positions[i * 3 + 2] = (Math.random() - 0.5) * 48;

        velocities.push({
          y: 0.02 + Math.random() * 0.03,
          x: (Math.random() - 0.5) * 0.01
        });
      }

      geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));

      const material = new THREE.PointsMaterial({
        color: 0xd97706,
        size: 0.35,
        transparent: true,
        opacity: 0.8
      });

      leavesParticles = new THREE.Points(geometry, material);
      leavesParticles.userData.velocities = velocities;
      scene.add(leavesParticles);
    }

    function createSparkles() {
      const sparkleCount = 80;
      const geometry = new THREE.BufferGeometry();
      const positions = new Float32Array(sparkleCount * 3);

      for (let i = 0; i < sparkleCount; i++) {
        positions[i * 3] = (Math.random() - 0.5) * 40;
        positions[i * 3 + 1] = 1 + Math.random() * 10;
        positions[i * 3 + 2] = (Math.random() - 0.5) * 40;
      }

      geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));

      const material = new THREE.PointsMaterial({
        color: 0xf59e0b,
        size: 0.2,
        transparent: true,
        opacity: 0.9
      });

      sparklesParticles = new THREE.Points(geometry, material);
      scene.add(sparklesParticles);
    }

    /* ==================== ANIMATED DANCERS & PARTY GUESTS ==================== */
    function createDancers() {
      const outfitColors = [0x7c2d12, 0xd97706, 0x1c1917, 0x065f46, 0x4c1d95, 0x9a3412];
      const positions = [
        { x: -2, z: -2, type: 'dance' },
        { x: -4, z: -1, type: 'dance' },
        { x: 0, z: -3, type: 'dance' },
        { x: -1, z: 1, type: 'dance' },
        { x: -3, z: -4, type: 'dance' },
        { x: 1, z: -1, type: 'dance' },
        { x: -16, z: 2, type: 'sway' }, // Cider bar
        { x: 16, z: 1, type: 'pose' }, // Photo booth
        { x: 9, z: 9, type: 'sit' },   // Mask table
        { x: -2, z: -11, type: 'dj' }   // DJ
      ];

      positions.forEach((p, idx) => {
        const group = new THREE.Group();
        const color = outfitColors[idx % outfitColors.length];

        // Body Mesh (Low-Poly Stylized Avatar)
        const torsoMat = new THREE.MeshStandardMaterial({ color: color, roughness: 0.5 });
        const skinMat = new THREE.MeshStandardMaterial({ color: 0xe0a96d, roughness: 0.6 });

        // Head
        const head = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.5, 0.5), skinMat);
        head.position.y = 2.4;
        group.add(head);

        // Masquerade Mask on Avatar
        const maskMat = new THREE.MeshStandardMaterial({ color: 0xd97706, metalness: 0.8 });
        const mask = new THREE.Mesh(new THREE.BoxGeometry(0.52, 0.16, 0.2), maskMat);
        mask.position.set(0, 2.42, 0.22);
        group.add(mask);

        // Torso
        const torso = new THREE.Mesh(new THREE.BoxGeometry(0.7, 1.1, 0.4), torsoMat);
        torso.position.y = 1.5;
        group.add(torso);

        // Arms
        const armMat = p.type === 'dj' ? skinMat : torsoMat;
        const leftArm = new THREE.Mesh(new THREE.BoxGeometry(0.2, 0.8, 0.2), armMat);
        leftArm.position.set(-0.48, 1.5, 0);
        group.add(leftArm);

        const rightArm = new THREE.Mesh(new THREE.BoxGeometry(0.2, 0.8, 0.2), armMat);
        rightArm.position.set(0.48, 1.5, 0);
        group.add(rightArm);

        // Glowing Stick in Right Hand!
        const stickGeo = new THREE.CylinderGeometry(0.04, 0.04, 0.8, 8);
        const stickMat = new THREE.MeshBasicMaterial({ color: selectedGlowColor });
        const glowStick = new THREE.Mesh(stickGeo, stickMat);
        glowStick.position.set(0, -0.4, 0.2);
        glowStick.rotation.x = Math.PI / 3;
        rightArm.add(glowStick);

        const glowLight = new THREE.PointLight(selectedGlowColor, 1.2, 4);
        glowLight.position.copy(glowStick.position);
        rightArm.add(glowLight);

        // Legs
        const legMat = new THREE.MeshStandardMaterial({ color: 0x1c1917 });
        const leftLeg = new THREE.Mesh(new THREE.BoxGeometry(0.25, 1.0, 0.25), legMat);
        leftLeg.position.set(-0.2, 0.5, 0);
        group.add(leftLeg);

        const rightLeg = new THREE.Mesh(new THREE.BoxGeometry(0.25, 1.0, 0.25), legMat);
        rightLeg.position.set(0.2, 0.5, 0);
        group.add(rightLeg);

        group.position.set(p.x, 0, p.z);
        scene.add(group);

        dancers.push({
          mesh: group,
          leftArm,
          rightArm,
          glowStick,
          glowLight,
          type: p.type,
          baseY: 0,
          speed: 1 + Math.random() * 0.8,
          phase: Math.random() * Math.PI * 2,
          baseRotY: Math.random() * Math.PI * 2
        });
      });
    }

    /* ==================== INTERACTIVE GLOW STICK SPAWNER ==================== */
    function setGlowColor(colorHex, elem) {
      selectedGlowColor = colorHex;
      document.querySelectorAll('.glow-dot').forEach(d => d.classList.remove('selected'));
      if(elem) elem.classList.add('selected');

      // Update all existing dancer glow stick colors
      dancers.forEach(d => {
        if(d.glowStick) d.glowStick.material.color.setHex(colorHex);
        if(d.glowLight) d.glowLight.color.setHex(colorHex);
      });

      showToast("✨ Glow Stick Color Changed!");
    }

    function spawnGlowStick(x, z, colorHex = selectedGlowColor) {
      const group = new THREE.Group();
      const geo = new THREE.CylinderGeometry(0.05, 0.05, 1.0, 10);
      const mat = new THREE.MeshBasicMaterial({ color: colorHex });
      const stick = new THREE.Mesh(geo, mat);
      stick.rotation.z = Math.PI / 2 + (Math.random() - 0.5) * 0.4;
      stick.rotation.y = Math.random() * Math.PI * 2;
      group.add(stick);

      const light = new THREE.PointLight(colorHex, 1.5, 5);
      light.position.y = 0.2;
      group.add(light);

      group.position.set(x, 0.15, z);
      scene.add(group);
      glowSticks.push({ group, light, mat });

      showToast("⚡ Glow Stick Dropped on Dance Floor!");
    }

    function dropRandomGlowStick() {
      const rx = (Math.random() - 0.5) * 12 - 2;
      const rz = (Math.random() - 0.5) * 12 - 2;
      spawnGlowStick(rx, rz);
    }

    function onSceneDoubleClick(event) {
      const raycaster = new THREE.Raycaster();
      const mouse = new THREE.Vector2();
      mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
      mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

      raycaster.setFromCamera(mouse, camera);
      const intersects = raycaster.intersectObjects(scene.children, true);

      if (intersects.length > 0) {
        const pt = intersects[0].point;
        if (Math.abs(pt.x) < 22 && Math.abs(pt.z) < 22) {
          spawnGlowStick(pt.x, pt.z);
        }
      }
    }

    function toggleNeonMode() {
      isNeonMode = !isNeonMode;
      const btn = document.getElementById('neon-toggle-btn');

      if (isNeonMode) {
        scene.background = new THREE.Color(0x020204);
        scene.fog.color = new THREE.Color(0x020204);
        mainSpot.intensity = 0.1;
        ambientLight.intensity = 0.08;

        dancers.forEach(d => { if (d.glowLight) d.glowLight.intensity = 3.0; });
        glowSticks.forEach(g => { if (g.light) g.light.intensity = 3.5; });

        btn.classList.add('active');
        btn.style.background = '#00ff88';
        btn.style.color = '#111';
        showToast("⚡ Dark Neon Rave Mode Activated!");
      } else {
        scene.background = new THREE.Color(0x0e0c0a);
        scene.fog.color = new THREE.Color(0x0e0c0a);
        mainSpot.intensity = 1.5;
        ambientLight.intensity = 0.8;

        dancers.forEach(d => { if (d.glowLight) d.glowLight.intensity = 1.2; });
        glowSticks.forEach(g => { if (g.light) g.light.intensity = 1.5; });

        btn.classList.remove('active');
        btn.style.background = '';
        btn.style.color = '';
        showToast("☀️ Standard Warm Lights Restored");
      }
    }

    /* ==================== ANIMATION LOOP UPDATE ==================== */
    function animate() {
      requestAnimationFrame(animate);

      // Animate Dancers & Avatars
      const time = Date.now() * 0.003;
      dancers.forEach(d => {
        const t = time * d.speed + d.phase;
        if (d.type === 'dance' || d.type === 'dj') {
          d.mesh.position.y = d.baseY + Math.abs(Math.sin(t * 2)) * 0.15;
          d.leftArm.rotation.z = Math.sin(t) * 0.8 + 0.4;
          d.rightArm.rotation.z = -Math.sin(t) * 0.8 - 0.4;
          d.leftArm.rotation.x = Math.cos(t) * 0.6;
          d.rightArm.rotation.x = -Math.cos(t) * 0.6;
          d.mesh.rotation.y = d.baseRotY + Math.sin(t * 0.5) * 0.3;
        } else if (d.type === 'sway') {
          d.mesh.rotation.z = Math.sin(t * 0.8) * 0.08;
          d.rightArm.rotation.x = Math.sin(t) * 0.3 - 0.4;
        }
      });

      // Animate Falling Leaves
      if(leavesParticles) {
        const positions = leavesParticles.geometry.attributes.position.array;
        const vels = leavesParticles.userData.velocities;

        for(let i = 0; i < vels.length; i++) {
          positions[i*3+1] -= vels[i].y;
          positions[i*3] += Math.sin(Date.now() * 0.001 + i) * 0.01;

          if(positions[i*3+1] < 0.2) {
            positions[i*3+1] = 15;
            positions[i*3] = (Math.random() - 0.5) * 48;
          }
        }
        leavesParticles.geometry.attributes.position.needsUpdate = true;
      }

      // Pulse Sparkles
      if(sparklesParticles) {
        sparklesParticles.rotation.y += 0.0005;
      }

      controls.update();
      updateHotspotPositions();
      renderer.render(scene, camera);
    }

    function focusZone(zoneKey) {
      currentFocusZone = zoneKey;
      const config = zonePositions[zoneKey];
      if(!config) return;

      // Update Active Navigation Button
      document.querySelectorAll('.nav-btn').forEach(btn => btn.classList.remove('active'));
      const activeBtn = Array.from(document.querySelectorAll('.nav-btn')).find(b => b.textContent.toLowerCase().includes(zoneKey.replace('ciderbar','cider').replace('photobooth','photo').replace('masktable','mask').replace('dancefloor','dance')));
      if(activeBtn) activeBtn.classList.add('active');

      // Camera Lerp Target Animation
      const duration = 1200;
      const startCam = { x: camera.position.x, y: camera.position.y, z: camera.position.z };
      const startTgt = { x: controls.target.x, y: controls.target.y, z: controls.target.z };
      const startTime = performance.now();

      function step(now) {
        const elapsed = now - startTime;
        const progress = Math.min(elapsed / duration, 1);
        const ease = 1 - Math.pow(1 - progress, 3); // Cubic ease out

        camera.position.x = startCam.x + (config.cam.x - startCam.x) * ease;
        camera.position.y = startCam.y + (config.cam.y - startCam.y) * ease;
        camera.position.z = startCam.z + (config.cam.z - startCam.z) * ease;

        controls.target.x = startTgt.x + (config.target.x - startTgt.x) * ease;
        controls.target.y = startTgt.y + (config.target.y - startTgt.y) * ease;
        controls.target.z = startTgt.z + (config.target.z - startTgt.z) * ease;

        if(progress < 1) requestAnimationFrame(step);
      }
      requestAnimationFrame(step);

      // Update Info Card Details
      updateInfoCard(zoneKey);
    }

    /* Map 3D Coordinates to 2D HTML Hotspots */
    function updateHotspotPositions() {
      const hotspots = [
        { id: 'hotspot-entrance', pos: new THREE.Vector3(0, 4, 18) },
        { id: 'hotspot-dancefloor', pos: new THREE.Vector3(-2, 1, -2) },
        { id: 'hotspot-ciderbar', pos: new THREE.Vector3(-18, 2, 0) },
        { id: 'hotspot-photobooth', pos: new THREE.Vector3(18, 3, 0) },
        { id: 'hotspot-masktable', pos: new THREE.Vector3(10, 2, 10) }
      ];

      const widthHalf = window.innerWidth / 2;
      const heightHalf = window.innerHeight / 2;

      hotspots.forEach(item => {
        const elem = document.getElementById(item.id);
        if(!elem) return;

        const tempVec = item.pos.clone();
        tempVec.project(camera);

        // Check if behind camera
        if(tempVec.z > 1) {
          elem.style.display = 'none';
          return;
        }

        elem.style.display = 'flex';
        const x = (tempVec.x * widthHalf) + widthHalf;
        const y = -(tempVec.y * heightHalf) + heightHalf;
        elem.style.left = `${x}px`;
        elem.style.top = `${y}px`;
      });
    }

    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    }

    /* ==================== UI & CONTENT LOGIC ==================== */
    const cardData = {
      overview: {
        title: "Venetian Autumn Ballroom",
        subtitle: "Full Venue View",
        body: "A complete transformation of the school gymnasium into a warm harvest wonderland with glowing string light canopy, rustic wooden accents, and cozy activity zones.",
        actionText: "<i class='fa-solid fa-mask'></i> Customize Mask",
        actionFn: "openMaskModal()"
      },
      entrance: {
        title: "Grand Velvet Entrance",
        subtitle: "Arrival Portal",
        body: "Students arrive through a majestic timber arch draped in golden maple leaves, fairy lights, and rich burgundy curtains. Red carpet photo check-in welcomes every guest.",
        actionText: "<i class='fa-solid fa-camera'></i> Take Arrival Photo",
        actionFn: "triggerCameraFlash()"
      },
      dancefloor: {
        title: "DJ Dance Floor",
        subtitle: "Main Stage & Sound",
        body: "Glossy dark wooden dance platform encircled by warm gold ambient lighting. Student-requested clean hits, group dances, and the grand unmasking countdown at 8:00 PM.",
        actionText: "<i class='fa-solid fa-music'></i> Play Sample Beat",
        actionFn: "playSampleSong()"
      },
      ciderbar: {
        title: "Warm Cider & Treat Bar",
        subtitle: "Refreshment Station",
        body: "Rustic wooden refreshment bar serving hot spiced apple cider with cinnamon sticks, mini pumpkin pie bites, pretzel twists, and gold mask cupcakes.",
        actionText: "<i class='fa-solid fa-mug-hot'></i> Pour Virtual Cider",
        actionFn: "pourCiderAction()"
      },
      photobooth: {
        title: "Masquerade Photo Lounge",
        subtitle: "Memories & Props",
        body: "Vintage ornate gold mirror backdrop with plush seating. Complete with masquerade feather boas, handheld golden masks, and instant print photo strips.",
        actionText: "<i class='fa-solid fa-camera-retro'></i> Snap Photo Flash",
        actionFn: "triggerCameraFlash()"
      },
      masktable: {
        title: "DIY Mask Crafting Table",
        subtitle: "Creative Station",
        body: "Every guest receives a blank Venetian mask upon entry! Use glitter pens, velvet ribbons, and metallic paint to personalize your unique masquerade look.",
        actionText: "<i class='fa-solid fa-wand-magic-sparkles'></i> Open Mask Studio",
        actionFn: "openMaskModal()"
      }
    };

    function updateInfoCard(key) {
      const data = cardData[key] || cardData.overview;
      document.getElementById('card-title').textContent = data.title;
      document.getElementById('card-subtitle').textContent = data.subtitle;
      document.getElementById('card-body').textContent = data.body;

      const actionBtn = document.getElementById('card-actions').children[0];
      actionBtn.innerHTML = data.actionText;
      actionBtn.setAttribute('onclick', data.actionFn);
    }

    function toggleDimLighting() {
      isNightMode = !isNightMode;
      const targetIntensity = isNightMode ? 0.2 : 0.8;

      scene.children.forEach(child => {
        if(child.isAmbientLight) child.intensity = targetIntensity;
      });

      showToast(isNightMode ? "🌙 Dance Lights Dimmed!" : "☀️ Evening Lights Restored!");
    }

    function triggerCameraFlash() {
      const flash = document.getElementById('flash-overlay');
      flash.style.opacity = '1';
      setTimeout(() => { flash.style.opacity = '0'; }, 150);
      showToast("📸 Photo Captured! Added to Class Gallery!");
    }

    function pourCiderAction() {
      showToast("🍎 Fresh Spiced Cider Poured!");
    }

    function playSampleSong() {
      showToast("🎵 DJ Queue: Autumn Lo-Fi Dance Remix Playing!");
    }

    function showToast(msg) {
      const toast = document.getElementById('toast');
      document.getElementById('toast-text').textContent = msg;
      toast.classList.add('active');
      setTimeout(() => toast.classList.remove('active'), 2500);
    }

    /* ==================== MASK CUSTOMIZER CANVAS ==================== */
    let maskColor = '#7C2D12';
    let maskTrim = '#F59E0B';

    function drawMaskPreview() {
      const cvs = document.getElementById('mask-canvas');
      const ctx = cvs.getContext('2d');
      ctx.clearRect(0, 0, cvs.width, cvs.height);

      ctx.save();
      ctx.translate(cvs.width / 2, cvs.height / 2);

      // Mask Body Shape
      ctx.fillStyle = maskColor;
      ctx.strokeStyle = maskTrim;
      ctx.lineWidth = 4;

      ctx.beginPath();
      ctx.moveTo(-100, -20);
      ctx.bezierCurveTo(-80, -60, -20, -50, 0, -20);
      ctx.bezierCurveTo(20, -50, 80, -60, 100, -20);
      ctx.bezierCurveTo(110, 20, 50, 45, 0, 15);
      ctx.bezierCurveTo(-50, 45, -110, 20, -100, -20);
      ctx.closePath();
      ctx.fill();
      ctx.stroke();

      // Left Eye Hole
      ctx.fillStyle = '#1C1917';
      ctx.beginPath();
      ctx.ellipse(-45, -10, 22, 12, -Math.PI/12, 0, Math.PI*2);
      ctx.fill();

      // Right Eye Hole
      ctx.beginPath();
      ctx.ellipse(45, -10, 22, 12, Math.PI/12, 0, Math.PI*2);
      ctx.fill();

      // Feather Accents
      ctx.fillStyle = maskTrim;
      for(let i = -2; i <= 2; i++) {
        ctx.beginPath();
        ctx.ellipse(i*12, -45, 6, 20, (i*0.2), 0, Math.PI*2);
        ctx.fill();
      }

      ctx.restore();
    }

    function setMaskColor(color, elem) {
      maskColor = color;
      document.querySelectorAll('.color-dot').forEach(d => d.classList.remove('selected'));
      elem.classList.add('selected');
      drawMaskPreview();
    }

    function setMaskTrim(trim) {
      maskTrim = trim;
      drawMaskPreview();
    }

    function openMaskModal() {
      document.getElementById('mask-modal').classList.add('active');
      drawMaskPreview();
    }

    function closeMaskModal() {
      document.getElementById('mask-modal').classList.remove('active');
    }

    function wearCustomMask() {
      closeMaskModal();
      showToast("✨ Custom Venetian Mask Equipped!");
    }

    /* ==================== WEB AUDIO API AMBIENCE SYNTH ==================== */
    let audioCtx, isAudioPlaying = false;

    function toggleAudio() {
      const btn = document.getElementById('audio-toggle-btn');
      
      if(!audioCtx) {
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      }

      if(!isAudioPlaying) {
        audioCtx.resume();
        startAmbientSynth();
        isAudioPlaying = true;
        btn.classList.add('active');
        btn.innerHTML = '<i class="fa-solid fa-volume-high"></i>';
        showToast("🎵 Cozy Ambience Audio On!");
      } else {
        audioCtx.suspend();
        isAudioPlaying = false;
        btn.classList.remove('active');
        btn.innerHTML = '<i class="fa-solid fa-volume-xmark"></i>';
        showToast("🔇 Audio Muted");
      }
    }

    function startAmbientSynth() {
      // Soft Warm Chord Synthesizer Pad
      const frequencies = [220, 277.18, 329.63, 440]; // A major warm chord
      frequencies.forEach(freq => {
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();

        osc.type = 'sine';
        osc.frequency.setValueAtTime(freq, audioCtx.currentTime);

        gain.gain.setValueAtTime(0.015, audioCtx.currentTime);

        osc.connect(gain);
        gain.connect(audioCtx.destination);
        osc.start();
      });
    }

    // Initialize 3D Scene on Window Load
    window.onload = () => {
      init3D();
    };
  </script>
</body>
</html>
