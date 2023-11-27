<script setup>

import * as THREE from 'three';
import * as dat from 'dat.gui';
import gsap from 'gsap';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader';

const props = defineProps({
    renderer: Object,
});
const { renderer } = props;

// 场景
const scene = new THREE.Scene();

// 相机
const camera = new THREE.PerspectiveCamera(
    45,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
);
camera.position.set(0, 0, 100);

// 控制器
const orbControls = new OrbitControls(camera, renderer.domElement);
orbControls.enableDamping = true;
// orbControls.dampingFactor = 0.01;

// gui
const gui = new dat.GUI();


function animate() {
    // orbControls.update();
    requestAnimationFrame(animate);
    renderer.render(scene, camera);
}

animate();

window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
});
</script>

<template></template>

<style scoped lang="less">
canvas {
    width: 100%;
}
</style>
