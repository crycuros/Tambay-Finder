<template>
  <canvas ref="canvas" class="three-canvas"></canvas>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import * as THREE from 'three';

const canvas = ref(null);
let scene, camera, renderer, globe, points;
let animationId;

onMounted(() => {
  init();
  animate();
});

onUnmounted(() => {
  if (animationId) cancelAnimationFrame(animationId);
  if (renderer) renderer.dispose();
});

const init = () => {
  scene = new THREE.Scene();
  
  camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
  camera.position.z = 3;

  renderer = new THREE.WebGLRenderer({ 
    canvas: canvas.value, 
    alpha: true, 
    antialias: true 
  });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

  const globeGeometry = new THREE.SphereGeometry(1, 64, 64);
  const globeMaterial = new THREE.MeshBasicMaterial({
    color: 0x6366f1,
    wireframe: true,
    transparent: true,
    opacity: 0.3
  });
  globe = new THREE.Mesh(globeGeometry, globeMaterial);
  scene.add(globe);

  const pointsGeometry = new THREE.BufferGeometry();
  const pointsCount = 300;
  const positions = new Float32Array(pointsCount * 3);
  
  for (let i = 0; i < pointsCount * 3; i += 3) {
    const radius = 1 + Math.random() * 0.5;
    const theta = Math.random() * Math.PI * 2;
    const phi = Math.acos(2 * Math.random() - 1);
    
    positions[i] = radius * Math.sin(phi) * Math.cos(theta);
    positions[i + 1] = radius * Math.sin(phi) * Math.sin(theta);
    positions[i + 2] = radius * Math.cos(phi);
  }
  
  pointsGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
  const pointsMaterial = new THREE.PointsMaterial({
    color: 0x6366f1,
    size: 0.02,
    transparent: true,
    opacity: 0.8
  });
  points = new THREE.Points(pointsGeometry, pointsMaterial);
  scene.add(points);

  window.addEventListener('resize', onResize);
};

const animate = () => {
  animationId = requestAnimationFrame(animate);
  
  if (globe) {
    globe.rotation.y += 0.002;
    globe.rotation.x += 0.001;
  }
  if (points) {
    points.rotation.y += 0.003;
    points.rotation.x += 0.001;
  }
  
  renderer.render(scene, camera);
};

const onResize = () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
};
</script>

<style scoped>
.three-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  pointer-events: none;
}
</style>