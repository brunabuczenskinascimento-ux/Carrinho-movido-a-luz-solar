<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Super Carrinho Solar</title>
<style>
*{box-sizing:border-box;margin:0;padding:0}
body{background:#101522;color:white;font-family:Arial,sans-serif;overflow:hidden}
canvas{display:block;width:100vw;height:100dvh}
.overlay{position:fixed;inset:0;display:flex;flex-direction:column;justify-content:center;align-items:center;text-align:center;padding:24px;background:linear-gradient(#160f20aa,#080d19ed)}
h1{font-size:clamp(38px,9vw,65px);color:#ffbd54;text-shadow:0 4px 20px #ff5b24}
p{margin:9px;color:#eee}
button{border:0;border-radius:14px;padding:15px 30px;background:linear-gradient(135deg,#ff9d35,#e83229);color:white;font-size:19px;font-weight:bold;margin-top:18px}
.hidden{display:none!important}
#hud{position:fixed;top:18px;left:20px;right:20px;display:none;justify-content:space-between;font-weight:bold;text-shadow:0 2px 5px #000;pointer-events:none}
.stat{font-size:13px}.stat strong{display:block;font-size:32px;color:#ffc34f}
#pauseBtn{position:fixed;top:16px;right:18px;display:none;margin:0;padding:9px 14px;background:#ffffff33}
#controls{position:fixed;bottom:25px;left:0;right:0;display:none;justify-content:space-between;padding:0 24px}
.ctrl{width:70px;height:65px;padding:0;margin:0;background:#ffffff30;border:2px solid #ffffff80;font-size:30px;touch-action:none}
#hint{position:fixed;bottom:105px;left:0;right:0;text-align:center;color:#fff;text-shadow:0 2px 5px #000;font-size:13px;pointer-events:none}
</style>
</head>
<body>
<canvas id="game"></canvas>
<div id="start" class="overlay">
 <h1>SUPER CARRO</h1><p>☀️ CORRIDA SOLAR</p>
 <p>O carrinho acelera pela estrada!</p>
 <button id="play">JOGAR</button>
 <p>Use as setas para mudar de faixa.</p>
</div>
<div id="hud">
 <div class="stat">SCORE<strong id="score">0</strong></div>
 <div class="stat" style="text-align:center">VELOCIDADE<strong id="speed">80 KM/H</strong></div>
 <div class="stat" style="text-align:right">TEMPO<strong id="timer">60</strong></div>
</div>
<button id="pauseBtn">Ⅱ</button>
<div id="hint" class="hidden">Use ◀ ▶ para movimentar o carrinho</div>
<div id="controls">
 <button class="ctrl" id="left">◀</button><button class="ctrl" id="right">▶</button>
</div>
<div id="over" class="overlay hidden">
 <h1>FIM DE JOGO</h1><p>Sua pontuação:</p><h2 id="finalScore">0</h2>
 <button id="again">JOGAR NOVAMENTE</button>
</div>
<div id="pause" class="overlay hidden"><h1>PAUSADO</h1><button id="resume">CONTINUAR</button></div>
<script>
const canvas=document.getElementById('game'),ctx=canvas.getContext('2d');
let W=innerWidth,H=innerHeight,dpr=1,active=false,paused=false;
let xLane=0,score=0,timeLeft=60,carTravel=0,last=0,roadPulse=0;
function resize(){dpr=Math.min(devicePixelRatio||1,2);W=innerWidth;H=innerHeight;canvas.width=W*dpr;canvas.height=H*dpr;ctx.setTransform(dpr,0,0,dpr,0,0);draw()}
addEventListener('resize',resize);
function poly(points,color){ctx.fillStyle=color;ctx.beginPath();ctx.moveTo(...points[0]);for(const p of points.slice(1))ctx.lineTo(...p);ctx.closePath();ctx.fill()}
function drawCar(cx,cy,s){
 ctx.save();ctx.translate(cx,cy);ctx.scale(s,s);
 ctx.fillStyle='#090d16a8';ctx.beginPath();ctx.ellipse(0,3,53,18,0,0,Math.PI*2);ctx.fill();
 // rodas
 ctx.fillStyle='#11151c';ctx.fillRect(-48,-55,13,59);ctx.fillRect(35,-55,13,59);
 // carro visto de trás
 poly([[-39,-92],[39,-92],[49,-52],[47,7],[-47,7],[-49,-52]],'#e82e35');
 poly([[-31,-87],[31,-87],[25,-48],[-25,-48]],'#182b3b');
 poly([[-25,-81],[25,-81],[22,-53],[-22,-53]],'#29465a');
 ctx.fillStyle='#ffcf68';ctx.fillRect(-39,-24,12,9);ctx.fillRect(27,-24,12,9);
 ctx.fillStyle='#971e2b';ctx.fillRect(-38,-10,76,10);
 ctx.fillStyle='#fff0b1';ctx.fillRect(-11,-7,22,5);
 ctx.fillStyle='#171b22';ctx.fillRect(-43,4,86,7);
 // reflexo/luz traseira
 ctx.shadowColor='#ff5a38';ctx.shadowBlur=18;ctx.fillStyle='#ff6a48';ctx.fillRect(-42,-25,7,8);ctx.fillRect(35,-25,7,8);ctx.shadowBlur=0;
 ctx.restore();
}
function draw(){
 // céu e pôr do sol
 let sky=ctx.createLinearGradient(0,0,0,H*.55);sky.addColorStop(0,'#f06b54');sky.addColorStop(.65,'#ffad70');sky.addColorStop(1,'#f7c28d');
 ctx.fillStyle=sky;ctx.fillRect(0,0,W,H);
 const sx=W*.72,sy=H*.19;
 let glow=ctx.createRadialGradient(sx,sy,4,sx,sy,H*.27);glow.addColorStop(0,'#fff0b3aa');glow.addColorStop(1,'#ff9c5b00');ctx.fillStyle=glow;ctx.fillRect(0,0,W,H*.55);
 ctx.fillStyle='#ffe0a0';ctx.beginPath();ctx.arc(sx,sy,H*.055,0,Math.PI*2);ctx.fill();
 // prédios estáticos
 for(let side=0;side<2;side++)for(let i=0;i<8;i++){
  let t=i/8,base=H*(.39+t*.17),bw=W*(.075+t*.035),bh=H*(.13+t*.18),bx=side===0?i*W*.045:W-i*W*.045-bw;
  ctx.fillStyle=i%2?'#302b36':'#49333a';ctx.fillRect(bx,base-bh,bw,bh);
  ctx.fillStyle='#ffbd78';for(let wy=0;wy<3;wy++)for(let wx=0;wx<2;wx++)ctx.fillRect(bx+bw*(.18+wx*.42),base-bh*.78+wy*bh*.23,bw*.12,bh*.07);
 }
 // estrada fixa
 const hy=H*.34;
 poly([[W*.43,hy],[W*.57,hy],[W*1.08,H],[W*-.08,H]],'#30343a');
 // calçadas/acostamentos
 poly([[W*.43,hy],[W*.39,hy],[W*.04,H],[W*.17,H]],'#55464a');
 poly([[W*.57,hy],[W*.61,hy],[W*.83,H],[W*.96,H]],'#55464a');
 // bordas
 ctx.strokeStyle='#f3d9aa';ctx.lineWidth=3;ctx.beginPath();ctx.moveTo(W*.43,hy);ctx.lineTo(W*.17,H);ctx.moveTo(W*.57,hy);ctx.lineTo(W*.83,H);ctx.stroke();
 // linhas centrais estáticas
 ctx.strokeStyle='#f5e6c9';ctx.lineWidth=Math.max(3,W*.006);ctx.setLineDash([H*.035,H*.045]);ctx.beginPath();ctx.moveTo(W*.5,hy);ctx.lineTo(W*.5,H);ctx.stroke();ctx.setLineDash([]);
 // faixa lateral vermelha e branca
 for(let i=0;i<14;i++){let y=hy+(H-hy)*i/14, y2=hy+(H-hy)*(i+1)/14;let xl=W*(.39-(.39-.17)*i/14),xr=W*(.39-(.39-.17)*(i+1)/14);poly([[xl,y],[W*(.405-(.405-.185)*i/14),y],[W*(.405-(.405-.185)*(i+1)/14),y2],[xr,y2]],i%2?'#f4e7d1':'#b7353b')}
 // carro avança no quadro, estrada e cenário ficam imóveis
 const travel=Math.sin(carTravel)*.075;
 const carY=H*(.80-travel);
 const scale=Math.min(W,H)/500*1.55*(1-travel*.18);
 // feixe de luz
 let cx=W/2+xLane*W*.20;
 let light=ctx.createRadialGradient(cx,carY,2,cx,carY,Math.min(W,H)*.24);light.addColorStop(0,'#ff9d4540');light.addColorStop(1,'#ff9d4500');ctx.fillStyle=light;ctx.fillRect(cx-Math.min(W,H)*.24,carY-Math.min(W,H)*.24,Math.min(W,H)*.48,Math.min(W,H)*.48);
 drawCar(cx,carY,scale);
}
function setVisible(id,on){document.getElementById(id).classList.toggle('hidden',!on)}
function start(){score=0;timeLeft=60;carTravel=0;xLane=0;active=true;paused=false;setVisible('start',false);setVisible('over',false);setVisible('pause',false);setVisible('hint',true);document.getElementById('hud').style.display='flex';document.getElementById('controls').style.display='flex';document.getElementById('pauseBtn').style.display='block';last=performance.now()}
function finish(){active=false;document.getElementById('finalScore').textContent=Math.floor(score);setVisible('over',true);document.getElementById('hud').style.display='none';document.getElementById('controls').style.display='none';document.getElementById('pauseBtn').style.display='none';setVisible('hint',false)}
function move(n){if(active&&!paused)xLane=Math.max(-.82,Math.min(.82,xLane+n*.22))}
document.getElementById('play').onclick=start;document.getElementById('again').onclick=start;
document.getElementById('left').onpointerdown=e=>{e.preventDefault();move(-1)};
document.getElementById('right').onpointerdown=e=>{e.preventDefault();move(1)};
document.getElementById('pauseBtn').onclick=()=>{if(active){paused=true;setVisible('pause',true)}};
document.getElementById('resume').onclick=()=>{paused=false;setVisible('pause',false);last=performance.now()};
addEventListener('keydown',e=>{if(e.key==='ArrowLeft'||e.key==='a')move(-1);if(e.key==='ArrowRight'||e.key==='d')move(1)});
function loop(now){let dt=Math.min(now-last||16,50);last=now;if(active&&!paused){timeLeft-=dt/1000;carTravel+=dt*.0015;score+=dt*.012;document.getElementById('score').textContent=Math.floor(score);document.getElementById('timer').textContent=Math.ceil(timeLeft);document.getElementById('speed').textContent=Math.floor(80+Math.sin(carTravel*2)*8)+' KM/H';if(timeLeft<=0)finish()}draw();requestAnimationFrame(loop)}
resize();requestAnimationFrame(loop);
</script>
</body>
</html>
