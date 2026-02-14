<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>SuperQuiz 1v1 Progressive Levels</title>
<style>
body { font-family: Arial; text-align: center; background: #46178f; color: white; overflow-x:hidden; }
.box { width: 520px; margin: 20px auto; background: #2b0a57; padding: 20px; border-radius: 12px; position: relative; }
button { margin: 6px; padding: 15px; border: none; border-radius: 8px; cursor: pointer; font-size: 15px; }
.red { background: #e21b3c; color:white; }
.blue { background: #1368ce; color:white; }
.green { background: #26890c; color:white; }
.yellow { background: #f2c80f; color:black; }
#timer { margin: 10px; font-size: 18px; height: 20px; }
#timerBar { height: 15px; background: #0f0; border-radius: 8px; transition: width 0.2s linear;}
#feedback { margin: 10px; font-size: 20px; height: 50px; }
.progress { width: 100%; background: #ddd; border-radius: 8px; margin-bottom: 10px; height: 15px; }
.progressFill { height: 100%; background: #f2c80f; width: 0%; border-radius: 8px; transition: width 0.5s;}
.firework { position: absolute; width: 10px; height: 10px; border-radius: 50%; animation: explode 0.8s forwards; }
@keyframes explode { 0% { transform: scale(1); opacity:1;} 100% { transform: scale(3); opacity:0;} }
 
/* Leaderboard table with black text and purple theme */
table { width: 90%; margin: 10px auto; border-collapse: collapse; border-radius:8px; overflow:hidden; }
th, td { padding: 8px; text-align: center; color: black; }  /* black text */
thead { background: #7b3de1; color: black; }       /* header */
tbody tr:nth-child(odd) { background: #5c2aa0; color: black; }  /* light purple odd rows */
tbody tr:nth-child(even){ background: #6b32c1; color: black; }  /* darker purple even rows */
tfoot { background: #7b3de1; color: black; font-weight: bold; } /* totals */
</style>
</head>
<body>
 
<!-- Sounds -->
<audio id="correctSound" src="https://freesound.org/data/previews/320/320181_5260877-lq.mp3"></audio>
<audio id="wrongSound" src="https://freesound.org/data/previews/256/256113_4486188-lq.mp3"></audio>
 
<div class="box" id="start">
<h2>SuperQuiz 1v1 vs Bot</h2>
<input id="player1" placeholder="Player 1 Name"><br><br>
<input id="player2" placeholder="Player 2 Name (leave blank for Bot)"><br><br>
<select id="botDifficulty">
  <option value="easy">Bot Difficulty: Easy</option>
  <option value="normal" selected>Normal</option>
  <option value="hard">Hard</option>
  <option value="extreme">Extreme</option>
</select><br><br>
<button onclick="startGame()">Start Quiz</button>
</div>
 
<div class="box" id="quiz" style="display:none;">
<div class="progress"><div id="levelProgress" class="progressFill"></div></div>
<h3 id="levelTitle"></h3>
<h3 id="question"></h3>
<div id="timer"><div id="timerBar"></div></div>
<div class="answers" id="answers"></div>
<div id="feedback"></div>
<div style="margin-top:10px;">
  <strong>Score:</strong> <span id="score1">0</span> - <span id="score2">0</span>
</div>
<div style="margin-top:10px;">
  <strong>Turn:</strong> <span id="turn"></span>
</div>
</div>
 
<div class="box" id="end" style="display:none;">
<h2 id="result"></h2>
<h3>Leaderboard</h3>
<table>
<thead>
<tr><th>Level</th><th id="th1"></th><th id="th2"></th></tr>
</thead>
<tbody id="leaderboardBody"></tbody>
<tfoot>
<tr><th>Total</th><th id="total1">0</th><th id="total2">0</th></tr>
</tfoot>
</table>
<button onclick="location.reload()">Play Again</button>
</div>
 
<script>
// Players & game state
let p1="", p2="", botDiff="normal";
let level=1, index=0, turn=1, timerValue=10, timer;
let score={1:0,2:0}, levelScores=[], questions=[];
 
// Questions per level
const levels = {
  1: [ {q:"2 + 2 = ?", o:["3","4","5","6"], a:"4"}, {q:"Water freezes at ?", o:["0°C","10°C","100°C","50°C"], a:"0°C"} ],
  2: [ {q:"5 × 6 = ?", o:["30","35","25","20"], a:"30"}, {q:"Past tense of 'run'?", o:["ran","runned","running","run"], a:"ran"} ],
  3: [ {q:"12 ÷ 4 = ?", o:["2","3","4","6"], a:"3"}, {q:"Chemical symbol for Oxygen?", o:["O","Ox","H","C"], a:"O"} ],
  4: [ {q:"Solve: 3x + 5 = 20, x = ?", o:["5","3","10","15"], a:"5"}, {q:"Largest planet?", o:["Jupiter","Earth","Saturn","Mars"], a:"Jupiter"} ],
  5: [ {q:"Solve: 5x - 7 = 18, x = ?", o:["5","3","7","4"], a:"5"}, {q:"Speed of light (km/s)?", o:["300,000","30,000","300","3,000"], a:"300,000"} ]
};
 
// Sounds
const correctSound=document.getElementById("correctSound");
const wrongSound=document.getElementById("wrongSound");
 
// Start game
function startGame(){
  p1=document.getElementById("player1").value||"Player 1";
  p2=document.getElementById("player2").value||"Bot";
  botDiff=document.getElementById("botDifficulty").value;
  document.getElementById("start").style.display="none";
  document.getElementById("quiz").style.display="block";
  level=1; levelScores=[];
  prepareLevel(); loadQuestion();
}
 
// Prepare level
function prepareLevel(){
  questions = levels[level].sort(()=>Math.random()-0.5);
  index = 0; score = {1:0,2:0};
  updateProgress();
}
 
// Load question
function loadQuestion(){
  clearInterval(timer);
  if(index >= questions.length){ nextLevel(); return; }
 
  timerValue=10;
  document.getElementById("timerBar").style.width="100%";
  document.getElementById("feedback").textContent="";
  document.getElementById("score1").textContent=score[1];
  document.getElementById("score2").textContent=score[2];
  document.getElementById("turn").textContent=(turn===1?p1:p2);
  document.getElementById("levelTitle").textContent="Level "+level;
 
  let q = questions[index];
  document.getElementById("question").textContent = q.q;
 
  let shuffled = q.o.slice().sort(()=>Math.random()-0.5);
  let ans=document.getElementById("answers"); ans.innerHTML="";
  let colors=["red","blue","green","yellow"];
  shuffled.forEach((opt,i)=>{
    let btn=document.createElement("button");
    btn.className=colors[i]; btn.textContent=opt; btn.onclick=()=>check(opt,q.a);
    ans.appendChild(btn);
  });
 
  if(p2==="Bot" && turn===2){ setTimeout(()=>botAnswer(q.a),botDelay()); }
 
  timer=setInterval(()=>{
    timerValue--;
    document.getElementById("timerBar").style.width=(timerValue*10)+"%";
    if(timerValue<=0) nextTurn();
  },1000);
}
 
// Bot logic
function botDelay(){ switch(botDiff){ case "easy": return 2500+Math.random()*1500; case "normal": return 1500+Math.random()*1000; case "hard": return 1000+Math.random()*500; case "extreme": return 500+Math.random()*500; } }
function botAccuracy(){ switch(botDiff){ case "easy": return 0.5; case "normal": return 0.7; case "hard": return 0.85; case "extreme": return 0.95; } }
function firework(correct){ const box=document.getElementById("quiz"); for(let i=0;i<15;i++){ const f=document.createElement("div"); f.className="firework"; f.style.background=correct?"#0f0":"#f00"; f.style.left=Math.random()*400+"px"; f.style.top=Math.random()*200+"px"; box.appendChild(f); setTimeout(()=>f.remove(),800);} }
function botAnswer(correct){ clearInterval(timer); if(Math.random()<botAccuracy()){ score[2]+=timerValue; document.getElementById("feedback").textContent="Bot ✅ Correct!"; correctSound.play(); firework(true);} else{ document.getElementById("feedback").textContent="Bot ❌ Wrong! 😢"; wrongSound.play(); firework(false);} document.getElementById("score1").textContent=score[1]; document.getElementById("score2").textContent=score[2]; setTimeout(nextTurn,1000); }
 
// Check player answer
function check(choice,answer){ clearInterval(timer); if(choice===answer){ score[turn]+=timerValue; document.getElementById("feedback").textContent="✅ Correct!"; correctSound.play(); firework(true);} else{ document.getElementById("feedback").textContent="❌ Wrong! 😢"; wrongSound.play(); firework(false);} document.getElementById("score1").textContent=score[1]; document.getElementById("score2").textContent=score[2]; setTimeout(nextTurn,1000); }
 
// Alternate turn
function nextTurn(){ if(turn===1){ turn=2; loadQuestion(); } else{ turn=1; index++; updateProgress(); loadQuestion(); } }
 
// Progress bar
function updateProgress(){ document.getElementById("levelProgress").style.width=((index)/questions.length)*100+"%"; }
 
// Next level
function nextLevel(){ levelScores.push({1:score[1],2:score[2]}); if(level<5){ level++; prepareLevel(); alert("Level "+level+" unlocked!"); loadQuestion();} else finish(); }
 
// Finish game with full purple leaderboard and black text
function finish(){
  if(levelScores.length < 5){
      levelScores.push({1:score[1],2:score[2]});
  }
 
  document.getElementById("quiz").style.display="none";
  document.getElementById("end").style.display="block";
 
  let total1=0, total2=0;
  let tbody="";
 
  for(let i=0; i<levelScores.length; i++){
      let s1 = levelScores[i][1];
      let s2 = levelScores[i][2];
      total1 += s1;
      total2 += s2;
 
      tbody += "<tr>"+
               "<td>Level "+(i+1)+"</td>"+
               "<td>"+s1+"</td>"+
               "<td>"+s2+"</td>"+
               "</tr>";
  }
 
  document.getElementById("leaderboardBody").innerHTML = tbody;
 
  document.getElementById("total1").textContent = total1;
  document.getElementById("total2").textContent = total2;
 
  document.getElementById("th1").textContent = p1;
  document.getElementById("th2").textContent = p2;
 
  let winner="";
  if(total1>total2) winner = p1+" wins!";
  else if(total2>total1) winner = p2+" wins!";
  else winner = "It's a tie!";
 
  document.getElementById("result").textContent = "Final Result: "+winner;
}
</script>
 
</body>
</html>
