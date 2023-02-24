/**
* @author Lcs
* @date 2023/02/21 10:45
* @Description: 3D模型
*/

<template>
  <div class="model-wrap">
    <div class="three-wrap"
         v-loading="$store.getters.loading"
         element-loading-text="数据加载中..."
         element-loading-spinner="el-icon-loading"
         element-loading-background="rgba(0, 0, 0, 0.5)"
    >
      <house360 v-if="isShow360House" />
      <div id="container" @click="onMouseClick" v-else></div>
    </div>
    <!--切换盒子-->
    <div class="toggle-wrap">
      <el-button
          type="primary"
          class="toggle-btn"
          :class="{active: curActive === index }"
          v-for="(item, index) in modelList"
          :key="index"
          @click="toggleModel(item, index)"
      >{{ item.title }}</el-button>
    </div>
  </div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"; //轨道控制器
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js"; //模型加载器
import house360 from "@/views/house360";
export default {
  components: {
    house360
  },
  data() {
    return {
      modelList: [
        {
          id: '01',
          modelUrl: "model/武士刀.glb",
          title: "武士刀",
          bg: require('@/assets/images/panorama/pImg21.png'),
          cameraPosition: { // 相机位置
            x: -10,
            y: 5,
            z: 10
          },
          modePosition: { // 模型位置
            x: 2,
            y: 0,
            z: -5
          },
          modeScale: { // 模型缩放
            x: 20,
            y: 20,
            z: 20
          }
        },
        {
          id: '02',
          modelUrl: "model/左轮手枪.glb",
          title: "左轮手枪",
          bg: require('@/assets/images/panorama/pImg20.png'),
          cameraPosition: { // 相机位置
            x: 0,
            y: 50,
            z: 100
          },
          modePosition: { // 模型位置
            x: 0,
            y: 0,
            z: 0
          },
          modeScale: { // 模型缩放
            x: 0.1,
            y: 0.1,
            z: 0.1
          }
        },
        {
          id: '03',
          modelUrl: "model/dji-mini-2.glb",
          title: "大疆 Mini 2",
          bg: require('@/assets/images/panorama/pImg8.png'),
          cameraPosition: { // 相机位置
            x: -10,
            y: 30,
            z: 10
          },
          modePosition: { // 模型位置
            x: 5,
            y: -280,
            z: -10
          },
          modeScale: { // 模型缩放
            x: 150,
            y: 150,
            z: 150
          }
        },
        {
          id: '04',
          title: '全景看房'
        }
      ],

      scene: null, // 场景
      camera: null, //相机
      renderer: null, //渲染器
      curActive: 0, // 选择的当前模型
      isShow360House: false
    }
  },

  mounted() {
    this.init(this.modelList[0])
  },

  methods: {
    init(data) {
      this.$nextTick(() => {
        // 新建场景
        this.scene = new THREE.Scene();
        const container = document.getElementById('container');
        // 场景宽度
        // let width = container.clientWidth;
        let width = window.innerWidth;
        // 场景高度
        let height = window.innerHeight;

        // 新建透视相机
        this.camera = new THREE.PerspectiveCamera(75, width / height, 0.01, 1000);
        // 设置相机的位置
        this.camera.position.set(data.cameraPosition.x, data.cameraPosition.y, data.cameraPosition.z);

        //相机观察目标指向Threejs 3D空间中某个位置
        this.camera.lookAt(0, 0, 0); //坐标原点

        this.renderer = new THREE.WebGLRenderer({
          antialias: true, // 抗锯齿
          alpha: true
        });

        // 设置渲染区域
        this.renderer.setSize(width, height);

        // 设置设备像素比。通常用于避免HiDPI设备上绘图模糊
        this.renderer.setPixelRatio(window.devicePixelRatio);
        // 将画布添加到container中
        container.appendChild(this.renderer.domElement);

        // 坐标系（辅助开发）红色代表 X 轴. 绿色代表 Y 轴. 蓝色代表 Z 轴
        // const axes = new THREE.AxesHelper(300);
        // axes.rotation.x = 0.1;
        // this.scene.add(axes);

        window.addEventListener( 'resize', this.onWindowResize);
        this.createPanorama(data.bg);  // 球体全景
        this.createOrbitControls();
        this.createLight();
        this.loadModel(data);
        this.renderAnimate()
      })
    },

    // 场景球体全景
    createPanorama(img) {
      // 创建一个球星几何对象Geometry
      const geometry = new THREE.SphereGeometry(500, 100, 100);
      // 添加材质
      const material = new THREE.MeshBasicMaterial({
        map: new THREE.TextureLoader().load(img), // 导入图片
        // color: 0xffffff,
        side: THREE.BackSide // 定义将要渲染哪一面: 正面，背面或两者
      });
      // 新建物体网格模型,并将几何对象和材质添加进去
      const mesh = new THREE.Mesh(geometry, material);
      this.scene.add(mesh)
    },

    // 创建轨道控制 可以使得相机围绕目标进行轨道运动
    createOrbitControls() {
      this.controls = new OrbitControls(this.camera, this.renderer.domElement);
      // 右键平移拖拽
      this.controls.enablePan = false;
      // 鼠标缩放
      this.controls.enableZoom = true;
      // 相机距离原点的距离范围
      this.controls.minDistance = 0;
      this.controls.maxDistance = 200;
      // 当.enableDamping设置为true的时候，阻尼惯性有多大。 默认 0.05.
      // 请注意，要使得这一值生效，你必须在你的动画循环里调用.update()
      this.controls.dampingFactor = 0.1
      // 你能够垂直旋转的角度的上限，范围是0到Math.PI，其默认值为Math.PI
      this.controls.maxPolarAngle = (Math.PI / 4) * 3;
      this.controls.minPolarAngle = Math.PI / 4;
      // 自动旋转
      this.controls.autoRotate = true;
      // 自转速度
      this.controls.autoRotateSpeed = 1;
    },

    // 渲染函数
    renderAnimate() {
      this.anId = requestAnimationFrame(this.renderAnimate);
      this.controls.update();
      this.renderer.render(this.scene, this.camera)
    },

    // 加载模型
    loadModel(data) {
      let lm = new Promise((resolve, reject) => {
        let loader = new GLTFLoader();
        loader.load(data.modelUrl, gltf => {
          resolve(gltf);
          reject('加载模型失败');
          this.$store.commit('SET_LOADING', false);
        });
      });

      // 加载成功
      lm.then(res => {
        res.scene.position.set(data.modePosition.x, data.modePosition.y, data.modePosition.z);
        res.scene.scale.set(data.modeScale.x, data.modeScale.y, data.modeScale.z);
        this.scene.add(res.scene);
        this.$store.commit('SET_LOADING', false);
      }).catch(error => {
        console.log('模型加载失败：', error);
        this.$store.commit('SET_LOADING', false);
      })
    },

    // 创建光源
    createLight() {
      const lightColor = new THREE.Color(0xffffff);
      // 环境光会均匀的照亮场景中的所有物体
      const ambient = new THREE.AmbientLight(lightColor);
      this.scene.add(ambient);

      let directionalLight1 = new THREE.DirectionalLight(lightColor);
      directionalLight1.position.set(0, 0, 500);
      this.scene.add(directionalLight1); //平行光源添加到场景中

      if (this.curActive !== 2) {
        let directionalLight2 = new THREE.DirectionalLight(lightColor);
        directionalLight2.position.set(0, 0, -500);
        this.scene.add(directionalLight2); //平行光源添加到场景中

        let directionalLight3 = new THREE.DirectionalLight(lightColor);
        directionalLight3.position.set(500, 0, 0);
        this.scene.add(directionalLight3); //平行光源添加到场景中

        let directionalLight4 = new THREE.DirectionalLight(lightColor);
        directionalLight4.position.set(-500, 0, 0);
        this.scene.add(directionalLight4); //平行光源添加到场景中

        let directionalLight5 = new THREE.DirectionalLight(lightColor);
        directionalLight5.position.set(0, 500, 0);
        this.scene.add(directionalLight5); //平行光源添加到场景中
      }

      let directionalLight6 = new THREE.DirectionalLight(lightColor);
      directionalLight6.position.set(0, -500, 0);
      this.scene.add(directionalLight6); //平行光源添加到场景中
    },

    onWindowResize() {
      if (this.camera) {
        this.camera.aspect = window.innerWidth / window.innerHeight;
        this.camera.updateProjectionMatrix();
        this.renderer.setSize( window.innerWidth, window.innerHeight );
      }
      // labelRenderer.setSize( window.innerWidth, window.innerHeight );
    },

    // 点击切换模型
    async toggleModel(item, index) {
      this.$store.commit('SET_LOADING', true);
      if (this.camera) {
        await this.destroyCom();
      }
      this.curActive = index;
      if (item.id === '04') {
        this.isShow360House = true;
      } else {
        this.isShow360House = false;
        this.init(item);
      }
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
      this.controls.reset();
      this.controls.dispose();
      // this.controls = null;
      document.getElementById('container').innerHTML = null;
      this.renderer = null;
    },

    onMouseClick(event) {
      const container = document.getElementById('container');
      let getBoundingClientRect = container.getBoundingClientRect()
      // 屏幕坐标转标准设备坐标
      let x = ((event.clientX - getBoundingClientRect .left) / container.offsetWidth) * 2 - 1;// 标准设备横坐标
      let y = -((event.clientY - getBoundingClientRect .top) / container.offsetHeight) * 2 + 1;// 标准设备纵坐标
      let standardVector = new THREE.Vector3(x, y, 1);// 标准设备坐标
      // 标准设备坐标转世界坐标
      let worldVector = standardVector.unproject(this.camera);
      // 射线投射方向单位向量(worldVector坐标减相机位置坐标)
      let ray = worldVector.sub(this.camera.position).normalize();
      // 创建射线投射器对象
      let rayCaster = new THREE.Raycaster(this.camera.position, ray);
      // 返回射线选中的对象 第二个参数如果不填 默认是false
      let intersects = rayCaster.intersectObjects(this.scene.children, true);
      if (intersects.length > 0) {
        console.log("point::", intersects[0].point);
      }
    }
  },
}
</script>

<style scoped lang="scss">
.model-wrap {
  width: 100%;
  height: 100%;
  .three-wrap {
    height: 100%;
    width: 100%;
    #container {
      width: 100%;
      height: 100%;
    }
  }
  .toggle-wrap {
    position: absolute;
    top: 0;
    right: 0;
    width: 150px;
    background-color: rgba(255, 255, 255, 0.8);

    .toggle-btn {
      border-radius: 0;
      width: 100%;
      margin-left: 0;
      margin-top: 10px;
      //box-shadow: 0px 0px 5px black;
      &:first-of-type {
        margin-top: 0;
      }
      &.active {
        color: #000;
        //box-shadow: 5px 5px 10px black;
        background-color: #ff8282;
        border: #ff8282;
        color: white;
      }
    }
  }
}
</style>
