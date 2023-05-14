<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";//导入轨道控制器
import { onMounted, ref } from "vue";

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
  //创建几何体
  const cubeGeometry = new THREE.BoxGeometry(1,1,1,200, 200);//设置长宽高为1,1,1；水平和垂直顶点数为200,200的立体
  
  //导入纹理
  const textureLoader = new THREE.TextureLoader();
  const texture = textureLoader.load('./MOYA.jpg');
  //导入置换贴图
  const heightTexture = textureLoader.load('./MOYA.jpg');

  //纹理显示设置
  texture.magFilter = THREE.NearestFilter;//纹素覆盖大于一个像素时的设置（使用时按原本像素填充，插值无效）
  texture.minFilter = THREE.NearestFilter;//纹素覆盖小于一个像素时的设置（使用时按中值算法填充像素）

  //创建材质
  const standardMaterial = new THREE.MeshStandardMaterial({
    color: "#ffffff",
    map: texture,
    displacementMap: heightTexture,
    displacementScale: 0.05,//置换距离
  });
  standardMaterial.side = THREE.DoubleSide;//设置内外面都有材质

  //用几何体和材质合成实体
  const cube = new THREE.Mesh(cubeGeometry,standardMaterial);
  //场景中加入实体实体
  scene.add(cube);
  
  //6.添加坐标轴辅助器
  const axesHelper = new THREE.AxesHelper(5);
  scene.add(axesHelper);

  //7.增加灯光
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.1);//增加环境光（无光源，四面八方都是）
  scene.add(ambientLight);//环境光不需要设置位置
  const directtionaLight = new THREE.DirectionalLight(0xffffff, 0.5);//增加直照光（有光源）
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
