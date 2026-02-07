<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>NEON STRIKER - HYPER FENNEC</title>
    <style>
        body { margin: 0; overflow: hidden; background: #000; font-family: 'Orbitron', sans-serif; color: white; }
        #menu, #hud { position: absolute; inset: 0; display: flex; flex-direction: column; justify-content: center; align-items: center; pointer-events: none; }
        #menu { background: radial-gradient(circle, rgba(10,10,30,0.95) 0%, rgba(0,0,0,1) 100%); pointer-events: all; z-index: 10; }
        .panel { background: rgba(20, 20, 40, 0.9); padding: 40px; border: 3px solid #00f2fe; border-radius: 20px; text-align: center; min-width: 400px; box-shadow: 0 0 50px rgba(0, 242, 254, 0.4); }
        h1 { font-size: 3rem; color: #00f2fe; text-shadow: 0 0 20px #00f2fe; margin-bottom: 10px; }
        button { background: transparent; border: 2px solid #00f2fe; color: #00f2fe; padding: 15px 30px; font-family: 'Orbitron'; cursor: pointer; border-radius: 8px; margin: 10px; pointer-events: all; transition: 0.3s; font-weight: bold; }
        button:hover { background: #00f2fe; color: #000; box-shadow: 0 0 20px #00f2fe; }
        #hud { display: none; }
        #scoreboard { position: absolute; top: 20px; font-size: 48px; font-weight: 900; background: rgba(0,0,0,0.5); padding: 10px 30px; border-radius: 10px; border: 1px solid #00f2fe; }
    </style>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;900&display=swap" rel="stylesheet">
</head>
<body>
    <div id="menu">
        <div class="panel">
            <h1>NEON STRIKER</h1>
            <p>FENNEC LOADOUT: HYPER TURN SPEED</p>
            <button onclick="startGame(0.04)">EASY</button>
            <button onclick="startGame(0.08)">MEDIUM</button>
            <button onclick="startGame(0.16)">PRO</button>
            <p style="font-size: 11px; color: #777; margin-top: 15px;">SPACE: Jump/Dodge | WASD: Hyper Steering</p>
        </div>
    </div>
    <div id="hud"><div id="scoreboard"><span id="s-blue" style="color:#00f2fe">0</span> - <span id="s-orange" style="color:#ff8c00">0</span></div></div>

    <script type="importmap">
        { "imports": { "three": "https://unpkg.com/three@0.160.0/build/three.module.js", "cannon": "https://cdn.jsdelivr.net/npm/cannon-es@0.20.0/+esm" } }
    </script>

    <script type="module">
        import * as THREE from 'three';
        import * as CANNON from 'cannon';

        let gameState = 'MENU', botDifficulty = 0.08, score = { blue: 0, orange: 0 }, jumpCount = 0;
        const keys = {};

        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        const world = new CANNON.World();
        world.gravity.set(0, -22, 0);

        scene.add(new THREE.GridHelper(400, 80, 0x00f2fe, 0x050515));
        const groundBody = new CANNON.Body({ mass: 0, shape: new CANNON.Plane() });
        groundBody.quaternion.setFromEuler(-Math.PI / 2, 0, 0);
        world.addBody(groundBody);

        const ballBody = new CANNON.Body({ mass: 2, shape: new CANNON.Sphere(2.5) });
        ballBody.position.set(0, 5, 0);
        world.addBody(ballBody);
        const ballMesh = new THREE.Mesh(new THREE.SphereGeometry(2.5, 32), new THREE.MeshStandardMaterial({ color: 0xffffff, emissive: 0x111111 }));
        scene.add(ballMesh);

        function createFennec(color) {
            const group = new THREE.Group();
            const mat = new THREE.MeshStandardMaterial({ color: color });
            const wheelMat = new THREE.MeshStandardMaterial({ color: 0x111111 });

            const base = new THREE.Mesh(new THREE.BoxGeometry(3.2, 1.2, 4.8), mat);
            base.position.y = 0.6;
            group.add(base);

            const cabin = new THREE.Mesh(new THREE.BoxGeometry(2.8, 1, 2.5), mat);
            cabin.position.set(0, 1.6, -0.5);
            group.add(cabin);

            const wheelGeo = new THREE.CylinderGeometry(0.6, 0.6, 0.4, 16);
            wheelGeo.rotateZ(Math.PI / 2);
            const wheelPos = [[1.8, 0.4, 1.6], [-1.8, 0.4, 1.6], [1.8, 0.4, -1.6], [-1.8, 0.4, -1.6]];
            const wheels = wheelPos.map(pos => {
                const w = new THREE.Mesh(wheelGeo, wheelMat);
                w.position.set(...pos);
                group.add(w);
                return w;
            });

            scene.add(group);
            return { group, wheels };
        }

        const player = createFennec(0x00eaff);
        const playerBody = new CANNON.Body({ mass: 35, shape: new CANNON.Box(new CANNON.Vec3(1.6, 1.1, 2.4)) });
        playerBody.position.set(0, 2, 35);
        playerBody.angularDamping = 0.99; // Hyper damping for hyper speed
        world.addBody(playerBody);

        const bot = createFennec(0xff8c00);
        const botBody = new CANNON.Body({ mass: 35, shape: new CANNON.Box(new CANNON.Vec3(1.6, 1.
