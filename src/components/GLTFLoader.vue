<script setup>

import * as THREE from 'three';
import * as dat from 'dat.gui';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader';
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader';

// GLTFLoader 以json(.gltf)或二进制(.glb)格式提供，json格式中，模型，贴图，bin数据是分开的，glb打包在一起。  
// 可传输多种类型文件，网格，材质，贴图，蒙版，骨架，动画，相机等等。

const props = defineProps({
    renderer: Object,
});
const { renderer } = props;

// 场景
const scene = new THREE.Scene();
const axesHelper = new THREE.AxesHelper(5); // 添加坐标辅助器，尺寸为5
scene.add(axesHelper);
scene.background = new THREE.Color(0x999999);

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

const gltfLoader = new GLTFLoader();

// 加载模型
// gltfLoader.load('./models/sci-fi_building_4_2k.glb', (model) => {
//     scene.add(model.scene);
// });


// 压缩模型的加载
const dracoLoader = new DRACOLoader();
// 设置解码器的提取路径，这里没有用到解码器。
dracoLoader.setDecoderPath('./draco/');
// 加载解码器
gltfLoader.setDRACOLoader(dracoLoader);
// 加载
gltfLoader.load('./models/sci-fi_building_4_2k.glb', (model) => {
    scene.add(model.scene);
});

// 没有光模型是黑色，加载环境球面贴图，四周散布光源。
const rgbeLoader = new RGBELoader();
rgbeLoader.load('./texture/gum_trees_2k.hdr', (texture) => {
    texture.mapping = THREE.EquirectangularReflectionMapping;
    scene.environment = texture; // 关键，环境贴图光源
    // scene.background = texture;
});


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
