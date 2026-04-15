<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>打字變旋轉3D文字</title>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            height: 100%;
            overflow: hidden;
            background: #0a0a1f;
            font-family: 'Microsoft YaHei', Arial, sans-serif;
            color: white;
        }
        #scene {
            perspective: 1200px;
            width: 100%;
            height: 100%;
            display: flex;
            align-items: center;
            justify-content: center;
            position: relative;
        }
        #textContainer {
            transform-style: preserve-3d;
            font-size: 4.5vw;
            font-weight: bold;
            text-shadow: 
                0 0 20px #ff00ff,
                0 0 40px #00ffff,
                0 0 60px #ffff00;
            transition: transform 0.05s linear;
            white-space: pre;
            text-align: center;
            padding: 20px;
            max-width: 90%;
        }
        #inputArea {
            position: absolute;
            bottom: 30px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(0,0,0,0.6);
            padding: 12px 20px;
            border-radius: 50px;
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255,255,255,0.2);
            width: 80%;
            max-width: 500px;
        }
        input {
            width: 100%;
            background: transparent;
            border: none;
            color: white;
            font-size: 18px;
            outline: none;
        }
        #controls {
            position: absolute;
            top: 20px;
            right: 20px;
            z-index: 100;
        }
        button {
            padding: 10px 16px;
            margin-left: 8px;
            background: rgba(255,255,255,0.1);
            color: white;
            border: 1px solid rgba(255,255,255,0.3);
            border-radius: 8px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div id="controls">
        <button onclick="clearText()">清除文字</button>
    </div>

    <div id="scene">
        <div id="textContainer">開始打字吧...</div>
    </div>

    <div id="inputArea">
        <input type="text" id="input" placeholder="在這裡打字..." autocomplete="off" autofocus>
    </div>

    <script>
        const textContainer = document.getElementById('textContainer');
        const input = document.getElementById('input');
        let currentText = '';
        let rotationX = 0;
        let rotationY = 0;
        let targetRotX = 15;
        let targetRotY = 30;

        function update3DText() {
            currentText = input.value || '開始打字吧...';
            
            // 逐字包 span 增加打字動畫感
            let html = '';
            for (let char of currentText) {
                if (char === ' ') {
                    html += '<span style="display:inline-block;width:0.6em;"> </span>';
                } else {
                    html += `<span style="display:inline-block; transition: transform 0.4s;">${char}</span>`;
                }
            }
            textContainer.innerHTML = html || '開始打字吧...';
        }

        function animateRotation() {
            // 自動緩慢旋轉
            rotationY += 0.35;           // Y軸持續旋轉
            rotationX = Math.sin(Date.now() / 2000) * 12 + targetRotX; // X軸輕微呼吸擺動

            textContainer.style.transform = 
                `rotateX(\( {rotationX}deg) rotateY( \){rotationY}deg) scale(1)`;
            
            requestAnimationFrame(animateRotation);
        }

        // 即時打字更新
        input.addEventListener('input', update3DText);

        // 按 Enter 清空輸入框（但保留文字）
        input.addEventListener('keydown', (e) => {
            if (e.key === 'Enter') {
                input.value = '';
                update3DText();
            }
        });

        window.clearText = function() {
            input.value = '';
            update3DText();
        };

        // 初始化
        update3DText();
        animateRotation();

        // 滑鼠移動時輕微改變角度（增加互動感）
        document.addEventListener('mousemove', (e) => {
            const xPercent = (e.clientX / window.innerWidth) - 0.5;
            const yPercent = (e.clientY / window.innerHeight) - 0.5;
            targetRotX = 15 + yPercent * 25;
            targetRotY = 30 + xPercent * 40;
        });

        // 手機觸控支援
        let touchStartX = 0;
        document.addEventListener('touchmove', (e) => {
            if (e.touches.length === 1) {
                const deltaX = (e.touches[0].clientX - touchStartX) * 0.2;
                targetRotY += deltaX;
                touchStartX = e.touches[0].clientX;
            }
        }, { passive: true });
    </script>
</body>
</html>
