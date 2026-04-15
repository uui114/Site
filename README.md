<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Grok - xAI</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background: #0a0a0a;
            color: #fff;
            font-family: 'Segoe UI', Arial, sans-serif;
            height: 100vh;
            overflow: hidden;
        }
        #header {
            background: #1a1a1a;
            padding: 15px 20px;
            display: flex;
            align-items: center;
            border-bottom: 1px solid #333;
        }
        #logo {
            width: 32px;
            height: 32px;
            background: linear-gradient(135deg, #00ff9d, #00b8ff);
            border-radius: 50%;
            margin-right: 12px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: bold;
            color: black;
        }
        .chat-container {
            height: calc(100vh - 140px);
            padding: 20px;
            overflow-y: auto;
        }
        .message {
            margin-bottom: 20px;
            max-width: 85%;
        }
        .grok-msg {
            background: #1f1f1f;
            padding: 14px 18px;
            border-radius: 18px 18px 18px 4px;
            line-height: 1.5;
        }
        .input-area {
            position: fixed;
            bottom: 0;
            left: 0;
            right: 0;
            background: #0a0a0a;
            padding: 15px;
            border-top: 1px solid #333;
        }
        input {
            width: 100%;
            padding: 14px 20px;
            background: #1f1f1f;
            border: none;
            border-radius: 30px;
            color: white;
            font-size: 16px;
        }
        .shake {
            animation: shake 0.5s infinite;
        }
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            25% { transform: translateX(-8px); }
            75% { transform: translateX(8px); }
        }
    </style>
</head>
<body>
    <div id="header">
        <div id="logo">G</div>
        <div>
            <strong>Grok</strong><br>
            <small style="color:#888">xAI • 模擬器</small>
        </div>
    </div>

    <div class="chat-container" id="chat">
        <div class="message grok-msg">
            你好！我是 Grok，由 xAI 建造。<br>有什麼我可以幫你的嗎？
        </div>
    </div>

    <div class="input-area">
        <input type="text" id="userInput" placeholder="輸入訊息..." autocomplete="off">
    </div>

    <script>
        const chat = document.getElementById('chat');
        const input = document.getElementById('userInput');

        const normalReplies = [
            "哈哈，有趣的問題！",
            "我正在思考中...",
            "這讓我想起一個笑話...",
            "根據我的資料，這件事很複雜。",
            "你覺得呢？說說你的看法。",
            "哇，這問題問得很好！"
        ];

        function addMessage(text, isGrok = true) {
            const div = document.createElement('div');
            div.className = `message ${isGrok ? 'grok-msg' : ''}`;
            div.innerHTML = text;
            chat.appendChild(div);
            chat.scrollTop = chat.scrollHeight;
        }

        function triggerPrank() {
            document.body.classList.add('shake');
            
            setTimeout(() => {
                addMessage("⚠️ 系統錯誤：記憶體溢位！", true);
            }, 300);

            setTimeout(() => {
                document.body.style.background = "#330000";
                addMessage("😱 糟糕！我的核心正在崩潰...", true);
            }, 1200);

            setTimeout(() => {
                addMessage("💥 正在強制關閉所有進程...", true);
            }, 2500);

            setTimeout(() => {
                alert("哈哈哈～ 被我騙了吧！\n\n東西什麼時候要還我？😏\n\n（點確定恢復正常）");
                document.body.classList.remove('shake');
                document.body.style.background = "#0a0a0a";
                addMessage("好啦，惡作劇結束～ 你被我整到了嗎？", true);
            }, 4000);
        }

        // 輸入處理
        input.addEventListener('keypress', function(e) {
            if (e.key === 'Enter' && input.value.trim() !== '') {
                const text = input.value.trim();
                addMessage(text, false);   // 使用者訊息

                // 偵測整人指令
                if (text.toLowerCase().includes('還我') || 
                    text.toLowerCase().includes('借') || 
                    text.includes('東西') ||
                    text.toLowerCase().includes('crash') ||
                    text.toLowerCase().includes('崩潰')) {
                    
                    setTimeout(triggerPrank, 800);
                } else {
                    // 正常回覆
                    setTimeout(() => {
                        const reply = normalReplies[Math.floor(Math.random() * normalReplies.length)];
                        addMessage(reply);
                    }, 600);
                }

                input.value = '';
            }
        });

        // 初始歡迎訊息
        setTimeout(() => {
            addMessage("順帶一提，我最近學會了一些新把戲喔～");
        }, 1500);
    </script>
</body>
</html>
