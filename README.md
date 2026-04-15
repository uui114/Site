<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>黑暗料理</title>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            overflow: hidden;
            background: #111;
            height: 100%;
            font-family: Arial, sans-serif;
            touch-action: none;
        }
        canvas {
            display: block;
        }
        #ui {
            position: absolute;
            top: 10px;
            left: 10px;
            background: rgba(0,0,0,0.7);
            padding: 10px;
            border-radius: 8px;
            color: white;
            z-index: 100;
            user-select: none;
        }
        button {
            margin: 5px 3px;
            padding: 8px 12px;
            background: #333;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        button.active {
            background: #f44;
        }
        #info {
            position: absolute;
            bottom: 10px;
            left: 10px;
            color: rgba(255,255,255,0.6);
            font-size: 14px;
        }
    </style>
</head>
<body>
    <div id="ui">
        <strong>元素：</strong><br>
        <button data-type="sand" class="active">沙子</button>
        <button data-type="water">水</button>
        <button data-type="stone">石頭</button>
        <button data-type="fire">火</button>
        <button data-type="plant">植物</button>
        <button data-type="meat">肉</button>
        <button data-type="erase">清除</button>
        <br>
        <button onclick="clearAll()">清空畫面</button>
        <button onclick="togglePause()">暫停/繼續</button>
    </div>
    <div id="info">拖曳畫元素 • 火烤肉、水煮肉試試看！</div>
    <canvas id="canvas"></canvas>

    <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d', { alpha: false });
        let width = 0, height = 0;
        let grid = [];
        let nextGrid = [];
        let isPaused = false;
        let currentType = 'sand';
        let brushSize = 8;

        const TYPES = {
            empty: { color: '#000000', name: '空' },
            sand: { color: '#e2c78f', name: '沙子', density: 3 },
            water: { color: '#4a90e2', name: '水', density: 2 },
            stone: { color: '#555555', name: '石頭', density: 10, fixed: true },
            fire: { color: '#ff4400', name: '火', density: 0 },
            plant: { color: '#228b22', name: '植物', density: 4 },
            meat: { color: '#8b4513', name: '肉', density: 3, cooked: 0 },
            smoke: { color: '#aaaaaa', name: '煙', density: -1 }
        };

        function resize() {
            width = Math.floor(window.innerWidth / 2) * 2;  // 保持偶數
            height = Math.floor(window.innerHeight / 2) * 2;
            canvas.width = width;
            canvas.height = height;

            grid = Array(height).fill().map(() => Array(width).fill(0));
            nextGrid = Array(height).fill().map(() => Array(width).fill(0));
        }

        function getType(id) {
            return Object.keys(TYPES)[id] || 'empty';
        }

        function drawPixel(x, y, typeId) {
            if (x < 0 || x >= width || y < 0 || y >= height) return;
            grid[y][x] = typeId;
        }

        function paint(x, y, type) {
            const typeId = Object.keys(TYPES).indexOf(type);
            const r = Math.floor(brushSize / 2);
            for (let dy = -r; dy <= r; dy++) {
                for (let dx = -r; dx <= r; dx++) {
                    if (dx*dx + dy*dy <= r*r) {
                        drawPixel(Math.floor(x + dx), Math.floor(y + dy), typeId);
                    }
                }
            }
        }

        function updateSimulation() {
            if (isPaused) return;

            // 複製到 nextGrid
            for (let y = 0; y < height; y++) {
                for (let x = 0; x < width; x++) {
                    nextGrid[y][x] = grid[y][x];
                }
            }

            for (let y = height - 2; y >= 0; y--) {
                for (let x = 0; x < width; x++) {
                    const id = grid[y][x];
                    if (id === 0) continue;
                    const typeName = getType(id);

                    if (typeName === 'stone') continue; // 固定不動

                    let moved = false;

                    // 向下掉
                    if (y + 1 < height && TYPES[getType(grid[y+1][x])].density < TYPES[typeName].density) {
                        if (Math.random() < 0.9) {
                            nextGrid[y][x] = grid[y+1][x];
                            nextGrid[y+1][x] = id;
                            moved = true;
                        }
                    }

                    if (!moved) {
                        // 左右流動 (水、火、煙)
                        if (typeName === 'water' || typeName === 'fire' || typeName === 'smoke') {
                            const dir = Math.random() < 0.5 ? -1 : 1;
                            if (x + dir >= 0 && x + dir < width && TYPES[getType(grid[y][x+dir])].density < TYPES[typeName].density) {
                                nextGrid[y][x] = grid[y][x+dir];
                                nextGrid[y][x+dir] = id;
                                moved = true;
                            }
                        }
                    }

                    // 特殊互動
                    if (typeName === 'fire') {
                        // 火向上變煙
                        if (Math.random() < 0.3 && y > 0) {
                            nextGrid[y][x] = Object.keys(TYPES).indexOf('smoke');
                        }
                        // 燒肉、植物
                        if (y+1 < height) {
                            const below = getType(grid[y+1][x]);
                            if (below === 'meat') {
                                nextGrid[y+1][x] = id; // 火取代肉位置，肉被燒
                                // 肉變熟（顏色我們在繪製時處理）
                            } else if (below === 'plant' && Math.random() < 0.4) {
                                nextGrid[y+1][x] = id;
                            }
                        }
                    }

                    if (typeName === 'meat') {
                        // 簡單煮熟/燒焦邏輯（顏色變化在繪製時）
                        if (grid[y-1] && getType(grid[y-1][x]) === 'fire') {
                            // 被火烤 → cooked 值增加（這裡用隨機模擬）
                        }
                    }

                    if (typeName === 'smoke') {
                        // 煙往上飄
                        if (y > 0 && Math.random() < 0.6 && TYPES[getType(grid[y-1][x])].density < 0) {
                            nextGrid[y][x] = grid[y-1][x];
                            nextGrid[y-1][x] = id;
                        } else if (Math.random() < 0.2) {
                            nextGrid[y][x] = 0; // 煙消散
                        }
                    }

                    if (typeName === 'plant') {
                        // 簡單生長（碰到水）
                        if (Math.random() < 0.05) {
                            if (y+1 < height && getType(grid[y+1][x]) === 'water') {
                                nextGrid[y+1][x] = id; // 往下長一點
                            }
                        }
                    }
                }
            }

            // 複製回 grid
            for (let y = 0; y < height; y++) {
                for (let x = 0; x < width; x++) {
                    grid[y][x] = nextGrid[y][x];
                }
            }
        }

        function render() {
            const imageData = ctx.createImageData(width, height);
            const data = imageData.data;

            for (let y = 0; y < height; y++) {
                for (let x = 0; x < width; x++) {
                    const idx = (y * width + x) * 4;
                    const typeName = getType(grid[y][x]);
                    let col = TYPES[typeName].color;

                    // 特殊顏色調整（黑暗料理視覺）
                    if (typeName === 'meat') {
                        // 越接近火越焦（這裡簡單用 y 位置模擬）
                        const darkness = Math.min(80, y / 5);
                        col = `rgb(${139 - darkness}, ${69 - darkness/2}, ${19})`;
                    } else if (typeName === 'fire') {
                        col = Math.random() < 0.5 ? '#ffaa00' : '#ff4400';
                    }

                    const rgb = hexToRgb(col || '#000000');
                    data[idx] = rgb.r;
                    data[idx+1] = rgb.g;
                    data[idx+2] = rgb.b;
                    data[idx+3] = 255;
                }
            }
            ctx.putImageData(imageData, 0, 0);
        }

        function hexToRgb(hex) {
            const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
            return result ? {
                r: parseInt(result[1], 16),
                g: parseInt(result[2], 16),
                b: parseInt(result[3], 16)
            } : {r:0, g:0, b:0};
        }

        function loop() {
            updateSimulation();
            render();
            requestAnimationFrame(loop);
        }

        // 事件處理
        let isDrawing = false;
        function handleDraw(e) {
            if (!isDrawing) return;
            const rect = canvas.getBoundingClientRect();
            const x = (e.clientX || e.touches[0].clientX) - rect.left;
            const y = (e.clientY || e.touches[0].clientY) - rect.top;
            paint(x, y, currentType);
        }

        canvas.addEventListener('mousedown', (e) => { isDrawing = true; handleDraw(e); });
        canvas.addEventListener('mousemove', handleDraw);
        canvas.addEventListener('mouseup', () => isDrawing = false);
        canvas.addEventListener('mouseleave', () => isDrawing = false);

        canvas.addEventListener('touchstart', (e) => { isDrawing = true; handleDraw(e); e.preventDefault(); });
        canvas.addEventListener('touchmove', (e) => { handleDraw(e); e.preventDefault(); });
        canvas.addEventListener('touchend', () => isDrawing = false);

        // UI 按鈕
        document.querySelectorAll('#ui button[data-type]').forEach(btn => {
            btn.addEventListener('click', () => {
                document.querySelectorAll('#ui button').forEach(b => b.classList.remove('active'));
                btn.classList.add('active');
                currentType = btn.dataset.type;
            });
        });

        window.clearAll = function() {
            grid = Array(height).fill().map(() => Array(width).fill(0));
        };

        window.togglePause = function() {
            isPaused = !isPaused;
        };

        // 初始化
        resize();
        window.addEventListener('resize', resize);

        // 初始放一點石頭當鍋底
        for (let x = width*0.3; x < width*0.7; x++) {
            drawPixel(x, height - 30, Object.keys(TYPES).indexOf('stone'));
        }

        loop();

        // 防止手機滾動
        document.addEventListener('touchmove', e => e.preventDefault(), { passive: false });
    </script>
</body>
</html>
