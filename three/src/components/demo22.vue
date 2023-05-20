<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";//导入轨道控制器
import { onMounted, ref } from "vue";
import * as dat from "dat.gui";//导入dat.gui

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
  const sphereGeometry = new THREE.SphereGeometry(1, 20, 20);//设置半径为1，顶点数为20,20

  //创建材质
  const material = new THREE.MeshStandardMaterial();

  //用几何体和材质合成实体
  const sphere = new THREE.Mesh(sphereGeometry, material);
  
  //球投射阴影
  sphere.castShadow = true;

  //场景中加入实体实体
  scene.add(sphere);

  //创建平面
  const planeGeometry = new THREE.PlaneGeometry(50, 50);
  const plane = new THREE.Mesh(planeGeometry, material);
  plane.receiveShadow = true;//平面接收阴影
  plane.position.set(0, -1, 0);
  plane.rotation.x = -Math.PI / 2;
  
  scene.add(plane);
  
  //6.添加坐标轴辅助器
  const axesHelper = new THREE.AxesHelper(5);
  scene.add(axesHelper);

  // 设置时钟
  const clock = new THREE.Clock();

  //7.增加灯光
  const pointLight = new THREE.PointLight(0xff0000, 1);//增加环境光,0.1强度（无光源，四面八方都是
  const smallBall= new THREE.Mesh(
    new THREE.SphereGeometry(0.1,20, 20),
    new THREE.MeshBasicMaterial({color: 0xff0000})
  );
  smallBall.position.set(2, 2, 2);
  smallBall.add(pointLight);
  scene.add(smallBall);//环境光不需要设置位置

  //增加灯光阴影
  pointLight.castShadow = true;//光线投下阴影
  pointLight.shadow.radius = 20;
  pointLight.shadow.mapSize.set(4096, 4096);//阴影贴图分辨率
  pointLight.distance = 0;//设置光照距离为多少的时候为0，但0为不衰减
  pointLight.penumbra = 0;//光照半影的衰减效果,但0为不衰减
  pointLight.decay = 1;//光沿着距离衰减，默认为1，为2时是现实世界的衰减情况,需要开启renderer.physicallyCorrectLights
  pointLight.intensity = 2;//调节光照强度

  //增加光的投射相机
  pointLight.shadow.camera.near = 0.5;
  pointLight.shadow.camera.far = 500;
  pointLight.shadow.camera.top = 5;
  pointLight.shadow.camera.bottom = -5;
  pointLight.shadow.camera.left = -5;
  pointLight.shadow.camera.right = 5;

  //初始化gui用户参数控制
  const gui = new dat.GUI();
  gui.add(pointLight.position, "x").min(-5).max(5).step(0.1);
  gui.add(pointLight, "distance").min(0).max(10).step(0.01);
  gui.add(pointLight, "decay").min(0).max(5).step(0.01);

  //8.初始化渲染器
  const renderer = new THREE.WebGLRenderer();
  
  //设置渲染的尺寸大小(相当于canvas画布)
  renderer.setSize(window.innerWidth, window.innerHeight);
  
  //开启环境阴影贴图
  renderer.shadowMap.enabled = true;
  //开启渲染器正确的物理渲染方法
  renderer.physicallyCorrectLights = true;

  //9.创建轨道控制器
  const controls = new OrbitControls( camera, renderer.domElement);

  //10.设置控制器阻尼，让控制器更有真实效果，必须在动画循环里调用.update()。
  controls.enableDamping = true;

  /***********实际渲染***********/
  function render(){//渲染函数 requestAnimationFrame默认传入时间
    let time = clock.getElapsedTime();
    smallBall.position.x = Math.sin(time) * 3;
    smallBall.position.z = Math.cos(time) * 3;
    smallBall.position.y = 2 + Math.sin(time*10) * 0.5;

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
