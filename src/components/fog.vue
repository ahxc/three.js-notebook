<script setup>

import * as THREE from 'three';
import * as dat from 'dat.gui';
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
orbControls.dampingFactor = 0.01;

const gui = new dat.GUI();

const boxGeometry = new THREE.BoxGeometry(1, 1, 100);
const boxMaterial = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
const box = new THREE.Mesh(boxGeometry, boxMaterial);

scene.add(box);
scene.background = new THREE.Color(0x999999);
const axesHelper = new THREE.AxesHelper(5); // 添加坐标辅助器，尺寸为5
scene.add(axesHelper);

// 添加雾
// 颜色，密度，距离摄像机多远生成
// scene.fog = new THREE.Fog(0x999999, 0.5, 50);
// 指数级雾，全景生成，效果更强。微调。
// 颜色，密度
scene.fog = new THREE.FogExp2(0x999999, 0.02);

function animate() {
    // orbControls.update();
    requestAnimationFrame(animate);
    renderer.render(scene, camera);
}

animate();

window.addEventListener('resize', () => {
    renderer.setSize(window.innerWidth, window.innerHeight);
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
});
</script>

<template></template>

<style scoped lang="less"></style>
