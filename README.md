<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>ROCKET LEAGUE 2.0</title>
    <style>
        body { margin: 0; overflow: hidden; background: #000; font-family: 'Orbitron', sans-serif; color: white; }
        #gui, #menu { position: absolute; inset: 0; display: flex; flex-direction: column; justify-content: center; align-items: center; pointer-events: none; }
        
        /* Menu Styling */
        #menu { background: radial-gradient(circle, rgba(10,10,30,0.95) 0%, rgba(0,0,0,1) 100%); pointer-events: all; z-index: 10; }
        .panel { background: rgba(20, 20, 40, 0.9); padding: 40px; border: 3px solid #00f2fe; border-radius: 20px; text-align: center; min-width: 450px; box-shadow: 0 0 50px rgba(0, 242, 254, 0.4); }
        h1 { font-size: 48px; margin-bottom: 10px; color: #00f2fe; text-shadow: 0 0 20px #00f2fe; letter-spacing: 3px; }
        h2 { font-size: 18px; color: #7000ff; margin-bottom: 30px; text-transform: uppercase; }
        
        .btn-group { display: flex; gap: 20px; margin-bottom: 30px; justify-content: center; }
        button { background: transparent; border: 2px solid #00f2fe; color: #00f2fe; padding: 15px 30px; font-family: 'Orbitron'; cursor: pointer; transition: 0.3s; font-weight: bold; border-radius: 8px; font-size: 16px; }
        button:hover { background: #00f2fe; color: #000; box-shadow: 0 0 30px #00f2fe; }
        button.active { background: #00f2fe; color: #000; }
        #start-btn { font-size: 24px; width: 100%; border-color: #ff00ff; color: #ff00ff; margin-top: 10px; }
        #start-btn:hover { background: #ff00ff; color: #fff; box-shadow: 0 0 40px #ff00ff; }

        /* HUD */
        #hud { position: absolute; inset: 0; pointer-events: none; display: none; }
        #scoreboard { position: absolute; top: 30px; left: 50%; transform: translateX(-50%); font-size: 42px; font-weight: 900; }
        #boost-container { position: absolute; bottom: 50px; right: 50px; width: 300px; text-align: right; }
        #boost-bar { height: 16px; background: rgba(0,242,254,0.1); border: 2px solid #00f2fe; border-radius: 20px; overflow: hidden; margin-top: 5px; }
        #boost-fill { height: 100%; width: 100%; background: linear-gradient(90deg, #00f2fe, #7000ff); box-shadow: 0 0 20px #00f2fe; }

        /* Goal Effects */
        #goal-flash { position: absolute; inset: 0; background: white; opacity: 0; pointer-events: none; z-index: 100; transition: opacity 0.1s; }
        #goal-text { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); font-size: 120px; font-weight: 900; display: none; z-index: 101; font-style: italic; }
    </style>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;900&display=swap" rel="stylesheet">
</head>
<body>

    <div id="goal-flash"></div>
    <div id="goal-text">GOAL!</div>

    <div id="menu">
        <div class="panel">
            <h1>ROCKET LEAGUE 2.0</h1>
            <h2>Car Garage*</h2>
            <div class="btn-group">
                <button id="btn-octane" class="active" onclick="selectCar('octane')">OCTANE-X</button>
                <button id="btn-dominus" onclick="selectCar('dominus')">DOMINUS-GT</button>
            </div>
            <button id="start-btn" onclick="startGame()">ENTER ARENA</button>
        </div>
    </div>

    <div id="hud">
        <div id="scoreboard">
            <span style="color:#00f2fe" id="s-blue">0</span> 
            <span style="color:white; margin: 0 20px;">-</span> 
            <span style="color:#ff8c00" id="s-orange">0</span>
        </div>
        <div id="boost-container">
            <span style="font-size: 14px; letter-spacing: 2px;">NITRO BOOST</span>
            <div id="boost-bar"><div id="boost-fill"></div></div>
        </div>
    </div>

    <script type="importmap">
        {
            "imports": {
                "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
                "cannon": "https://cdn.jsdelivr.net/npm/cannon-es@0.20.0/+esm"
            }
        }
    </script>

    <script type="module">
        import * as THREE from 'three';
        import * as CANNON from 'cannon';

        let gameState = 'MENU';
        let score = { blue: 0, orange: 0 };
        let boost = 100;
        let ballCam = true;
        let shake = 0;
        const keys = {};

        // --- SETUP ---
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        const world = new CANNON.World();
        world.gravity.set(0, -25, 0);

        // --- ARENA ---
        scene.add(new THREE.GridHelper(400, 80, 0x00f2fe, 0x050515));
        const groundBody = new CANNON.Body({ mass: 0, shape: new CANNON.Plane() });
        groundBody.quaternion.setFromEuler(-Math.PI / 2, 0, 0);
        world.addBody(groundBody);

        // --- BALL ---
        const ballBody = new CANNON.Body({ mass: 2, shape: new CANNON.Sphere(2.5), material: new CANNON.Material({restitution: 0.9}) });
        ballBody.position.set(0, 10, 0);
        world.addBody(ballBody);
        const ballMesh = new THREE.Mesh(new THREE.SphereGeometry(2.5, 32, 32), new THREE.MeshStandardMaterial({ color: 0xffffff, emissive: 0x00f2fe, emissiveIntensity: 0.5 }));
        scene.add(ballMesh);

        // --- CAR CONSTRUCTION ---
        let carBody, carMesh;
        window.selectCar = (type) => {
            if(carBody) world.removeBody(carBody);
            if(carMesh) scene.remove(carMesh);

            const isOctane = type === 'octane';
            document.getElementById('btn-octane').className = isOctane ? 'active' : '';
            document.getElementById('btn-dominus').className = isOctane ? '' : 'active';

            const dim = isOctane ? {w:1.6, h:1.1, l:2.4} : {w:1.7, h:0.7, l:2.8};
            carBody = new CANNON.Body({ mass: 35, shape: new CANNON.Box(new CANNON.Vec3(dim.w, dim.h, dim.l)) });
            carBody.position.set(0, 2, 30);
            world.addBody(carBody);

            carMesh = new THREE.Group();
            const mat = new THREE.MeshStandardMaterial({ color: isOctane ? 0x00eaff : 0xff0044, metalness: 0.9, roughness: 0.1 });
            
            // Chassis
            const body = new THREE.Mesh(new THREE.BoxGeometry(dim.w*1.9, dim.h*0.8, dim.l*1.8), mat);
            carMesh.add(body);
            
            // Cabin
            const cabin = new THREE.Mesh(new THREE.BoxGeometry(dim.w*1.3, dim.h*0.7, dim.l*0.7), new THREE.MeshStandardMaterial({color:0x000000, metalness:1}));
            cabin.position.set(0, dim.h*0.7, isOctane ? 0.2 : 0.4);
            carMesh.add(cabin);

            // Wheels
            const wGeo = new THREE.CylinderGeometry(0.6, 0.6, 0.5, 16);
            const wMat = new THREE.MeshStandardMaterial({color:0x111111});
            const wPos = [[1.5, -0.4, 1.4], [-1.5, -0.4, 1.4], [1.5, -0.4, -1.4], [-1.5, -0.4, -1.4]];
            wPos.forEach(p => {
                const w = new THREE.Mesh(wGeo, wMat);
                w.rotation.z = Math.PI/2;
                w.position.set(p[0], p[1], p[2]);
                carMesh.add(w);
            });

            scene.add(carMesh);
        };

        window.startGame = () => {
            gameState = 'PLAYING';
            document.getElementById('menu').style.display = 'none';
            document.getElementById('hud').style.display = 'block';
        };

        // Initialize Car
        selectCar('octane');

        // Lights
        scene.add(new THREE.AmbientLight(0xffffff, 0.5));
        const sun = new THREE.DirectionalLight(0xffffff, 1.5);
        sun.position.set(10, 50, 10);
        scene.add(sun);

        // Input
        window.addEventListener('keydown', e => {
            keys[e.code] = true;
            if(e.code === 'KeyK' && Math.abs(carBody.velocity.y) < 0.2) carBody.velocity.y = 14;
        });
        window.addEventListener('keyup', e => keys[e.code] = false);

        function triggerGoal(team) {
            shake = 2.0;
            const flash = document.getElementById('goal-flash');
            const text = document.getElementById('goal-text');
            flash.style.opacity = '1';
            text.style.display = 'block';
            text.style.color = team === 'blue' ? '#00f2fe' : '#ff8c00';
            
            setTimeout(() => flash.style.opacity = '0', 100);
            setTimeout(() => {
                text.style.display = 'none';
                ballBody.position.set(0, 10, 0);
                ballBody.velocity.set(0,0,0);
                carBody.position.set(0, 2, 30);
                carBody.velocity.set(0,0,0);
            }, 2000);
        }

        function update() {
            requestAnimationFrame(update);

            if(gameState === 'PLAYING') {
                world.fixedStep();
                
                // Controls
                const fwd = new THREE.Vector3(0,0,-1).applyQuaternion(carMesh.quaternion);
                if(keys['KeyW']) carBody.velocity.addScaledVector(0.7, new CANNON.Vec3(fwd.x, fwd.y, fwd.z), carBody.velocity);
                if(keys['KeyS']) carBody.velocity.addScaledVector(-0.4, new CANNON.Vec3(fwd.x, fwd.y, fwd.z), carBody.velocity);
                if(keys['KeyA']) carBody.angularVelocity.y = 3.5;
                else if(keys['KeyD']) carBody.angularVelocity.y = -3.5;
                else carBody.angularVelocity.y *= 0.9;

                // Nitro
                if(keys['ShiftLeft'] && boost > 0) {
                    carBody.velocity.addScaledVector(0.9, new CANNON.Vec3(fwd.x, fwd.y, fwd.z), carBody.velocity);
                    boost -= 0.8;
                } else if(boost < 100) boost += 0.2;

                // Sync Visuals
                carMesh.position.copy(carBody.position);
                carMesh.quaternion.copy(carBody.quaternion);
                ballMesh.position.copy(ballBody.position);
                
                // Goals
                if(ballBody.position.z < -60) { score.blue++; triggerGoal('blue'); ballBody.position.z = 0; }
                if(ballBody.position.z > 60) { score.orange++; triggerGoal('orange'); ballBody.position.z = 0; }

                // HUD
                document.getElementById('s-blue').innerText = score.blue;
                document.getElementById('s-orange').innerText = score.orange;
                document.getElementById('boost-fill').style.width = boost + "%";

                // Camera
                const camPos = carMesh.position.clone().add(new THREE.Vector3(0, 6, 14).applyQuaternion(carMesh.quaternion));
                camera.position.lerp(camPos, 0.1);
                if(shake > 0) {
                    camera.position.x += (Math.random()-0.5) * shake;
                    camera.position.y += (Math.random()-0.5) * shake;
                    shake *= 0.9;
                }
                camera.lookAt(ballMesh.position);
            } else {
                carMesh.rotation.y += 0.01;
                camera.position.set(10, 5, 10);
                camera.lookAt(carMesh.position);
            }

            renderer.render(scene, camera);
        }

        update();
    </script>
</body>
</html>
