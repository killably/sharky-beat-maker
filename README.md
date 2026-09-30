# sharky-beat-maker
from pathlib import Path
import zipfile

html = r'''<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Sharky Beat Maker</title>
<style>
*{box-sizing:border-box}body{margin:0;background:#090909;color:#fff;font-family:Arial,sans-serif;padding:20px}
.app{max-width:900px;margin:auto}.brand{font-size:28px;font-weight:900;margin-bottom:18px;letter-spacing:1px}
.brand b{color:#ff7a00}.bar,.seq{background:#151515;border:1px solid #292929;border-radius:16px;padding:16px;margin-bottom:15px}
.controls{display:flex;gap:9px;flex-wrap:wrap;align-items:center}
button{background:#242424;color:white;border:1px solid #3a3a3a;border-radius:9px;padding:10px 15px;font-weight:700;cursor:pointer}
button:hover{background:#303030}.play{background:#ff7a00;color:#111;border-color:#ff7a00}
input{background:#202020;color:white;border:1px solid #444;border-radius:7px;padding:8px;width:60px}
.grid{display:grid;grid-template-columns:85px repeat(16,1fr);gap:5px;overflow:auto}
.cell,.num{height:36px;border-radius:6px}.num{display:grid;place-items:center;color:#777;font-size:11px}
.name{display:flex;align-items:center;color:#bbb;font-size:13px;font-weight:bold}
.cell{background:#222;border:1px solid #303030;cursor:pointer}.cell.on{background:#ff7a00;border-color:#ff7a00}
.cell.now{outline:2px solid white}.hint{color:#777;font-size:12px;margin-top:12px}
@media(max-width:600px){.grid{grid-template-columns:70px repeat(16,34px)}}
</style>
</head>
<body>
<div class="app">
<div class="brand">SHARKY<b>•</b>BEAT MAKER</div>
<div class="bar"><div class="controls">
<button class="play" id="play">▶ Play</button><button id="stop">■ Stop</button>
<button id="clear">Clear</button>
<label>BPM <input id="bpm" type="number" value="120" min="50" max="200"></label>
</div></div>
<div class="seq"><h3>Drum Sequencer</h3><div class="grid" id="grid"></div>
<div class="hint">Tap the squares to make your first beat • 16 steps</div></div>
</div>
<script>
const sounds=[["Kick",100,"sine"],["Snare",180,"triangle"],["Hi-Hat",6000,"square"],["Clap",300,"triangle"]],N=16,pat=sounds.map(()=>Array(N).fill(0));
let ac,gain,timer,step=0,running=false;
function draw(){
 grid.innerHTML='<div></div>'+Array.from({length:N},(_,i)=>'<div class="num">'+(i+1)+'</div>').join('');
 sounds.forEach((x,r)=>{
  grid.innerHTML+='<div class="name">'+x[0]+'</div>'+
  Array.from({length:N},(_,s)=>'<div class="cell '+(pat[r][s]?'on':'')+'" data-r="'+r+'" data-s="'+s+'"></div>').join('')
 });
 document.querySelectorAll('.cell').forEach(c=>c.onclick=()=>{
  let r=c.dataset.r,s=c.dataset.s;pat[r][s]^=1;c.classList.toggle('on')
 });
}
function init(){
 if(!ac){ac=new (window.AudioContext||window.webkitAudioContext)();gain=ac.createGain();gain.gain.value=.7;gain.connect(ac.destination)}
 if(ac.state==='suspended')ac.resume();
}
function hit(r){
 init();let o=ac.createOscillator(),g=ac.createGain(),t=ac.currentTime;
 o.type=sounds[r][2];o.frequency.setValueAtTime(sounds[r][1],t);
 if(r===0)o.frequency.exponentialRampToValueAtTime(45,t+.12);
 g.gain.setValueAtTime(r<2?.5:.12,t);g.gain.exponentialRampToValueAtTime(.001,t+(r<2?.16:.06));
 o.connect(g);g.connect(gain);o.start(t);o.stop(t+.2);
}
function tick(){
 document.querySelectorAll('.now').forEach(x=>x.classList.remove('now'));
 document.querySelectorAll('[data-s="'+step+'"]').forEach(x=>x.classList.add('now'));
 pat.forEach((v,r)=>v[step]&&hit(r));step=(step+1)%N;
}
play.onclick=()=>{
 if(running)return;init();running=true;tick();
 timer=setInterval(tick,60000/(+bpm.value||120)/4);
};
stop.onclick=()=>{
 running=false;clearInterval(timer);
 document.querySelectorAll('.now').forEach(x=>x.classList.remove('now'));step=0;
};
clear.onclick=()=>{pat.forEach(r=>r.fill(0));draw()};
draw();
</script>
</body>
</html>'''

path=Path("/mnt/data/index.html")
path.write_text(html,encoding="utf-8")
zip_path=Path("/mnt/data/Sharky_Beat_Maker_GitHub.zip")
with zipfile.ZipFile(zip_path,"w",zipfile.ZIP_DEFLATED) as z:
    z.write(path,arcname="index.html")
print(path)
print(zip_path)
