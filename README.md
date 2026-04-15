<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Mini Games</title>

<style>
body{
  margin:0;
  font-family:Arial;
  background:white;
  overflow:hidden;
}

/* 主畫面 */
#menu{
  height:100vh;
  overflow-y:auto;
  display:flex;
  flex-direction:column;
  align-items:center;
  padding-top:80px;
}

/* 右上角代碼框 */
#codeBox{
  position:fixed;
  top:20px;
  right:20px;
  display:flex;
  gap:10px;
  z-index:999;
}

#codeInput{
  border:3px solid black;
  font-size:18px;
  padding:10px;
}

#codeBox button{
  border:3px solid black;
  background:white;
  font-size:18px;
  padding:10px 15px;
  cursor:pointer;
}

/* 主畫面按鈕 */
.gameBtn{
  width:90%;
  height:33vh;
  margin:20px;
  font-size:6vw;
  border:5px solid black;
  background:white;
  border-radius:25px;
  cursor:pointer;
}

.locked{
  opacity:0.4;
}

/* 遊戲畫面 */
#game{
  display:none;
  height:100vh;
  flex-direction:column;
  align-items:center;
  justify-content:flex-start;
}

/* 標題 */
h1{
  font-size:60px;
  margin:20px;
}

/* 進度條 */
#barBox{
  width:70%;
  height:50px;
  border:5px solid black;
}

#bar{
  height:100%;
  width:50%;
  background:white;
}

/* 按鈕 */
.clickBtn{
  width:300px;
  height:300px;
  margin-top:60px;
  border:5px solid black;
  background:white;
  font-size:40px;
  cursor:pointer;
}

/* GIF */
#gif{
  display:none;
  width:100%;
  height:100vh;
  object-fit:contain;
}
</style>
</head>

<body>

<!-- 右上角代碼 -->
<div id="codeBox">
  <input id="codeInput" placeholder="Enter Code">
  <button onclick="checkCode()">OK</button>
</div>

<!-- 主畫面 -->
<div id="menu">
  <button class="gameBtn" onclick="startGame()">SAVE THE CAT!!!</button>
  <button class="gameBtn">COMING SOON</button>
  <button id="level3Btn" class="gameBtn locked" onclick="startLevel3()">🔒 LEVEL 3</button>
</div>

<!-- 遊戲 -->
<div id="game">
  <h1>SAVE THE CAT!!!</h1>
  <div id="barBox"><div id="bar"></div></div>
  <button class="clickBtn" onclick="addProgress()">CLICK ME</button>
</div>

<img id="gif">

<script>

let progress = 50;
let drain;
let level3Unlocked = localStorage.getItem("level3Unlocked")==="true";

/* 更新第三關 */
function updateLevel3(){
  if(level3Unlocked){
    const btn=document.getElementById("level3Btn");
    btn.classList.remove("locked");
    btn.innerText="LEVEL 3";
  }
}
updateLevel3();

/* 開始第一關 */
function startGame(){
  document.getElementById("menu").style.display="none";
  document.getElementById("game").style.display="flex";

  progress=50;
  updateBar();

  drain=setInterval(()=>{
    progress-=0.4 + progress/200;
    updateBar();
    checkLose();
  },50);
}

/* 點擊增加 */
function addProgress(){
  progress+=15;
  if(progress>100) progress=100;
  updateBar();
  if(progress>=100) win();
}

/* 更新條 */
function updateBar(){
  document.getElementById("bar").style.width=progress+"%";
}

/* 勝利 */
function win(){
  clearInterval(drain);
  showGif("https://media1.tenor.com/m/7aod08iYQuMAAAAC/mane-im-dead.gif");
}

/* 失敗 */
function checkLose(){
  if(progress<=0){
    clearInterval(drain);
    showGif("https://media.tenor.com/ioyb8tciIyQAAAAi/cat-explode-cat-meme.gif");
  }
}

/* 顯示GIF */
function showGif(src){
  document.getElementById("game").style.display="none";
  let gif=document.getElementById("gif");
  gif.src=src;
  gif.style.display="block";

  setTimeout(()=>{
    gif.style.display="none";
    document.getElementById("menu").style.display="flex";
  },5000);
}

/* 代碼解鎖 */
function checkCode(){
  if(document.getElementById("codeInput").value==="ihatesixseven!!!"){
    level3Unlocked=true;
    localStorage.setItem("level3Unlocked",true);
    updateLevel3();
    alert("LEVEL 3 UNLOCKED!");
  }
}

/* 第三關 */
function startLevel3(){
  if(!level3Unlocked){
    alert("Locked!");
    return;
  }
  alert("Level 3 coming soon 😈");
}

</script>
</body>
</html>
