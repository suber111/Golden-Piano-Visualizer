<!DOCTYPE html>
<html lang="pl">
<head>
  <meta charset="UTF-8" />
  <title>Golden Visuals</title>
  <script src="https://cdn.jsdelivr.net/npm/tone@14.7.77/build/Tone.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@tonejs/midi@2.0.27"></script>
  <style>
    body {
      margin: 0;
      min-height: 100vh;
      background: linear-gradient(120deg,#23243e 0%,#30323d 100%);
      overflow-x: hidden;
      color: #fff;
      font-family: Arial, sans-serif;
    }
    #ui {
      position: fixed;
      z-index: 2;
      top: 10px;
      left: 12px;
      background: rgba(44,46,70,0.95);
      border-radius: 14px;
      box-shadow: 0 2px 28px #2229;
      padding: 12px 18px 18px 18px;
      max-width: 560px;
      user-select: none;
    }
    #controls, #extra, #morefx, #fx2 {margin-bottom:12px;}
    #controls input, #controls select,
    #extra input, #extra label,
    #morefx input, #morefx select,
    #fx2 input {
      margin:6px 9px 6px 7px;
    }
    #extra, #fx2 {margin-top: 2px;}
    button, select, input[type="checkbox"], input[type="number"], input[type="range"] {
      cursor: pointer;
    }
    canvas {
      display: block;
      margin: 44px auto 0 auto;
      border-radius: 18px;
      box-shadow: 0 4px 64px #000a, 0 0 32px #ffd95a44;
      outline: none;
      background: transparent;
    }
    .track-color-label {
      margin-right: 16px;
      display: inline-block;
      font-size: 15px;
    }
    .control-label, .extra-label, .morefx-label {
      font-size: 15px;
    }
    #status {font-size: 17px; color: #ffd95a; margin-left: 36px;}
    .rec-btn {
      font-size: 17px; background:#222; color:#ffe660;
      border-radius:7px; border:none; padding:7px 24px; margin-right:10px; 
      transition: background 0.14s;
    }
    .rec-btn:hover {background:#33313a;}
    ::selection {background: #ffd95a22;}
  </style>
</head>
<body>
<div id="ui">
  <button id="recWebmBtn" class="rec-btn">▶️ Record WebM</button>
  <span id="recWebmStatus"></span>
  <div id="controls">
    <label class="control-label">Particle style: 
      <select id="particleType">
        <option value="classic">balls</option>
        <option value="star">stars</option>
        <option value="rect">rectangles</option>
        <option value="ring">rings</option>
        <option value="snowflake">❄️ snowflake</option>
      </select>
    </label>
    <label class="control-label">amount: <input id="particleAmount" type="number" min="0" max="500" value="80"></label>
    <label class="control-label">size: <input id="particleSize" type="number" min="0" max="12" step="0.1" value="1.2"></label>
    <label class="control-label">speed: <input id="particleSpeed" type="number" step="0.1" min="0." max="3" value="0.8"></label>
    <label class="control-label">Alpha: <input id="particleAlpha" type="number" step="0.01" min="0.0" max="100" value="0.7"></label>
    <label class="control-label">Disable particles: <input id="disableParticles" type="checkbox"></label>
    <button id="resetParticles">Reset Particle</button>
    <button id="clearMidi">Clear MIDI</button>
  </div>
  <div id="extra">
    <label class="extra-label">piano width: <input id="pianoWidth" type="number" min="400" max="1920" step="10" value="1920"></label>
    <label class="extra-label">piano alpha: <input id="pianoAlpha" type="range" min="0.2" max="1.0" step="0.01" value="1" style="width:70px;"></label>
    <label class="extra-label">Night Mode: <input id="nightMode" type="checkbox" checked></label>
    <label class="extra-label">Tempo: <input id="tempo" type="number" min="30" max="220" value="120" style="width:48px;"></label>
    <label class="extra-label">Autoplay: <input id="autoplay" type="checkbox"></label>
    <label class="extra-label">note delay: <input id="noteDelay" type="number" min="0" max="10" step="0.1" value="1.0" style="width:48px;"></label>
    <label class="extra-label"><button id="pauseBtn">Pause</button></label>
    <label class="extra-label">mark keys notes: <input id="markKeysNotes" type="checkbox"></label>
    <span id="status"></span>
  </div>
  <div id="morefx">
    <label class="morefx-label">note gradient: <input id="nutGradient" type="checkbox"></label>
    <label class="morefx-label">Blur FX particle: <input id="particleBlur" type="checkbox"></label>
    <label class="morefx-label">piano shadow: <input id="shadowDepth" type="range" min="0" max="48" value="20" style="width:70px;"></label>
    <label class="morefx-label" style="display:none;">gold lettering: <input id="showTitle" type="checkbox" checked></label>
    <label class="morefx-label">Saber line style: 
      <select id="saberStyle">
        <option value="neon">Neon</option>
        <option value="gold">Gold</option>
        <option value="dynamic">Pulse</option>
      </select>
    </label>
    <label class="morefx-label">demo: <input id="demoMode" type="checkbox"></label>
  </div>
  <div id="fx2">
    <label class="morefx-label">Rainbow mode: <input id="rainbowMode" type="checkbox"></label>
    <label class="morefx-label">Bounce particles: <input id="bounceParticles" type="checkbox"></label>
    <label class="morefx-label">Ghost Notes: <input id="ghostNotes" type="checkbox"></label>
    <label class="morefx-label">Gravity: <input id="gravity" type="number" min="0" max="0.35" step="0.01" value="0.07"></label>
    <label class="morefx-label">Invert background: <input id="invertBg" type="checkbox"></label>
    <label class="morefx-label"><button id="randomColors">Randomize Track Colors</button></label>
    <label class="morefx-label"><button id="snapshotBtn">Snapshot</button></label>
  </div>
  <div id="colors"></div>
  <label>midi file: <input type="file" id="midiFile"></label>
  <button id="start">Start</button>
  <button id="stop">Stop</button>
  <label class="extra-label">Lag Disabler: <input id="lagDisabler" type="checkbox" checked></label>
</div>
<canvas id="waterfall" width="3840" height="2160"></canvas>
<script>
let pianoHeight = 180;
let waterfallHeight = 1080;
let canvas = document.getElementById('waterfall');
let ctx = canvas.getContext('2d');

function setCanvasSize(){
  canvas.width = parseInt(document.getElementById('pianoWidth').value);
  canvas.height = waterfallHeight;
}
setCanvasSize();

const waterfallScale = 180;
let pianoY = canvas.height - pianoHeight;
let saberY = pianoY - 2;
const waterfallRadius = 6, whiteRadius = 5, blackRadius = 5;
const defaultTrackColors = [
  "#49aaff", "#ff6666", "#ffd95a", "#62ebad", "#b16aff", "#ff90c0"
];
let trackCount = 0, trackColors = [];
const firstMidi = 21, lastMidi = 108;
const keysTotal = 88, keyNames = ["A","A#","B","C","C#","D","D#","E","F","F#","G","G#"];
const keyMap = [];
let pitch = firstMidi, whiteIdx = 0;
for(let i=0;i<keysTotal;i++,pitch++){
  let noteIdx=(pitch-21)%12, octave=Math.floor((pitch-12)/12);
  let isBlack=keyNames[noteIdx].includes('#');
  keyMap.push({midi:pitch, note:keyNames[noteIdx], octave:octave,
    type: isBlack?"black":"white", wIdx:isBlack?whiteIdx-1:whiteIdx});
  if(!isBlack) whiteIdx++;
}
let whiteKeys = keyMap.filter(k=>k.type==="white");
let blackKeys = keyMap.filter(k=>k.type==="black");
let whiteW = canvas.width/whiteKeys.length, whiteH = pianoHeight;
let blackW = whiteW*0.58, blackH = whiteH*0.62;

function updatePianoSize(){
  setCanvasSize();
  pianoY = canvas.height - pianoHeight;
  saberY = pianoY - 2;
  whiteW = canvas.width/whiteKeys.length;
  whiteH = pianoHeight;
  blackW = whiteW*0.58;
  blackH = whiteH*0.62;
  drawWaterfall(0);
}
document.getElementById('pianoWidth').oninput = updatePianoSize;

let notes = [], midiLoaded = false, midiDuration = 0;
let playing = false, paused = false, startTime = 0;
let scheduled = [];
let particles = [];
let demoMode = false, rainbowMode = false, bounceParticles = false, ghostNotes = false, gravity = 0.07, invertBg = false;
let particleType = "classic"; let particleAmount = 80; let particleSize = 1.2; let particleSpeed = 0.8; let particleAlpha = 0.7;
let particleBlur = false; let pianoAlpha = 1; let nightMode = true; let tempoValue = 120; let autoplay = false;
let nutGradient = false; let shadowDepth = 20; let showTitle = false; let saberStyle = "neon";
let noteDelay = 1.0; let markKeysNotes = false;
let disableParticles = false;

let lagDisablerEnabled = true;
let maxParticles = 300;  // ZMNIEJSZONE DLA WIĘKSZEJ PŁYNNOŚCI

document.getElementById('particleType').onchange = e => particleType = e.target.value;
document.getElementById('particleAmount').oninput = e => particleAmount = Math.min(500, parseInt(e.target.value));
document.getElementById('particleSize').oninput = e => particleSize = parseFloat(e.target.value);
document.getElementById('particleSpeed').oninput = e => particleSpeed = parseFloat(e.target.value);
document.getElementById('particleAlpha').oninput = e => particleAlpha = parseFloat(e.target.value);
document.getElementById('disableParticles').onchange = e => disableParticles = !!e.target.checked;
document.getElementById('particleBlur').onchange = e => particleBlur = !!e.target.checked;
document.getElementById('pianoAlpha').oninput = e => pianoAlpha = parseFloat(e.target.value);
document.getElementById('nightMode').onchange = e => nightMode = !!e.target.checked;
document.getElementById('tempo').oninput = e => tempoValue = parseInt(e.target.value);
document.getElementById('resetParticles').onclick = () => particles = [];
document.getElementById('clearMidi').onclick = () => {notes = []; midiLoaded = false; particles = []; stopWaterfall();}
document.getElementById('autoplay').onchange = e => autoplay = !!e.target.checked;
document.getElementById('nutGradient').onchange = e => nutGradient = !!e.target.checked;
document.getElementById('shadowDepth').oninput = e => shadowDepth = parseInt(e.target.value);
document.getElementById('saberStyle').onchange = e => saberStyle = e.target.value;
document.getElementById('demoMode').onchange = e => demoMode = !!e.target.checked;
document.getElementById('rainbowMode').onchange = e => rainbowMode = !!e.target.checked;
document.getElementById('bounceParticles').onchange = e => bounceParticles = !!e.target.checked;
document.getElementById('ghostNotes').onchange = e => ghostNotes = !!e.target.checked;
document.getElementById('gravity').oninput = e => gravity = parseFloat(e.target.value);
document.getElementById('invertBg').onchange = e => invertBg = !!e.target.checked;
document.getElementById('pauseBtn').onclick = () => { paused = !paused; if(paused) Tone.Transport.pause(); else Tone.Transport.start(); };
document.getElementById('randomColors').onclick = () => {for(let i=0;i<trackColors.length;i++) trackColors[i]= "#"+Math.floor(Math.random()*16777215).toString(16).padStart(6,'0');};
document.getElementById('snapshotBtn').onclick = () => {
  let link = document.createElement('a');
  link.download = "golden_piano_waterfall.png";
  link.href = canvas.toDataURL();
  link.click();
};
document.getElementById('noteDelay').oninput = function(e){ noteDelay = parseFloat(e.target.value); };
document.getElementById('markKeysNotes').onchange = e => markKeysNotes = !!e.target.checked;
document.getElementById('lagDisabler').onchange = e => lagDisablerEnabled = e.target.checked;

let webmRecorder, webmChunks = [], webmIsRecording = false;
const recWebmBtn = document.getElementById('recWebmBtn');
const recWebmStatus = document.getElementById('recWebmStatus');
function startWebmRecording() {
  let stream = canvas.captureStream(60);
  webmChunks = [];
  webmRecorder = new MediaRecorder(stream, {mimeType:"video/webm; codecs=vp9"});
  webmRecorder.ondataavailable = (e)=>{ if(e.data.size) webmChunks.push(e.data); };
  webmRecorder.onstop = ()=>{
    const blob = new Blob(webmChunks, { type: "video/webm" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url; a.download = "golden_piano_waterfall.webm";
    document.body.appendChild(a); a.click();
    setTimeout(()=>{URL.revokeObjectURL(url); a.remove();},800);
    recWebmStatus.textContent = "recording is ready!";
    recWebmBtn.textContent = "▶️ record WebM";
    webmIsRecording = false;
  };
  webmRecorder.start();
  webmIsRecording = true;
  recWebmStatus.textContent = "video recording...";
  recWebmBtn.textContent = "⏹️ Stop and download";
}
recWebmBtn.onclick = ()=>{
  if(webmIsRecording){
    webmRecorder.stop();
    recWebmStatus.textContent = "Finalizing...";
    recWebmBtn.disabled = true;
    setTimeout(()=>{recWebmBtn.disabled = false;}, 1400);
  }else{
    startWebmRecording();
  }
};

function renderColorPickers(tracks){
  const colorsDiv = document.getElementById('colors');
  colorsDiv.innerHTML = '';
  trackColors.length = tracks;
  for(let i=0;i<tracks;i++){
    trackColors[i] = trackColors[i] || defaultTrackColors[i%defaultTrackColors.length];
    const label = document.createElement('label');
    label.className = "track-color-label";
    label.innerText = `Track ${i+1}: `;
    const input = document.createElement('input');
    input.type = "color";
    input.value = trackColors[i];
    input.dataset.track = i;
    input.oninput = function() {
      trackColors[parseInt(this.dataset.track)] = this.value;
    };
    label.appendChild(input);
    colorsDiv.appendChild(label);
  }
}

function loadMidiFile(arrayBuffer){
  const midi = new Midi(arrayBuffer);
  notes = [];
  midiDuration = midi.duration;
  trackCount = midi.tracks.length;
  renderColorPickers(trackCount);
  midi.tracks.forEach((track, trackIdx) => {
    track.notes.forEach(n => {
      if(n.midi<firstMidi || n.midi>lastMidi) return;
      notes.push({midi: n.midi, t0: n.time, dur: n.duration,
        velocity: Math.max(0.18, n.velocity), track: trackIdx});
    });
  });
  notes.sort((a,b)=>a.t0-b.t0);
  midiLoaded = true;
  if(autoplay) playWaterfall();
}
document.getElementById('midiFile').addEventListener("change",function(e){
  const file = e.target.files[0];
  if(!file) return;
  const reader = new FileReader();
  reader.onload = (ev)=>{ loadMidiFile(ev.target.result); };
  reader.readAsArrayBuffer(file);
});

function fillRoundRect(ctx, x, y, w, h, r, fillStyle, gradient=null) {
  ctx.save();
  ctx.beginPath();
  ctx.moveTo(x + r, y);
  ctx.lineTo(x + w - r, y);
  ctx.arcTo(x + w, y, x + w, y + r, r);
  ctx.lineTo(x + w, y + h - r);
  ctx.arcTo(x + w, y + h, x + w - r, y + h, r);
  ctx.lineTo(x + r, y + h);
  ctx.arcTo(x, y + h, x, y + h - r, r);
  ctx.lineTo(x, y + r);
  ctx.arcTo(x, y, x + r, y, r);
  ctx.closePath();
  ctx.fillStyle = gradient ? gradient : fillStyle;
  ctx.fill();
  ctx.restore();
}

function rainbowColor(t){
  let h = (t*7)%360; return "hsl("+h+",85%,56%)";
}

function drawParticleClassic(ctx, p) {
  ctx.save();
  ctx.globalAlpha = p.alpha;
  ctx.shadowColor = p.color;
  ctx.shadowBlur = particleBlur ? 8 : 4;
  ctx.translate(p.x, p.y);
  ctx.beginPath();
  ctx.arc(0, 0, p.radius, 0, 2 * Math.PI);
  ctx.fillStyle = rainbowMode ? rainbowColor(p.x + p.y + p.life) : p.color;
  ctx.fill();
  ctx.restore();
}

function drawParticleStar(ctx, p) {
  ctx.save();
  ctx.globalAlpha = p.alpha;
  ctx.shadowBlur = particleBlur ? 12 : 6;
  ctx.shadowColor = p.color;
  ctx.fillStyle = rainbowMode ? rainbowColor(p.x + p.y + p.life*2) : p.color;
  ctx.translate(p.x, p.y);
  for(let i=0; i<5; i++){
    ctx.rotate(Math.PI/2.5);
    ctx.beginPath();
    ctx.moveTo(0, 0);
    ctx.lineTo(0, -p.radius);
    ctx.lineTo(p.radius/3, -p.radius/2);
    ctx.fill();
  }
  ctx.restore();
}

function drawParticleRect(ctx, p) {
  ctx.save();
  ctx.globalAlpha = p.alpha;
  ctx.shadowBlur = particleBlur ? 12 : 6;
  ctx.shadowColor = p.color;
  ctx.fillStyle = rainbowMode ? rainbowColor(p.x + p.life*3) : p.color;
  ctx.fillRect(p.x - p.radius/2, p.y - p.radius/2, p.radius, p.radius/2);
  ctx.restore();
}

function drawParticleRing(ctx, p) {
  ctx.save();
  ctx.globalAlpha = p.alpha;
  ctx.shadowBlur = particleBlur ? 10 : 5;
  ctx.shadowColor = p.color;
  ctx.strokeStyle = rainbowMode ? rainbowColor(p.y + p.life*5) : p.color;
  ctx.lineWidth = p.radius/3;
  ctx.beginPath();
  ctx.arc(p.x, p.y, p.radius, 0, 2*Math.PI);
  ctx.stroke();
  ctx.restore();
}

// SNOWFLAKE Z LATANIEM (wzorzec śniegu)
function drawParticleSnowflake(ctx, p) {
  ctx.save();
  ctx.globalAlpha = p.alpha * 0.85;
  ctx.shadowColor = particleBlur ? p.color + "88" : p.color + "cc";
  ctx.shadowBlur = particleBlur ? 12 : 6;
  ctx.strokeStyle = rainbowMode ? rainbowColor(p.x + p.y + p.life * 1.5) : p.color;
  ctx.lineWidth = 1;
  ctx.lineCap = "round";
  ctx.lineJoin = "round";
  ctx.translate(p.x, p.y);
  ctx.rotate(p.life * 0.015 + Math.sin(p.life * 0.08) * 0.2);
  
  // Główny sześciokąt
  ctx.beginPath();
  for(let i = 0; i < 6; i++) {
    let angle = (i * Math.PI) / 3;
    let r = p.radius;
    let x = Math.cos(angle) * r;
    let y = Math.sin(angle) * r;
    if(i === 0) ctx.moveTo(x, y);
    else ctx.lineTo(x, y);
  }
  ctx.closePath();
  ctx.stroke();
  
  // Gałązki (uproszczone dla wydajności)
  for(let i = 0; i < 6; i++) {
    let angle = (i * Math.PI) / 3;
    let innerR = p.radius * 0.35;
    let outerR = p.radius * 1.3;
    ctx.beginPath();
    ctx.moveTo(Math.cos(angle) * innerR, Math.sin(angle) * innerR);
    ctx.lineTo(Math.cos(angle) * outerR, Math.sin(angle) * outerR);
    ctx.stroke();
  }
  
  ctx.shadowBlur = 0;
  ctx.restore();
}

function spawnParticles(x, y, color) {
  if(disableParticles) return;
  if(lagDisablerEnabled && particles.length > maxParticles) return;
  
  // Specjalny wzorzec ruchu dla snowflake
  for (let i = 0; i < particleAmount; i++) {
    let base = {
      x: x + (Math.random()-0.5)*4,
      y: y + Math.random()*4,
      vx: (Math.random()-0.5)*particleSpeed * (particleType === 'snowflake' ? 0.4 : 1),
      vy: -Math.random()*particleSpeed*0.8-0.05,
      alpha: particleAlpha + Math.random()*0.1,
      color: color,
      radius: particleSize + Math.random()*1.5,
      life: Math.random()*25 + 40,
      type: particleType,
      sway: particleType === 'snowflake' ? Math.random()*0.03 : 0  // delikatne falowanie dla śniegu
    };
    particles.push(base);
  }
}

function shadeColor(color, percent) {
  let R = parseInt(color.slice(1,3),16);
  let G = parseInt(color.slice(3,5),16);
  let B = parseInt(color.slice(5,7),16);
  R = Math.min(255,Math.max(0,R + percent));
  G = Math.min(255,Math.max(0,G + percent));
  B = Math.min(255,Math.max(0,B + percent));
  return '#' + ((1<<24)|(R<<16)|(G<<8)|B).toString(16).slice(1);
}

function gradientGold(ctx, x, y, w, h, base){
  let grad = ctx.createLinearGradient(x, y, 0, y + h);
  grad.addColorStop(0, base);
  grad.addColorStop(0.45, "#fff7a1");
  grad.addColorStop(0.6, "#ffd700ee");
  grad.addColorStop(1, "#b8860b99");
  return grad;
}

function drawWaterfall(curT){
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  
  if(invertBg){
    ctx.fillStyle = "#fcfcfc";
    ctx.fillRect(0, 0, canvas.width, canvas.height);
  }
  if(nightMode && !invertBg){
    ctx.fillStyle = "#121323";
    ctx.fillRect(0, 0, canvas.width, canvas.height);
  }
  
  let _activeNotes = 0;
  for(let n of notes){
    let k = keyMap[n.midi - 21];
    let x = k.wIdx * whiteW, w = whiteW - 2;
    let isBlackKey = k.type === "black";
    if(isBlackKey) { x += whiteW - blackW / 2; w = blackW; }
    let h = n.dur * waterfallScale;
    let y = pianoY - h - (n.t0 + noteDelay - curT) * waterfallScale;
    if(y + h < 0 || y > pianoY - 12) continue;
    
    let trCol = trackColors[n.track % trackColors.length] || defaultTrackColors[n.track % defaultTrackColors.length];
    let color = isBlackKey
      ? shadeColor(nightMode ? trCol : trCol, nightMode ? -42 : -22)
      : nightMode ? shadeColor(trCol, 15) : trCol;
    if(rainbowMode) color = rainbowColor(n.midi * 12 + curT * 20 + n.track * 48);
    
    let ghost = ghostNotes && n.velocity < 0.25;
    ctx.globalAlpha = ghost ? 0.14 : n.velocity;
    if(nutGradient && !ghost){
      fillRoundRect(ctx, x, y, w, h, Math.min(w, waterfallRadius), color, gradientGold(ctx, x, y, w, h, color));
    } else if(ghost){
      fillRoundRect(ctx, x, y, w, h, Math.min(w, waterfallRadius), "#bbb");
    } else{
      fillRoundRect(ctx, x, y, w, h, Math.min(w, waterfallRadius), color);
    }
    
    if (markKeysNotes && ['C','D','E','F','G','A','B'].includes(k.note.replace('#', ''))) {
      ctx.save();
      ctx.font = "bold 14px Arial";
      ctx.fillStyle = "#643b00";
      ctx.globalAlpha = ghost ? 0.21 : n.velocity * 0.8;
      ctx.textAlign = "center";
      ctx.textBaseline = "middle";
      ctx.fillText(k.note.replace('#', ''), x + w/2, y + h/2);
      ctx.restore();
    }
    
    if(curT >= n.t0 + noteDelay && curT <= (n.t0 + n.dur + noteDelay)) _activeNotes++;
    ctx.globalAlpha = 1;
  }
  
  // Saber line (zoptymalizowany)
  ctx.save();
  ctx.lineCap = "round";
  const pulse = (Math.sin(performance.now() / 250) + 1) / 2;
  ctx.shadowColor = saberStyle === "gold" ? "#ffd600" : "#35ffe0";
  ctx.shadowBlur = 10 + 8 * pulse;
  ctx.strokeStyle = saberStyle === "gold" ? "#ffd700" : "#17ffff";
  ctx.lineWidth = 8 + 4 * pulse;
  ctx.globalAlpha = 0.7 + 0.2 * pulse;
  ctx.beginPath();
  ctx.moveTo(0, saberY);
  ctx.lineTo(canvas.width, saberY);
  ctx.stroke();
  ctx.restore();

  if(shadowDepth > 0){
    ctx.save();
    ctx.globalAlpha = 0.2;
    ctx.fillStyle = "#000";
    ctx.shadowBlur = shadowDepth;
    ctx.shadowColor = "#000";
    ctx.fillRect(0, pianoY + whiteH - 1, canvas.width, 20);
    ctx.restore();
  }

  ctx.globalAlpha = pianoAlpha;
  for(let i = 0; i < whiteKeys.length; i++){
    const k = whiteKeys[i], x = i * whiteW;
    ctx.save();
    ctx.fillStyle = nightMode ? '#e8e4dc' : '#fffefc';
    ctx.fillRect(x + 1, pianoY, whiteW - 2, whiteH);
    ctx.strokeStyle = "#444";
    ctx.lineWidth = 1;
    ctx.strokeRect(x + 1, pianoY, whiteW - 2, whiteH);
    ctx.restore();
    
    if (markKeysNotes && ['C','D','E','F','G','A','B'].includes(k.note.replace('#',''))) {
      ctx.save();
      ctx.font = "bold 18px Arial";
      ctx.fillStyle = "#3e50ad";
      ctx.globalAlpha = 0.6 * pianoAlpha;
      ctx.textAlign = "center";
      ctx.textBaseline = "top";
      ctx.fillText(k.note.replace('#',''), x + whiteW/2, pianoY + 3);
      ctx.restore();
    }
  }

  for(let k of blackKeys){
    const x = k.wIdx * whiteW + whiteW - blackW / 2;
    ctx.save();
    ctx.fillStyle = nightMode ? '#2a2d3a' : '#44475a';
    ctx.fillRect(x + 1, pianoY, blackW - 2, blackH);
    ctx.strokeStyle = "#555";
    ctx.lineWidth = 1;
    ctx.strokeRect(x + 1, pianoY, blackW - 2, blackH);
    ctx.restore();
  }
  ctx.globalAlpha = 1;

  for(let n of notes){
    if(curT >= n.t0 + noteDelay && curT <= (n.t0 + n.dur + noteDelay)){
      const k = keyMap[n.midi - 21];
      let xshow, wshow, isBlackKey = k.type === "black", trCol;
      trCol = trackColors[n.track % trackColors.length] || defaultTrackColors[n.track % defaultTrackColors.length];
      let noteCol = isBlackKey ? shadeColor(trCol, nightMode ? -42 : -22) : shadeColor(trCol, nightMode ? 15 : 0);
      if(rainbowMode) noteCol = rainbowColor(n.midi * 12 + curT * 20 + n.track * 48);
      
      if(!isBlackKey){
        xshow = k.wIdx * whiteW;
        wshow = whiteW;
        ctx.save();
        ctx.globalAlpha = 0.4 * pianoAlpha;
        ctx.fillStyle = noteCol;
        ctx.fillRect(xshow + 1, pianoY, wshow - 2, whiteH);
        ctx.restore();
      } else {
        xshow = k.wIdx * whiteW + whiteW - blackW / 2;
        wshow = blackW;
        ctx.save();
        ctx.globalAlpha = 0.6 * pianoAlpha;
        ctx.fillStyle = noteCol;
        ctx.fillRect(xshow + 1, pianoY, wshow - 2, blackH);
        ctx.restore();
      }
      
      let partX = !isBlackKey ? k.wIdx * whiteW + whiteW / 2 : xshow + blackW / 2;
      let partY = pianoY - 4;
      if(Math.abs(curT - (n.t0 + noteDelay)) < 0.04){
        spawnParticles(partX, partY, noteCol);
      }
    }
  }

  if(!disableParticles){
    for(let i = particles.length - 1; i >= 0; i--){
      let p = particles[i];
      
      // Rysowanie particle
      if(p.type === 'classic') drawParticleClassic(ctx, p);
      else if(p.type === 'star') drawParticleStar(ctx, p);
      else if(p.type === 'rect') drawParticleRect(ctx, p);
      else if(p.type === 'ring') drawParticleRing(ctx, p);
      else if(p.type === 'snowflake') drawParticleSnowflake(ctx, p);
      
      // Aktualizacja ruchu SNOWFLAKE (lata + faluje)
      if(p.type === 'snowflake'){
        p.x += p.vx + Math.sin(p.life * 0.1 + p.sway) * 0.8;
        p.y += p.vy + Math.sin(p.life * 0.07) * 0.3;
      } else {
        p.x += p.vx;
        p.y += p.vy;
      }
      
      if(bounceParticles && p.y > pianoY - 10){
        p.vy *= -0.3;
        p.y = pianoY - 10;
      }
      p.vy += gravity * 0.8;
      p.alpha -= 0.01;
      p.radius *= 0.99;
      p.life--;
      
      if (p.alpha <= 0.01 || p.radius <= 0.2 || p.life < 0 || p.y > canvas.height) {
        particles.splice(i, 1);
      }
    }
  }

  ctx.save();
  ctx.globalAlpha = 0.9;
  ctx.font = "bold 16px Arial";
  ctx.fillStyle = "#ffd95a";
  ctx.textAlign = "right";
  let end = playing ? Math.max(0, (midiDuration + noteDelay - (curT || 0))).toFixed(1) : "0.0";
  let left = playing ? notes.filter(n => n.t0 + noteDelay > curT).length : 0;
  ctx.fillText(`Notes: ${left} | Time: ${end}s`, canvas.width - 20, 30);
  ctx.restore();

  document.getElementById('status').textContent = playing ? `Notes: ${left}, Time: ${end}s` : '';
}

async function playWaterfall(){
  if(!midiLoaded || notes.length === 0) return;
  playing = true;
  paused = false;
  startTime = performance.now() / 1000;
  scheduled = [];
  const synth = new Tone.PolySynth(Tone.Synth).toDestination();
  Tone.Transport.bpm.value = tempoValue;
  Tone.Transport.seconds = 0;
  for(const n of notes){
    scheduled.push(Tone.Transport.schedule((time) => {
      synth.triggerAttackRelease(Tone.Frequency(n.midi, "midi"), n.dur, time, n.velocity);
    }, n.t0 + noteDelay));
  }
  setTimeout(() => { Tone.Transport.start(); }, noteDelay * 1000);
}

function stopWaterfall(){
  playing = false;
  Tone.Transport.stop();
  Tone.Transport.cancel();
  scheduled = [];
}

document.getElementById('start').onclick = () => { playWaterfall(); };
document.getElementById('stop').onclick = () => { stopWaterfall(); };

function demoTick(t){
  if(disableParticles) return;
  for(let d = 0; d < 4; d++){
    let x = 100 + d * whiteW * 8;
    let y = Math.abs(Math.sin(t / 120 + d * 2)) * canvas.height * 0.5 + 50;
    let w = whiteW * 0.6;
    let col = rainbowMode ? rainbowColor(t + d * 50) : ["#ffd95a", "#ff3cc0", "#30d1ff", "#7aff72"][d % 4];
    fillRoundRect(ctx, x, y, w, 25, 6, col);
    if(t % 50 < 2) spawnParticles(x + w / 2, y + 8, col);
  }
}

function animate(){
  requestAnimationFrame(animate);
  let curT = 0;
  if(demoMode && !midiLoaded) {
    drawWaterfall(0);
    demoTick(Math.floor(performance.now() / 10));
  } else if(playing && !paused){
    curT = (performance.now() / 1000) - startTime;
    drawWaterfall(curT);
    if(curT > midiDuration + noteDelay + 1) stopWaterfall();
  } else {
    drawWaterfall(0);
  }
}
animate();
</script>
</body>
</html>

