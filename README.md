# Redline-Mobile
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no,viewport-fit=cover">
<meta name="theme-color" content="#080808">
<title>REDLINE ARENA MOBILE</title>
<style>
*{box-sizing:border-box;-webkit-tap-highlight-color:transparent}
html,body{margin:0;width:100%;height:100%;overflow:hidden;background:#080808;color:#fff;font-family:Arial,sans-serif;touch-action:none}
canvas{display:block;width:100%;height:100%}
#enter{position:fixed;z-index:10;top:50%;left:50%;transform:translate(-50%,-50%);background:#e51b35;color:#fff;border:3px solid white;border-radius:16px;padding:22px 38px;font-size:clamp(22px,6vw,32px);font-weight:900;box-shadow:0 0 28px #ff1735;white-space:nowrap}
#status{position:fixed;z-index:8;left:0;right:0;bottom:calc(10px + env(safe-area-inset-bottom));text-align:center;pointer-events:none;font-weight:bold;font-size:12px;text-shadow:0 0 8px #000;padding:0 12px}
#controls{display:none;position:fixed;inset:0;z-index:6;pointer-events:none}
#joystick{position:absolute;left:22px;bottom:calc(30px + env(safe-area-inset-bottom));width:132px;height:132px;border:2px solid #ffffff80;border-radius:50%;background:#111a;pointer-events:auto;touch-action:none}
#stick{position:absolute;left:50%;top:50%;width:54px;height:54px;transform:translate(-50%,-50%);border-radius:50%;background:#e51b35;border:2px solid #fff;box-shadow:0 0 15px #ff1735}
#jump,#interact{position:absolute;right:24px;border:2px solid #fff;color:#fff;background:#a51027e8;border-radius:50%;font-weight:900;pointer-events:auto;touch-action:none;box-shadow:0 0 14px #ff1735}
#jump{width:76px;height:76px;bottom:calc(38px + env(safe-area-inset-bottom));font-size:14px}
#interact{width:66px;height:66px;bottom:calc(126px + env(safe-area-inset-bottom));font-size:11px}
#lookHint{position:absolute;right:18px;top:18px;color:#ffffff99;font-size:11px;text-shadow:0 1px 5px #000;pointer-events:none}
#crosshair{position:absolute;left:50%;top:50%;width:10px;height:10px;transform:translate(-50%,-50%);border:1px solid #ff4055;border-radius:50%;pointer-events:none;opacity:.8}
@media (max-width:430px){#joystick{width:116px;height:116px;left:16px}#stick{width:48px;height:48px}#jump{right:18px;width:68px;height:68px}#interact{right:24px;width:60px;height:60px}}
</style>
</head>
<body>
<button id="enter">ENTER GAME</button>
<div id="controls">
 <div id="lookHint">DRAG RIGHT SIDE TO LOOK</div>
 <div id="crosshair"></div>
 <div id="joystick"><div id="stick"></div></div>
 <button id="interact">INTERACT</button>
 <button id="jump">JUMP</button>
</div>
<div id="status">REDLINE ARENA MOBILE — CONNECTING...</div>

<script type="module">
import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';
import {initializeApp} from 'https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js';
import {getFirestore,doc,getDoc,setDoc,serverTimestamp} from 'https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore.js';
import {getAuth,signInAnonymously} from 'https://www.gstatic.com/firebasejs/10.12.2/firebase-auth.js';

// SAME FIREBASE PROJECT — no second Firebase project is needed.
const firebaseConfig={
 apiKey:"AIzaSyCvB7N_XK2kLYhwn8xwHI5It4TS7dMjKQk",
 authDomain:"redline-arena.firebaseapp.com",
 projectId:"redline-arena",
 storageBucket:"redline-arena.firebasestorage.app",
 messagingSenderId:"94653220727",
 appId:"1:94653220727:web:f8d3f1184189bb451b4372",
 measurementId:"G-8SCPFK600M"
};
const firebaseApp=initializeApp(firebaseConfig);
const db=getFirestore(firebaseApp);
const auth=getAuth(firebaseApp);
let currentRoomCode="";
let firebaseReady=false;
function setStatus(message){document.getElementById("status").textContent=message}
const authReady=signInAnonymously(auth).then(()=>{
 firebaseReady=true;setStatus("REDLINE ARENA MOBILE — FIREBASE CONNECTED");
}).catch(error=>{
 console.error(error);setStatus("FIREBASE ERROR: "+error.message);
 throw error;
});

// THREE.JS SCENE
const scene=new THREE.Scene();
scene.background=new THREE.Color(0x090b12);
scene.fog=new THREE.Fog(0x090b12,22,55);
const playerRig=new THREE.Group();scene.add(playerRig);
const camera=new THREE.PerspectiveCamera(70,innerWidth/innerHeight,.1,100);
camera.position.set(0,1.6,0);playerRig.add(camera);
const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,1.75));
renderer.shadowMap.enabled=true;
renderer.shadowMap.type=THREE.PCFSoftShadowMap;
document.body.appendChild(renderer.domElement);
scene.add(new THREE.HemisphereLight(0xffffff,0x333344,2));
const light=new THREE.PointLight(0xff1735,35,30);light.position.set(0,4,-3);scene.add(light);

// ARENA
const floor=new THREE.Mesh(new THREE.PlaneGeometry(18,18),new THREE.MeshStandardMaterial({color:0x30343e,roughness:.9}));
floor.rotation.x=-Math.PI/2;floor.position.z=-2;floor.receiveShadow=true;scene.add(floor);
const grid=new THREE.GridHelper(18,36,0xff1735,0x555966);grid.position.set(0,.025,-2);scene.add(grid);
function wall(x,y,z,w,h,d){
 const m=new THREE.Mesh(new THREE.BoxGeometry(w,h,d),new THREE.MeshStandardMaterial({color:0x252832}));
 m.position.set(x,y,z);m.castShadow=true;m.receiveShadow=true;scene.add(m);
}
wall(0,2,-11,18,4,.3);wall(-9,2,-2,.3,4,18);wall(9,2,-2,.3,4,18);wall(0,4,-2,18,.3,18);

// HELPERS
function makeText(message,color="#ffffff"){
 const c=document.createElement("canvas");c.width=1024;c.height=512;
 const ctx=c.getContext("2d");ctx.fillStyle="#10131c";ctx.fillRect(0,0,1024,512);
 const lines=String(message).split("\n");ctx.fillStyle=color;ctx.textAlign="center";ctx.textBaseline="middle";
 const size=lines.length>4?38:55;ctx.font="bold "+size+"px Arial";
 const lineHeight=size*1.35,start=256-(lines.length-1)*lineHeight/2;
 lines.forEach((line,i)=>ctx.fillText(line,512,start+i*lineHeight,950));
 const texture=new THREE.CanvasTexture(c);texture.colorSpace=THREE.SRGBColorSpace;return texture;
}
function solidBox(w,h,d,color,x,y,z,parent){
 const mesh=new THREE.Mesh(new THREE.BoxGeometry(w,h,d),new THREE.MeshStandardMaterial({color,roughness:.8}));
 mesh.position.set(x,y,z);mesh.castShadow=true;mesh.receiveShadow=true;parent.add(mesh);return mesh;
}

// COMPUTER AND SHELF — no tables
const computer=new THREE.Group();computer.position.set(0,0,-7);scene.add(computer);
solidBox(2.9,.16,1.5,0x454650,0,.9,0,computer);
solidBox(2.92,.055,.06,0xff1735,0,.82,.72,computer);
for(const [x,z] of [[-1.2,-.58],[1.2,-.58],[-1.2,.58],[1.2,.58]])solidBox(.16,.82,.16,0x252832,x,.4,z,computer);
solidBox(2,1.45,.15,0x111827,0,2.05,-.22,computer);
const screenFace=new THREE.Mesh(new THREE.PlaneGeometry(1.85,1.3),new THREE.MeshBasicMaterial({map:makeText("REDLINE ARENA","#ff263f")}));
screenFace.position.set(0,2.05,-.135);computer.add(screenFace);
function screenMessage(message){
 if(screenFace.material.map)screenFace.material.map.dispose();
 screenFace.material.map=makeText(message,"#ff4055");screenFace.material.needsUpdate=true;
}
const menuButtons=[];
function clearMenu(){
 for(const button of menuButtons){
  computer.remove(button);
  button.traverse(child=>{
   if(child.geometry)child.geometry.dispose();
   if(child.material){if(child.material.map)child.material.map.dispose();child.material.dispose()}
  });
 }
 menuButtons.length=0;
}
function addComputerButton(label,x,y,action,width=.62){
 const button=solidBox(width,.22,.12,0xa51027,x,y,.03,computer);button.userData.action=action;
 const labelMesh=new THREE.Mesh(new THREE.PlaneGeometry(width-.04,.17),new THREE.MeshBasicMaterial({map:makeText(label)}));
 labelMesh.position.set(0,0,.066);button.add(labelMesh);menuButtons.push(button);return button;
}

// ROOMS
let gameMode="MENU",enteredRoomCode="";
async function createPrivateRoom(){
 try{
  setStatus("CONNECTING TO FIREBASE...");await authReady;
  for(let attempt=0;attempt<10;attempt++){
   const code=String(Math.floor(1000+Math.random()*9000)),roomRef=doc(db,"rooms",code);
   const existing=await getDoc(roomRef);if(existing.exists())continue;
   await setDoc(roomRef,{code,hostUid:auth.currentUser.uid,status:"open",createdAt:serverTimestamp()});
   currentRoomCode=code;showWaitingRoom();return;
  }
  setStatus("COULD NOT CREATE ROOM — TRY AGAIN");
 }catch(error){console.error(error);setStatus("ROOM ERROR ["+(error.code||"unknown")+"]: "+error.message)}
}
async function joinPrivateRoom(code){
 code=String(code||"").trim();
 if(!/^\d{4}$/.test(code)){setStatus("PLEASE ENTER A 4-DIGIT CODE");return}
 try{
  setStatus("CHECKING ROOM "+code+"...");await authReady;
  const roomSnap=await getDoc(doc(db,"rooms",code));
  if(!roomSnap.exists()){screenMessage("ROOM NOT FOUND\nCHECK YOUR CODE");setStatus("ROOM "+code+" DOES NOT EXIST");return}
  if(roomSnap.data().status!=="open"){screenMessage("ROOM CLOSED");setStatus("THIS ROOM IS NOT OPEN");return}
  currentRoomCode=code;showWaitingRoom();
 }catch(error){console.error(error);setStatus("JOIN ERROR ["+(error.code||"unknown")+"]: "+error.message)}
}
function showMainMenu(){
 clearMenu();gameMode="MENU";screenMessage("REDLINE ARENA\nSELECT AN OPTION");
 addComputerButton("CREATE",-.68,1.04,createPrivateRoom);
 addComputerButton("JOIN",0,1.04,showJoinKeyboard);
 addComputerButton("PRACTICE",.68,1.04,()=>{
  gameMode="PRACTICE";clearMenu();screenMessage("PRACTICE MODE\nHAVE FUN!");setStatus("PRACTICE MODE — MOVE AND JUMP!");
 });
}
function showJoinKeyboard(){
 clearMenu();enteredRoomCode="";screenMessage("ENTER ROOM CODE\n_ _ _ _");
 const digits=[["1",-.68,1.04],["2",0,1.04],["3",.68,1.04],["4",-.68,.76],["5",0,.76],["6",.68,.76],["7",-.68,.48],["8",0,.48],["9",.68,.48],["0",0,.20]];
 for(const [digit,x,y] of digits)addComputerButton(digit,x,y,()=>{
  if(enteredRoomCode.length>=4)return;enteredRoomCode+=digit;
  screenMessage("ENTER ROOM CODE\n"+enteredRoomCode.padEnd(4,"_").split("").join(" "));
 },.55);
 addComputerButton("ENTER",-.48,-.12,()=>{
  if(enteredRoomCode.length!==4){setStatus("ENTER ALL 4 DIGITS FIRST");return}
  joinPrivateRoom(enteredRoomCode);
 },.85);
 addComputerButton("CANCEL",.48,-.12,showMainMenu,.85);
}
function showWaitingRoom(){
 clearMenu();gameMode="PRIVATE_WAITING";
 const uid=auth.currentUser?auth.currentUser.uid:"NOT SIGNED IN";
 screenMessage("PRIVATE WAITING ROOM\nROOM CODE: "+currentRoomCode+"\nLEADERBOARD\nYOU: "+uid.slice(0,16)+"\nPLAYER SYNC NOT ENABLED");
 setStatus("PRIVATE ROOM "+currentRoomCode+" — WAITING ROOM");
 addComputerButton("BACK TO MENU",0,1.04,showMainMenu,1.25);
}
showMainMenu();computer.visible=false;

// GORILLA AVATAR — black with subtle red lava accents
const bodyMaterial=new THREE.MeshStandardMaterial({color:0x08090c,roughness:.82});
const faceMaterial=new THREE.MeshStandardMaterial({color:0x090a0d,roughness:.9});
const chestMaterial=new THREE.MeshStandardMaterial({color:0x111216,roughness:.85});
const lavaMaterial=new THREE.MeshStandardMaterial({color:0xff102c,emissive:0xff001b,emissiveIntensity:1.35,roughness:.4});
const darkEyeMaterial=new THREE.MeshStandardMaterial({color:0x050505,roughness:.3});
const torso=new THREE.Mesh(new THREE.SphereGeometry(.28,24,20),bodyMaterial);torso.scale.set(.86,.72,.68);torso.castShadow=true;scene.add(torso);
const head=new THREE.Mesh(new THREE.SphereGeometry(.17,24,20),bodyMaterial);head.castShadow=true;scene.add(head);
const face=new THREE.Mesh(new THREE.SphereGeometry(.115,20,16),faceMaterial);face.scale.set(1.05,.9,.4);scene.add(face);
for(let i=0;i<2;i++){const eye=new THREE.Mesh(new THREE.SphereGeometry(.025,16,12),darkEyeMaterial);eye.position.set(i===0?-.04:.04,.015,-.045);face.add(eye)}
const chest=new THREE.Mesh(new THREE.SphereGeometry(.13,20,16),chestMaterial);chest.scale.set(1,1,.18);scene.add(chest);
const chestLava=[];
for(let i=0;i<2;i++){const line=new THREE.Mesh(new THREE.BoxGeometry(.015,.11,.015),lavaMaterial);line.rotation.z=i===0?-.35:.35;scene.add(line);chestLava.push(line)}
// Mobile version has a visible avatar, but no disconnected VR controller arms.
const arms=[];
function makeArm(){
 const arm=new THREE.Mesh(new THREE.CylinderGeometry(.085,.105,1,16),bodyMaterial);
 const hand=new THREE.Mesh(new THREE.SphereGeometry(.09,16,12),bodyMaterial);
 const bands=[];
 for(let i=0;i<2;i++){const band=new THREE.Mesh(new THREE.TorusGeometry(.095,.012,8,24),lavaMaterial);band.rotation.x=Math.PI/2;scene.add(band);bands.push(band)}
 scene.add(arm);scene.add(hand);return {arm,hand,bands};
}
arms.push(makeArm(),makeArm());
function setArmVisible(a,v){a.arm.visible=v;a.hand.visible=v;a.bands.forEach(b=>b.visible=v)}
arms.forEach(a=>setArmVisible(a,true));
function updateSegment(mesh,start,end){
 const direction=new THREE.Vector3().subVectors(end,start),length=direction.length();
 mesh.position.copy(start).add(end).multiplyScalar(.5);mesh.scale.y=length;
 mesh.quaternion.setFromUnitVectors(new THREE.Vector3(0,1,0),direction.normalize());
}

// MOBILE INPUT
const controls=document.getElementById("controls"),joystick=document.getElementById("joystick"),stick=document.getElementById("stick");
const jumpButton=document.getElementById("jump"),interactButton=document.getElementById("interact");
let joyX=0,joyY=0,joyPointer=null,jumpHeld=false;
let yaw=0,pitch=0,lookPointer=null,lastLookX=0,lastLookY=0;
const lookSensitivity=.004;
function updateStick(e){
 const r=joystick.getBoundingClientRect(),cx=r.left+r.width/2,cy=r.top+r.height/2;
 let dx=e.clientX-cx,dy=e.clientY-cy,max=r.width*.34;
 const len=Math.hypot(dx,dy);if(len>max){dx=dx/len*max;dy=dy/len*max}
 joyX=dx/max;joyY=dy/max;
 stick.style.left=`calc(50% + ${dx}px)`;stick.style.top=`calc(50% + ${dy}px)`;
}
function resetStick(){joyX=0;joyY=0;joyPointer=null;stick.style.left="50%";stick.style.top="50%"}
joystick.addEventListener("pointerdown",e=>{e.preventDefault();joyPointer=e.pointerId;joystick.setPointerCapture(e.pointerId);updateStick(e)});
joystick.addEventListener("pointermove",e=>{if(e.pointerId===joyPointer){e.preventDefault();updateStick(e)}});
joystick.addEventListener("pointerup",e=>{if(e.pointerId===joyPointer)resetStick()});
joystick.addEventListener("pointercancel",resetStick);
jumpButton.addEventListener("pointerdown",e=>{e.preventDefault();jumpHeld=true});
["pointerup","pointercancel","pointerleave"].forEach(ev=>jumpButton.addEventListener(ev,()=>jumpHeld=false));

// Drag anywhere on the right half (except buttons) to rotate the camera.
renderer.domElement.addEventListener("pointerdown",e=>{
 if(!gameStarted||e.clientX<innerWidth*.38)return;
 lookPointer=e.pointerId;lastLookX=e.clientX;lastLookY=e.clientY;
 renderer.domElement.setPointerCapture(e.pointerId);
});
renderer.domElement.addEventListener("pointermove",e=>{
 if(e.pointerId!==lookPointer)return;
 const dx=e.clientX-lastLookX,dy=e.clientY-lastLookY;lastLookX=e.clientX;lastLookY=e.clientY;
 yaw-=dx*lookSensitivity;pitch-=dy*lookSensitivity;
 pitch=THREE.MathUtils.clamp(pitch,-1.15,1.15);
 camera.rotation.set(pitch,yaw,0,"YXZ");
});
function stopLook(e){if(e.pointerId===lookPointer)lookPointer=null}
renderer.domElement.addEventListener("pointerup",stopLook);
renderer.domElement.addEventListener("pointercancel",stopLook);

// TOUCH INTERACTION: aim the center crosshair at the computer and tap INTERACT.
const raycaster=new THREE.Raycaster();
function interact(){
 if(!gameStarted)return;
 const origin=new THREE.Vector3();camera.getWorldPosition(origin);
 const direction=new THREE.Vector3(0,0,-1).applyQuaternion(camera.getWorldQuaternion(new THREE.Quaternion()));
 raycaster.set(origin,direction);raycaster.far=5;
 const hits=raycaster.intersectObjects(menuButtons,false);
 if(hits.length&&hits[0].object.userData.action){hits[0].object.userData.action();return}
 setStatus("AIM THE CENTER DOT AT A COMPUTER BUTTON");
}
interactButton.addEventListener("pointerdown",e=>{e.preventDefault();interact()});

// PLAYER MOVEMENT
let gameStarted=false,verticalVelocity=0,grounded=true,jumpWasDown=false;
const gravity=9.8,jumpPower=4.2,walkSpeed=3.0;
function movePlayer(delta){
 const forward=new THREE.Vector3(0,0,-1).applyAxisAngle(new THREE.Vector3(0,1,0),yaw);
 const right=new THREE.Vector3(1,0,0).applyAxisAngle(new THREE.Vector3(0,1,0),yaw);
 const dead=.12;
 if(Math.abs(joyX)>dead||Math.abs(joyY)>dead){
  playerRig.position.addScaledVector(forward,-joyY*walkSpeed*delta);
  playerRig.position.addScaledVector(right,joyX*walkSpeed*delta);
 }
 if(jumpHeld&&!jumpWasDown&&grounded){verticalVelocity=jumpPower;grounded=false}
 jumpWasDown=jumpHeld;
 verticalVelocity-=gravity*delta;playerRig.position.y+=verticalVelocity*delta;
 if(playerRig.position.y<=0){playerRig.position.y=0;verticalVelocity=0;grounded=true}
 // Keep the player inside the arena.
 playerRig.position.x=THREE.MathUtils.clamp(playerRig.position.x,-8.2,8.2);
 playerRig.position.z=THREE.MathUtils.clamp(playerRig.position.z,-10.1,6.1);
}
const headPosition=new THREE.Vector3(),handPosition=new THREE.Vector3(),shoulder=new THREE.Vector3();
let playerHeight=1.6;
function updateBody(){
 camera.position.y=playerHeight;
 camera.getWorldPosition(headPosition);
 torso.position.copy(headPosition);torso.position.y-=playerHeight*.40;torso.scale.set(.86,.72*(playerHeight/1.6),.68);
 head.position.copy(headPosition);head.position.y-=.13;
 face.position.copy(head.position);face.position.y+=.005;face.position.z-=.095;
 chest.position.copy(torso.position);chest.position.z-=.18;
 chestLava[0].position.set(torso.position.x-.035,torso.position.y+.01,torso.position.z-.205);
 chestLava[1].position.set(torso.position.x+.035,torso.position.y-.01,torso.position.z-.205);
 // Short regular gorilla arms, with subtle motion based on movement input.
 for(let i=0;i<2;i++){
  const side=i===0?-1:1;
  shoulder.copy(headPosition);shoulder.x+=side*.19;shoulder.y-=playerHeight*.22;shoulder.z+=.035;
  handPosition.set(headPosition.x+side*.48,headPosition.y-.48+Math.sin(performance.now()*.006+(i*Math.PI))*.035,headPosition.z-.16);
  updateSegment(arms[i].arm,shoulder,handPosition);arms[i].hand.position.copy(handPosition);
  const dir=new THREE.Vector3().subVectors(handPosition,shoulder),len=dir.length();dir.normalize();
  arms[i].bands.forEach((band,b)=>{band.position.copy(shoulder).addScaledVector(dir,len*(b===0?.32:.7));band.quaternion.copy(arms[i].arm.quaternion);band.rotateX(Math.PI/2)});
 }
}

// ENTER GAME
document.getElementById("enter").addEventListener("click",async()=>{
 gameStarted=true;document.getElementById("enter").style.display="none";controls.style.display="block";
 computer.visible=true;showMainMenu();playerRig.position.set(0,0,0);yaw=0;pitch=0;camera.rotation.set(0,0,0);
 setStatus("REDLINE ARENA MOBILE — USE JOYSTICK TO MOVE");
 try{await authReady}catch(error){console.error(error)}
});

// LOOP
let lastTime=performance.now(),nearComputer=false;
renderer.setAnimationLoop(()=>{
 const now=performance.now(),delta=Math.min((now-lastTime)/1000,.05);lastTime=now;
 if(gameStarted){
  movePlayer(delta);updateBody();
  const target=new THREE.Vector3();camera.getWorldPosition(target);
  const comp=new THREE.Vector3();computer.getWorldPosition(comp);comp.y+=1.5;
  const near=target.distanceTo(comp)<3.5;
  if(near&&!nearComputer)setStatus("COMPUTER NEARBY — AIM AT A BUTTON AND TAP INTERACT");
  if(!near&&nearComputer)setStatus("WALK TO THE COMPUTER");
  nearComputer=near;
 }
 renderer.render(scene,camera);
});
window.addEventListener("resize",()=>{
 camera.aspect=innerWidth/innerHeight;camera.updateProjectionMatrix();
 renderer.setSize(innerWidth,innerHeight);
});
authReady.catch(error=>console.error("Firebase is not ready:",error));
</script>
</body>
</html>
