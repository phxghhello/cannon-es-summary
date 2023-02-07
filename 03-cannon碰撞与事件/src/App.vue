<template>
  <div></div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import * as CANNON from "cannon-es";

// 初始化物理世界
const world = new CANNON.World();

// 设置物理世界允许休眠
world.allowSleep = true;
// 设置重力
world.gravity.set(0, -9.82, 0);

// 初始化3D场景
const scene = new THREE.Scene();
// 初始化相机
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(0, 5, 10);

// 初始化渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 初始化控制器
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;

let phyMeshes = [];
let meshes = [];

// 创建物理立方体
const boxShape = new CANNON.Box(new CANNON.Vec3(0.5, 0.5, 0.5));

// 创建一个刚体
const boxBody = new CANNON.Body({
  shape: boxShape,
  position: new CANNON.Vec3(-2, 5, 0),
  mass: 1,
  material: boxMaterialCon,
  collisionFilterGroup: GROUP1,
  collisionFilterMask: GROUP1 | GROUP2 | GROUP3 | GROUP4,
});
// 设置立方体允许休眠
boxBody.allowSleep = true;
// 设置立方体如果速度小于0.1,则休眠
boxBody.sleepSpeedLimit = 0.5;
// 设置立方体如果速度小于0.1之后,1秒后进入休眠状态
boxBody.sleepTimeLimit = 1;
// 将刚体添加到物理世界
world.addBody(boxBody);
phyMeshes.push(boxBody);

// 监听休眠事件
boxBody.addEventListener("sleepy", (e) => {
  console.log("即将进入休眠", e);
});
boxBody.addEventListener("sleep", (e) => {
  console.log("进入休眠", e);
});

boxBody.addEventListener("collide", (e) => {
  console.log("碰撞了", e);
  let impactStrength = e.contact.getImpactVelocityAlongNormal();
  console.log("impactStrength", impactStrength);
});

// 创建立方体几何体
const boxGeometry = new THREE.BoxGeometry(1, 1, 1);
// 创建立方体材质
const boxMaterial = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
// 创建立方体网格
const boxMesh = new THREE.Mesh(boxGeometry, boxMaterial);
// 将网格添加到3D场景
scene.add(boxMesh);
meshes.push(boxMesh);

// 创建物理球
const sphereShape = new CANNON.Sphere(0.5);
// 创建一个刚体
const sphereBody = new CANNON.Body({
  shape: sphereShape,
  position: new CANNON.Vec3(0, 0.5, 0),
  mass: 1,
  material: boxMaterialCon,
  collisionFilterGroup: GROUP2,
  collisionFilterMask: GROUP1 | GROUP4,
});
// 将刚体添加到物理世界
world.addBody(sphereBody);
phyMeshes.push(sphereBody);

// 创建球几何体
const sphereGeometry = new THREE.SphereGeometry(0.5, 32, 32);
// 创建球材质
const sphereMaterial = new THREE.MeshBasicMaterial({ color: 0x0000ff });
// 创建球网格
const sphereMesh = new THREE.Mesh(sphereGeometry, sphereMaterial);
// 将网格添加到3D场景
scene.add(sphereMesh);
meshes.push(sphereMesh);

// 创建物理圆柱体
const cylinderShape = new CANNON.Cylinder(0.5, 0.5, 1, 32);
// 创建一个刚体
const cylinderBody = new CANNON.Body({
  shape: cylinderShape,
  position: new CANNON.Vec3(2, 0.5, 0),
  mass: 1,
  material: boxMaterialCon,
  collisionFilterGroup: GROUP3,
  collisionFilterMask: GROUP1 | GROUP4,
});
// 将刚体添加到物理世界
world.addBody(cylinderBody);
phyMeshes.push(cylinderBody);

// 创建圆柱体几何体
const cylinderGeometry = new THREE.CylinderGeometry(0.5, 0.5, 1, 32);
// 创建圆柱体材质
const cylinderMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000 });
// 创建圆柱体网格
const cylinderMesh = new THREE.Mesh(cylinderGeometry, cylinderMaterial);
// 将网格添加到3D场景
scene.add(cylinderMesh);
meshes.push(cylinderMesh);

// 设置立方体的初始速度
// boxBody.velocity.set(2, 0, 0);

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
