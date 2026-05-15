<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>لعبة X O</title>
<style>
* { box-sizing: border-box; margin: 0; padding: 0; }
body { font-family: Arial, sans-serif; background: #f5f3ff; display: flex; justify-content: center; align-items: center; min-height: 100vh; }
#game { display: flex; flex-direction: column; align-items: center; padding: 2rem 1rem; gap: 1rem; }
#title { font-size: 28px; font-weight: bold; color: #3C3489; }
#subtitle { font-size: 14px; color: #888; margin-top: -0.5rem; }
#status-box { background: #EEEDFE; border-radius: 999px; padding: 8px 24px; font-size: 15px; font-weight: bold; color: #3C3489; min-width: 200px; text-align: center; transition: background 0.3s; }
#status-box.win { background: #EAF3DE; color: #27500A; }
#status-box.lose { background: #FAECE7; color: #712B13; }
#status-box.draw { background: #FAEEDA; color: #633806; }
#board { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; }
.cell { width: 90px; height: 90px; border-radius: 18px; background: #EEEDFE; border: 2px solid #AFA9EC; display: flex; align-items: center; justify-content: center; font-size: 36px; cursor: pointer; transition: background 0.15s, transform 0.1s; user-select: none; }
.cell:hover:not(.taken) { background: #CECBF6; transform: scale(1.05); }
.cell.x-cell { background: #FBEAF0; border-color: #ED93B1; color: #993356; animation: pop 0.2s ease; }
.cell.o-cell { background: #E6F1FB; border-color: #85B7EB; color: #185FA5; animation: pop 0.2s ease; }
.cell.win-cell { background: #C0DD97; border-color: #639922; transform: scale(1.08); }
.cell.taken { cursor: default; }
@keyframes pop { 0%{transform:scale(0.7)} 70%{transform:scale(1.1)} 100%{transform:scale(1)} }
#scores { display: flex; gap: 16px; }
.score-card { background: #fff; border-radius: 12px; padding: 10px 20px; text-align: center; min-width: 80px; box-shadow: 0 1px 4px #0001; }
.score-label { font-size: 12px; color: #888; margin-bottom: 4px; }
.score-num { font-size: 22px; font-weight: bold; color: #333; }
#restart { border: 1.5px solid #AFA9EC; background: transparent; color: #534AB7; border-radius: 999px; padding: 8px 28px; font-size: 14px; cursor: pointer; transition: background 0.15s; }
#restart:hover { background: #EEEDFE; }
</style>
</head>
<body>
<div id="game">
  <div id="title">⭕ X · O ✕</div>
  <div id="subtitle">أنت = ✕  |  الكمبيوتر = ⭕</div>
  <div id="status-box">دورك! 🎯</div>
  <div id="scores">
    <div class="score-card"><div class="score-label">أنت ✕</div><div class="score-num" id="score-x">0</div></div>
    <div class="score-card"><div class="score-label">تعادل</div><div class="score-num" id="score-d">0</div></div>
    <div class="score-card"><div class="score-label">كمبيوتر ⭕</div><div class="score-num" id="score-o">0</div></div>
  </div>
  <div id="board">
    <div class="cell" data-i="0"></div><div class="cell" data-i="1"></div><div class="cell" data-i="2"></div>
    <div class="cell" data-i="3"></div><div class="cell" data-i="4"></div><div class="cell" data-i="5"></div>
    <div class="cell" data-i="6"></div><div class="cell" data-i="7"></div><div class="cell" data-i="8"></div>
  </div>
  <button id="restart">🔄 جولة جديدة</button>
</div>
<script>
const wins=[[0,1,2],[3,4,5],[6,7,8],[0,3,6],[1,4,7],[2,5,8],[0,4,8],[2,4,6]];
let board,active,scores={x:0,o:0,d:0};
const cells=document.querySelectorAll('.cell');
const status=document.getElementById('status-box');
function init(){board=Array(9).fill(null);active=true;cells.forEach(c=>{c.className='cell';c.textContent='';});status.className='';status.id='status-box';status.textContent='دورك! 🎯';}
function checkWin(b,p){return wins.find(w=>w.every(i=>b[i]===p))||null;}
function highlight(combo){combo.forEach(i=>cells[i].classList.add('win-cell'));}
function minimax(b,isMax){const xw=checkWin(b,'X'),ow=checkWin(b,'O');if(ow)return 10;if(xw)return -10;if(b.every(v=>v))return 0;let best=isMax?-Infinity:Infinity;for(let i=0;i<9;i++){if(!b[i]){b[i]=isMax?'O':'X';const v=minimax(b,!isMax);b[i]=null;best=isMax?Math.max(best,v):Math.min(best,v);}}return best;}
function aiMove(){let best=-Infinity,move=-1;for(let i=0;i<9;i++){if(!board[i]){board[i]='O';const v=minimax(board,false);board[i]=null;if(v>best){best=v;move=i;}}}return move;}
function endGame(msg,cls){status.textContent=msg;status.className=cls;active=false;}
function updateScores(){document.getElementById('score-x').textContent=scores.x;document.getElementById('score-o').textContent=scores.o;document.getElementById('score-d').textContent=scores.d;}
cells.forEach(cell=>{cell.addEventListener('click',()=>{const i=+cell.dataset.i;if(!active||board[i])return;board[i]='X';cell.textContent='✕';cell.classList.add('x-cell','taken');const xw=checkWin(board,'X');if(xw){highlight(xw);scores.x++;updateScores();endGame('🎉 يسطا فزت!','win');return;}if(board.every(v=>v)){scores.d++;updateScores();endGame('🤝 تعادل!','draw');return;}status.textContent='الكمبيوتر يفكر... 🤔';setTimeout(()=>{const ai=aiMove();if(ai===-1)return;board[ai]='O';cells[ai].textContent='⭕';cells[ai].classList.add('o-cell','taken');const ow=checkWin(board,'O');if(ow){highlight(ow);scores.o++;updateScores();endGame('😅 الكمبيوتر فاز!','lose');return;}if(board.every(v=>v)){scores.d++;updateScores();endGame('🤝 تعادل!','draw');return;}status.textContent='دورك! 🎯';},400);});});
document.getElementById('restart').addEventListener('click',init);
init();
</script>
</body>
</html>
