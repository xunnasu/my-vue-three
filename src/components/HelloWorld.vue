<!-- 这个组件实现了一个 旋转的彩色立方体 ：
1. 创建场景、相机、渲染器
2. 创建一个 2x2x2 的立方体，使用法线材质（彩虹色）
3. 每帧旋转立方体并渲染
4. 组件卸载时清理资源 -->
<template>
  <div class="hello">
    <h1>Vue3 + Vite + Three.js</h1>
    <div ref="canvasContainer" class="three-box"></div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'

const canvasContainer = ref(null) // ref 引用，指向模板中的 div 容器


let scene, camera, renderer, cube, animateId

onMounted(() => {
  // 1. 场景
  scene = new THREE.Scene()

  // 2. 透视相机 (模拟人眼视角)
  camera = new THREE.PerspectiveCamera(
    75, //视野角度
    canvasContainer.value.clientWidth / canvasContainer.value.clientHeight, //宽高比
    0.1, //近裁剪面（最近可见距离）
    1000 //远裁剪面（最远可见距离）
  )

  camera.position.z = 5 //相机向后移动 5 个单位 （将相机放在 z 轴上，距离原点 5个单位）

  // 3. 渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true }) //开启抗锯齿，让边缘更平滑
  // 设置渲染尺寸
  renderer.setSize(
    canvasContainer.value.clientWidth,
    canvasContainer.value.clientHeight
  )
  canvasContainer.value.appendChild(renderer.domElement) //将 canvas 元素添加到 DOM 中

  // 4. 立方体
  const geometry = new THREE.BoxGeometry(2, 2, 2) //立方体几何体：2x2x2 的立方体
  const material = new THREE.MeshNormalMaterial() //法线材质(根据面的朝向显示不同颜色)：法线材质（彩虹色）
  cube = new THREE.Mesh(geometry, material) // 网格对象：将几何体和材质组合起来
  cube.position.z = -5 // 将立方体向前移动 5 个单位 （将立方体放在 z 轴上，距离原点 5 个单位）
  scene.add(cube) // 添加到场景

  // 动画
  animate()
})

function animate() {
  animateId = requestAnimationFrame(animate) // 浏览器 API，每帧调用一次 animate 函数
  cube.rotation.x += 0.008 // 绕 x 轴旋转
  cube.rotation.y += 0.008 // 绕 y 轴旋转
  renderer.render(scene, camera) // 渲染场景，将相机视图渲染到 canvas 元素上
}

onUnmounted(() => {
  cancelAnimationFrame(animateId) //停止动画循环
  renderer?.dispose() //释放渲染器占用的资源
})
</script>

<style scoped>
.hello {
  text-align: center;
  padding: 20px;
}
.three-box {
  width: 100%;
  height: 500px;
  margin: 0 auto;
  background: #111;
}
</style>