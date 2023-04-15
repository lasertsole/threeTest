<script setup>
import * as THREE from "three"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { onMounted, ref, resolveTransitionHooks } from "vue";

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
  const cubeGeometry = new THREE.BoxGeometry(1,1,1);//创建几何体形状
  const cubeMaterial = new THREE.MeshBasicMaterial({color: 0xffff00});//创建白色基本网格材质
  const cube = new THREE.Mesh(cubeGeometry,cubeMaterial);//根据几何体材质创建物体
  cube.scale.set(3,2,1);
  cube.rotation.set(Math.PI / 4, 0, 0, "ZYX");//物体绕x轴旋转45deg,绕y和轴旋转0deg，旋转顺序为先Z再Y后X
  //将几何体往场景添加
  scene.add(cube);
  
  //添加坐标轴辅助器
  const axesHelper = new THREE.AxesHelper(5);
  scene.add(axesHelper);

  //初始化渲染器
  const renderer = new THREE.WebGLRenderer();
  //设置渲染的尺寸大小(相当于canvas画布)
  renderer.setSize(window.innerWidth, window.innerHeight);

  //创建轨道控制器
  const controls = new OrbitControls( camera, renderer.domElement);
  
  // 设置时钟
  const clock = new THREE.Clock();
  /***********实际渲染***********/
  function render(){//渲染函数 requestAnimationFrame默认传入时间
    // let deltaTime = clock.getDelta();//两次获取时间的间隔时间,单位为秒
    let time = clock.getElapsedTime();//获取时钟运行的总时长,单位为秒
    // console.log("两次获取时间的间隔时间:", deltaTime);
    // console.log("时钟运行总时长:", time);
    //物体移动
    cube.position.x = time * 1 % 5;
    cube.rotation.x = time * 1 % Math.PI;

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
</script>

<template>
  <div ref="demo" class="demo"></div>
</template>

<style scoped>
 .demo{
  width: 100vw;
  height: 100vh;
 }
</style>
