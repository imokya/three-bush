<script setup lang="ts">
import * as THREE from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";

let scene: THREE.Scene,
  camera: THREE.PerspectiveCamera,
  renderer: THREE.WebGLRenderer;

let controls: OrbitControls;

const createObject = () => {
  const mat = new THREE.MeshBasicMaterial({ color: 0xff0000 });
  const geo = new THREE.BoxGeometry(1, 1, 1);
  const mesh = new THREE.Mesh(geo, mat);
  scene.add(mesh);
};

const createScene = () => {
  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(
    35,
    window.innerWidth / window.innerHeight,
    0.1,
    500
  );
  camera.position.set(0, 0, 10);
  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);
};

const render = () => {
  controls.update();
  renderer.render(scene, camera);
  requestAnimationFrame(render);
};

const onResize = () => {
  const w = window.innerWidth;
  const h = window.innerHeight;
  renderer.setSize(w, h);
  camera.aspect = w / h;
  camera.updateProjectionMatrix();
};

const createListener = () => {
  window.addEventListener("resize", onResize);
};

const createControls = () => {
  controls = new OrbitControls(camera, renderer.domElement);
};

createScene();
createObject();
createControls();
createListener();
render();
</script>

<template></template>

<style scoped></style>
