import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { VRButton } from 'three/addons/webxr/VRButton.js';
import { FBXLoader } from 'three/addons/loaders/FBXLoader.js'; // 🔹 Cargador FBX

// Escena
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x222222);

// Cámara
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.set(0, 2, 6);

// Renderizador
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);
renderer.xr.enabled = true;
renderer.xr.setReferenceSpaceType('local');
document.body.appendChild(VRButton.createButton(renderer));

// Controles
const controls = new OrbitControls(camera, renderer.domElement);
controls.target.set(0, 1.5, 0);
controls.update();

// LUZ
const light = new THREE.HemisphereLight(0xffffff, 0x444444, 1.2);
scene.add(light);

// Material cuadriculado
const gridTexture = new THREE.TextureLoader().load('https://threejs.org/examples/textures/checker.png');
gridTexture.wrapS = gridTexture.wrapT = THREE.RepeatWrapping;
gridTexture.repeat.set(10, 10);
const basicMaterial = new THREE.MeshStandardMaterial({ map: gridTexture });


//Cube map
const path = 'CubeMap/';
				const format = '.png';
				const urls = [
					path + 'px' + format, path + 'nx' + format,
					path + 'py' + format, path + 'ny' + format,
					path + 'pz' + format, path + 'nz' + format
				];

				const reflectionCube = new THREE.CubeTextureLoader().load( urls );
                //scene = new THREE.Scene();
				scene.background = reflectionCube;


// Piso
const floor = new THREE.Mesh(new THREE.BoxGeometry(10, 0.1, 10), basicMaterial);
floor.position.y = 0;
scene.add(floor);

// Techo
const ceiling = new THREE.Mesh(new THREE.BoxGeometry(10, 0.1, 6   ), basicMaterial);
ceiling.position.y = 5;
ceiling.position.z = -2;
scene.add(ceiling);

// Paredes
function createWall(pos, rot) {
  const wall = new THREE.Mesh(new THREE.BoxGeometry(10, 5, 0.1), basicMaterial);
  wall.position.set(pos.x, pos.y, pos.z);
  wall.rotation.y = rot;
  scene.add(wall);
}
function createWall2(pos, rot) {
  const wall = new THREE.Mesh(new THREE.BoxGeometry(6, 5, 0.1), basicMaterial);
  wall.position.set(pos.x, pos.y, pos.z);
  wall.rotation.y = rot;
  scene.add(wall);
}
createWall({ x: 0, y: 2.5, z: -5 }, 0);               // Fondo
createWall2({ x: -5, y: 2.5, z: -2 }, Math.PI / 2);     // Izquierda
createWall2({ x: 5, y: 2.5, z: -2 }, Math.PI / 2);      // Derecha

// Plataforma interna
function createPlatformBlock2(x, y, z) {
  const block = new THREE.Mesh(new THREE.BoxGeometry(1, 1, 1), basicMaterial);
  block.position.set(x, y, z);
  scene.add(block);
}
createPlatformBlock2(4.5, 2.5, -4.5);

// Jugador (visual)
const player = new THREE.Mesh(
  new THREE.CylinderGeometry(0.5, 0.5, 1.8, 32),
  new THREE.MeshStandardMaterial({ color: 0x5555ff })
);
player.position.set(0, 0.9, 4.5);
scene.add(player);

// Raycaster y targets
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();
const targets = [];

// 🔢 Variables de puntaje y precisión
let score = 0;
let shotsFired = 0;
let hits = 0;

const scoreDisplay = document.getElementById('score');
const accuracyDisplay = document.getElementById('accuracy');

function updateHUD() {
  scoreDisplay.textContent = `Puntaje: ${score}`;
  const precision = shotsFired > 0 ? (hits / shotsFired * 100).toFixed(1) : 0;
  accuracyDisplay.textContent = `Precisión: ${precision}%`;
}

function createTarget() {
  const sphere = new THREE.Mesh(
    new THREE.SphereGeometry(0.3, 16, 16),
    new THREE.MeshStandardMaterial({ color: 0x00ffff })
  );

  let x, y, z;
  do {
    x = Math.random() * 8 - 4;
    y = Math.random() * 3 + 1.2;
    z = Math.random() * -4 - 1;
  } while (x > 4 && z < -2); // Evita zona del bloque interno

  sphere.position.set(x, y, z);
  scene.add(sphere);
  targets.push(sphere);
}
for (let i = 0; i < 5; i++) createTarget();

function shootRay() {
  shotsFired++;
  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObjects(targets);
  if (intersects.length > 0) {
    const hit = intersects[0].object;
    scene.remove(hit);
    targets.splice(targets.indexOf(hit), 1);
    score += 5;
    hits++;
    createTarget();
  }
  updateHUD();
}

// Mouse
window.addEventListener('mousedown', (event) => {
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
  shootRay();
});

// Gamepad
let gamepadIndex = null;
function checkGamepadInput() {
  const gamepads = navigator.getGamepads();
  if (!gamepads) return;

  for (let i = 0; i < gamepads.length; i++) {
    const gp = gamepads[i];
    if (gp && gp.connected) {
      if (gamepadIndex === null) gamepadIndex = i;
      if (gp.buttons[0].pressed) {
        mouse.x = 0;
        mouse.y = 0;
        shootRay();
      }
    }
  }
}

// 🔫 FBX Arma
const loader = new FBXLoader();
loader.load('pistol/source/glock_model.fbx', (fbx) => {
  fbx.scale.set(0.01, 0.01, 0.01); // Ajustar según el tamaño del modelo
  fbx.position.set(0.2, -0.2, -0.5); // Ajuste típico POV
  fbx.rotation.set(0, Math.PI, 0); // Corrige orientación del modelo
  camera.add(fbx); // Se fija a la cámara
  scene.add(camera); // Asegura que el modelo se renderice
});




// 🎯 Mira al centro
const crosshairGeometry = new THREE.RingGeometry(0.005, 0.01, 16);
const crosshairMaterial = new THREE.MeshBasicMaterial({ color: 0xffffff, side: THREE.DoubleSide });
const crosshair = new THREE.Mesh(crosshairGeometry, crosshairMaterial);
crosshair.position.z = -1;
camera.add(crosshair);

// Render loop
renderer.setAnimationLoop(() => {
  checkGamepadInput();
  renderer.render(scene, camera);
});


// Resize
window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
