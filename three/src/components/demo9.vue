<script setup>
import * as THREE from "three";
import gsap from "gsap";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";//导入轨道控制器
import * as dat from "dat.gui";//导入dat.gui
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
  
  //添加物体
  //创建几何体
  for(let i=0;i<50;i++){
    //每个三角形，需要3个顶点，每个顶点需要3个值
    const geometry = new THREE.BufferGeometry();//创建缓存几何体
    const positionArray = new Float32Array(9);//共有9个值
    for(let j=0;j<9;j++){
        positionArray[j] = Math.random() * 10 - 5;
    }
    geometry.setAttribute("position", new THREE.BufferAttribute(positionArray, 3));//设置几何体属性，每三组值为一个面属性
    let color = new THREE.Color(Math.random(),Math.random(),Math.random());//创建随机颜色
    const material = new THREE.MeshBasicMaterial({color:color, transparent:true, opacity: 0.5});
    //根据几何体和材质创建物体
    const mesh = new THREE.Mesh(geometry, material);
    scene.add(mesh);
  }

  //添加坐标轴辅助器
  const axesHelper = new THREE.AxesHelper(5);
  scene.add(axesHelper);

  //初始化渲染器
  const renderer = new THREE.WebGLRenderer();
  //设置渲染的尺寸大小(相当于canvas画布)
  renderer.setSize(window.innerWidth, window.innerHeight);

  //创建轨道控制器
  const controls = new OrbitControls( camera, renderer.domElement);
  //设置控制器阻尼，让控制器更有真实效果，必须在动画循环里调用.update()。
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
