<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ZEST - Retro Paint</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=VT323&display=swap');

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            background: #008080; /* Windows 95 teal desktop */
            font-family: 'MS Sans Serif', 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            font-size: 12px;
            height: 100vh;
            overflow: hidden;
            image-rendering: pixelated; /* crisp retro feel */
            display: flex;
            flex-direction: column;
        }

        .window {
            flex: 1;
            display: flex;
            flex-direction: column;
            margin: 8px;
            border: 2px solid #dfdfdf;
            border-top-color: #000;
            border-left-color: #000;
            background: #c0c0c0;
            box-shadow: 
                inset 1px 1px 0 #fff, 
                inset -1px -1px 0 #000;
        }

        .title-bar {
            background: linear-gradient(to right, #000080, #1084d0);
            color: white;
            padding: 3px 8px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            user-select: none;
        }

        .title-bar h1 {
            font-size: 13px;
            font-weight: normal;
        }

        .buttons {
            display: flex;
            gap: 4px;
        }

        .btn-window {
            width: 18px;
            height: 18px;
            background: #c0c0c0;
            border: 1px solid #fff;
            border-bottom-color: #000;
            border-right-color: #000;
            font-size: 10px;
            line-height: 16px;
            text-align: center;
            cursor: pointer;
        }

        .toolbar {
            background: #c0c0c0;
            border-bottom: 2px solid #000;
            border-top: 2px solid #fff;
            padding: 4px;
            display: flex;
            gap: 6px;
            flex-wrap: wrap;
        }

        .tool-group {
            display: flex;
            flex-direction: column;
            gap: 2px;
            border: 1px solid #000;
            border-top-color: #fff;
            border-left-color: #fff;
            background: #c0c0c0;
            padding: 4px;
        }

        button, .tool-btn {
            width: 48px;
            height: 48px;
            background: #c0c0c0;
            border: 2px solid;
            border-color: #fff #000 #000 #fff;
            font-size: 20px;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        button:active, .tool-btn.active {
            border-color: #000 #fff #fff #000;
            background: #808080;
            color: white;
        }

        #canvas-container {
            flex: 1;
            background: 
                linear-gradient(45deg, #808080 25%, transparent 25%) 0 0 / 16px 16px,
                linear-gradient(45deg, transparent 75%, #808080 75%) 8px 8px / 16px 16px,
                #c0c0c0; /* subtle dither */
            position: relative;
            border: 2px inset #fff;
            border-top-color: #000;
            border-left-color: #000;
        }

        #canvas {
            position: absolute;
            inset: 0;
            width: 100%;
            height: 100%;
            background: white;
            image-rendering: pixelated;
            cursor: crosshair;
            touch-action: none;
        }

        .color-palette {
            background: #c0c0c0;
            border-top: 2px solid #000;
            padding: 6px;
            display: flex;
            flex-wrap: wrap;
            gap: 4px;
            justify-content: center;
        }

        .color-swatch {
            width: 24px;
            height: 24px;
            border: 2px solid #000;
            cursor: pointer;
        }

        .status-bar {
            background: #c0c0c0;
            border-top: 2px solid #fff;
            padding: 4px 8px;
            font-size: 11px;
            color: #000;
        }

        footer {
            text-align: center;
            padding: 6px;
            background: #000080;
            color: white;
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
                <button id="penBtn" class="tool-btn active" title="Pencil">✏️</button>
                <button id="eraserBtn" class="tool-btn" title="Eraser">🧽</button>
            </div>
            <div class="tool-group">
                <button id="undoBtn" title="Undo">↺</button>
                <button id="redoBtn" title="Redo">↻</button>
            </div>
            <div class="tool-group">
                <button id="clearBtn" title="Clear">🗑️</button>
                <button id="saveBtn" title="Save">💾</button>
            </div>
            <label title="Brush Size">
                Size: <input type="range" id="brushSize" min="1" max="40" value="4" style="width:120px;">
            </label>
        </div>

        <div id="canvas-container">
            <canvas id="canvas"></canvas>
        </div>

        <div class="color-palette">
            <!-- Classic Windows 95 / MS Paint 20-color palette -->
            <div class="color-swatch" style="background:#000000;" data-color="#000000"></div>
            <div class="color-swatch" style="background:#808080;" data-color="#808080"></div>
            <div class="color-swatch" style="background:#C0C0C0;" data-color="#C0C0C0"></div>
            <div class="color-swatch" style="background:#FFFFFF;" data-color="#FFFFFF"></div>
            <div class="color-swatch" style="background:#800000;" data-color="#800000"></div>
            <div class="color-swatch" style="background:#FF0000;" data-color="#FF0000"></div>
            <div class="color-swatch" style="background:#808000;" data-color="#808000"></div>
            <div class="color-swatch" style="background:#FFFF00;" data-color="#FFFF00"></div>
            <div class="color-swatch" style="background:#008000;" data-color="#008000"></div>
            <div class="color-swatch" style="background:#00FF00;" data-color="#00FF00"></div>
            <div class="color-swatch" style="background:#008080;" data-color="#008080"></div>
            <div class="color-swatch" style="background:#00FFFF;" data-color="#00FFFF"></div>
            <div class="color-swatch" style="background:#000080;" data-color="#000080"></div>
            <div class="color-swatch" style="background:#0000FF;" data-color="#0000FF"></div>
            <div class="color-swatch" style="background:#800080;" data-color="#800080"></div>
            <div class="color-swatch" style="background:#FF00FF;" data-color="#FF00FF"></div>
            <div class="color-swatch" style="background:#A0522D;" data-color="#A0522D"></div>
            <div class="color-swatch" style="background:#D2691E;" data-color="#D2691E"></div>
            <div class="color-swatch" style="background:#F4A460;" data-color="#F4A460"></div>
            <div class="color-swatch" style="background:#FFD700;" data-color="#FFD700"></div>
        </div>

        <div class="status-bar">
            For Pooja • Retro ZEST Paint • Use mouse or touch to draw
        </div>
    </div>

    <footer>Made with ❤️ in 90s style for Pooja</footer>

    <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        const brushSize = document.getElementById('brushSize');
        const penBtn = document.getElementById('penBtn');
        const eraserBtn = document.getElementById('eraserBtn');
        const undoBtn = document.getElementById('undoBtn');
        const redoBtn = document.getElementById('redoBtn');
        const clearBtn = document.getElementById('clearBtn');
        const saveBtn = document.getElementById('saveBtn');

        let painting = false;
        let isEraser = false;
        let history = [];
        let historyIndex = -1;
        const MAX_HISTORY = 25;

        function resizeCanvas() {
            const current = canvas.toDataURL();
            canvas.width = canvas.offsetWidth;
            canvas.height = canvas.offsetHeight;

            if (current && history.length > 0) {
                const img = new Image();
                img.src = current;
                img.onload = () => ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
            } else {
                ctx.fillStyle = 'white';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
            }
        }

        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();

        function saveState() {
            if (historyIndex < history.length - 1) {
                history = history.slice(0, historyIndex + 1);
            }
            history.push(canvas.toDataURL());
            if (history.length > MAX_HISTORY) history.shift();
            historyIndex = history.length - 1;
        }

        function draw(e) {
            if (!painting) return;
            const rect = canvas.getBoundingClientRect();
            const x = (e.clientX || (e.touches && e.touches[0].clientX)) - rect.left;
            const y = (e.clientY || (e.touches && e.touches[0].clientY)) - rect.top;

            ctx.lineWidth = brushSize.value;
            ctx.lineCap = 'round';
            ctx.lineJoin = 'round';

            if (isEraser) {
                ctx.globalCompositeOperation = 'destination-out';
            } else {
                ctx.globalCompositeOperation = 'source-over';
            }

            ctx.lineTo(x, y);
            ctx.stroke();
            ctx.beginPath();
            ctx.moveTo(x, y);
        }

        function startDrawing(e) {
            e.preventDefault();
            painting = true;
            const rect = canvas.getBoundingClientRect();
            const x = (e.clientX || (e.touches && e.touches[0].clientX)) - rect.left;
            const y = (e.clientY || (e.touches && e.touches[0].clientY)) - rect.top;

            ctx.beginPath();
            ctx.moveTo(x, y);

            if (!isEraser) ctx.strokeStyle = e.target.dataset?.color || '#000000';
            draw(e);
        }

        function endDrawing() {
            if (painting) {
                painting = false;
                ctx.beginPath();
                saveState();
            }
        }

        canvas.addEventListener('mousedown', startDrawing);
        canvas.addEventListener('mousemove', draw);
        canvas.addEventListener('mouseup', endDrawing);
        canvas.addEventListener('mouseout', endDrawing);
        canvas.addEventListener('touchstart', startDrawing);
        canvas.addEventListener('touchmove', draw);
        canvas.addEventListener('touchend', endDrawing);
        canvas.addEventListener('touchcancel', endDrawing);

        penBtn.addEventListener('click', () => {
            isEraser = false;
            penBtn.classList.add('active');
            eraserBtn.classList.remove('active');
        });

        eraserBtn.addEventListener('click', () => {
            isEraser = true;
            eraserBtn.classList.add('active');
            penBtn.classList.remove('active');
        });

        document.querySelectorAll('.color-swatch').forEach(swatch => {
            swatch.addEventListener('click', (e) => {
                ctx.strokeStyle = e.target.dataset.color;
                penBtn.click(); // switch to pen when picking color
            });
        });

        undoBtn.addEventListener('click', () => {
            if (historyIndex > 0) {
                historyIndex--;
                const img = new Image();
                img.src = history[historyIndex];
                img.onload = () => {
                    ctx.clearRect(0, 0, canvas.width, canvas.height);
                    ctx.drawImage(img, 0, 0);
                };
            }
        });

        redoBtn.addEventListener('click', () => {
            if (historyIndex < history.length - 1) {
                historyIndex++;
                const img = new Image();
                img.src = history[historyIndex];
                img.onload = () => {
                    ctx.clearRect(0, 0, canvas.width, canvas.height);
                    ctx.drawImage(img, 0, 0);
                };
            }
        });

        clearBtn.addEventListener('click', () => {
            if (confirm("Clear canvas?")) {
                ctx.fillStyle = 'white';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                history = [];
                historyIndex = -1;
                saveState();
            }
        });

        saveBtn.addEventListener('click', () => {
            const link = document.createElement('a');
            link.download = `zest-retro-paint-${new Date().toISOString().slice(0,10)}.png`;
            link.href = canvas.toDataURL();
            link.click();
        });

        // Start with blank canvas & save initial state
        ctx.fillStyle = 'white';
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.strokeStyle = '#000000'; // default black
        saveState();
    </script>
</body>
</html>
