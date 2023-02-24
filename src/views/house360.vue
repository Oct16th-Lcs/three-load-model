/**
* @author Lcs
* @date 2023/02/24 09:44
* @Description: vr看房
*/
<template>
  <div class="container" ref="container"></div>
</template>

<script>
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader";
export default {
  data() {
    return {
      scene: null,
      camera: null,
      renderer: null,
      controls: null,
      anId: null
    }
  },
  mounted() {
    this.init();
  },

  destroyed() {
    this.destroyCom()
  },
  methods: {
    init() {
      const container = this.$refs.container;
       // 新建场景
      this.scene = new THREE.Scene();

      // 新建相机
      this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

      // 设置相机位置
      this.camera.position.z = 0.1;

      // 初始化渲染器
      this.renderer = new THREE.WebGLRenderer({
        antialias: true, // 抗锯齿
        alpha: true
      });
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      //设置设备像素比,避免HiDPI设备上绘图模糊
      this.renderer.setPixelRatio(window.devicePixelRatio);
      container.appendChild(this.renderer.domElement);

      if (this.controls) {
        this.controls.reset();
      }
      // 添加控制器
      this.controls = new OrbitControls(this.camera, container);
      // 滑动阻尼
      this.controls.enableDamping = true;
      // 自动旋转
      this.controls.autoRotate = true;
      // 自转速度
      this.controls.autoRotateSpeed = 1;
      // 鼠标缩放
      this.controls.enableZoom = true;

      window.addEventListener( 'resize', this.onWindowResize);

      // 添加立方体
      this.createCube();

      // 添加球体
      // this.createPanorama();

      // 添加光源
      const light = new THREE.AmbientLight( 0xffffff );
      this.scene.add(light);

      const animate = () => {
        this.anId = requestAnimationFrame(animate);
        this.camera.updateProjectionMatrix();
        this.controls.update();
        this.renderer.render(this.scene, this.camera);
      }
      animate();
    },

    // 创建立方体（一种实现方式， 也可以用球体加载一张全景图）
    createCube() {
      const geometry = new THREE.BoxGeometry(10, 10, 10);
      // 添加材质
      // const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
      // const cube = new THREE.Mesh(geometry, material);
      // this.scene.add(cube);

      // 引入6张图片贴在立方体上
      // const arr = ['4_l', '4_r', '4_u', '4_d', '4_b', '4_f'];
      const arr = ['4_r', '4_l', '4_u', '4_d', '4_f', '4_b'];
      let boxMaterials = [];

      arr.forEach(item => {
        // 纹理加载
        let url = require(`@/assets/images/living/${item}.jpg`);
        let loader  = new THREE.TextureLoader().load(url, (texture) => {
          this.$store.commit('SET_LOADING', false);
          // loader.matrixAutoUpdate = true;
          // 创建材质
          if (item === '4_u' || item === '4_d') {
            // loader.rotation = Math.PI;
            loader.center = new THREE.Vector2(0.5, 0.5);
            boxMaterials.push(new THREE.MeshBasicMaterial({ map: texture }));
          } else {
            boxMaterials.push(new THREE.MeshBasicMaterial({ map: texture }));
          }
        });
      });

      const cube = new THREE.Mesh(geometry, boxMaterials);
      cube.geometry.scale(1, 1, -1);
      this.scene.add(cube);
    },

    // 创建球体
    createPanorama() {
      // 创建一个球体几何对象
      const geometry = new THREE.SphereGeometry(5, 32, 32);
      // 引入hdr
      const loader = new RGBELoader();
      // let url = require('@/assets/images/hdr/Living.hdr');
      // console.log('url::', url);
      loader.load('./images/hdr/Living.hdr', (texture) => {
        this.$store.commit('SET_LOADING', false);
        const material = new THREE.MeshBasicMaterial({ map: texture, side: THREE.BackSide });
        const sphere = new THREE.Mesh(geometry, material);
        this.scene.add(sphere);
      });
    },

    // 销毁场景
    destroyCom() {
      cancelAnimationFrame(this.anId);
      // 强制失去上下文
      this.renderer.forceContextLoss();
      this.renderer.dispose();
      this.scene.clear();
      this.scene = null;
      this.camera = null;
      // this.controls = null;
      this.controls.reset();
      this.controls.dispose();
      // this.$refs.container.innerHTML = null;
      this.renderer = null;
    },

    onWindowResize() {
      if (this.camera) {
        this.camera.aspect = window.innerWidth / window.innerHeight;
        this.camera.updateProjectionMatrix();
        this.renderer.setSize( window.innerWidth, window.innerHeight );
      }
    },

  },
}
</script>

<style scoped>
.container {
  height: 100vh;
  width: 100vw;
  background-color: #f0f0f0;
}
</style>
