<template>
  <div></div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import * as CANNON from "cannon-es";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { Body } from "cannon-es";
// 初始化物理世界
const world = new CANNON.World();
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
camera.position.set(0, 0, 10);
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

// 设置碰撞组,数值要用2的幂
const GROUP1 = 1;
const GROUP2 = 2;
const GROUP3 = 4;
const GROUP4 = 8;

// 设置立方体材质
const boxMaterialCon = new CANNON.Material("boxMaterial");
boxMaterialCon.friction = 0;
boxMaterialCon.restitution = 0.1;

// 创建一个物理世界的平面
// const planeShape = new CANNON.Plane();
const planeShape = new CANNON.Box(new CANNON.Vec3(5, 0.1, 5));
// 创建一个刚体
const planeBody = new CANNON.Body({
  // mass: 0,
  shape: planeShape,
  position: new CANNON.Vec3(0, -0.1, 0),
  type: CANNON.Body.STATIC,
  material: boxMaterialCon,
  collisionFilterGroup: GROUP1,
  collisionFilterMask: GROUP1 | GROUP2 | GROUP3,
});
// planeBody.quaternion.setFromAxisAngle(new CANNON.Vec3(1, 0, 0), 0.1);

// 将刚体添加到物理世界
world.addBody(planeBody);

// 创建一个平面几何体
// const planeGeometry = new THREE.PlaneGeometry(10, 10);
const planeGeometry = new THREE.BoxGeometry(10, 0.2, 10);
// 创建一个平面材质
const planeMaterial = new THREE.MeshBasicMaterial({ color: 0xffff00 });
// 创建一个平面网格
const planeMesh = new THREE.Mesh(planeGeometry, planeMaterial);
planeMesh.position.y = -0.1;
// x轴旋转90度
// planeMesh.rotation.x = 0.1;
// 将网格添加到3D场景
scene.add(planeMesh);

// 创建坐标系
const axesHelper = new THREE.AxesHelper(5);
scene.add(axesHelper);
// 创建一个立方体几何体
let previousBody;
// 循环十个并排的立方体
// 创建一个立方体几何体
const boxGeometry = new THREE.BoxGeometry(1, 1, 1);
// 创建一个立方体材质
const boxMaterial = new THREE.MeshBasicMaterial({ color: 0x00ff00 });

const boxShape = new CANNON.Box(new CANNON.Vec3(0.5, 0.5, 0.5));
for (let i = 0; i < 10; i++) {
  // 创建一个立方体网格
  const boxMesh = new THREE.Mesh(boxGeometry, boxMaterial);
  boxMesh.position.x = i;
  boxMesh.position.y = 1;
  meshes.push(boxMesh);
  scene.add(boxMesh);

  // 创建一个刚体
  const boxBody = new CANNON.Body({
    mass: 1,
    shape: boxShape,
    position: new CANNON.Vec3(i + 0.5 * i, 1, 0),
    material: boxMaterialCon,
    collisionFilterGroup: GROUP2,
    collisionFilterMask: GROUP1 | GROUP2 | GROUP3,
  });
  // 将刚体添加到物理世界
  world.addBody(boxBody);
  phyMeshes.push(boxBody);
  // 将刚体添加到上一个刚体的约束中
  if (previousBody) {
    const constraint = new CANNON.LockConstraint(boxBody, previousBody);
    world.addConstraint(constraint);
    world.removeConstraint(constraint);
  }
  previousBody = boxBody;
}

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
