---
title: Threejs学习和总结基础篇 - 几何体Geometry
date: 2021-07-29
sidebar: 'auto'
categories:
  - Javascript
tags:
  - Threejs
  - 3D
  - WebGL
  - Geometry
author: aigisss
location: blog
summary: 一个模型是由几何体 Geometry 和材质 Material 组成。Three.js 内置了很多的几何体种类，如立方体、三棱锥、球、八面体、十二面体、二十面体等等，将介绍这些几何体的模型创建和几何体的通用方法。
---

# 几何体Geometry
一个模型是由几何体 Geometry 和材质 Material 组成。
Three.js 内置了很多的几何体种类，如立方体、三棱锥、球、八面体、十二面体、二十面体等等，将介绍这些几何体的模型创建和几何体的通用方法。

![](https://aigisss.com/blog/posts/89cc5dea/geometry-1570631173098.png)

## Geometry 和 BufferGeometry
当前 Three.js 内置了这两种几何体类型，这两个几何体类型都用于存储模型的顶点位置、面的索引、法向量、颜色、UV 纹理以及一些自定义的属性。

它们两个的区别是：BufferGeometry 存储的都是一些原始的数据，性能比 Geometry 高，很适合存储一些放入场景内不需要再额外操作的模型。
而 Geometry 的优势刚好相反，Geometry 比 BufferGeometry 更友好，使用了 Three.js 提供的 `THREE.Vector3` 或者 THREE.Color 这样的对象来存储数据（顶点位置、面、颜色等），
这些对象易于阅读和编辑，但效率低于 BufferGeometry 使用的类型化数组。

所以，我们可以根据项目的大小来使用不同的几何体，小项目可以使用 Geometry 实现，中大型的项目还是推荐 BufferGeometry。

我们将使用较为简单的 Geometry 来实现案例。

### Geometry 和 BufferGeometry 互转
在这里插一点内容，两种几何体类型可以互转，所以，不要担心现在使用的是哪种。  
- Geometry 转换成 BufferGeometry

转换代码，如下：
```javascript
//实例化一个Geometry对象
var geo = new THREE.Geometry(); 
//调用对象的fromBufferGeometry方法，并将需要转换的geometry传入
var bufferGeo = geo.fromBufferGeometry(geometry);
//返回的对象转换成的BufferGeometry
```
- BufferGeometry转换成Geometry
```javascript
//实例化一个BufferGeometry对象
var bufferGeo = new THREE.BufferGeometry(); 
//调用对象的fromGeometry方法，并将需要转换的bufferGeometry传入
var geo = bufferGeo.fromGeometry(bufferGeometry);
//返回的对象转换成的Geometry
```
接下来，我们将讲解一下 Three.js 内置几何体。

### 立方体 BoxGeometry 和 BoxBufferGeometry
立方体是最早接触的几何体，可以通过设置长宽高来创建各种各样的立方体。

看下面的案例，代码：
```javascript
var geometry = new THREE.BoxGeometry( 1, 1, 1 );
var material = new THREE.MeshBasicMaterial( {color: 0x00ff00} );
var cube = new THREE.Mesh( geometry, material );
scene.add( cube );
```
案例中的构造函数：
```
BoxGeometry(width : '浮点类型', height : '浮点类型', depth : '浮点类型', widthSegments : '整数类型', heightSegments : '整数类型', depthSegments : '整数类型')
```
各参数的含义：  
- width：沿 X 轴的宽度，默认值为1
- height：沿 Y 轴的高度，默认值为1
- depth：沿 Z 轴的深度，默认值为1
- widthSegments：可选，沿着边的宽度的分割面的数量。默认值为1
- heightSegments：可选，沿着边的高度的分割面的数量。默认值为1
- depthSegments：可选，沿着边的深度的分割面的数量。缺省值是1

案例查看：[请点击这里](https://threejs.org/docs/scenes/geometry-browser.html#BoxGeometry) 。

### 圆 CircleGeometry 和 CircleBufferGeometry
> 在 WebGL 里，所有的模型都是通过三角形面组成，圆形是由多个三角形分段构成，这些三角形分段围绕一个中心点延伸并且延伸到给定半径以外。
> 它从起始角度和给定的中心角度逆时针方向构建。它也可以用来创建规则的多边形，其中线段的数量决定了边的数量。

看下面案例，代码如下：
```javascript
var geometry = new THREE.CircleGeometry( 5, 32 );
var material = new THREE.MeshBasicMaterial( { color: 0xffff00 } );
var circle = new THREE.Mesh( geometry, material );
scene.add( circle );
```
案例中的构造函数，如下：
```
CircleGeometry(radius : '浮点类型', segments : '整数类型', thetaStart : '浮点类型', thetaLength : '浮点类型')
```
各参数的含义：  
- radius：圆的半径，默认值为1
- segments：段数（三角形），最小值为3，默认值为8
- thetaStart：第一段的起始角度，默认值为0
- thetaLength：圆形扇形的中心角，通常称为 theta。默认值是 2 * Pi，画出一个整圆。

案例查看：[请点击这里](https://threejs.org/docs/scenes/geometry-browser.html#CircleGeometry) 。

### 圆锥 ConeGeometry 和 ConeBufferGeometry
这是一个可以创建圆锥体的类。

看下面案例，代码如下：
```javascript
var geometry = new THREE.ConeGeometry( 5, 20, 32 );
var material = new THREE.MeshBasicMaterial( {color: 0xffff00} );
var cone = new THREE.Mesh( geometry, material );
scene.add( cone );
```
案例中的构造函数，如下：
```
ConeGeometry(radius : '浮点类型', height : '浮点类型', radialSegments : '整数类型', heightSegments : '整数类型', openEnded : '布尔类型', thetaStart : '浮点类型', thetaLength : '浮点类型')
```
各参数含义：  
- radius：底部圆锥的半径，默认值为1
- height：圆锥体的高度，默认值为1
- radialSegments：圆锥周围的分段面数，默认值为8
- heightSegments：沿圆锥体高度的面的行数，默认值为1
- openEnded：圆锥体底部是是隐藏还是显示，默认值为 false，显示
- thetaStart：第一段的起始角度，默认值是0（Three.js 的0度位置）
- thetaLength — 圆形扇形的中心角，通常称为 theta。默认值是2 * Pi，画出一个整圆。

案例查看：[请点击这里](https://threejs.org/docs/scenes/geometry-browser.html#ConeGeometry) 。

### 圆柱 CylinderGeometry 和 CylinderBufferGeometry
这是一个可以创建圆柱几何体的类。

看下面案例，代码如下：
```javascript
var geometry = new THREE.CylinderGeometry( 5, 5, 20, 32 );
var material = new THREE.MeshBasicMaterial( {color: 0xffff00} );
var cylinder = new THREE.Mesh( geometry, material );
scene.add( cylinder );
```
案例中的构造函数，如下：
```
CylinderGeometry(radiusTop : '浮点类型', radiusBottom : '浮点类型', height : '浮点类型', radialSegments : '整数类型', heightSegments : '整数类型', openEnded : '布尔类型', thetaStart : '浮点类型', thetaLength : '浮点类型')
```
各参数含义：  
- radiusTop：顶部圆柱体的半径。默认值为1
- radiusBottom：底部圆柱体的半径。默认值为1
- height：圆柱体的高度。默认值为1
- radialSegments：圆柱周围的分段面数。默认值为8
- heightSegments：沿圆柱体高度的面的行数。默认值为1
- openEnded：圆柱体的两端是否显示，默认值是 false，显示
- thetaStart：第一段的起始角度，默认值是0（Three.js 的0度位置）。
- thetaLength — 圆形扇形的中心角，通常称为 theta。默认值是2 * Pi，画出一个整圆。

案例查看：[请点击这里](https://threejs.org/docs/scenes/geometry-browser.html#CylinderGeometry) 。

### 球 SphereGeometry 和 SphereBufferGeometry
这是一个可以创建球体几何体的类。

看下面案例，代码如下：
```javascript
var geometry = new THREE.SphereGeometry( 5, 32, 32 );
var material = new THREE.MeshBasicMaterial( {color: 0xffff00} );
var sphere = new THREE.Mesh( geometry, material );
scene.add( sphere );
```
案例中的构造函数，如下 ：
```
SphereGeometry(radius : '浮点类型', widthSegments : '整数类型', heightSegments : '整数类型', phiStart : '浮点类型', phiLength : '浮点类型', thetaStart : '浮点类型', thetaLength : '浮点类型')
```
各参数含义：  
- radius：球体半径。默认值是1
- widthSegments：水平线段的数量。最小值是3，默认值是8
- heightSegments：垂直段的数量。最小值是2，默认值是6
- phiStart：指定水平渲染起始角度。默认值为0
- phiLength：指定水平渲染角度大小。默认值是 Math.PI * 2
- thetaStart：指定垂直渲染起始角度。默认值为0
- thetaLength：指定垂直渲染角度大小。默认是 Math.PI。

默认是渲染整个的圆，如果线段越多，显得球体越圆滑。  
案例查看：[请点击这里](https://threejs.org/docs/scenes/geometry-browser.html#SphereGeometry) 。

### 平面 PlaneGeometry 和 SphereBufferGeometry
这是一个可以创建平面几何体的类。

看下面案例，代码如下：
```javascript
var geometry = new THREE.PlaneGeometry( 5, 20, 32 );
var material = new THREE.MeshBasicMaterial( {color: 0xffff00, side: THREE.DoubleSide} );
var plane = new THREE.Mesh( geometry, material );
scene.add( plane );
```
案例中的构造函数，如下 ：
```
PlaneGeometry(width : '浮点类型', height : '浮点类型', widthSegments : '整数类型', heightSegments : '整数类型')
```
各参数含义：  
- width：沿 X 轴的宽度。默认值为1
- height：沿着 Y 轴的高度。默认值为1
- widthSegments：宽度的分段数，可选。默认值为1
- heightSegments：高度的分段数，可选。默认值为1。
案例查看：请点击这里。

### 圆环 TorusGeometry 和 TorusBufferGeometry
一个可以创建圆环几何体的类。

看下面案例，代码如下：
```javascript
var geometry = new THREE.TorusGeometry( 10, 3, 16, 100 );
var material = new THREE.MeshBasicMaterial( { color: 0xffff00 } );
var torus = new THREE.Mesh( geometry, material );
scene.add( torus );
```
案例中的构造函数，如下 ：
```
TorusGeometry(radius : '浮点类型', tube : '浮点类型', radialSegments : '整数类型', tubularSegments : '整数类型', arc : '浮点类型')
```
各参数含义：  
- radius：圆环的半径，从圆环的中心到管的中心。默认值为1；
- tube：管的半径。默认值是0.4；
- radialSegments：横向分段数，默认值是8；
- tubularSegments：纵向分段数，默认值是6；
- arc — 绘制的弧度。默认值是 Math.PI * 2，绘制整个圆环。

案例查看：[请点击这里](https://threejs.org/docs/scenes/geometry-browser.html#TorusGeometry) 。

Three.js 还内置了一些其他的几何体模型（如字体几何体、拉伸几何体、车床几何体等），由于篇幅原因和难度原因，在这里不一一讲解。

如果迫不及待得想了解这一块的内容可以通过官方文档或者我的博客中关于 Three.js 的笔记来了解更多。

## 常用方法
Geometry 和 BufferGeomety 内置了一些常用的方法，在每一种几何体上面，我们都可以调用相关的方法来达到我们的目的。

### center()
此方法为居中方法，可以根据边界框居中几何图形。

### computeBoundingBox()
此方法可以计算几何的边界框，方法调用后，会更新 `Geometry.boundingBox` 属性，我们可以通过 `Geometry.boundingBox` 属性获取到一个包围几何体的立方体的每个轴向的最大值和最小值。

### dispose()
将几何体从内存中删除，这个方法必须记得使用。如果频繁的删除模型，一定要记得将几何体从内存中删除掉。

## 总结案例
最后，我将上面介绍的所有几何体放到一个场景内，制作了一个小案例，如下：
```js
import * as dat from "dat.gui";
import * as Stats from "stats.js";
import * as THREE from "three";
// import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

//声明一些全局变量
let renderer,
  camera,
  scene,
  stats,
  controls,
  gui,
  rotate = true;
//声明一些几何体模型变量
let box, circle, cone, cylinder, sphere, plane, torus;
const appWidth = window.innerWidth * 0.79;
const appHeight = window.innerHeight;
const app = document.getElementById("app");
//初始化渲染器
function initRenderer() {
  renderer = new THREE.WebGLRenderer(); //实例化渲染器
  renderer.setSize(appWidth, appHeight); //设置宽和高

  app.appendChild(renderer.domElement); //添加到dom
}
//初始化场景
function initScene() {
  scene = new THREE.Scene(); //实例化场景
}
//初始化相机
function initCamera() {
  camera = new THREE.PerspectiveCamera(45, appWidth / appHeight, 0.1, 200); //实例化相机
  camera.position.set(0, 0, 150);
}
//创建模型
function initMesh() {
  var material = new THREE.MeshNormalMaterial();
  //创建立方体
  box = new THREE.Mesh(new THREE.BoxGeometry(5, 5, 5), material);
  box.position.set(-50, 20, 0);
  scene.add(box);
  //创建圆
  circle = new THREE.Mesh(new THREE.CircleGeometry(5, 32), material);
  circle.position.set(-20, 20, 0);
  scene.add(circle);
  //创建圆锥
  cone = new THREE.Mesh(new THREE.ConeGeometry(5, 20, 32), material);
  cone.position.set(20, 20, 0);
  scene.add(cone);
  //创建圆柱
  cylinder = new THREE.Mesh(new THREE.CylinderGeometry(5, 5, 20, 32), material);
  cylinder.position.set(50, 20, 0);
  scene.add(cylinder);
  //创建球
  sphere = new THREE.Mesh(new THREE.SphereGeometry(5, 32, 32), material);
  sphere.position.set(-35, -20, 0);
  scene.add(sphere);
  //创建平面
  plane = new THREE.Mesh(new THREE.PlaneGeometry(5, 5), material);
  plane.position.set(0, -20, 0);
  scene.add(plane);
  //创建圆环
  torus = new THREE.Mesh(new THREE.TorusGeometry(10, 3, 16, 4), material);
  torus.position.set(35, -20, 0);
  scene.add(torus);
}
//运行动画
var step = 0; //记录旋转的角度
function animate() {
  requestAnimationFrame(animate); //循环调用函数
  //判断是否可以旋转
  if (rotate) {
    step += 0.02;
    //设置每一个模型的转向
    box.rotation.set(step, step, step);
    circle.rotation.set(step, step, step);
    cone.rotation.set(step, step, step);
    cylinder.rotation.set(step, step, step);
    sphere.rotation.set(step, step, step);
    plane.rotation.set(step, step, step);
    torus.rotation.set(step, step, step);
  }
  stats.update(); //更新性能检测框
  renderer.render(scene, camera); //渲染界面
}
//性能检测框
function initStats() {
  stats = new Stats();
  app.appendChild(stats.dom);
}
//创建调试框
function initGui() {
  controls = {
    rotate: true
  };
  gui = new dat.GUI();
  console.log("Go: initGui -> gui", dat.GUI);
  gui
    .add(controls, "rotate")
    .name("旋转")
    .onChange(function(e) {
      rotate = e;
    });
}

export default function second() {
  var stats = new Stats();
  console.log("Go: second -> stats", stats);

  // const gui = new dat.GUI();
  // console.log("Go: second -> gui", gui);

  $("#app").empty();
  initRenderer();
  initScene();
  initCamera();
  initMesh();
  initStats();
  initGui();
  animate();
}
```

---
收录时间: 2021-07-29

<Vssue :title="$title" />
