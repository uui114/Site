<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <title>Roblox 使用者查詢 - 顯示正在玩什麼</title>
  <style>
    body { font-family: Arial, sans-serif; text-align: center; padding: 50px; background: #f4f4f4; }
    input, button { padding: 12px; font-size: 18px; margin: 10px; }
    #result { margin-top: 40px; padding: 20px; background: white; border-radius: 10px; max-width: 600px; margin-left: auto; margin-right: auto; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
    img { width: 180px; border-radius: 50%; border: 4px solid #00b0f0; }
    .game-link { color: #00b0f0; text-decoration: none; font-weight: bold; }
    .game-link:hover { text-decoration: underline; }
  </style>
</head>
<body>
  <h1>Roblox 使用者查詢</h1>
  <p>輸入 Roblox 使用者名稱，查看基本資料與正在玩的遊戲</p>
  
  <input type="text" id="username" placeholder="例如: Builderman 或你的帳號" style="width: 300px;" />
  <button onclick="searchUser()">查詢</button>

  <div id="result"></div>

  <script>
    async function searchUser() {
      const usernameInput = document.getElementById('username').value.trim();
      if (!usernameInput) return alert('請輸入 Roblox 使用者名稱');

      const resultDiv = document.getElementById('result');
      resultDiv.innerHTML = '<p>查詢中，請稍候...</p>';

      try {
        // 1. 用 username 取得 User ID
        const idRes = await fetch('https://users.roblox.com/v1/usernames/users', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ usernames: [usernameInput], excludeBannedUsers: false })
        });
        const idData = await idRes.json();

        if (!idData.data || idData.data.length === 0) {
          resultDiv.innerHTML = `<p style="color:red;">找不到這個使用者名稱</p>`;
          return;
        }

        const userId = idData.data[0].id;

        // 2. 取得使用者基本資料
        const userRes = await fetch(`https://users.roblox.com/v1/users/${userId}`);
        const user = await userRes.json();

        // 3. 取得 Presence（正在玩什麼）
        const presenceRes = await fetch('https://presence.roblox.com/v1/presence/users', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ userIds: [userId] })
        });
        const presenceData = await presenceRes.json();
        const presence = presenceData.userPresences && presenceData.userPresences[0];

        let presenceHTML = '';
        if (presence) {
          const type = presence.userPresenceType;
          let status = '離線';
          let gameInfo = '';

          if (type === 0) status = '離線';
          else if (type === 1) status = '線上（不在遊戲中）';
          else if (type === 2 || type === 3) {
            status = '正在玩遊戲';
            if (presence.placeId) {
              const placeRes = await fetch(`https://games.roblox.com/v1/places/${presence.placeId}`);
              const place = await placeRes.json();
              const gameName = place.name || '未知遊戲';
              const gameUrl = `https://www.roblox.com/games/${presence.placeId}`;
              gameInfo = `<br><strong>遊戲名稱：</strong> <a href="\( {gameUrl}" target="_blank" class="game-link"> \){gameName}</a>`;
            }
          } else {
            status = '其他狀態';
          }

          presenceHTML = `
            <p><strong>目前狀態：</strong> ${status}</p>
            ${gameInfo}
            ${presence.lastLocation ? `<p><strong>位置：</strong> ${presence.lastLocation}</p>` : ''}
          `;
        }

        // 顯示所有結果
        resultDiv.innerHTML = `
          <img src="https://www.roblox.com/headshot-thumbnail/image?userId=${userId}&width=180&height=180&format=png" alt="頭像">
          <h2>\( {user.displayName} (@ \){user.name})</h2>
          <p><strong>User ID：</strong> ${userId}</p>
          <p><strong>描述：</strong> ${user.description || '無描述'}</p>
          <p><strong>創建日期：</strong> ${new Date(user.created).toLocaleDateString('zh-TW')}</p>
          ${presenceHTML}
        `;
      } catch (err) {
        resultDiv.innerHTML = `<p style="color:red;">發生錯誤，請稍後再試或檢查網路</p>`;
        console.error(err);
      }
    }

    // 按 Enter 也可以查詢
    document.getElementById('username').addEventListener('keypress', function(e) {
      if (e.key === 'Enter') searchUser();
    });
  </script>
</body>
</html>
