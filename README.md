<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>彈球填滿螢幕</title>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            overflow: hidden;
            background: #000;
            height: 100%;
            touch-action: none;
        }
        canvas {
            display: block;
        }
        #info {
            position: absolute;
            top: 10px;
            left: 10px;
            color: rgba(255,255,255,0.7);
            font-family: Arial, sans-serif;
            pointer-events: none;
            z-index: 10;
        }
    </style>
</head>
<body>
    <div id="info">球數量: <span id="count">1</span></div>
    <canvas id="canvas"></canvas>

    <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        const countEl = document.getElementById('count');

        let balls = [];
        let width, height;

        class Ball {
            constructor(x, y, vx, vy, radius, color) {
                this.x = x;
                this.y = y;
                this.vx = vx;
                this.vy = vy;
                this.radius = radius;
                this.color = color;
            }

            update() {
                this.x += this.vx;
                this.y += this.vy;

                // 撞邊界時反彈 + 分裂成兩顆
                let split = false;

                if (this.x - this.radius <= 0 || this.x + this.radius >= width) {
                    this.vx = -this.vx * 0.98; // 輕微能量損失
                    this.x = Math.max(this.radius, Math.min(width - this.radius, this.x));
                    split = true;
                }

                if (this.y - this.radius <= 0 || this.y + this.radius >= height) {
                    this.vy = -this.vy * 0.98;
                    this.y = Math.max(this.radius, Math.min(height - this.radius, this.y));
                    split = true;
                }

                if (split && balls.length < 800) {  // 限制最大數量避免卡死
                    // 分裂成兩顆，速度隨機微調
                    const newVx = this.vx * (0.8 + Math.random() * 0.6);
                    const newVy = this.vy * (0.8 + Math.random() * 0.6);

                    balls.push(new Ball(
                        this.x,
                        this.y,
                        newVx + (Math.random() - 0.5) * 2,
                        newVy + (Math.random() - 0.5) * 2,
                        this.radius * 0.95,  // 稍微變小一點
                        `hsl(${Math.random()*360}, 90%, 65%)`
                    ));
                }
            }

            draw() {
                ctx.save();
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.shadowBlur = 15;
                ctx.shadowColor = this.color;
                ctx.fill();
                ctx.restore();
            }
        }

        function resize() {
            width = window.innerWidth;
            height = window.innerHeight;
            canvas.width = width;
            canvas.height = height;
        }

        function createInitialBall() {
            const radius = 18;
            const x = width / 2;
            const y = height / 3;
            const speed = 4.5;

            balls = [];
            balls.push(new Ball(
                x, y,
                (Math.random() - 0.5) * speed * 2,
                speed * 0.8,
                radius,
                `hsl(${Math.random()*360}, 90%, 65%)`
            ));
        }

        function animate() {
            // 半透明黑色背景，產生拖尾效果
            ctx.fillStyle = 'rgba(0, 0, 0, 0.15)';
            ctx.fillRect(0, 0, width, height);

            for (let i = 0; i < balls.length; i++) {
                balls[i].update();
                balls[i].draw();
            }

            countEl.textContent = balls.length;
            requestAnimationFrame(animate);
        }

        // 初始化
        window.addEventListener('resize', () => {
            resize();
        });

        // 點擊畫面可手動加入一顆球（可選）
        canvas.addEventListener('click', (e) => {
            const radius = 12 + Math.random() * 10;
            balls.push(new Ball(
                e.clientX,
                e.clientY,
                (Math.random() - 0.5) * 8,
                (Math.random() - 0.5) * 8,
                radius,
                `hsl(${Math.random()*360}, 90%, 65%)`
            ));
        });

        // 啟動
        resize();
        createInitialBall();
        animate();

        // 防止手機滾動
        document.addEventListener('touchmove', (e) => e.preventDefault(), { passive: false });
    </script>
</body>
</html>
