<script setup lang="ts">
import { onMounted } from "vue";
import * as THREE from "three/webgpu";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
import * as BufferGeometryUtils from "three/addons/utils/BufferGeometryUtils.js";
import { VertexNormalsHelper } from "three/addons/helpers/VertexNormalsHelper.js";
import Stats from "three/addons/libs/stats.module.js";
import {
  texture,
  time,
  vec3,
  sin,
  uv,
  mod,
  positionWorld,
  positionLocal,
} from "three/tsl";

let scene: THREE.Scene,
  camera: THREE.PerspectiveCamera,
  renderer: THREE.WebGPURenderer;

let controls: OrbitControls;
let bushes: THREE.Object3D[] = [];
let dummy = new THREE.Object3D();
let instancedMesh: THREE.InstancedMesh;
let stats: Stats;
const rows = 10;
const cols = 10;

const createScene = () => {
  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(
    35,
    window.innerWidth / window.innerHeight,
    0.1,
    500
  );
  camera.position.set(0, 30, 30);

  renderer = new THREE.WebGPURenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.outputColorSpace = THREE.SRGBColorSpace;
  renderer.setAnimationLoop(render);
  renderer.setClearColor(new THREE.Color(0x000000));
  document.body.appendChild(renderer.domElement);

  stats = new Stats();
  document.body.appendChild(stats.dom);
};

const updateBushes = () => {
  // for (let bush of bushes) {
  //   const bushPos = new THREE.Vector3(0, 0, 1);
  //   const cameraPos = camera.position.clone().setY(0);
  //   const angle = bushPos.angleTo(cameraPos);
  //   bushPos.cross(cameraPos);
  //   const toAngle = bushPos.y < 0 ? -angle : angle;
  //   bush.rotation.y = toAngle;
  // }
  let index = 0;

  for (let i = 0; i < rows; i++) {
    for (let j = 0; j < cols; j++) {
      dummy.position.set(i * 3 - rows * 1.5, 0, j * 3 - cols * 1.5);
      dummy.updateMatrix();

      const bushPos = new THREE.Vector3(0, 0, 1);
      const cameraPos = camera.position.clone().setY(0);
      const angle = bushPos.angleTo(cameraPos);
      bushPos.cross(cameraPos);
      const toAngle = bushPos.y < 0 ? -angle : angle;
      dummy.rotation.y = toAngle;

      instancedMesh.setMatrixAt(index++, dummy.matrix);
    }
  }
};

const render = () => {
  updateBushes();
  controls.update();
  renderer.renderAsync(scene, camera);
  stats.update();
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

  controls.maxPolarAngle = Math.PI / 2;
  controls.minPolarAngle = Math.PI / 4;
  controls.enablePan = false;
  controls.enableDamping = true;
};

const createFog = () => {
  scene.fog = new THREE.Fog(new THREE.Color(0x000000), 0.1, 150);
};

const createHelper = () => {
  const helper = new THREE.GridHelper(200, 200);
  scene.add(helper);
};

const createBush = () => {
  const count = 120;
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
    //  plane.rotateX(Math.random() * 9999);
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
  const perlinTexture = new THREE.TextureLoader().load("assets/perlin.png");

  const material = new THREE.MeshMatcapNodeMaterial({
    matcap: matcap,
    alphaTest: 0.1,
    alphaMap,
  });

  const perlinUv = mod(positionWorld.xz.mul(0.2).add(sin(time.mul(0.05))), 1);
  const perlinColor = texture(perlinTexture, perlinUv)
    .sub(0.5)
    .mul(positionWorld.y);

  material.positionNode = positionLocal.add(
    vec3(perlinColor.r, 0, perlinColor.r)
  );

  let geometry = BufferGeometryUtils.mergeGeometries(planes, true);
  // const mesh = new THREE.Mesh(geometry, material);
  // mesh.position.copy(pos);

  // 查看normal
  // const vetexHelper = new VertexNormalsHelper(mesh, 0.2, 0x00ff00);
  // scene.add(vetexHelper);

  instancedMesh = new THREE.InstancedMesh(geometry, material, rows * cols);

  scene.add(instancedMesh);
};

const createBushes = () => {
  // for (let i = 0; i < 10; i++) {
  //   for (let j = 0; j < 10; j++) {
  //     const x = i * 3 - 15;
  //     const z = j * 3 - 15;
  //     const position = new THREE.Vector3(x, 0.5, z);
  //     const delay = Math.random() * 2;
  //     createBush(position, delay);
  //   }
  // }
  createBush();
};

onMounted(() => {
  createScene();
  createFog();
  createControls();
  createListener();
  createBushes();
  createHelper();
  render();
});
</script>

<template></template>

<style scoped></style>
