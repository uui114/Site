<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title></title>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            height: 100%;
            background: #000;
            overflow: hidden;
            color: white;
            font-family: Arial, sans-serif;
        }
        #scare {
            position: fixed;
            top: 0; left: 0;
            width: 100%;
            height: 100%;
            background: rgba(255, 0, 0, 0.9);
            display: none;
            align-items: center;
            justify-content: center;
            flex-direction: column;
            z-index: 999;
            animation: flash 0.15s infinite;
        }
        #scare-text {
            font-size: 48px;
            font-weight: bold;
            text-align: center;
            text-shadow: 0 0 20px #fff;
        }
        @keyframes flash {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.6; }
        }
        .shake {
            animation: shake 80ms infinite linear;
        }
        @keyframes shake {
            0% { transform: translate(0, 0); }
            10% { transform: translate(-12px, 8px); }
            20% { transform: translate(12px, -8px); }
            30% { transform: translate(-10px, 6px); }
            40% { transform: translate(10px, -6px); }
            50% { transform: translate(-8px, 4px); }
            60% { transform: translate(8px, -4px); }
            70% { transform: translate(-6px, 2px); }
            80% { transform: translate(6px, -2px); }
            90% { transform: translate(-4px, 1px); }
            100% { transform: translate(0, 0); }
        }
    </style>
</head>
<body>

    <div id="scare">
        <div id="scare-text">do you hear the whistle?</div>
        <div style="margin-top: 40px; font-size: 24px;">快跑！</div>
    </div>

    <script>
        // 頁面載入後立即彈出輸入框
        window.onload = function() {
            setTimeout(() => {
                const answer = prompt("do you hear the whistle");

                // 不管輸入什麼，都觸發驚嚇效果
                if (answer !== null) {
                    triggerScare();
                } else {
                    // 如果按取消，也一樣觸發
                    triggerScare();
                }
            }, 800); // 稍微延遲一下，避免太突然
        };

        function triggerScare() {
            const scareDiv = document.getElementById('scare');
            const body = document.body;

            scareDiv.style.display = 'flex';
            body.classList.add('shake');

            // 播放幾次閃爍
            let count = 0;
            const interval = setInterval(() => {
                count++;
                if (count > 12) {
                    clearInterval(interval);
                    // 結束後顯示最終畫面
                    scareDiv.innerHTML = `
                        <div style="font-size: 60px; color: #ff0000; text-shadow: 0 0 30px #fff;">
                            you hear it
                        </div>
                        <div style="margin-top: 30px; font-size: 28px;">...</div>
                    `;
                }
            }, 180);
        }

        // 額外：按任意鍵或點擊畫面也可以再次觸發（方便測試）
        document.addEventListener('click', () => {
            if (document.getElementById('scare').style.display === 'flex') {
                triggerScare();
            }
        });
    </script>
</body>
</html>
