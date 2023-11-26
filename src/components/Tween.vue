<script setup>

import * as THREE from 'three';
import * as dat from 'dat.gui';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import * as TWEEN from 'three/examples/jsm/libs/tween.module';

// 补间动画，即几何体从起点到终点的过度阶段。
// 推荐用独立库，gsap

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

const tween = new TWEEN.Tween(sphere.position);
tween.to({ x: 2 }, 2000);
// .onUpdate(() => {
//     console.log(sphere.position.x);
//     更新后的回调
// });

// 设置缓动函数，各个时间点速度不同
tween.easing(TWEEN.Easing.Quadratic.InOut);

// 链式动画，按照加载顺序依次执行
const tween2 = new TWEEN.Tween(sphere.position);
tween2.to({ y: 4 }, 2000);
// .onUpdate(() => {
//     console.log(sphere.position.y);
// });

tween.chain(tween2);

tween.onComplete(() => { });
tween.onStop(() => { });
tween.onStart(() => { });

tween2.start();
// yoyo倒回去，也会有delay
tween.repeat(Infinity).yoyo(true).delay(1000).start(); // 重复无限次

const events = {
    stop: function (params) {
        tween.stop();
    },
    start: function (params) {
        tween.start();
    }
};

gui.add(events, 'stop').name('暂停');
gui.add(events, 'start').name('开始');

function animate() {
    // 更新
    tween.update();
    // tween2.update();
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
