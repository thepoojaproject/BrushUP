<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ZEST - Draw & Create</title>
    <style>
        body {
            margin: 0;
            background: #29825d;
            font-family: Arial, sans-serif;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }
        header {
            text-align: center;
            color: white;
            padding: 15px 0 5px;
            background: rgba(0,0,0,0.15);
        }
        #logo {
            max-width: 320px;
            width: 80%;
            height: auto;
            display: block;
            margin: 0 auto 8px;
            filter: drop-shadow(0 4px 12px rgba(0,0,0,0.4));
        }
        #toolbar {
            background: #ffffff;
            padding: 12px;
            text-align: center;
            box-shadow: 0 4px 12px rgba(0,0,0,0.3);
            z-index: 10;
            overflow-x: auto;
            white-space: nowrap;
        }
        #toolbar label {
            margin: 0 14px;
            font-weight: bold;
            color: #29825d;
        }
        #canvas-container {
            flex: 1;
            position: relative;
            background: #f0f0f0;
            overflow: hidden;
        }
        #canvas {
            position: absolute;
            top: 0; left: 0;
            width: 100%;
            height: 100%;
            background: white;
            box-shadow: 0 6px 20px rgba(0,0,0,0.35);
            cursor: crosshair;
            touch-action: none;
        }
        #canvas.eraser-cursor {
            cursor: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 32 32"><circle cx="16" cy="16" r="14" fill="white" stroke="black" stroke-width="2"/><text x="16" y="22" font-size="18" text-anchor="middle" fill="black">🧽</text></svg>') 16 16, auto;
        }
        button {
            padding: 9px 16px;
            margin: 4px 6px;
            background: #29825d;
            color: white;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-weight: bold;
            transition: 0.2s;
            min-width: 80px;
        }
        button:hover {
            background: #1f6245;
        }
        button.active {
            background: #A8DF8E;
            color: #1a3c2e;
        }
        footer {
            text-align: center;
            padding: 12px;
            color: white;
            font-size: 0.95em;
            background: rgba(0,0,0,0.25);
        }
    </style>
</head>
<body>
    <header>
        <img id="logo" src="https://i.ibb.co/21Szz69V/Brush-Up.png" alt="Brush Up - ZEST Logo">
    </header>
    <div id="toolbar">
        <label>Color: <input type="color" id="colorPicker" value="#A8DF8E"></label>
        <label>Size: <input type="range" id="brushSize" min="1" max="60" value="6"></label>
        <button id="penBtn" class="active">✏️ Pen</button>
        <button id="eraserBtn">🧽 Eraser</button>
        <button id="undoBtn">↺ Undo</button>
        <button id="redoBtn">↻ Redo</button>
        <button id="clearBtn">🗑️ Clear</button>
        <button id="saveBtn">💾 Save</button>
    </div>
    <div id="canvas-container">
        <canvas id="canvas"></canvas>
    </div>
    <footer>
        Made with ❤️ for Pooja
    </footer>

    <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        const colorPicker = document.getElementById('colorPicker');
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
        let historyIndex = -1;  // -1 means at latest state
        const MAX_HISTORY = 30;

        function resizeCanvas() {
            // Save current drawing state
            const currentState = canvas.toDataURL();

            // Resize
            canvas.width = canvas.offsetWidth;
            canvas.height = canvas.offsetHeight;

            // Restore previous content (scaled to fit new size)
            if (history.length > 0 || currentState) {
                const img = new Image();
                img.src = currentState;
                img.onload = () => {
                    ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                };
            } else {
                // First time / cleared canvas
                ctx.fillStyle = 'white';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
            }
        }

        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();

        function saveState() {
            // Remove any future states (redo becomes invalid after new action)
            if (historyIndex < history.length - 1) {
                history = history.slice(0, historyIndex + 1);
            }

            history.push(canvas.toDataURL());
            if (history.length > MAX_HISTORY) {
                history.shift();
            }
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
                ctx.strokeStyle = '#000000'; // doesn't matter for erase
            } else {
                ctx.globalCompositeOperation = 'source-over';
                ctx.strokeStyle = colorPicker.value;
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
            draw(e);
        }

        function endDrawing() {
            if (painting) {
                painting = false;
                ctx.beginPath();
                saveState();
            }
        }

        // Mouse events
        canvas.addEventListener('mousedown', startDrawing);
        canvas.addEventListener('mousemove', draw);
        canvas.addEventListener('mouseup', endDrawing);
        canvas.addEventListener('mouseout', endDrawing);

        // Touch events
        canvas.addEventListener('touchstart', startDrawing);
        canvas.addEventListener('touchmove', draw);
        canvas.addEventListener('touchend', endDrawing);
        canvas.addEventListener('touchcancel', endDrawing);

        // Tool selection
        penBtn.addEventListener('click', () => {
            isEraser = false;
            penBtn.classList.add('active');
            eraserBtn.classList.remove('active');
            canvas.classList.remove('eraser-cursor');
            colorPicker.value = '#A8DF8E';
        });

        eraserBtn.addEventListener('click', () => {
            isEraser = true;
            eraserBtn.classList.add('active');
            penBtn.classList.remove('active');
            canvas.classList.add('eraser-cursor');
        });

        undoBtn.addEventListener('click', () => {
            if (historyIndex > 0) {
                historyIndex--;
                const img = new Image();
                img.src = history[historyIndex];
                img.onload = () => {
                    ctx.clearRect(0, 0, canvas.width, canvas.height);
                    ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
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
                    ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                };
            }
        });

        clearBtn.addEventListener('click', () => {
            if (confirm("Clear the entire canvas? This will reset undo history.")) {
                ctx.globalCompositeOperation = 'source-over';
                ctx.fillStyle = 'white';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                history = [];
                historyIndex = -1;
                saveState(); // save the cleared state
            }
        });

        saveBtn.addEventListener('click', () => {
            const link = document.createElement('a');
            link.download = `zest-drawing-${new Date().toISOString().slice(0,10)}.png`;
            link.href = canvas.toDataURL('image/png');
            link.click();
        });

        // Initialize
        saveState();
    </script>
</body>
</html>
