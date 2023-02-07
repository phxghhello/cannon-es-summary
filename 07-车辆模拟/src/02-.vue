<template>
  <div></div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import * as CANNON from "cannon-es";
// 初始化物理世界
const world = new CANNON.World();
// 设置重力
world.gravity.set(0, -9.82, 0);

// Sweep and prune broadphase
world.broadphase = new CANNON.SAPBroadphase(world);

// Disable friction by default
world.defaultContactMaterial.friction = 0;

// 初始化3D场景
const scene = new THREE.Scene();
// 初始化相机
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(0, 10, 20);
camera.lookAt(0, 0, 0);

// 初始化渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 初始化控制器
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;

let phyMeshes = [];
let meshes = [];

// 创建地面
// const groundShape = new CANNON.Plane();
const groundPhyMaterial = new CANNON.Material("ground");
const groundShape = new CANNON.Box(new CANNON.Vec3(50, 0.2, 50));
const groundBody = new CANNON.Body({
  mass: 0,
  shape: groundShape,
  material: groundPhyMaterial,
});
// groundBody.quaternion.setFromAxisAngle(new CANNON.Vec3(1, 0, 0), -Math.PI / 2);
world.addBody(groundBody);
phyMeshes.push(groundBody);

const groundMesh = new THREE.Mesh(
  // new THREE.PlaneGeometry(100, 100),
  new THREE.BoxGeometry(100, 0.4, 100),
  new THREE.MeshBasicMaterial({
    color: 0x666666,
  })
);
// groundMesh.rotation.x = -Math.PI / 2;
scene.add(groundMesh);
meshes.push(groundMesh);

// 创建车身
const chassisShape = new CANNON.Box(new CANNON.Vec3(2, 0.5, 1));
const chassisBody = new CANNON.Body({
  mass: 150,
  shape: chassisShape,
});
chassisBody.position.set(0, 14, 0);
world.addBody(chassisBody);
phyMeshes.push(chassisBody);

// three.js的车身
const chassisMesh = new THREE.Mesh(
  new THREE.BoxGeometry(4, 1, 2),
  new THREE.MeshBasicMaterial({
    color: 0x660066,
  })
);
scene.add(chassisMesh);
meshes.push(chassisMesh);

// 创建拥有悬架的车辆
const vehicle = new CANNON.RaycastVehicle({
  chassisBody,
});

const wheelOptions = {
  // 轮子半径
  radius: 1,
  // 本地坐标系中的轮子位置
  // directionLocal: new CANNON.Vec3(0, -1, 0),
  suspensionStiffness: 30, // 悬架的刚度
  suspensionRestLength: 0.3, // 悬架的休息长度
  // frictionSlip: 1.4, // 滑动摩擦力
  dampingRelaxation: 2.3, // 振动
  dampingCompression: 4.4, // 压缩
  maxSuspensionForce: 100000, // 最大悬架力
  // rollInfluence: 0.1, // 滚动影响
  axleLocal: new CANNON.Vec3(0, 0, 1), // 本地坐标系中的轴
  chassisConnectionPointLocal: new CANNON.Vec3(-1, 0, 1), // 本地坐标系中的车身连接点
  maxSuspensionTravel: 0.2, // 最大悬架行程
  // customSlidingRotationalSpeed: -30,
  // useCustomSlidingRotationalSpeed: true,
};

vehicle.addWheel({
  ...wheelOptions,
  chassisConnectionPointLocal: new CANNON.Vec3(-1, 0, 1),
});

vehicle.addWheel({
  ...wheelOptions,
  chassisConnectionPointLocal: new CANNON.Vec3(-1, 0, -1),
});

vehicle.addWheel({
  ...wheelOptions,
  chassisConnectionPointLocal: new CANNON.Vec3(1, 0, 1),
});

vehicle.addWheel({
  ...wheelOptions,
  chassisConnectionPointLocal: new CANNON.Vec3(1, 0, -1),
});

vehicle.addToWorld(world);

const wheelBodies = [];

// 创建轮子
// 创建车轮形状
const wheelShape = new CANNON.Cylinder(0.5, 0.5, 0.2, 20);
// const wheelShape = new CANNON.Sphere(0.5);
// three.js的车轮
const wheelMaterial = new THREE.MeshBasicMaterial({
  color: 0x660000,
  wireframe: true,
});
// const wheelGeometry = new THREE.SphereGeometry(0.5, 8, 8);
const wheelGeometry = new THREE.CylinderGeometry(0.5, 0.5, 0.2, 20);

const wheelPhyMaterial = new CANNON.Material("wheel");

const wheel_ground = new CANNON.ContactMaterial(
  wheelPhyMaterial,
  groundPhyMaterial,
  {
    friction: 0,
    restitution: 0,
    contactEquationStiffness: 1000,
  }
);
world.addContactMaterial(wheel_ground);

for (let i = 0; i < vehicle.wheelInfos.length; i++) {
  const wheelBody = new CANNON.Body({
    mass: 0,
    // shape: wheelShape,
    // type: CANNON.Body.KINEMATIC,
    // collisionFilterGroup: 0, // turn off collisions
    material: wheelPhyMaterial,
  });
  wheelBody.type = CANNON.Body.KINEMATIC;
  wheelBody.collisionFilterGroup = 0;
  // wheelBody.addShape(wheelShape);
  wheelBodies.push(wheelBody);
  // world.addBody(wheelBody);
  phyMeshes.push(wheelBody);

  const wheelMesh = new THREE.Mesh(wheelGeometry, wheelMaterial);
  wheelMesh.rotation.x = -Math.PI / 2;
  const wheelObj = new THREE.Object3D();
  wheelObj.add(wheelMesh);
  scene.add(wheelObj);
  meshes.push(wheelObj);
}

world.addEventListener("postStep", () => {
  // 更新车轮的位置
  wheelBodies.forEach((wheelBody, index) => {
    vehicle.updateWheelTransform(index);
    wheelBody.position.copy(vehicle.wheelInfos[index].worldTransform.position);
    wheelBody.quaternion.copy(
      vehicle.wheelInfos[index].worldTransform.quaternion
    );
  });
});

// 设置完毕之后，将车子添加到世界中
// vehicle.addToWorld(world);

// console.log(vehicle);
// console.log(world);

// 控制车子
window.addEventListener("keydown", (event) => {
  // console.log(event.key);
  if (event.key === "w") {
    // setWheelForce(力，轮子编号)
    vehicle.applyEngineForce(1000, 0);
    vehicle.applyEngineForce(1000, 1);
    // vehicle.applyEngineForce(10000, 2);
    // vehicle.applyEngineForce(10000, 3);
  }
  if (event.key === "s") {
    vehicle.applyEngineForce(-1000, 0);
    vehicle.applyEngineForce(-1000, 1);
  }
  if (event.key == "a") {
    vehicle.setSteeringValue(Math.PI / 4, 2);
    vehicle.setSteeringValue(Math.PI / 4, 3);
  }
  if (event.key == "d") {
    vehicle.setSteeringValue(-Math.PI / 4, 2);
    vehicle.setSteeringValue(-Math.PI / 4, 3);
  }
});
window.addEventListener("keyup", (event) => {
  if (event.key === "w") {
    vehicle.applyEngineForce(0, 0);
    vehicle.applyEngineForce(0, 1);
  }
  if (event.key === "s") {
    vehicle.applyEngineForce(0, 0);
    vehicle.applyEngineForce(0, 1);
  }
  if (event.key == "a") {
    vehicle.setSteeringValue(0, 2);
    vehicle.setSteeringValue(0, 3);
  }
  if (event.key == "d") {
    vehicle.setSteeringValue(0, 2);
    vehicle.setSteeringValue(0, 3);
  }
});

// 渲染
let clock = new THREE.Clock();
function animate() {
  let delta = clock.getDelta();
  world.step(1 / 60, delta);

  for (let i = 0; i < phyMeshes.length; i++) {
    meshes[i].position.copy(phyMeshes[i].position);
    meshes[i].quaternion.copy(phyMeshes[i].quaternion);
  }

  controls.update();
  renderer.render(scene, camera);
  requestAnimationFrame(animate);
}

animate();
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

canvas {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
}
</style>
