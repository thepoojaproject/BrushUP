<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ZEST - Retro Paint</title>

<style>
@import url('https://fonts.googleapis.com/css2?family=VT323&display=swap');

* { box-sizing: border-box; margin: 0; padding: 0; }

body {
    background: #008080;
    font-family: 'MS Sans Serif','Segoe UI',Tahoma,sans-serif;
    font-size: 12px;
    height: 100vh;
    overflow: hidden;
    display: flex;
    flex-direction: column;
}

.window {
    flex: 1;
    margin: 8px;
    border: 2px solid #dfdfdf;
    border-top-color: #000;
    border-left-color: #000;
    background: #c0c0c0;
    display: flex;
    flex-direction: column;
}

.title-bar {
    background: linear-gradient(to right,#000080,#1084d0);
    color: #fff;
    padding: 4px 8px;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.title-bar h1 { font-size: 13px; font-weight: normal; }

.buttons { display: flex; gap: 4px; }
.btn-window {
    width: 18px; height: 18px;
    background: #c0c0c0;
    border: 1px solid #fff;
    border-right-color: #000;
    border-bottom-color: #000;
    text-align: center;
    font-size: 10px;
    line-height: 16px;
}

.toolbar {
    background: #c0c0c0;
    padding: 6px;
    display: flex;
    gap: 6px;
    flex-wrap: wrap;
    border-bottom: 2px solid #000;
    border-top: 2px solid #fff;
}

.tool-group {
    display: flex;
    flex-direction: column;
    gap: 4px;
    padding: 4px;
    border: 1px solid #000;
    border-top-color: #fff;
    border-left-color: #fff;
}

.tool-btn, button {
    width: 48px;
    height: 48px;
    background: #c0c0c0;
    border: 2px solid;
    border-color: #fff #000 #000 #fff;
    cursor: pointer;
    font-size: 20px;
}

.tool-btn.active, button:active {
    border-color: #000 #fff #fff #000;
    background: #808080;
    color: #fff;
}

#canvas-container {
    flex: 1;
    position: relative;
    background:
        linear-gradient(45deg,#808080 25%,transparent 25%) 0 0/16px 16px,
        linear-gradient(45deg,transparent 75%,#808080 75%) 8px 8px/16px 16px,
        #c0c0c0;
    border: 2px inset #fff;
    border-top-color: #000;
    border-left-color: #000;
}

canvas {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    background: #fff;
    image-rendering: pixelated;
    cursor: crosshair;
}

.color-palette {
    padding: 6px;
    display: flex;
    flex-wrap: wrap;
    gap: 4px;
    justify-content: center;
    border-top: 2px solid #000;
    background: #c0c0c0;
}

.color-swatch {
    width: 24px;
    height: 24px;
    border: 2px solid #000;
    cursor: pointer;
}

.status-bar {
    padding: 4px 8px;
    background: #c0c0c0;
    border-top: 2px solid #fff;
    font-size: 11px;
}

footer {
    text-align: center;
    padding: 6px;
    background: #000080;
    color: #fff;
    font-size: 11px;
}
</style>
</head>

<body>

<div class="window">
    <div class="title-bar">
        <h1>Untitled - ZEST Paint</h1>
        <div class="buttons">
            <div class="btn-window">_</div>
            <div class="btn-window">□</div>
            <div class="btn-window">×</div>
        </div>
    </div>

    <div class="toolbar">
        <div class="tool-group">
            <button id="penBtn" class="tool-btn active">✏️</button>
            <button id="eraserBtn" class="tool-btn">🧽</button>
        </div>
        <div class="tool-group">
            <button id="undoBtn">↺</button>
            <button id="redoBtn">↻</button>
        </div>
        <div class="tool-group">
            <button id="uploadBtn">📁</button>
            <button id="clearBtn">🗑️</button>
            <button id="saveBtn">💾</button>
        </div>
        Size <input type="range" id="brushSize" min="1" max="40" value="4">
    </div>

    <input type="file" id="imageUpload" accept="image/*" hidden>

    <div id="canvas-container">
        <canvas id="canvas"></canvas>
    </div>

    <div class="color-palette">
        <div class="color-swatch" data-color="#000000" style="background:#000"></div>
        <div class="color-swatch" data-color="#808080" style="background:#808080"></div>
        <div class="color-swatch" data-color="#C0C0C0" style="background:#C0C0C0"></div>
        <div class="color-swatch" data-color="#FFFFFF" style="background:#FFF"></div>
        <div class="color-swatch" data-color="#FF0000" style="background:#F00"></div>
        <div class="color-swatch" data-color="#00FF00" style="background:#0F0"></div>
        <div class="color-swatch" data-color="#0000FF" style="background:#00F"></div>
        <div class="color-swatch" data-color="#FFFF00" style="background:#FF0"></div>
        <div class="color-swatch" data-color="#FF00FF" style="background:#F0F"></div>
        <div class="color-swatch" data-color="#00FFFF" style="background:#0FF"></div>
    </div>

    <div class="status-bar">For Pooja • Retro ZEST Paint</div>
</div>

<footer>Made with ❤️ in 90s style</footer>

<script>
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

let painting = false;
let isEraser = false;
let currentColor = "#000000";
let history = [];
let historyIndex = -1;
const MAX_HISTORY = 25;

function resizeCanvas() {
    const ratio = window.devicePixelRatio || 1;
    const rect = canvas.getBoundingClientRect();

    const temp = document.createElement("canvas");
    temp.width = canvas.width;
    temp.height = canvas.height;
    temp.getContext("2d").drawImage(canvas,0,0);

    canvas.width = rect.width * ratio;
    canvas.height = rect.height * ratio;
    ctx.scale(ratio, ratio);
    ctx.imageSmoothingEnabled = false;
    ctx.drawImage(temp,0,0);
}

window.addEventListener("resize", resizeCanvas);
resizeCanvas();

function saveState() {
    history = history.slice(0, historyIndex + 1);
    history.push(canvas.toDataURL());
    if (history.length > MAX_HISTORY) history.shift();
    historyIndex = history.length - 1;
}

function getPos(e) {
    const r = canvas.getBoundingClientRect();
    const x = (e.clientX || e.touches[0].clientX) - r.left;
    const y = (e.clientY || e.touches[0].clientY) - r.top;
    return { x, y };
}

function startDraw(e) {
    e.preventDefault();
    painting = true;
    const {x,y} = getPos(e);
    ctx.beginPath();
    ctx.moveTo(x,y);
}

function draw(e) {
    if (!painting) return;
    const {x,y} = getPos(e);

    ctx.lineWidth = brushSize.value;
    ctx.lineCap = "round";
    ctx.globalCompositeOperation = isEraser ? "destination-out" : "source-over";
    ctx.strokeStyle = currentColor;

    ctx.lineTo(x,y);
    ctx.stroke();
}

function endDraw() {
    if (!painting) return;
    painting = false;
    ctx.beginPath();
    saveState();
}

canvas.addEventListener("mousedown", startDraw);
canvas.addEventListener("mousemove", draw);
canvas.addEventListener("mouseup", endDraw);
canvas.addEventListener("mouseout", endDraw);
canvas.addEventListener("touchstart", startDraw);
canvas.addEventListener("touchmove", draw);
canvas.addEventListener("touchend", endDraw);

penBtn.onclick = () => { isEraser=false; penBtn.classList.add("active"); eraserBtn.classList.remove("active"); }
eraserBtn.onclick = () => { isEraser=true; eraserBtn.classList.add("active"); penBtn.classList.remove("active"); }

document.querySelectorAll(".color-swatch").forEach(s =>
    s.onclick = () => { currentColor = s.dataset.color; penBtn.click(); }
);

undoBtn.onclick = () => {
    if (historyIndex > 0) load(--historyIndex);
};
redoBtn.onclick = () => {
    if (historyIndex < history.length - 1) load(++historyIndex);
};

function load(i) {
    const img = new Image();
    img.src = history[i];
    img.onload = () => ctx.drawImage(img,0,0);
}

uploadBtn.onclick = () => imageUpload.click();
imageUpload.onchange = e => {
    const img = new Image();
    img.onload = () => { ctx.drawImage(img,0,0,canvas.width,canvas.height); saveState(); };
    img.src = URL.createObjectURL(e.target.files[0]);
};

clearBtn.onclick = () => {
    if(confirm("Clear canvas?")) {
        ctx.clearRect(0,0,canvas.width,canvas.height);
        saveState();
    }
};

saveBtn.onclick = () => {
    const a = document.createElement("a");
    a.download = "zest-paint.png";
    a.href = canvas.toDataURL();
    a.click();
};

ctx.fillStyle="#fff";
ctx.fillRect(0,0,canvas.width,canvas.height);
saveState();
</script>

</body>
</html>
