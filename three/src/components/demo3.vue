<script setup>
import * as THREE from "three";
import gsap from "gsap";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
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
  const cubeGeometry = new THREE.BoxGeometry(1,1,1);//创建几何体形状
  const cubeMaterial = new THREE.MeshBasicMaterial({color: 0xffff00});//创建白色基本网格材质
  const cube = new THREE.Mesh(cubeGeometry,cubeMaterial);//根据几何体材质创建物体
  cube.scale.set(3,2,1);
  cube.rotation.set(0, 0, 0, "ZYX");//物体绕x轴旋转45deg,绕y和轴旋转0deg，旋转顺序为先Z再Y后X
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

  // 设置动画
  let animate1 = gsap.to(cube.position, {
    x:5,//运动最终状态 
    duration: 5,//持续时间 
    ease: "power4.in",//运动方式
    onStart:()=>{//开始触发函数
      console.log("唯心主义最强强者汉偶武，终将超越计划！");
    },
    onComplete:()=>{//结束触发函数
      console.log("最强强者汉偶武，在期末考试降临之际，一口气超越计划的传奇故事，你！不！要！忘！记！啦！！！！");
    },
    delay: 2,//延迟开始时间
    repeat: 2,// 重复次数，-1为无限次
    yoyo: true,//往返运动
  });

  gsap.to(cube.rotation, {x: 2 * Math.PI, duration: 5, ease: "none", repeat:-1});
  /***********实际渲染***********/
  function render(){//渲染函数 requestAnimationFrame默认传入时间
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

  function changeAnimate(){
    if(animate1.isActive()){//判断animate1是否在运动状态
      animate1.pause();//停止animate1
    }
    else{
      animate1.resume();//恢复animate1
    }
  }
</script>

<template>
  <div ref="demo" class="demo" @click="changeAnimate"></div>
</template>

<style scoped>
 .demo{
  width: 100vw;
  height: 100vh;
 }
</style>
