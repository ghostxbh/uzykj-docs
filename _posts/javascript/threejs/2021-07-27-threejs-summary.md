---
title: Threejs学习和总结基础篇 - 概述
date: 2021-07-27
tags:
    - Threejs
    - 3D
    - WebGL
author: aigisss
location: blog
summary: 使用 Three.js 显示创建的内容，我们必须需要的三大件是：渲染器、相机和场景。
---

# 概述
使用 Three.js 显示创建的内容，我们必须需要的三大件是：渲染器、相机和场景。相机获取到场景内显示的内容，然后再通过渲染器渲染到画布上面。

要在屏幕上展示3D图形，思路大体上都是这样的：

- 1、构建一个三维空间
    - Three中称之为场景(Scene)
- 2、选择一个观察点，并确定观察方向/角度等
    - Three中称之为相机(Camera)
- 3、在场景中添加供观察的物体
    - Three中的物体有很多种，包括Mesh,Line,Points等，它们都继承自Object3D类
- 4、将观察到的场景渲染到屏幕上的指定区域
    - Three中使用Renderer完成这一工作
> 拿电影来类比的话，场景对应于整个布景空间，相机是拍摄镜头，渲染器用来把拍摄好的场景转换成胶卷。

## 场景 Scene
场景是所有物体的容器，也对应着我们创建的三维世界。场景允许你设置哪些对象被Three.js渲染以及渲染在哪里。在场景中放置对象、灯光和相机。

## 相机 Camera
```javascript
// 初始化相机
function initCamera() {
  camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 200); // 实例化相机
  camera.position.set(0, 0, 15);
}
```

### 相机
Three中的相机有两种，分别是正投影相机`THREE.OrthographicCamera`和透视投影相机`THREE.PerspectiveCamera`。  
![](https://aigisss.com/blog/posts/89cc5dea/v2-3a0604ded6e3edc25e5b12fd63ba97bb_hd-1570630416011.png)

正交投影与透视投影的区别如上图所示，左图是正交投影，物体发出的光平行地投射到屏幕上，远近的方块都是一样大的；右图是透视投影，近大远小，符合我们平时看东西的感觉。
维基百科：[三维投影](https://link.zhihu.com/?target=https%3A//zh.wikipedia.org/wiki/%E4%B8%89%E7%BB%B4%E6%8A%95%E5%BD%B1)

#### 正交投影相机
![](https://aigisss.com/blog/posts/89cc5dea/v2-62ede52e0bb0d8b49f6cf2e41debc247_hd-1570630416011.jpg)

可以近似地认为，视景体里的物体平行投影到近平面上，然后近平面上的图像被渲染到屏幕上。

#### 透视投影相机
![](https://aigisss.com/blog/posts/89cc5dea/v2-3b160a77bda7661c4dd3920ddeaae605_hd-1570630416012.jpg)

### 坐标系
Camera是三维世界中的观察者，为了观察这个世界，首先我们要描述空间中的位置，Three中使用采用常见的[右手坐标系](https://link.zhihu.com/?target=https%3A//zh.wikipedia.org/wiki/%E7%AC%9B%E5%8D%A1%E5%84%BF%E5%9D%90%E6%A0%87%E7%B3%BB%23.E4.B8.89.E7.B6.AD.E7.A9.BA.E9.96.93) 定位：  
![](https://aigisss.com/blog/posts/89cc5dea/zbx-1570630416012.png)

我们这里使用到的是 `THREE.PerspectiveCamera`- ，这个相机模拟人眼看到的效果，就是具有透视的效果，近大远小。- - 

第一行，我们实例化了一个透视相机，需要四个值，分别是视野、宽高比、近裁面和远裁面。

- 视野：当前相机视野的宽度，值越大，渲染出来的内容也会更多。
- 宽高比：默认是按照画布显示的宽高比例来设置，如果比例设置的不对，会发现渲染出来的画面有拉伸或者压缩的感觉。
- 近裁面和远裁面：这个是设置相机可以看到的场景内容的范围，如果场景内的内容位置不在这两个值内的话，将不会被显示到渲染的画面中。

第二行，我们设置了相机的位置。

## 渲染器 Renderer
```javascript
//初始化渲染器
function initRenderer() {
    renderer = new THREE.WebGLRenderer(); //实例化渲染器
    renderer.setSize(window.innerWidth, window.innerHeight); //设置宽和高
    document.body.appendChild(renderer.domElement); //添加到dom
}
```
第一行实例化了一个 `THREE.WebGLRenderer`，这是一个基于 WebGL 渲染的渲染器，当然，Three.js 向下兼容，还有 CanvasRenderer、CSS2DRenderer、CSS3DRenderer 和 SVGRenderer，这四个渲染器分别基于 canvas2D、CSS2D、CSS3D 和 SVG 渲染的渲染器。由于，作为 3D 渲染，WebGL 渲染的效果最好，并且支持的功能更多。

第二行，调用了一个设置函数 setSize 方法，这个是设置需要显示的窗口大小。案例是基于浏览器全屏显示，所以设置了浏览器窗口的宽和高。

第三行，`renderer.domElement` 是在实例化渲染器时生成的一个 Canvas 画布，渲染器渲染界面生成的内容，都将在这个画布上显示。所以，我们将这个画布添加到了 DOM 当中，来显示渲染的内容。

## 模型 Object3D
渲染器，场景和相机都全了，是不是就能显示东西了？不能！因为场景内没有内容，即使渲染出来也是一片漆黑，所以我们需要往场景里面添加内容。接下来，我们将查看 initMesh 方法，看看如何创建一个最简单的模型：
```javascript
//创建模型
function initMesh() {
    geometry = new THREE.BoxGeometry( 2, 2, 2 ); //创建几何体
    material = new THREE.MeshNormalMaterial(); //创建材质

    mesh = new THREE.Mesh( geometry, material ); //创建网格
    scene.add( mesh ); //将网格添加到场景
}
```
创建一个网格（模型）需要两种对象：几何体和材质。

- 几何体代表模型的形状，它是由固定的点的位置组成，点绘制出面，面组成了模型。
- 材质是我们看到当前模型显示出来的效果，如显示的颜色，质感等。

Three中供显示的物体有很多，它们都继承自Object3D类

### Mesh
我们都知道，计算机的世界里，一条弧线是由有限个点构成的有限条线段连接得到的。线段很多时，看起来就是一条平滑的弧线了。
计算机中的三维模型也是类似的，普遍的做法是用三角形组成的网格来描述，我们把这种模型称之为Mesh模型。

![](https://aigisss.com/blog/posts/89cc5dea/v2-d1ac417178674e359837b5795edca3b3_hd-1570630416012.jpg)

> 这是那只著名的斯坦福兔子。它在3D图形中的地位与数字图像处理领域中著名的Lenna是类似的。
> 看这只兔子，随着三角形数量的增加，它的表面越来越平滑

在Three中，Mesh的构造函数是这样的：Mesh( geometry, material )  
geometry是它的形状，material是它的材质。  
不止是Mesh，创建很多物体都要用到这两个属性。下面我们来看看这两个重要的属性。Material和Geometry是相辅相成的，必须结合使用。  

#### Geometry
Geometry，形状，相当直观。Geometry通过存储模型用到的点集和点间关系(哪些点构成一个三角形)来达到描述物体形状的目的。  
Three提供了立方体(其实是长方体)、平面(其实是长方形)、球体、圆形、圆柱、圆台等许多基本形状；  
你也可以通过自己定义每个点的位置来构造形状；  
对于比较复杂的形状，我们还可以通过外部的模型文件导入。  

#### Material
Material，材质，这就没有形状那么直观了。  
材质其实是物体表面除了形状以为所有可视属性的集合，例如色彩、纹理、光滑度、透明度、反射率、折射率、发光度。  
这里讲一下材质(Material)、贴图(Map)和纹理(Texture)的关系。  
材质上面已经提到了，它包括了贴图以及其它。  
贴图其实是‘贴’和‘图’，它包括了图片和图片应当贴到什么位置。  
纹理嘛，其实就是‘图’了。  
Three提供了多种材质可供选择，能够自由地选择漫反射/镜面反射等材质。  

## 光影Light
神说：要有光！  
光影效果是让画面丰富的重要因素。  
Three提供了包括环境光AmbientLight、点光源PointLight、 聚光灯SpotLight、方向光DirectionalLight、半球光HemisphereLight等多种光源。  
只要在场景中添加需要的光源就好了。

## 让场景动起来
动画，就是多幅图片一直切换便可显示动画的效果。为了能显示动画的效果，我们首先要了解一个函数 requestAnimationFrame，这个函数专门为了动画而出现。
它与 setInterval 相比，优势在于不需要设置多长时间重新渲染，而是在当前线程内 JS 空闲时自动渲染，并且最大帧数控制在一秒60帧。
所以，我们书写了一个可以循环调用的函数：

```javascript
function animate() {
    requestAnimationFrame(animate); //循环调用函数
    // ...
}
```
在循环调用的函数中，每一帧我们都让页面重新渲染相机拍摄下来的内容：
```javascript
renderer.render( scene, camera ); //渲染界面
```
渲染的 render 方法需要两个值，第一个值是场景对象，第二个值是相机对象。
这意味着，你可以有多个相机和多个场景，可以通过渲染不同的场景和相机让画布上显示不同的画面。

但是，如果现在一直渲染的话，我们发现就一个立方体在那，也没有动，我们需要做的是让立方体动起来：
```javascript
mesh.rotation.x += 0.01; //每帧网格模型的沿x轴旋转0.01弧度
mesh.rotation.y += 0.02; //每帧网格模型的沿y轴旋转0.02弧度
```
每一个实例化的网格对象都有一个 rotation 的值，通过设置这个值可以让立方体旋转起来。
在每一帧里，我们让立方体沿 x 轴方向旋转0.01弧度，沿 y 轴旋转0.02弧度（1π 弧度等于180度角度）。

## Threejs性能检测插件
在 Three.js 里面，遇到最多的问题就是性能问题，所以我们需要时刻检测当前的 Three.js 的性能。
现在 Three.js 常使用的一款插件叫 stats。接下来我们看看如何将 stats 插件在 Three.js 的项目中使用。

1、首先在页面中引入插件代码：
```html
<script src="http://www.wjceo.com/lib/js/libs/stats.min.js"></script>
```
这是 一个 CDN 的地址，直接引入即可。

2、然后，我们需要实例化一个 stats 对象，然后把对象内生成的 DOM 添加到页面当中。
```javascript
stats = new Stats();
document.body.appendChild(stats.dom);
```

3、最后一步，我们需要在 requestAnimationFrame 的回调里面更新每次渲染的时间：
```javascript
function animate() {
    requestAnimationFrame(animate); //循环调用函数
    stats.update(); //更新性能插件
    renderer.render( scene, camera ); //渲染界面
}
```
使用了性能检测插件以后，整个代码如下：
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta
      name="viewport"
      content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0"
    />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Stats插件案例</title>
    <style>
      body {
        margin: 0;
      }

      canvas {
        width: 100%;
        height: 100%;
        display: block;
      }
    </style>
  </head>
  <body onload="init()">
    <script src="https://cdn.bootcss.com/three.js/92/three.js"></script>
    <script src="http://www.wjceo.com/lib/js/libs/stats.min.js"></script>
    <script>
      //声明一些全局变量
      var renderer, camera, scene, geometry, material, mesh, stats;
      //初始化渲染器
      function initRenderer() {
        renderer = new THREE.WebGLRenderer(); //实例化渲染器
        renderer.setSize(window.innerWidth, window.innerHeight); //设置宽和高
        document.body.appendChild(renderer.domElement); //添加到dom
      }

      //初始化场景
      function initScene() {
        scene = new THREE.Scene(); //实例化场景
      }

      //初始化相机
      function initCamera() {
        camera = new THREE.PerspectiveCamera(
          45,
          window.innerWidth / window.innerHeight,
          0.1,
          200
        ); //实例化相机
        camera.position.set(0, 0, 15);
      }

      //创建模型
      function initMesh() {
        geometry = new THREE.BoxGeometry(2, 2, 2); //创建几何体
        material = new THREE.MeshNormalMaterial(); //创建材质
        mesh = new THREE.Mesh(geometry, material); //创建网格
        scene.add(mesh); //将网格添加到场景
      }

      //运行动画
      function animate() {
        requestAnimationFrame(animate); //循环调用函数
        mesh.rotation.x += 0.01; //每帧网格模型的沿x轴旋转0.01弧度
        mesh.rotation.y += 0.02; //每帧网格模型的沿y轴旋转0.02弧度
        stats.update(); //更新性能检测框
        renderer.render(scene, camera); //渲染界面
      }

      //性能检测框
      function initStats() {
        stats = new Stats();
        document.body.appendChild(stats.dom);
      }

      //初始化函数，页面加载完成是调用
      function init() {
        initRenderer();
        initScene();
        initCamera();
        initMesh();
        initStats();
        animate();
      }
    </script>
  </body>
</html>
```

---
收录时间: 2021-07-27

<Vssue :title="$title" />
