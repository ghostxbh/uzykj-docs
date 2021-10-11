---
title: Threejs学习和总结基础篇 - 材质Material
date: 2021-08-04
sidebar: 'auto'
categories:
  - Javascript
tags:
  - Threejs
  - 3D
  - WebGL
  - Material
author: aigisss
location: blog
summary: 模型的表现，也就是我们看到的的模型外观——材质。
---

# 材质Material
模型的表现，也就是我们看到的的模型外观——材质。

![](https://aigisss.com/blog/posts/89cc5dea/materials.png)

简单的说，就是物体看起来是什么质地。材质可以看成是材料和质感的结合。
在渲染程序中，它是表面各种可视属性的结合，这些可视属性是指表面的色彩、纹理、光滑度、透明度、反射率、折射率、发光度等。
Three.js 给我们封装好了大部分的材质效果，避免我们使用复杂的 Shader 语言自己去实现。
接下来我们先介绍下 Material 常用的一些属性和方法

## 基本属性和方法
### needsUpdate
如果修改了 Material 内的内容，需要将 needsUpdate 属性设置为 true，
Three.js 会在下一帧里将修改内容同步到 WebGL 的显存内。切记不要在 requestAnimationFrame 方法内更新，
会浪费性能，只需要在更新 Material 属性后设置一次即可。

### side
此属性可以定义当前面的哪个方向会被渲染，默认值是 THREE.FrontSide（只渲染正面），
可选值有：`THREE.BackSide`（只渲染背面）和 `THREE.DoubleSide`（正面和背面都会渲染）。

### transparent
此属性定义了材质是否可以透明，因为对于透明需要材质进行特殊处理，并在不透明的物体渲染完成后再渲染透明物体。
当设置此属性为true后，可以通过设置opacity来调整透明度，默认为false。

### opacity
此属性可以定义材质的透明度，必须将材质的 transparent 设置为 true 才可使透明度管用。
取值范围为 0.0 到 1.0。默认值是1.0，也就是不透明。

### map
此属性可以配置当前材质的纹理贴图，是一个 `THREE.Texture` 对象，下面我们将会讲解如何给材质贴图。
这是大部分材质都会有的属性，只有极其个别的材质如 LineBasicMaterial（线材质）等没有这个属性。

### wireframe
是否将模型渲染成线框，默认为 false。个别材质也没有这个属性。

### dispose()
此方法用于将材质从内存中删除，在不需要使用当前材质时使用，但不会将材质的纹理贴图删除，如果需要将纹理贴图也删除，
需要调用 `material.map.dispose()`。

## 配置纹理贴图
由于经常使用纹理贴图，所以在这里单独讲解一下如何实现一个纹理贴图。 实现纹理贴图有以下两种方式。

第一种，使用 `THREE.TextureLoader` 进行生成纹理对象：
```js
var texture = new THREE.TextureLoader().load( "textures/water.jpg" ); 
material.map = texture; //将纹理赋值给材质
```
或者直接实例化：
```js
var texture = new THREE.Texture(canvas); //实例化的第一个对象可以是`img`、`canvas`和`video`。
material.map = texture; //将纹理赋值给材质
```
### 纹理重复问题
如果图片不是标准的2的幂数（2、4、8、16、32、64、128、256、512、1024、2048……），
在控制台会给我们提示：“THREE.WebGLRenderer: image is not power of two”，
意思就是说图片不是标准格式高宽不是2的幂数。我们需要的水平方向和垂直方向上设置的图片重复显示。
需要配置的两个属性是：`texture.wrapS`（水平方向重复）和 `texture.wrapT`（垂直方向重复），
默认值是：`THREE.ClampToEdgeWrapping`，即纹理的最后一个像素延伸到网格的边缘。
可选项有：`THREE.RepeatWrapping`，表示纹理将重复无穷大；MirroredRepeatWrapping，表示镜像重复，可以理解为重复时，
反着绘制一个然后正着绘制一个，达到的效果就是没有强烈的过渡感觉。

### needsUpdate 属性
如果更新了纹理的相关属性，需要将此属性设置为 true，将数据同步到 WebGL。

### repeat
纹理在整个表面水平方向和垂直方向重复多少次，也会受纹理重复设置的影响，设置方式为：
```js
var texture = new THREE.TextureLoader().load( "textures/water.jpg" );
texture.wrapS = THREE.RepeatWrapping; //设置水平方向无限循环
texture.wrapT = THREE.RepeatWrapping; //设置垂直方向无限循环
texture.repeat.set( 4, 4 ); //水平方向和垂直方向都重复四次
```

## 内置常用材质
在讲解常用材质之前，我们先讲解一下如何实例化一个材质和一些需要注意的地方。
我们使用第一个讲到的材质 MeshBasicMaterial 作为例子。

### MeshBasicMaterial 和设置颜色的方法
这种材质是一种简单的材质，不会受到光的影响，直接看到的效果就是整个物体的颜色都是一样，没有立体的感觉。
在实例化材质时，我们可以传入一个对象，设置材质的相关属性可以通过对象属性的方式传入，
但是属性 color（颜色）例外，实例化的时候可以传入十六进制数，也可以写十六进制字符串。
实例化完成后再修改需要重新赋值 `THREE.Color` 对象，或者调用 `material.color.set` 方法赋值。
```js
var material = new THREE.MeshBasicMaterial({color:0x00ffff});
var geometry = new THREE.BoxGeometry(1, 1, 1);

var mesh = new THREE.Mesh(geometry, material);
scene.add(mesh);
```
上面的案例就是使用 MeshBasicMaterial 材质创建了一个立方体，我们设置了显示颜色为一种浅蓝色，
除了上面实例化的时候进行设置，后面也可以再修改：
```js
var material = new THREE.MeshBasicMaterial({color:0x00ffff}); //设置初始的颜色为浅蓝色
material.color.set(0xff00ff); //将颜色修改为紫色
```
我们也可以直接赋值一个新的 `THREE.Color` 对象，如：
```js
var material = new THREE.MeshBasicMaterial({color:0x00ffff}); //设置初始的颜色为浅蓝色
material.color = new THREE.Color(0xff00ff); //将颜色修改为紫色
```
我们可以通过 `new THREE.Color` 创建一个颜色对象，Three.js 支持的颜色书写方式比较丰富，如：
```js
//直接传入十六进制数或者字符串
var color = new THREE.Color( 0xff0000 );
var color = new THREE.Color( "#ff0000" );

//RGB 字符串
var color = new THREE.Color("rgb(255, 0, 0)");
var color = new THREE.Color("rgb(100%, 0%, 0%)");

//支持一百四十多中颜色名称
var color = new THREE.Color( 'skyblue' );

//HSL 字符串
var color = new THREE.Color("hsl(0, 100%, 50%)");

//支持RGB值设置在0到1之间的方式
var color = new THREE.Color( 1, 0, 0 );
```

### MeshNormalMaterial 法向材质
这种材质会根据面的方向不同自动改变颜色，也是我们之前一直在用的材质。此材质不受灯光影响。
```js
geometry = new THREE.BoxGeometry( 2, 2, 2 ); //创建几何体
material = new THREE.MeshNormalMaterial(); //创建材质

mesh = new THREE.Mesh( geometry, material ); //创建网格
scene.add( mesh ); //将网格添加到场景
```

### LineBasicMaterial 线条材质
在上一篇我们讲几何体时，没有讲解如何画直线，是由于直线需要单独的材质进行实现，所以我们将直线放到了材质这一篇中进行讲解。
注意，由于 Windows 系统的原因，线的宽度只能为1。

要绘制线段，我们需要确定两个点，就是起点和终点，案例中我们使用了四个顶点创建了三条线。
然后 Geometry 对象使用这组顶点配置几何体，实例化线的材质，最后使用 THREE.Line 生成线。
```js
//添加直线
var pointsArr = [
    new THREE.Vector3( -10, 0, -5 ),
    new THREE.Vector3( -5, 15, 5 ),
    new THREE.Vector3( 20, 15, -5 ),
    new THREE.Vector3( 10, 0, 5 )
];

var lineGeometry = new THREE.Geometry(); //实例化几何体
lineGeometry.setFromPoints(pointsArr); //使用当前点的属性配置几何体

var lineMaterial = new THREE.LineBasicMaterial({color:0x00ff00}); //材质

line = new THREE.Line(lineGeometry, lineMaterial);
scene.add(line);
```

### LineDashedMaterial 虚线
我们也可以创建虚线，这里我们来点新花样，就是实现曲线。曲线也和直线一样，在 Windows 系统线的粗度只能为1。

要创建曲线，我们需要使用到 `THREE.CatmullRomCurve3` 来生成一个 curve 对象，这是一个曲线对象，
可以从对象获取生成的曲线的点的集合，在这里科普一下，曲线也是由无数段的直线组成的，段数分的越清晰，曲线过渡越顺滑。
```js
var pointsArr = [
    new THREE.Vector3( -10, 0, -5 ),
    new THREE.Vector3( -5, 15, 5 ),
    new THREE.Vector3( 20, 15, -5 ),
    new THREE.Vector3( 10, 0, 5 )
];
//指定一些用于生成曲线线的三维顶点
var curve = new THREE.CatmullRomCurve3(pointsArr);

var points = curve.getPoints( 50 ); //使用getPoints获取当前曲线分成50段后的所有顶点
var curveGeometry = new THREE.BufferGeometry().setFromPoints( points ); //使用顶点生成几何体

var curveMaterial = new THREE.LineDashedMaterial( { color : 0xff0000 } ); //创建一条红色的线材质

// 使用THREE.Line创建线
curveLine = new THREE.Line( curveGeometry, curveMaterial );
curveLine.computeLineDistances(); //需要重新计算位置才能显示出虚线
scene.add(curveLine);
```

### 添加光
由于 MeshBasicMaterial 不会受光的影响，即使有光也不会影响它的效果，前面我们也没有添加光。
但是后面介绍的材质会受到光源的影响，在介绍之前，我们需要添加一个光源，来影响材质的显示效果。
```js
//创建灯光
function initLight() {
    var light = new THREE.DirectionalLight(0xffffff); //添加了一个白色的平行光
    light.position.set(20, 50, 50); //设置光的方向
    scene.add(light); //添加到场景

    //添加一个全局环境光
    scene.add(new THREE.AmbientLight(0x222222));
}
```
上面我们添加了一个模拟太阳光线的平行光和一个对每一个物理都造成影响的环境光，具体的内容将会在下一篇讲解。

下面介绍的材质都是对光有反应的，而且如果场景内没有光，模型将无法显示。

### MeshLambertMaterial 兰伯特材质
这种材质会对光有反应，但是不会出现高光，可以模拟一些粗糙的材质的物体，比如木头或者石头。

实现案例，如下：
```js
geometry = new THREE.BoxGeometry( 2, 2, 2 ); //创建几何体
material = new THREE.MeshLambertMaterial({color:0x00ffff}); //创建材质

mesh = new THREE.Mesh( geometry, material ); //创建网格
scene.add( mesh ); //将网格添加到场景
```

### MeshPhongMaterial 高光材质
这种材质具有高光效果，可以模拟一些光滑的物体的材质效果，比如油漆面，瓷瓦等光滑物体。

实现案例如下：
```js
geometry = new THREE.BoxGeometry( 2, 2, 2 ); //创建几何体
material = new THREE.MeshPhongMaterial({color:0x00ffff}); //创建材质

mesh = new THREE.Mesh( geometry, material ); //创建网格
scene.add( mesh ); //将网格添加到场景
```

### MeshStandardMaterial 基于物理的渲染（PBR）材质
这种材质基于物理的渲染（PBR）材质，生成的材质效果更佳，但是相应也占用更多的计算量。
这种材质我们可以定义它的粗糙度来确定反光效果，经常用于模拟金属的质感，使金属质感更加真实。

实现案例如下：
```js
geometry = new THREE.BoxGeometry( 2, 2, 2 ); //创建几何体
material = new THREE.MeshPhongMaterial({color:0x00ffff}); //创建材质
material.metalness = 0.1; //设置的值的范围为0-1，值越小，材质越光滑，高光越明显
material.metalnessMap = 0.1; //设置的值的范围为0-1，值越大，越有生锈金属的质感，值越小反光越清晰

mesh = new THREE.Mesh( geometry, material ); //创建网格
scene.add( mesh ); //将网格添加到场景
```

---
收录时间: 2021-08-04

<Vssue :title="$title" />
