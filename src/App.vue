<script setup>
import * as THREE from 'three';

import Base from './components/base.vue';
import Texture from './components/texture.vue';
import Fog from './components/fog.vue';
import GLTFLoader from './components/GLTFLoader.vue';
import RayCaster from './components/RayCaster.vue';
import Tween from './components/tween.vue';
import Light from './components/light.vue';
import PracticeChristmas from './components/PracticeChristmas.vue';

// 渲染器，渲染成一个canvas
const renderer = new THREE.WebGLRenderer({
  antialias: true // 抗锯齿
});
renderer.setSize(window.innerWidth, window.innerHeight); // 设置尺寸
document.body.appendChild(renderer.domElement)

</script>

<template>
  <!-- <Base :renderer="renderer" /> -->
  <!-- <Texture :renderer="renderer" /> -->
  <!-- <Fog :renderer="renderer" /> -->
  <!-- <GLTFLoader :renderer="renderer" /> -->
  <!-- <RayCaster :renderer="renderer" /> -->
  <!-- <Tween :renderer="renderer" /> -->
  <!-- <Light :renderer="renderer" /> -->
  <PracticeChristmas :renderer="renderer" />
</template>

<style scoped lang="less"></style>
