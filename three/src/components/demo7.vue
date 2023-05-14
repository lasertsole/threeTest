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
  const cubeGeometry = new THREE.BoxGeometry(1,1,1);//创建几何体形状
  const cubeMaterial = new THREE.MeshBasicMaterial({color: 0xffff00});//创建白色基本网格材质
  const cube = new THREE.Mesh(cubeGeometry,cubeMaterial);//根据几何体材质创建物体
  cube.scale.set(3,2,1);
  cube.rotation.set(0, 0, 0, "ZYX");//物体绕x轴旋转45deg,绕y和轴旋转0deg，旋转顺序为先Z再Y后X
  //将几何体往场景添加
  scene.add(cube);
  
  /*********************以下是用户自定义设置*********************/
  //初始化gui用户参数控制
  const gui = new dat.GUI();
  gui.add(cube.position, "x").min(0).max(5).step(0.1).name("x轴移动").onChange((value)=>{//修改物体位置
    console.log("值被修改了:", value);
  }).onFinishChange((value)=>{
    console.log("完全停下来:", value);
  });

  const params = {//修改物体颜色
    color: "#ffff00",
    fn:()=>{//让立方体运动起来
        gsap.to(cube.position, {x:5, duration:2, yoyo:true, repeat:-1});
    }
  }
  gui.addColor(params, 'color').name("颜色").onChange((value)=>{
    console.log("值被修改:", value);
    cube.material.color.set(value);
  });

  gui.add(cube, "visible").name("是否显示");//设置物体是否显示

  gui.add(params,"fn");//触发物体往返运动
  
  let folder = gui.addFolder("设置立方体");//添加折叠文件夹

  folder.add(cube.material, "wireframe");//设置线框属性

  /*********************以上是用户自定义设置*********************/

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
