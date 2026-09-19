<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Bubble Shooter</title>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
  :root{
    --bg1:#1b1035;
    --bg2:#0d0620;
    --panel:#ffffff20;
    --text:#f5f0ff;
    --accent:#ff5da2;
  }
  *{box-sizing:border-box;}
  html,body{
    margin:0;padding:0;height:100%;
    background:radial-gradient(ellipse at top, var(--bg1), var(--bg2));
    color:var(--text);
    font-family:'Segoe UI', system-ui, -apple-system, sans-serif;
    overflow:hidden;
    -webkit-tap-highlight-color: transparent;
  }
  #wrap{
    display:flex;
    flex-direction:column;
    align-items:center;
    justify-content:flex-start;
    height:100%;
    padding:10px;
  }
  #hud{
    width:100%;
    max-width:480px;
    display:flex;
    justify-content:space-between;
    align-items:center;
    padding:6px 12px;
    font-size:15px;
    font-weight:600;
    letter-spacing:0.5px;
  }
  #hud .score{color:var(--accent);}
  #stage{
    position:relative;
    border-radius:14px;
    overflow:hidden;
    box-shadow:0 10px 40px rgba(0,0,0,0.5), inset 0 0 0 2px #ffffff22;
    touch-action:none;
    background:#00000030;
  }
  canvas{display:block;}
  #overlay{
    position:absolute;
    inset:0;
    display:none;
    align-items:center;
    justify-content:center;
    flex-direction:column;
    background:rgba(10,5,25,0.85);
    text-align:center;
    padding:20px;
  }
  #overlay h1{
    margin:0 0 8px;
    font-size:28px;
  }
  #overlay p{
    margin:0 0 18px;
    opacity:0.8;
  }
  button{
    background:linear-gradient(135deg,#ff5da2,#7b5dff);
    color:white;
    border:none;
    padding:12px 26px;
    border-radius:30px;
    font-size:15px;
    font-weight:700;
    cursor:pointer;
    box-shadow:0 6px 18px rgba(123,93,255,0.4);
  }
  button:active{transform:scale(0.96);}
  #hint{
    margin-top:8px;
    font-size:12px;
    opacity:0.55;
    text-align:center;
    max-width:400px;
  }
</style>
</head>
<body>
<div id="wrap">
  <div id="hud">
    <div>Score: <span class="score" id="scoreVal">0</span></div>
    <div id="nextLabel" style="opacity:0.7;font-weight:500;">Next: <span id="nextDot" style="display:inline-block;width:12px;height:12px;border-radius:50%;vertical-align:middle;"></span></div>
  </div>
  <div id="stage">
    <canvas id="game"></canvas>
    <div id="overlay">
      <h1 id="overlayTitle">Game Over</h1>
      <p id="overlayText">The bubbles got you this time.</p>
      <button id="restartBtn">Play Again</button>
    </div>
  </div>
  <div id="hint">Aim with mouse / finger, click or tap to shoot. Match 3+ same-color bubbles to pop them!</div>
</div>

<script>
(function(){
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');
  const overlay = document.getElementById('overlay');
  const overlayTitle = document.getElementById('overlayTitle');
  const overlayText = document.getElementById('overlayText');
  const restartBtn = document.getElementById('restartBtn');
  const scoreVal = document.getElementById('scoreVal');
  const nextDot = document.getElementById('nextDot');

  // ---- Config ----
  const COLORS = ['#ff5252','#ffca28','#66bb6a','#42a5f5','#ab47bc','#ff7043'];
  const ROWS_VISIBLE = 12;
  const COLS = 10;
  let R = 18; // bubble radius, recalculated on resize
  let stageW = 400, stageH = 560;

  function resize(){
    const maxW = Math.min(window.innerWidth - 20, 420);
    const maxH
