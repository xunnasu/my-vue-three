<!-- 点云渲染 -->
---
name: threejs-pointcloud
description: Three.js 点云渲染、PCD 加载、点云材质、性能优化、交互、动画
---

# Three.js 点云渲染

## PCD 文件加载

### 基础加载
```javascript
import { PCDLoader } from 'three/addons/loaders/PCDLoader.js'

const loader = new PCDLoader()
loader.load('/binary-00.pcd', (points) => {
  scene.add(points)
  
  // 设置点大小
  points.material.size = 0.1
  points.material.color.set(0x00ffff)
})
```

### 异步加载
```javascript
const loader = new PCDLoader()
const points = await loader.loadAsync('/binary-00.pcd')
scene.add(points)
```

### 加载管理器
```javascript
const manager = new THREE.LoadingManager()
manager.onLoad = () => console.log('点云加载完成')
const loader = new PCDLoader(manager)
```

## 点云材质

### 基础材质
```javascript
const material = new THREE.PointsMaterial({
  color: 0xffffff,
  size: 0.1,
  sizeAttenuation: true, // 近大远小
  transparent: true,
  opacity: 0.8,
  alphaTest: 0.1
})
```

### 顶点颜色
```javascript
const material = new THREE.PointsMaterial({
  size: 0.1,
  vertexColors: true, // 使用顶点颜色
  sizeAttenuation: true
})

// 设置顶点颜色
const colors = []
for (let i = 0; i < count; i++) {
  colors.push(Math.random(), Math.random(), Math.random())
}
geometry.setAttribute('color', new THREE.Float32BufferAttribute(colors, 3))
```

### 纹理点云
```javascript
const texture = new THREE.TextureLoader().load('/particle.png')
const material = new THREE.PointsMaterial({
  size: 0.5,
  map: texture,
  transparent: true,
  alphaTest: 0.01,
  blending: THREE.AdditiveBlending
})
```

## 从数据创建点云

### 从数组创建
```javascript
const positions = []
for (let i = 0; i < 1000; i++) {
  positions.push(
    Math.random() * 10 - 5,
    Math.random() * 10 - 5,
    Math.random() * 10 - 5
  )
}

const geometry = new THREE.BufferGeometry()
geometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3))

const material = new THREE.PointsMaterial({ size: 0.1, color: 0x00ff00 })
const points = new THREE.Points(geometry, material)
scene.add(points)
```

### 带颜色的点云
```javascript
const positions = []
const colors = []

for (let i = 0; i < 1000; i++) {
  positions.push(
    Math.random() * 10 - 5,
    Math.random() * 10 - 5,
    Math.random() * 10 - 5
  )
  
  colors.push(
    Math.random(),
    Math.random(),
    Math.random()
  )
}

const geometry = new THREE.BufferGeometry()
geometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3))
geometry.setAttribute('color', new THREE.Float32BufferAttribute(colors, 3))

const material = new THREE.PointsMaterial({
  size: 0.1,
  vertexColors: true
})

const points = new THREE.Points(geometry, material)
scene.add(points)
```

## 点云交互

### 拾取点
```javascript
const raycaster = new THREE.Raycaster()
const mouse = new THREE.Vector2()

function onMouseClick(event) {
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1
  
  raycaster.setFromCamera(mouse, camera)
  const intersects = raycaster.intersectObject(points)
  
  if (intersects.length > 0) {
    const index = intersects[0].index
    console.log('点击了点', index)
  }
}

window.addEventListener('click', onMouseClick)
```

### 高亮选中的点
```javascript
// 保存原始颜色
const originalColors = geometry.attributes.color.array.slice()

// 高亮点
function highlightPoint(index) {
  const colors = geometry.attributes.color.array
  colors[index * 3] = 1     // R
  colors[index * 3 + 1] = 0 // G
  colors[index * 3 + 2] = 0 // B
  geometry.attributes.color.needsUpdate = true
}

// 恢复颜色
function resetColors() {
  const colors = geometry.attributes.color.array
  for (let i = 0; i < colors.length; i++) {
    colors[i] = originalColors[i]
  }
  geometry.attributes.color.needsUpdate = true
}
```

## 性能优化

### LOD（Level of Detail）
```javascript
import { mergeGeometries } from 'three/addons/utils/BufferGeometryUtils.js'

// 创建不同细节级别的点云
const highDetail = createPointCloud(100000)
const medDetail = createPointCloud(50000)
const lowDetail = createPointCloud(10000)

const lod = new THREE.LOD()
lod.addLevel(highDetail, 0)
lod.addLevel(medDetail, 50)
lod.addLevel(lowDetail, 100)
scene.add(lod)
```

### 点云降采样
```javascript
function downsamplePointCloud(geometry, factor) {
  const positions = geometry.attributes.position.array
  const colors = geometry.attributes.color?.array
  
  const newPositions = []
  const newColors = colors ? [] : null
  
  for (let i = 0; i < positions.length; i += 3 * factor) {
    newPositions.push(
      positions[i],
      positions[i + 1],
      positions[i + 2]
    )
    
    if (colors) {
      newColors.push(
        colors[i],
        colors[i + 1],
        colors[i + 2]
      )
    }
  }
  
  const newGeometry = new THREE.BufferGeometry()
  newGeometry.setAttribute('position', new THREE.Float32BufferAttribute(newPositions, 3))
  
  if (newColors) {
    newGeometry.setAttribute('color', new THREE.Float32BufferAttribute(newColors, 3))
  }
  
  return newGeometry
}
```

### Frustum Culling
```javascript
points.frustumCulled = true // 默认开启
geometry.computeBoundingSphere() // 确保边界球正确
```

## 点云动画

### 旋转动画
```javascript
function animate() {
  requestAnimationFrame(animate)
  points.rotation.y += 0.001
  renderer.render(scene, camera)
}
```

### 粒子动画
```javascript
const velocities = []
for (let i = 0; i < count; i++) {
  velocities.push(
    (Math.random() - 0.5) * 0.01,
    (Math.random() - 0.5) * 0.01,
    (Math.random() - 0.5) * 0.01
  )
}

function animate() {
  requestAnimationFrame(animate)
  
  const positions = geometry.attributes.position.array
  for (let i = 0; i < positions.length; i += 3) {
    positions[i] += velocities[i]
    positions[i + 1] += velocities[i + 1]
    positions[i + 2] += velocities[i + 2]
  }
  
  geometry.attributes.position.needsUpdate = true
  renderer.render(scene, camera)
}
```

## Vue3 + Three.js 点云组件

### 完整组件结构
```vue
<template>
  <div ref="container" class="point-cloud-container"></div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'
import { PCDLoader } from 'three/addons/loaders/PCDLoader.js'

const container = ref(null)
let scene, camera, renderer, points, animationId

onMounted(async () => {
  // 场景
  scene = new THREE.Scene()
  
  // 相机
  camera = new THREE.PerspectiveCamera(75, 1, 0.1, 1000)
  camera.position.z = 5
  
  // 渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(container.value.clientWidth, container.value.clientHeight)
  container.value.appendChild(renderer.domElement)
  
  // 加载点云
  const loader = new PCDLoader()
  points = await loader.loadAsync('/binary-00.pcd')
  points.material.size = 0.1
  points.material.color.set(0x00ffff)
  scene.add(points)
  
  // 动画
  animate()
})

function animate() {
  animationId = requestAnimationFrame(animate)
  renderer.render(scene, camera)
}

onUnmounted(() => {
  cancelAnimationFrame(animationId)
  renderer?.dispose()
  points?.geometry?.dispose()
  points?.material?.dispose()
})
</script>

<style scoped>
.point-cloud-container {
  width: 100%;
  height: 500px;
}
</style>
```

## 常见问题

### 点云不显示
- 检查相机位置是否正确
- 检查点大小是否太小
- 检查材质颜色是否与背景相同
- 检查点云是否在相机视野内

### 性能问题
- 减少点的数量
- 使用 LOD
- 关闭不必要的功能（如阴影）
- 使用简单的材质

### 内存泄漏
- 组件卸载时 dispose 所有资源
- 清理事件监听器
- 清理动画帧

## 参考资源
- Three.js Points 文档
- PCD 文件格式规范
- 点云处理最佳实践