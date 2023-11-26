<script setup>
import * as THREE from 'three';

import Base from './components/base.vue';
import Texture from './components/texture.vue';
import Fog from './components/fog.vue';
import GLTFLoader from './components/GLTFLoader.vue';
import Raycaster from './components/raycaster.vue';
import Tween from './components/Tween.vue';


// 渲染器，渲染成一个canvas
const renderer = new THREE.WebGL1Renderer();
renderer.setSize(window.innerWidth, window.innerHeight); // 设置尺寸
document.body.appendChild(renderer.domElement); // body插入一个canvas

</script>

<template>
  <!-- <Base :renderer="renderer" /> -->
  <!-- <Texture :renderer="renderer" /> -->
  <!-- <Fog :renderer="renderer" /> -->
  <!-- <GLTFLoader :renderer="renderer" /> -->
  <!-- <Raycaster :renderer="renderer" /> -->
  <Tween :renderer="renderer" />
</template>

<style scoped lang="less"></style>
