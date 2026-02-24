<template>
  <div class="point-cloud-container">
    <!-- Three.js 渲染容器 -->
    <div ref="container" class="three-container"></div>
    
    <!-- 点大小调整控件 -->
    <div class="controls-panel">
      <label>
        点大小:
        <input 
          type="range" 
          v-model.number="pointSize" 
          min="0.1" 
          max="5" 
          step="0.1"
          @input="updatePointSize"
        >
        <span>{{ pointSize }}</span>
      </label>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { PCDLoader } from 'three/addons/loaders/PCDLoader.js';

// 容器引用
const container = ref<HTMLDivElement | null>(null);
// 点大小响应式变量
const pointSize = ref<number>(1.0);

// Three.js 核心对象
let scene: THREE.Scene | null = null;
let camera: THREE.PerspectiveCamera | null = null;
let renderer: THREE.WebGLRenderer | null = null;
let controls: OrbitControls | null = null;
let pointCloud: THREE.Points | null = null;
let animationId: number | null = null;

// PCD 文件路径
const pcdFilePath = '/binary-00.pcd';

/**
 * 初始化 Three.js 场景
 */
const initThree = async () => {
  if (!container.value) return;

  // 1. 创建场景
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0x111111);

  // 2. 创建相机
  camera = new THREE.PerspectiveCamera(
    75,
    container.value.clientWidth / container.value.clientHeight,
    0.1,
    1000
  );
  camera.position.set(0, 0, 5);

  // 3. 创建渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(container.value.clientWidth, container.value.clientHeight);
  renderer.setPixelRatio(window.devicePixelRatio);
  container.value.appendChild(renderer.domElement);

  // 4. 添加光照
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.8);
  scene.add(ambientLight);

  const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
  directionalLight.position.set(5, 5, 5);
  scene.add(directionalLight);

  // 5. 添加轨道控制器
  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.dampingFactor = 0.05;
  controls.target.set(0, 0, 0);
  controls.update();

  try {
    // 6. 使用 PCDLoader 加载 PCD 文件
    const loader = new PCDLoader();
    const points = await loader.loadAsync(pcdFilePath);
    
    // 7. 创建点材质
    const material = new THREE.PointsMaterial({
      size: pointSize.value,
      // vertexColors: true, // 使用顶点颜色
      color:0x00ffff, // 固定青色
      sizeAttenuation: true, // 点大小随距离衰减
      transparent: true,
      alphaTest: 0.1
    });

    // 8. 创建点云对象并添加到场景
    pointCloud = new THREE.Points(points.geometry, material);
    scene.add(pointCloud);

    // 自动调整相机位置以适配点云
    const box = points.geometry.boundingBox;
    if (box) {
      const center = box.getCenter(new THREE.Vector3());
      const size = box.getSize(new THREE.Vector3());
      const maxDim = Math.max(size.x, size.y, size.z);
      const fov = camera.fov * (Math.PI / 180);
      let cameraZ = Math.abs(maxDim / (2 * Math.tan(fov / 2)));
      cameraZ *= 1.5; // 稍微拉远一点

      camera.position.copy(center);
      camera.position.z += cameraZ;
      controls.target.copy(center);
      controls.update();
    }

  } catch (error) {
    console.error('点云加载失败:', error);
    alert(`点云加载失败: ${(error as Error).message}`);
  }

  // 10. 窗口大小适配
  window.addEventListener('resize', onWindowResize);

  // 11. 启动渲染循环
  animate();
};

/**
 * 更新点大小
 */
const updatePointSize = () => {
  if (pointCloud && pointCloud.material instanceof THREE.PointsMaterial) {
    pointCloud.material.size = pointSize.value;
    pointCloud.material.needsUpdate = true;
  }
};

/**
 * 渲染循环
 */
const animate = () => {
  animationId = requestAnimationFrame(animate);
  controls?.update();
  renderer?.render(scene!, camera!);
};

/**
 * 窗口大小调整处理
 */
const onWindowResize = () => {
  if (!camera || !renderer || !container.value) return;
  
  camera.aspect = container.value.clientWidth / container.value.clientHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(container.value.clientWidth, container.value.clientHeight);
};

/**
 * 清理 Three.js 资源
 */
const disposeThree = () => {
  // 取消动画循环
  if (animationId) {
    cancelAnimationFrame(animationId);
    animationId = null;
  }

  // 移除事件监听
  window.removeEventListener('resize', onWindowResize);

  // 清理控制器
  if (controls) {
    controls.dispose();
    controls = null;
  }

  // 清理点云资源
  if (pointCloud) {
    if (pointCloud.geometry) {
      pointCloud.geometry.dispose();
    }
    if (pointCloud.material) {
      (pointCloud.material as THREE.Material).dispose();
    }
    scene?.remove(pointCloud);
    pointCloud = null;
  }

  // 清理渲染器
  if (renderer) {
    renderer.dispose();
    if (container.value && renderer.domElement) {
      container.value.removeChild(renderer.domElement);
    }
    renderer = null;
  }

  // 清空核心对象
  scene = null;
  camera = null;
};

// 组件挂载时初始化
onMounted(() => {
  initThree();
});

// 组件销毁时清理资源
onUnmounted(() => {
  disposeThree();
});

defineOptions({
  name: 'PointCloudViewer'
});
</script>

<style scoped>
.point-cloud-container {
  width: 100%;
  height: 100vh;
  position: relative;
}

.three-container {
  width: 100%;
  height: 100%;
}

.controls-panel {
  position: absolute;
  top: 20px;
  left: 20px;
  background: rgba(0, 0, 0, 0.7);
  padding: 15px;
  border-radius: 8px;
  color: white;
  z-index: 100;
}

.controls-panel label {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 14px;
}

.controls-panel input {
  width: 150px;
}

.controls-panel span {
  min-width: 35px;
  text-align: right;
}
</style>