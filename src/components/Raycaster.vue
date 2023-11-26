<script setup>

import * as THREE from 'three';
import * as dat from 'dat.gui';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

// 光线投射用于鼠标与三维物体交互后的事件。通过一条射线打到目标物体上，实现交互。

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
camera.position.set(5, 5, 10);

// 控制器
const orbControls = new OrbitControls(camera, renderer.domElement);
orbControls.enableDamping = true;
orbControls.dampingFactor = 0.01;

const gui = new dat.GUI();

// 创建一个球几何体交互
const sphere = new THREE.Mesh(
    new THREE.SphereGeometry(1, 32, 32), // 半径，宽高分段
    new THREE.MeshBasicMaterial({ color: 0x00ff00 })
);
scene.add(sphere);

// 创建射线
const raycaster = new THREE.Raycaster();
// 二维向量
const mouse = new THREE.Vector2();

let clicked = false;
sphere.material._preColor = sphere.material.color.getHex();
window.addEventListener('click', (e) => {
    // 设置鼠标向量的x，y值，因为window 包括canvas 的event事件原点是屏幕左上角。正方向y是向下的。
    // 而three.js，原点也是左上角，但他的y轴正方向是向上的。即屏幕是位于笛卡尔坐标系第四象限。
    // 所以向量的位置需要转换。但
    mouse.x = e.clientX / window.innerWidth * 2 - 1;
    mouse.y = e.clientY / window.innerHeight * 2 - 1;
    // 通过相机和鼠标计算射线，向量，相机射线源头
    raycaster.setFromCamera(mouse, camera);

    // 指定可刺穿对象
    // const intersects = raycaster.intersectObjects([scene.children]);
    const intersects = raycaster.intersectObjects([sphere]);

    // 如果intersects刺穿了指定物体，改变颜色，在点击则变回去
    if (intersects.length > 0) {
        clicked = !clicked;
        console.log(intersects[0].object.material, clicked);
        // 备份前色
        intersects[0].object.material.color.set(clicked ? 0xff0000 : intersects[0].object._preColor);
    }
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
