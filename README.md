<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>NEON STRIKER - PRO FENNEC</title>
    <style>
        body { margin: 0; overflow: hidden; background: #000; font-family: 'Orbitron', sans-serif; color: white; }
        #menu, #hud { position: absolute; inset: 0; display: flex; flex-direction: column; justify-content: center; align-items: center; pointer-events: none; }
        
        /* Menu Styling */
        #menu { background: radial-gradient(circle, rgba(10,10,30,0.95) 0%, rgba(0,0,0,1) 100%); pointer-events: all; z-index: 10; }
        .panel { background: rgba(20, 20, 40, 0.9); padding: 40px; border: 3px solid #00f2fe; border-radius: 20px; text-align: center; min-width: 400px; box-shadow: 0 0 50px rgba(0, 242, 254, 0.4); }
        h1 { font-size: 3.5rem; margin-bottom: 10px; color: #00f2fe; text-shadow: 0 0 20px #00f2fe; }
        
        button { background: transparent; border: 2px solid #00f2fe; color: #00f2fe; padding: 15px 30px; font-family: 'Orbitron'; cursor: pointer; border-radius: 8px; margin: 10px; pointer-events: all; transition: 0.3s; font-weight: bold; }
        button:hover { background: #00f2fe; color: #000; box-shadow: 0 0 20px #00f2fe; }
        
        /* HUD Styling */
        #hud { display: none; }
        #scoreboard { position: absolute; top: 20px; font-size: 48px; font-weight: 900; background: rgba(0,0,0,0.5); padding: 10px 30px; border-radius: 10px; border: 1px solid #00f2fe; pointer-events: none; }
    </style>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;900&display=swap" rel="stylesheet">
</head>
<body>

    <div id="menu">
        <div class="panel">
            <h1>NEON STRIKER</h1>
            <p
