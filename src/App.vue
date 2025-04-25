<script setup lang="ts">
import * as THREE from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
import * as BufferGeometryUtils from "three/addons/utils/BufferGeometryUtils.js";
import { VertexNormalsHelper } from "three/addons/helpers/VertexNormalsHelper.js";

let scene: THREE.Scene,
  camera: THREE.PerspectiveCamera,
  renderer: THREE.WebGLRenderer;

let controls: OrbitControls;
let bushes: THREE.Object3D[] = [];

const createScene = () => {
  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(
    35,
    window.innerWidth / window.innerHeight,
    0.1,
    100
  );
  camera.position.set(0, 0, 10);
  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.outputColorSpace = THREE.SRGBColorSpace;
  // renderer.toneMapping = THREE.ACESFilmicToneMapping;

  document.body.appendChild(renderer.domElement);
};

const updateBushes = () => {
  for (let bush of bushes) {
    bush.lookAt(camera.position.x, 0, camera.position.z);
  }
};

const render = () => {
  // updateBushes();
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

const createFog = () => {
  const fog = new THREE.Fog(new THREE.Color(0x000000), 0.01, 50);
  scene.fog = fog;
};

const createHelper = () => {
  const helper = new THREE.GridHelper(100, 100);
  scene.add(helper);
};

const createBush = () => {
  const count = 180;
  const planes: THREE.PlaneGeometry[] = [];
  for (let i = 0; i < count; i++) {
    const plane = new THREE.PlaneGeometry(1, 1);
    planes.push(plane);

    const spherical = new THREE.Spherical(
      1 - Math.pow(Math.random(), 3),
      Math.PI * 2 * Math.random(),
      Math.PI * Math.random()
    );
    const position = new THREE.Vector3().setFromSpherical(spherical);
    plane.rotateX(Math.random() * 9999);
    plane.rotateY(0);
    plane.rotateZ(Math.random() * 9999);
    const y = position.y < 0 ? 0 : position.y;
    plane.translate(position.x, y, position.z);

    // 重新计算normal
    const normal = position.clone().normalize();

    const normalArray = new Float32Array(12);
    for (let i = 0; i < 4; i++) {
      const i3 = i * 3;
      const position = new THREE.Vector3(
        plane.attributes.position.array[i3],
        plane.attributes.position.array[i3 + 1],
        plane.attributes.position.array[i3 + 2]
      );

      const mixedNormal = position.lerp(normal, 0.4);
      normalArray[i3] = mixedNormal.x;
      normalArray[i3 + 1] = mixedNormal.y;
      normalArray[i3 + 2] = mixedNormal.z;
    }

    plane.setAttribute("normal", new THREE.BufferAttribute(normalArray, 3));
  }

  const matcap = new THREE.TextureLoader().load("assets/matcap.png");
  matcap.colorSpace = THREE.SRGBColorSpace;

  const alphaMap = new THREE.TextureLoader().load("assets/alpha.png");

  const material = new THREE.MeshMatcapMaterial({
    matcap: matcap,
    alphaTest: 0.01,
    alphaMap,
    // side: THREE.DoubleSide,
  });

  let geometry = BufferGeometryUtils.mergeGeometries(planes, true);
  const mesh = new THREE.Mesh(geometry, material);

  // 查看normal
  // const vetexHelper = new VertexNormalsHelper(mesh, 0.2, 0x00ff00);
  // scene.add(vetexHelper);
  bushes.push(mesh);
  scene.add(mesh);
};

createScene();
createFog();
createControls();
createListener();
createBush();
createHelper();
render();
</script>

<template></template>

<style scoped></style>
