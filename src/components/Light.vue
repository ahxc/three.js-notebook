<script setup>

import * as THREE from 'three';
import * as dat from 'dat.gui';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import * as TWEEN from 'three/examples/jsm/libs/tween.module';

// 光需要更高级的材质，光效果会更好，但也更耗性能，基础材质不会有光反射

// 材质要满足对光有反射效果
// 开启渲染器阴影计算。 renderer.shadowMap.enabled=true
// 光照投影，directionalLight.castshadow=true
// 物体投射阴影，sphere.castshadow=true
// 物体接受阴影，plane.receiveshadow=true

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

const material = new THREE.MeshPhysicalMaterial({ color: 0xffffff });
// 创建一个球几何体交互
const sphere = new THREE.Mesh(
    new THREE.SphereGeometry(0.5, 32, 32), // 半径，宽高分段
    material,
);
sphere.castShadow = true;
sphere.position.set(2, 2, 2);

// 创建一个平面反射光
const planeGeometry = new THREE.PlaneGeometry(10, 10);
const plane = new THREE.Mesh(planeGeometry, material);
plane.position.set(0, -1, 0);
plane.rotation.x = -Math.PI / 2; // 平面有正反，所以反向旋转。
plane.receiveShadow = true;

// 环境光
const ambientLight = new THREE.AmbientLight(0xffffff, 0.5); // 柔和的白光

// 点光源，萤火虫
const pointLight = new THREE.PointLight(0xffffff, 0.5); // 柔和的白光

// 加入球体
sphere.add(pointLight);

// 聚光灯
const spotLight = new THREE.SpotLight(0xffffff, 0.5);
spotLight.position.set(5, 5, 5);
spotLight.castShadow = true;
spotLight.shadow.radius = 10;// 模糊半径
spotLight.shadow.mapSize.set(4096, 4096);// 定义阴影像素密度
// 如果想要改变灯光方向创建一个新的Object3D对象  
const targetObject = sphere;// new THREE.Object3D();
// 设置Object3D的位置  
// targetObject.position.set(0, 0, 0);
// 将Object3D添加到场景中  
scene.add(targetObject);
// 设置SpotLight的目标为新创建的Object3D  
spotLight.target = targetObject;
spotLight.decay = 0.1; // 光线随距离增加的衰减两， 默认为2，容易看不到效果

// 平行光
const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
directionalLight.position.set(5, 5, 5);
directionalLight.castShadow = true;
directionalLight.shadow.mapSize.set(1024, 1024);// 定义阴影像素密度
directionalLight.shadow.radius = 5;// 模糊半径
// 设置平行光阴影的大小，光源相机任何属性修改后需更新矩阵
directionalLight.shadow.camera.near = 0.5; // 相机近点
directionalLight.shadow.camera.far = 32; // 相机远点，直接影响大小，最大为物体大小，不能超过物体。
directionalLight.shadow.camera.top = 5; // 平行光窗口大小
directionalLight.shadow.camera.bottom = -5;
directionalLight.shadow.camera.left = -5;
directionalLight.shadow.camera.right = 5;
directionalLight.shadow.camera.updateProjectionMatrix(); // 更新

const folder = gui.addFolder('阴影相机');
folder.add(directionalLight.shadow.camera, 'near').max(5).min(0).step(0.01).onChange(() => {
    directionalLight.shadow.camera.updateProjectionMatrix();
});
folder.add(directionalLight.shadow.camera, 'far').max(32).min(0).step(1).onChange(() => {
    directionalLight.shadow.camera.updateProjectionMatrix();
});

const folderSpotLight = gui.addFolder('聚光灯朝向');
folderSpotLight.add(sphere.position, 'x').max(10).min(0).step(1).onChange(() => {
    spotLight.shadow.camera.updateProjectionMatrix();
});
folderSpotLight.add(sphere.position, 'y').max(10).min(0).step(1).onChange(() => {
    spotLight.shadow.camera.updateProjectionMatrix();
});
folderSpotLight.add(sphere.position, 'z').max(10).min(0).step(1).onChange(() => {
    spotLight.shadow.camera.updateProjectionMatrix();
});
// 聚光灯扩散弧度，不应该超过九十度
folderSpotLight.add(spotLight, 'angle').max(Math.PI / 2).min(0).step(Math.PI / 90).name('角度').onChange(() => {
    spotLight.shadow.camera.updateProjectionMatrix();
});

scene
    // .add(directionalLight)
    // .add(spotLight)
    // .add(ambientLight)
    // .add(pointLight)
    .add(sphere)
    .add(plane);

// 开启阴影渲染
renderer.shadowMap.enabled = true;
// 真实物理光照模式
renderer.physicallyCorrecLights = true;

// gui.add(renderer, 'physicallyCorrecLights').onChange(() => {
//     animate();
// });

const clock = new THREE.Clock();
function animate() {
    const time = clock.getElapsedTime();
    sphere.position.x = Math.sin(time) * 3;
    sphere.position.z = Math.cos(time) * 3; // 数学的sin，cos曲线，绕x轴圆周远动
    sphere.position.y = Math.sin(time) * 3; // 数学的sin，cos曲线，绕x轴圆周远动
    orbControls.update();
    requestAnimationFrame(animate);
    renderer.render(scene, camera);
}

animate();

window.addEventListener('resize', () => {
    renderer.setSize(window.innerWidth, window.innerHeight);
    camera.aspect = window.innerWidth / window.innerHeight;
});
</script>

<template></template>

<style scoped lang="less"></style>
