<!DOCTYPE html>
<html lang="fa">
<head>
<meta charset="UTF-8">
<title>🛰️ Multi Spacecraft Fleet</title>
<style>
html,body{margin:0;overflow:hidden;background:#00030a;font-family:system-ui}
canvas{display:block}
.hud{
  
  position:absolute;left:16px;top:16px;
  padding:12px 16px;border-radius:14px;
  background:rgba(255,255,255,0.06);
  backdrop-filter:blur(10px);
  color:#d9f3ff;
  font-size:13px;
}
</style>
</head>
<body>

<canvas id="c"></canvas>
<div class="hud" id="hud"></div>

<script>
const c=document.getElementById("c");
const ctx=c.getContext("2d");
let w,h;function r(){w=c.width=innerWidth;h=c.height=innerHeight}
r();addEventListener("resize",r);

// Earth
const earth={x:w/2,y:h/2,r:60,mu:9000};

// Fleet
const targetAlt = 180;
const fleet=[];
const colors=["#9ff0ff","#ffd29f","#c3ff9f","#ff9fd2"];

for(let i=0;i<4;i++){
  fleet.push({
    x: earth.x,
    y: earth.y - (targetAlt + i*12),
    vx: 2 + i*0.12,
    vy: 0,
    battery: 100,
    integral: 0,
    prevErr: 0,
    color: colors[i]
  });
}

// PID constants
const Kp=0.015, Ki=0.0001, Kd=0.04;

// Gravity
function gravity(ship){
  const dx=earth.x-ship.x;
  const dy=earth.y-ship.y;
  const d=Math.hypot(dx,dy);
  const f=earth.mu/(d*d);
  ship.vx+=f*dx/d;
  ship.vy+=f*dy/d;
}

// Autopilot PID
function autopilot(ship){
  const dx=ship.x-earth.x;
  const dy=ship.y-earth.y;
  const dist=Math.hypot(dx,dy);
  const alt=dist-earth.r;

  const err=targetAlt-alt;
  ship.integral+=err;
  const der=err-ship.prevErr;
  ship.prevErr=err;

  let out=Kp*err+Ki*ship.integral+Kd*der;
  out=Math.max(-0.05,Math.min(0.05,out));

  const tx=-dy/dist;
  const ty= dx/dist;

  ship.vx+=tx*out;
  ship.vy+=ty*out;

  ship.battery-=Math.abs(out)*0.6;
}

// Update
function update(){
  fleet.forEach(s=>{
    gravity(s);
    autopilot(s);
    s.x+=s.vx;
    s.y+=s.vy;
    s.battery=Math.max(0,s.battery);
  });
}

// Draw
function draw(){
  ctx.fillStyle="rgba(0,3,10,0.35)";
  ctx.fillRect(0,0,w,h);

  update();

  // Earth
  ctx.beginPath();
  ctx.arc(earth.x,earth.y,earth.r,0,Math.PI*2);
  ctx.fillStyle="#0b3d91";
  ctx.fill();

  // Fleet
  fleet.forEach((s,i)=>{
    ctx.beginPath();
    ctx.arc(s.x,s.y,4,0,Math.PI*2);
    ctx.fillStyle=s.color;
    ctx.fill();
  });

  // HUD
  document.getElementById("hud").innerHTML =
    `🛰️ Fleet Size: ${fleet.length}<br>`+
    fleet.map((s,i)=>`Ship ${i+1}: 🔋 ${s.battery.toFixed(0)}%`).join("<br>");

  requestAnimationFrame(draw);
}

draw();
</script>
</body>
</html>
