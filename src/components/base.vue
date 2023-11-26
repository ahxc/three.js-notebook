<script setup>
/* three.js */
// 基础：点、线、面、几何体、材质、物体、场景、相机、渲染器、动画、控制器，渲染器等。
// 入门：PBR，Physically-Based Rendering基于物理的渲染。
// 环境贴图、凹凸贴图、置换贴图、放射光、,环境贴图、金属贴图、粗糙度贴图等。
// 进阶：着色器，控制GPU渲染，实现three.js底层封装原理，图形渲染原理，顶点着色器，片元着色器：绘制旗帜，云，水波。

// 画布元素绘制过程核心四步骤：1.构造几何体，2.创建材质。3.关联材质到几何体。4.加入场景。

/* GUI */
// 方法：1.addFolder加文件夹，2.add加单个控制器选项，3.加颜色控制addColor
// 属性：domElement

import * as THREE from 'three';
import * as dat from 'dat.gui';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

const props = defineProps({
    renderer: Object,
});
const { renderer } = props;


// 场景
const scene = new THREE.Scene();

// 相机
const camera = new THREE.PerspectiveCamera(
    45, // 视角大小
    window.innerWidth / window.innerHeight, // 宽高比
    0.1,// 近平面
    1000,//远平面
);

// box盒子几何体
const geometryBox = new THREE.BoxGeometry(1, 1, 1);

// 网格材质
const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
const pmaterial = new THREE.MeshBasicMaterial({ color: 0xff0000 });

// 关联材质，这个材质可以是六个组合成的array，因为立方体有六个面，实际也是六个面渲染的，只是共用一个材质。
const cube = new THREE.Mesh(geometryBox, material);
cube.position.x = 2;
const axesHelper = new THREE.AxesHelper(5); // 添加坐标辅助器，尺寸为5

// 创建父立方体
const parentCube = new THREE.Mesh(geometryBox, pmaterial);
parentCube.add(cube); // 父元素加入子元素，后续修改子元素位置将相对父元素，父元素则为世界坐标

// 缩放
cube.scale.set(2, 2, 2); // 三轴都放大两倍，相对父元素，父元素放大了，子元素更大

// 加入网格
// scene.add(cube).add(axesHelper);
scene.add(parentCube).add(axesHelper);

// 需要定义设置相机摆放位置，Z轴，屏幕向外，y轴垂直向上，x轴水平向右
camera.position.x = 5;
camera.position.y = 5;
camera.position.z = 5;
// 也可以直接接收 三维向量对象
camera.position.set(5, 5, 5);
camera.lookAt(0, 0, 0,); // 看向原点，默认

// 轨道控制器，定义后可通过监听元素鼠标事件控制元素轨道
// 此处是设置camera轨道，监听render的canvas事件控制
const orbControls = new OrbitControls(camera, renderer.domElement);
// 设置带阻尼的惯性，阻尼：摇荡或震荡系统中遇到的阻碍，使系统趋于稳定。越大系统静止速度越快
orbControls.enableDamping = true;
orbControls.dampingFactor = 0.01;
// orbControls.autoRotate = true;// 自动旋转

// 渲染函数动画
function animate() {
    orbControls.update(); // 启动控制轨道
    // 浏览器api，为动画请求动画帧，平衡帧率，避免cpu重绘和浪费。
    // animate，隐式函数参数，接收一个时间戳，timestamp请求动画帧的时刻。也可以换成封装的动画
    requestAnimationFrame(animate);
    // cube.rotation.x += 0.01; // 旋转 
    cube.rotation.x = Math.PI / 4;// 45度，pi等于180度，方向是右手法则，在父元素旋转的基础上再次旋转。
    renderer.render(scene, camera); // 渲染
}


// 使用dat.gui 提拱图形界面交互操作
const eventObj = {

    fullscreen: function fullscreen(params) {
        document.body.requestFullscreen();
    },

    exitFullscreen: function exitFullscreen(params) {
        document.exitFullscreen();
    }
};

const gui = new dat.GUI();
// gui中添加控制器和自定义事件，事件对象，属性名，修改别名。
gui.add(eventObj, 'fullscreen').name('全屏');
gui.add(eventObj, 'exitFullscreen').name('退出全屏');

// 添加cube位置的控制器，会自己判断类型。x属性。最小10。最大10。步骤1。别名。
gui.add(cube.position, 'x').min(-10).max(10).step(1).name('立方体x轴位置').onChange((val) => {
    console.log(`x改变回调，${val}`);
}).onFinishChange((val) => {
    console.log(`用户停止操作后回调，${val}`);
});
gui.add(cube.position, 'y').min(-10).max(10).step(1).name('立方体x轴位置');
gui.add(cube.position, 'z').min(-10).max(10).step(1).name('立方体x轴位置');

// 创建文件夹菜单，囊括控制器。
const folder = gui.addFolder('文件夹');
folder.add(cube.position, 'x').min(-10).max(10).step(1).name('立方体x轴位置');

// 如果添加的是原生布尔值属性，如线框wireframe，则添加一个复选框决定是否采用这个控制器。
// 注，js中所有的面都是三角形构成的，包括圆形。通过三个顶点一个面
folder.add(material, 'wireframe').name('父盒子线框模式');

// 添加颜色控制器
const colorParams = {
    cubeColor: "#ff0000" // 定义一个初始颜色
};
folder.addColor(colorParams, 'cubeColor').name('立方体颜色').onChange((val) => {
    // 设置颜色
    cube.material.color.set(val);
});

// 缓冲区几何体，可自定义顶点，面，法向量，颜色等，自定义缓存属性减少开销
const geometryBuffer = new THREE.BufferGeometry();

// 创建顶点数据，下面是两个三角拼成的一个正方形面。
// 顶点是有顺序的，面是有正反的，顶点逆时针构建的面为正面
const vertices = new Float32Array([
    -1.0, -1.0, 1.0,
    1.0, -1.0, 1.0,
    1.0, 1.0, 1.0,

    1.0, 1.0, 1.0, // 
    -1.0, 1.0, 1.0,
    -1.0, -1.0, 1.0//
]);

// 创建索引
const indices = new Uint16Array([0, 1, 2, 2, 3, 0]);
// 关联索引属性至几何体
geometryBuffer.setIndex(new THREE.BufferAttribute(indices, 1));

// 设置多个顶点组分割渲染，并分别关联材质。
// 顶点vertices索引，顶点总数，当前材质array的索引
geometryBuffer.addGroup(0, 3, 0);
geometryBuffer.addGroup(3, 3, 1);

// 创建buffer的网格材质
const materialBuffer_1 = new THREE.MeshBasicMaterial({
    color: 0x00ff00,
    wireframe: true, // 线条
});
const materialBuffer_2 = new THREE.MeshBasicMaterial({
    color: 0xffffff,
    wireframe: true, // 线条
});
// 加载多个材质到几何体
const plane = new THREE.Mesh(geometryBuffer, [materialBuffer_1, materialBuffer_2]);

// 设置buffer位置属性，itemSize尺寸设置为3
geometryBuffer.setAttribute('position', new THREE.BufferAttribute(vertices, 3));

// 加入场景
scene.add(plane);

animate();

// 监听窗口变化动态设置画布。元素大小不会改变
window.addEventListener('resize', () => {
    renderer.setSize(window.innerWidth, window.innerHeight);
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix(); // 更新相机矩阵

});
// renderer.domElement.requestFullscreen(); // 设置渲染器全屏，只能由用户操作触发，比如按钮点击再回调。注意会遮挡元素。
// document.body.requestFullscreen()
// document.exitFullscreen(); // 退出全屏





</script>

<template></template>

<style scoped lang="less">
canvas {
    display: block;
    position: fixed;
    left: 0;
    top: 0;
    width: 100vw;
    height: 100vh;
}
</style>
