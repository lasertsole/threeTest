<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";//导入轨道控制器
import { onMounted, ref } from "vue";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader";

  //1.创建场景
  const scene = new THREE.Scene();

  //2.创建相机
  const camera = new THREE.PerspectiveCamera(
      75,//视锥体角度
      window.innerWidth/window.innerHeight,//宽高比
      0.1,//近端极限
      1000//远端极限
  );

  //3.测试相机属性
  camera.position.set(0, 0, 10);

  //4.场景内添加相机
  scene.add(camera);
  
  //5.添加物体
  //设置cube纹理加载器
  let envMapTexture;

  //加载hdr环境图
  const rgbeLoader = new RGBELoader();
  rgbeLoader.loadAsync('./blue_photo_studio_4k.hdr').then((texture)=>{
    texture.mapping = THREE.EquirectangularReflectionMapping;//改变纹理映射为正方形映射为为球形
    scene.background = texture;
    scene.environment = texture;
    envMapTexture = texture;
  });

  //创建几何体
  const sphereGeometry = new THREE.SphereGeometry(1, 20, 20);//设置半径为1，顶点数为20,20

  //创建材质
  const material = new THREE.MeshStandardMaterial({
    metalness: 0.7,
    roughness: 0.1,
    envMap: envMapTexture,//贴上环境贴图
  });

  //用几何体和材质合成实体
  const cube = new THREE.Mesh(sphereGeometry, material);
  
  //场景中加入实体实体
  scene.add(cube);
  
  //6.添加坐标轴辅助器
  const axesHelper = new THREE.AxesHelper(5);
  scene.add(axesHelper);

  //7.增加灯光
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.1);//增加环境光（无光源，四面八方都是）
  scene.add(ambientLight);//环境光不需要设置位置
  const directtionaLight = new THREE.DirectionalLight(0xffffff, 0.9);//增加直照光（有光源）
  directtionaLight.position.set(10, 10, 10);
  scene.add(directtionaLight);

  //8.初始化渲染器
  const renderer = new THREE.WebGLRenderer();
  //设置渲染的尺寸大小(相当于canvas画布)
  renderer.setSize(window.innerWidth, window.innerHeight);

  //9.创建轨道控制器
  const controls = new OrbitControls( camera, renderer.domElement);

  //10.设置控制器阻尼，让控制器更有真实效果，必须在动画循环里调用.update()。
  controls.enableDamping = true;

  /***********实际渲染***********/
  function render(){//渲染函数 requestAnimationFrame默认传入时间
    controls.update();
    renderer.render(scene, camera);
    requestAnimationFrame(render);//1.请求动画帧，每帧重新渲染。浏览器原生函数 2.下一帧时函数自调用
  }

  const demo = ref();
  onMounted(()=>{
    //将渲染的canvas内容添加到demo上
    demo.value.appendChild(renderer.domElement);
    //使用渲染器，通过相机将场景渲染进来
    render();
  });

  // 监听画面变化，更新渲染画面
  window.addEventListener("resize", ()=>{
    // 更新摄像头
    camera.aspect = window.innerWidth / window.innerHeight;
    // 更新摄像机的投影矩阵(保证图像比例正确)
    camera.updateProjectionMatrix();
    // 更新渲染器
    renderer.setSize(window.innerWidth, window.innerHeight);
    // 设置渲染器的像素比
    renderer.setPixelRatio(window.devicePixelRatio);
  });

  /*鼠标双击全屏或退出全屏*/
  function fullScreenOrNot(){
    if(!document.fullscreenElement){//若不存在已全屏dom,则进入全屏。分钟退出全屏
        renderer.domElement.requestFullscreen();
    }
    else{
        document.exitFullscreen();
    }
  }
</script>

<template>
  <div ref="demo" class="demo" @click="changeAnimate" @dblclick="fullScreenOrNot"></div>
</template>

<style scoped>
 .demo{
  width: 100vw;
  height: 100vh;
 }
</style>
