<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>NEON STRIKER: FIXED EDITION</title>
    <style>
        body { margin: 0; overflow: hidden; background: #000; font-family: 'Orbitron', sans-serif; color: white; }
        #menu, #hud { position: absolute; inset: 0; display: flex; flex-direction: column; justify-content: center; align-items: center; pointer-events: none; }
        #menu { background: rgba(0,0,0,0.8); pointer-events: all; z-index: 10; }
        .panel { background: #111; padding: 30px; border: 3px solid #00f2fe; border-radius: 20px; text-align: center; }
        button { background: transparent; border: 2px solid #00f2fe; color: #00f2fe; padding: 12px 20px; cursor: pointer; font-family: 'Orbitron'; margin: 5px; pointer-events: all; }
        #garage-trigger { position: absolute; bottom: 20px; left: 20px; color: #ff00ff; border-color: #ff00ff; pointer-events: all; }
        #scoreboard { position: absolute; top: 20px; font-size: 32px; background: rgba(0,0,0,0.5); padding: 10px; border-radius: 10px; }
    </style>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;900&display=swap" rel="stylesheet">
</head>
<body>
    <button id="garage-trigger" onclick="location.reload()">🏠 RESET / GARAGE</button>

    <div id="menu">
        <div class="panel">
            <h1>NEON STRIKER</h1>
            <p id="car-name">MODEL: FENNEC</p>
            <button onclick="startGame(0.05)">START GAME</button>
        </div>
    </div>

    <div id="hud"><div id="scoreboard"><span id="s-blue" style="color:#00f2fe">0</span> - <span id="s-orange" style="color:#ff8c00">0</span></div></div>

    <script type="importmap">
        { "imports": { "three": "https://unpkg.com/three@0.160.0/build/three.module.js", "cannon": "https://cdn.jsdelivr.net/npm/cannon-es@0.20.0/+esm" } }
    </script>

    <script type="module">
        import * as THREE from 'three';
        import * as CANNON from 'cannon';

        let gameState = 'MENU', score = { blue: 0, orange: 0 };
        const keys = {};

        // --- SCENE SETUP ---
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        const world = new CANNON.World();
        world.gravity.set(0, -20, 0);

        // --- FIXED GROUND (No more floating grid!) ---
        const floorGeo = new THREE.BoxGeometry(100, 1, 160);
        const floorMat = new THREE.MeshStandardMaterial({ color: 0x111122, roughness: 0.1 });
        const floorMesh = new THREE.Mesh(floorGeo, floorMat);
        floorMesh.position.y = -0.5;
        scene.add(floorMesh);

        const floorBody = new CANNON.Body({ mass: 0, shape: new CANNON.Box(new CANNON.Vec3(50, 0.5, 80)) });
        world.addBody(floorBody);

        // Grid lines on top of solid floor
        const grid = new THREE.GridHelper(160, 40, 0x00f2fe, 0x004444);
        grid.position.y = 0.05;
        scene.add(grid);

        // --- CAR ---
        const carGroup = new THREE.Group();
        const bodyMesh = new THREE.Mesh(new THREE.BoxGeometry(3, 1.2, 4.5), new THREE.MeshStandardMaterial({color: 0x00f2fe}));
        bodyMesh.position.y = 0.6;
        carGroup.add(bodyMesh);
        scene.add(carGroup);

        const carBody = new CANNON.Body({ mass: 50, shape: new CANNON.Box(new CANNON.Vec3(1.5, 0.6, 2.25)) });
        carBody.position.set(0, 2, 40);
        carBody.angularDamping = 0.9; // Prevents spinning forever
        world.addBody(carBody);

        // --- BALL ---
        const ballMesh = new THREE.Mesh(new THREE.SphereGeometry(2.5), new THREE.MeshStandardMaterial({color: 0xffffff}));
        scene.add(ballMesh);
        const ballBody = new CANNON.Body({ mass: 5, shape: new CANNON.Sphere(2.5) });
        ballBody.position.set(0, 5, 0);
        world.addBody(ballBody);

        scene.add(new THREE.AmbientLight(0xffffff, 1));

        // --- CONTROLS & START ---
        window.startGame = () => {
            gameState = 'PLAYING';
            document.getElementById('menu').style.display = 'none';
        };

        window.addEventListener('keydown', e => keys[e.code] = true);
        window.addEventListener('keyup', e => keys[e.code] = false);

        function update() {
            requestAnimationFrame(update);
            if(gameState === 'PLAYING') {
                world.fixedStep();

                // Movement Logic (FORCE APPLIED)
                const fwd = new THREE.Vector3(0,0,-1).applyQuaternion(carGroup.quaternion);
                if(keys['KeyW']) {
                    carBody.velocity.x += fwd.x * 0.8;
                    carBody.velocity.z += fwd.z * 0.8;
                }
                if(keys['KeyS']) {
                    carBody.velocity.x -= fwd.x * 0.5;
                    carBody.velocity.z -= fwd.z * 0.5;
                }
                if(keys['KeyA']) carBody.angularVelocity.y = 5;
                else if(keys['KeyD']) carBody.angularVelocity.y = -5;
                else carBody.angularVelocity.y *= 0.9;

                // Sync Visuals
                carGroup.position.copy(carBody.position);
                carGroup.quaternion.copy(carBody.quaternion);
                ballMesh.position.copy(ballBody.position);

                // Camera
                const camOffset = new THREE.Vector3(0, 10, 25).applyQuaternion(carGroup.quaternion);
                camera.position.lerp(carGroup.position.clone().add(camOffset), 0.1);
                camera.lookAt(ballMesh.position);
            }
            renderer.render(scene, camera);
        }
        update();
    </script>
</body>
</html>
