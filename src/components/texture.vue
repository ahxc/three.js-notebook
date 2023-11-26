<script setup>
// 基础材质不会受到光照影响，如果需要，则需要用到贴图。

/* 贴图，模拟各种光照效果，但实际没有用到光。 */
// map 贴图，基础贴图
// specular map 高光贴图，镜面贴图。黑色表示不光滑无光，白色表示光滑反光。
// alpha map，透明贴图。蒙版覆盖设置基础贴图的透明度。黑色表示完全透明看不到基础贴图。白色完全不透明显示原图。0.5半透明。
// env map，环境贴图。贴图内部反射环境贴图里的内容。
// light map，光照贴图，和镜面贴图类似，黑色表示无光，白色表示有光照射。
// AO map，环境光遮蔽贴图。和光照贴图类似，表示物体各种夹角暗处的阴影部分，更细微，黑色表示阴影，白色表示无影响。

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
const axesHelper = new THREE.AxesHelper(5); // 添加坐标辅助器，尺寸为5
scene.add(axesHelper);

// 相机
const camera = new THREE.PerspectiveCamera(
    45,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
);

camera.position.set(0, 0, 100);
const orbControls = new OrbitControls(camera, renderer.domElement);
orbControls.enableDamping = true;
orbControls.dampingFactor = 0.01;

const gui = new dat.GUI();

// 纹理加载器
const textureLoader = new THREE.TextureLoader();
// 默认静态资源public下，如果在src目录下，则使用require('url')，不能共用贴图一个，得新开loader
const texture = textureLoader.load('./texture/6a05551b45c51296.png');
const lightMaptexture = textureLoader.load('./texture/6a05551b45c51296.png');
const aoMaptexture = textureLoader.load('./texture/6a05551b45c51296.png');

// 颜色空间，设置颜色显示标准。默认THREE.NoColorSpace
// texture.colorSpace = THREE.SRGBColorSpace; // srgb看起来更暗，按感知光强度均匀分布灰度条，中间值就是一半黑一半白。
// texture.colorSpace = THREE.LinearSRGBColorSpace; // 线性看起来更亮，发白。按发射光强度均匀分布灰度条，灰度部分较少，贴近真实颜色。

// 可以将六张贴图合成为一张。也专门用来加载hdr，exr格式。
// hdr环境贴图格式为HDRI，比传统rgb多了夜色，亮度，纹理，深度等，可以生成近乎逼真的效果。
const rgbeLoader = new RGBELoader();

rgbeLoader.load('./texture/gum_trees_2k.hdr', (envTexture) => {
    // 设置球形贴图，并可以四周观察。
    envTexture.mapping = THREE.EquirectangularReflectionMapping;
    // // 设置几何体的环境贴图，反射环境。
    // planeMaterial.envMap = envTexture;
    scene.background = envTexture; // 场景球面伪3d背景
    // scene.environment = envTexture; // 场景环境贴图，光源。
});

// 平面几何体，平面几何体通常面越少越好
// 宽，高，宽分块数，高分块数
const planeGeometry = new THREE.PlaneGeometry(5, 5);
// 网格基本材质贴图才会生效
const planeMaterial = new THREE.MeshBasicMaterial({
    color: 0xffffff,
    transparent: true,// 将png透明部分设置为透明而不是基础色。
    map: texture, // 基础贴图 // 一个texture不能设置个多个贴图，这里图方便。
    // specularMap: texture, // 镜面贴图
    // alphaMap: texture, // 透明贴图
    // envMap: backgroundTexture, // 环境贴图 用rgbeloader此处放入无效
    lightMap: lightMaptexture, // 光照贴图
    aoMap: aoMaptexture, // 环境光遮挡贴图
    aoMapIntensity: 1, // 环境遮挡贴图的强度
    reflectivity: 0.2, // 环境贴图的反射强度
});

const plane = new THREE.Mesh(planeGeometry, planeMaterial);
scene.add(plane);

// gui控制环境遮挡贴图程度
gui.add(planeMaterial, 'aoMapIntensity').min(0).max(1).name('环境贴图');
// gui调整颜色空间
gui.add(texture, 'colorSpace', {
    SRGB: THREE.SRGBColorSpace,
    linear: THREE.LinearSRGBColorSpace,
    noColorSpace: THREE.NoColorSpace, // 初始会自动匹配默认值
}).name('颜色空间').onChange(() => {
    texture.needsUpdate = true;
});

function animate() {
    orbControls.update();
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
