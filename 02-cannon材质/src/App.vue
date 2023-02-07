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

// 设置立方体材质
const boxMaterialCon = new CANNON.Material("boxMaterial");
boxMaterialCon.friction = 0.7;
boxMaterialCon.restitution = 1;

// 创建一个物理世界的平面
// const planeShape = new CANNON.Plane();
const planeShape = new CANNON.Box(new CANNON.Vec3(5, 0.1, 5));
// 创建一个刚体
const planeBody = new CANNON.Body({
  // mass: 0,
  shape: planeShape,
  position: new CANNON.Vec3(0, 0, 0),
  type: CANNON.Body.STATIC,
  material: boxMaterialCon,
});
planeBody.quaternion.setFromAxisAngle(new CANNON.Vec3(1, 0, 0), 0.1);

// 将刚体添加到物理世界
world.addBody(planeBody);

// 创建一个平面几何体
// const planeGeometry = new THREE.PlaneGeometry(10, 10);
const planeGeometry = new THREE.BoxGeometry(10, 0.2, 10);
// 创建一个平面材质
const planeMaterial = new THREE.MeshBasicMaterial({ color: 0xffff00 });
// 创建一个平面网格
const planeMesh = new THREE.Mesh(planeGeometry, planeMaterial);
// x轴旋转90度
planeMesh.rotation.x = 0.1;
// 将网格添加到3D场景
scene.add(planeMesh);

let phyMeshes = [];
let meshes = [];

// 创建物理立方体
const boxShape = new CANNON.Box(new CANNON.Vec3(0.5, 0.5, 0.5));

// 创建一个刚体
const boxBody = new CANNON.Body({
  shape: boxShape,
  position: new CANNON.Vec3(0, 5, 0),
  mass: 1,
  material: boxMaterialCon,
});
// 将刚体添加到物理世界
world.addBody(boxBody);
phyMeshes.push(boxBody);

// 创建立方体几何体
const boxGeometry = new THREE.BoxGeometry(1, 1, 1);
// 创建立方体材质
const boxMaterial = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
// 创建立方体网格
const boxMesh = new THREE.Mesh(boxGeometry, boxMaterial);
// 将网格添加到3D场景
scene.add(boxMesh);
meshes.push(boxMesh);

// 创建第二个物理立方体
// const boxShape2 = new CANNON.Box(new CANNON.Vec3(0.5, 0.5, 0.5));
// 设置立方体材质
const boxSlipperyMaterial = new CANNON.Material("boxSlipperyMaterial");
boxSlipperyMaterial.friction = 0;

// 创建一个刚体
const boxBody2 = new CANNON.Body({
  shape: boxShape,
  position: new CANNON.Vec3(1, 5, 0),
  mass: 1,
  material: boxSlipperyMaterial,
});
// 将刚体添加到物理世界
world.addBody(boxBody2);
phyMeshes.push(boxBody2);

// 创建立方体几何体
// const boxGeometry2 = new THREE.BoxGeometry(1, 1, 1);
// // 创建立方体材质
// const boxMaterial2 = new THREE.MeshBasicMaterial({ color: 0x0000ff });
// 创建立方体网格
const boxMesh2 = new THREE.Mesh(boxGeometry, boxMaterial);
// 将网格添加到3D场景
scene.add(boxMesh2);
meshes.push(boxMesh2);

// 创建第三个物理立方体
// const boxShape3 = new CANNON.Box(new CANNON.Vec3(0.5, 0.5, 0.5));
// 设置立方体材质
const boxBouncyMaterial = new CANNON.Material("boxBouncyMaterial");
// boxBouncyMaterial.friction = 0.1;
// boxBouncyMaterial.restitution = 1;

const boxBody3 = new CANNON.Body({
  shape: boxShape,
  position: new CANNON.Vec3(2, 5, 0),
  mass: 1,
  material: boxBouncyMaterial,
});
// 将刚体添加到物理世界
world.addBody(boxBody3);
phyMeshes.push(boxBody3);

// 创建立方体几何体
// const boxGeometry3 = new THREE.BoxGeometry(1, 1, 1);
// // 创建立方体材质
// const boxMaterial3 = new THREE.MeshBasicMaterial({ color: 0xff0000 });
// 创建立方体网格
const boxMesh3 = new THREE.Mesh(boxGeometry, boxMaterial);
// 将网格添加到3D场景
scene.add(boxMesh3);
meshes.push(boxMesh3);

// 定义接触材质
const material3toplane = new CANNON.ContactMaterial(
  boxMaterialCon,
  boxBouncyMaterial,
  {
    friction: 0,
    restitution: 0.1,
  }
);
// 将接触材质添加到物理世界
world.addContactMaterial(material3toplane);

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
