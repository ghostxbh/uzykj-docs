---
title: Threejs学习和总结基础篇 - 相机Camera 
date: 2021-08-05 
tags:
    - Threejs 
    - 3D 
    - WebGL 
    - Camera 
author: aigisss 
location: blog 
summary: 相机是 Three.js 抽象出来的一个对象，使用此对象，我们可以定义显示的内容，并且可以通过移动相机位置来显示不同的内容。
---

# 相机Camera

相机是 Three.js 抽象出来的一个对象，使用此对象，我们可以定义显示的内容，并且可以通过移动相机位置来显示不同的内容。

下面讲解一下 Three.js 中相机的通用属性和常用的相机对象。

## 相机通用属性和方法

我们常用的相机有正交相机（OrthographicCamera）和透视相机（PerspectiveCamera）两种，用于来捕获场景内显示的物体模型。它们有一些通用的属性和方法。

### Object3D 的所有属性和方法

由于相机都继承自 `THREE.Object3D` 对象，所以像设置位置的 position 属性、rotation 旋转和 scale 缩放属性，可以直接对相机对象设置。 我们甚至还可以使用 add()
方法，给相机对象添加子类，移动相机它的子类也会跟随着一块移动，我们可以使用这个特性制作一些比如 HUD 类型的显示界面。

### target 焦点属性和 lookAt() 方法

这两个方法的效果一定，都是调整相机的朝向，可以设置一个 `THREE.Vector3`（三维向量）点的位置：

```js
camera.target = new THREE.Vector3(0, 0, 0);
camera.lookAt(new THREE.Vector3(0, 0, 0));
```

上面两个都是朝向了原点，我们也可以将相机的朝向改为模型网格的 position，如果物体的位置发生了变化，相机的焦点方向也会跟随变动：

```js
var mesh = new THREE.Mesh(geometry, material);
camera.target = mesh.position;
//或者
camera.lookAt(mesh.position);
```

### getWorldDirection()

`getWorldDirection()` 方法可以获取当前位置到 target 位置的世界中的方向。方向也可以使用 THREE.Vector3 对象表示，所以该方法返回一个三维向量。

## OrthographicCamera 正交相机

使用正交相机 OrthographicCamera 渲染出来的场景，所有的物体和模型都按照它固有的尺寸和精度显示，一般使用在工业要求精度或者 2D 平面中，因为它能完整的显示物体应有的尺寸。

### 创建正交相机

![](https://aigisss.com/blog/posts/89cc5dea/54accde0-7573-11e8-ae69-7f0c27c81098)

上面的图片可以清晰的显示出正交相机显示的范围，它显示的内容是一个立方体结构，通过图片我们发现，只要确定 top、left、right、bottom、near 和 far
六个值，我们就能确定当前相机捕获场景的区域，在这个区域外面的内容不会被渲染，所以，我们创建相机的方法就是：

```js
new THREE.OrthographicCamera(left, right, top, bottom, near, far);
```

下面我们创建了一个显示场景中相机位置前方长宽高都为4的盒子内的物体的正交相机：

```js
var orthographicCamera = new THREE.OrthographicCamera(-2, 2, 2, -2, 0, 4);
scene.add(orthographicCamera); //一般不需要将相机放置到场景当中，如果需要添加子元素等一些特殊操作，还是需要add到场景内
```

正常情况相机显示的内容需要和窗口显示的内容为同样的比例才能够显示没有被拉伸变形的效果：

```js
var frustumSize = 1000; //设置显示相机前方1000高的内容
var aspect = window.innerWidth / window.innerHeight; //计算场景的宽高比
var orthographicCamera = new THREE.OrthographicCamera(frustumSize * aspect / -2, frustumSize * aspect / 2, frustumSize / 2, frustumSize / -2, 1, 2000); //根据比例计算出left，top，right，bottom的值
```

### 动态修改正交相机属性

我们也可以动态的修改正交相机的一些属性，注意修改完以后需要调用相机 updateProjectionMatrix() 方法来更新相机显存里面的内容，代码如下：

```js
var frustumSize = 1000; //设置显示相机前方1000高的内容
var aspect = window.innerWidth / window.innerHeight; //计算场景的宽高比
var orthographicCamera = new THREE.OrthographicCamera(); //实例化一个空的正交相机
orthographicCamera.left = frustumSize * aspect / -2; //设置left的值
orthographicCamera.right = frustumSize * aspect / 2; //设置right的值
orthographicCamera.top = frustumSize / 2; //设置top的值
orthographicCamera.bottom = frustumSize / -2; //设置bottom的值
orthographicCamera.near = 1; //设置near的值
orthographicCamera.far = 2000; //设置far的值

//注意，最后一定要调用updateProjectionMatrix()方法更新
orthographicCamera.updateProjectionMatrix();
```

### 窗口变动后的更新

由于浏览器的窗口可以随意修改，我们有时候需要监听浏览器窗口的变化，然后获取到最新的宽高比，再重新设置相关属性：

```js
var aspect = window.innerWidth / window.innerHeight; //重新获取场景的宽高比

//重新设置left right top bottom 四个值
orthographicCamera.left = frustumSize * aspect / -2; //设置left的值
orthographicCamera.right = frustumSize * aspect / 2; //设置right的值
orthographicCamera.top = frustumSize / 2; //设置top的值
orthographicCamera.bottom = frustumSize / -2; //设置bottom的值

//最后，记得一定要更新数据
orthographicCamera.updateProjectionMatrix();

//显示区域尺寸变了，我们也需要修改渲染器的比例
renderer.setSize(window.innerWidth, window.innerHeight);
```

## PerspectiveCamera 透视相机

透视相机是最常用的也是模拟人眼视角的一种相机，它所渲染生成的页面是一种近大远小的效果。

### 创建透视相机

![](https://aigisss.com/blog/posts/89cc5dea/e27d7170-7577-11e8-a337-9bdf53091bce)

上面的图片就是一个透视相机的生成原理，我们先看看渲染的范围是如何生成的：

- 首先，我们需要确定一个 fov 值，这个值是用来确定相机前方的垂直视角，角度越大，我们能够查看的内容就越多。
- 然后，我们又确定了一个渲染的宽高比，这个宽高比最好设置成页面显示区域的宽高比，这样我们查看生成画面才不会出现拉伸变形的效果，这时，我们可以确定前面生成内容的范围是一个四棱锥的区域。
- 最后，我们需要确定的就是相机渲染范围的最小值 near 和最大值 far，注意，这两个值都是距离相机的距离，确定完数值后，相机会显示的范围就是一个近小远大的四棱柱的范围，我们能够看到的内容都是在这个范围内的。

通过上面的原理，我们需要设置 fov 垂直角度，aspect 视角宽高比例和 near 最近渲染距离 far 最远渲染距离，就能确定当前透视相机的渲染范围。

下面代码展示了一个透视相机的创建方法：

```js
var perspectiveCamera = new THREE.PerspectiveCamera(45, width / height, 1, 1000);
scene.add(perspectiveCamera);
```

我们设置了前方的视角为45度，宽度和高度设置成显示窗口的宽度除以高度的比例即可，显示距离为1到1000距离以内的物体。

### 动态修改透视相机的属性

透视相机的属性创建完成后我们也可以根据个人需求随意修改，但是注意，相机的属性修改完成后，以后要调用 `updateProjectionMatrix()` 方法来更新：

```js
var perspectiveCamera = new THREE.PerspectiveCamera(45, width / height, 1, 1000);
scene.add(perspectiveCamera);

//下面为修改当前相机属性
perspectiveCamera.fov = 75; //修改相机的fov
perspectiveCamera.aspect = window.innerWidth / window.innerHeight; //修改相机的宽高比
perspectiveCamera.near = 100; //修改near
perspectiveCamera.far = 500; //修改far

//最后更新
perspectiveCamera.updateProjectionMatrix();
```

### 显示窗口变动后的回调

如果当前场景浏览器的显示窗口变动了，比如修改了浏览器的宽高后，我们需要设置场景自动更新，下面是一个常用的案例：

```js
function onWindowResize() {
  camera.aspect = window.innerWidth / window.innerHeight; //重新设置宽高比
  camera.updateProjectionMatrix(); //更新相机
  renderer.setSize(window.innerWidth, window.innerHeight); //更新渲染页面大小
}

window.onresize = onWindowResize;
```

最后，我写了一个案例来查看透视相机和正交相机的显示区别：[点击这里案例 Demo](https://johnson2heng.github.io/GitChat-Three.js/07%E7%AC%AC%E4%B8%83%E8%8A%82%20camera/index.html)
。

左侧是透视相机显示的效果，这种更符合人眼看到的效果，场景更加的立体。而右侧则是正交相机实现的效果，渲染出来的数值更加准确，但是却不符合人眼查看的效果。

案例代码:

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>正交相机和透视相机的对比</title>
    <style type="text/css">
        html, body {
            margin: 0;
            height: 100%;
        }

        canvas {
            display: block;
        }

    </style>
</head>
<body onload="draw();">

</body>
<script src="../js/three.js"></script>
<script src="../js/stats.min.js"></script>
<script src="../js/dat.gui.min.js"></script>
<script>
    var renderer, camera, scene, gui, stats;
    var group, perspectiveCamera, orthographicCamera;

    function initRender() {
        renderer = new THREE.WebGLRenderer({antialias: true});
        renderer.setSize(window.innerWidth, window.innerHeight);

        renderer.autoClear = false; //设置场景不自动清除内容，才能够让两个相机同时渲染页面
        //告诉渲染器需要阴影效果
        renderer.shadowMap.enabled = true;
        renderer.shadowMap.type = THREE.PCFSoftShadowMap; // 默认的是，没有设置的这个清晰 THREE.PCFShadowMap
        document.body.appendChild(renderer.domElement);
    }

    function initCamera() {

        var aspect = window.innerWidth / window.innerHeight / 2;

        //创建透视相机
        perspectiveCamera = new THREE.PerspectiveCamera(45, aspect, 0.1, 1000);

        //创建正交相机
        var size = 200;
        orthographicCamera = new THREE.OrthographicCamera(size * aspect / -2, size * aspect / 2, size / 2, size / -2, 0.1, 1000);

        //两个相机设置相同的位置
        perspectiveCamera.position.set(0, 100, 200);
        orthographicCamera.position.set(0, 100, 200);

        //设置两个相机焦点都为原点
        perspectiveCamera.lookAt(new THREE.Vector3());
        orthographicCamera.lookAt(new THREE.Vector3());

        //添加到场景
        scene.add(perspectiveCamera);
        scene.add(orthographicCamera);

    }

    function initScene() {
        scene = new THREE.Scene();
    }

    function initGui() {
        //声明一个保存需求修改的相关数据的对象
        gui = {
            auto: false, //自动旋转
            reset: function () {
                group.rotation.y = 0;
            }
        };

        var datGui = new dat.GUI();
        //将设置属性添加到gui当中，gui.add(对象，属性，最小值，最大值）
        datGui.add(gui, "auto").name("自动旋转");
        datGui.add(gui, "reset").name("重置");
    }

    function initLight() {
        var ambientLight = new THREE.AmbientLight("#111111");
        scene.add(ambientLight);

        var directionalLight = new THREE.DirectionalLight("#ffffff");
        directionalLight.position.set(40, 60, 10);

        directionalLight.shadow.camera.near = 0; //产生阴影的最近距离
        directionalLight.shadow.camera.far = 200; //产生阴影的最远距离
        directionalLight.shadow.camera.left = -50; //产生阴影距离位置的最左边位置
        directionalLight.shadow.camera.right = 50; //最右边
        directionalLight.shadow.camera.top = 50; //最上边
        directionalLight.shadow.camera.bottom = -50; //最下面

        //这两个值决定生成阴影密度 默认512
        directionalLight.shadow.mapSize.height = 1024;
        directionalLight.shadow.mapSize.width = 1024;

        //告诉平行光需要开启阴影投射
        directionalLight.castShadow = true;

        scene.add(directionalLight);
    }

    function initModel() {

        group = new THREE.Group(); //创建一个模型组

        //球体
        var sphereGeometry = new THREE.SphereGeometry(5, 24, 16);
        var sphereMaterial = new THREE.MeshPhongMaterial({color: 0xff00ff});

        var sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);

        sphere.position.x = -10;
        sphere.position.z = -10;

        sphere.castShadow = true; //开启阴影

        group.add(sphere);

        //立方体
        var cubeGeometry = new THREE.BoxGeometry(10, 10, 10);

        var cubeMaterial = new THREE.MeshLambertMaterial({color: 0x00ffff});

        var cube = new THREE.Mesh(cubeGeometry, cubeMaterial);
        cube.position.x = 20;
        cube.position.z = 20;

        cube.castShadow = true; //开启阴影

        group.add(cube);

        //底部平面
        var planeGeometry = new THREE.PlaneGeometry(100, 100);
        var planeMaterial = new THREE.MeshLambertMaterial({color: 0xaaaaaa});

        var plane = new THREE.Mesh(planeGeometry, planeMaterial);
        plane.rotation.x = -0.5 * Math.PI;
        plane.position.y = -5;

        plane.receiveShadow = true; //可以接收阴影

        group.add(plane);

        scene.add(group);

    }

    function initStats() {
        stats = new Stats();
        document.body.appendChild(stats.dom);
    }

    function render() {

        if (gui.auto) {
            group.rotation.y += 0.01;
        }

        renderer.clear(true, true, true); //手动清除场景

        var size = renderer.getSize(); //获取到当前的显示区域的宽高

        renderer.setViewport(0, 0, size.width / 2, size.height); //设置视口，从 (x, y) 到 (x + width, y + height)。
        renderer.render(scene, perspectiveCamera);

        renderer.setViewport(size.width / 2, 0, size.width / 2, size.height);
        renderer.render(scene, orthographicCamera);
    }

    //窗口变动触发的函数
    function onWindowResize() {
        var aspect = window.innerWidth / window.innerHeight / 2;

        //更新透视相机
        perspectiveCamera.aspect = aspect;
        perspectiveCamera.updateProjectionMatrix();

        //更新正交相机
        var size = 200;
        orthographicCamera.left = size * aspect / -2;
        orthographicCamera.right = size * aspect / 2;
        orthographicCamera.top = size / 2;
        orthographicCamera.bottom = size / -2;
        orthographicCamera.updateProjectionMatrix();

        renderer.setSize(window.innerWidth, window.innerHeight);
    }

    function animate() {
        //更新控制器
        render();

        //更新性能插件
        stats.update();

        requestAnimationFrame(animate);
    }

    function draw() {
        initGui();
        initRender();
        initScene();
        initCamera();
        initLight();
        initModel();
        initStats();

        animate();
        window.onresize = onWindowResize;
    }
</script>
</html>
```

制作相机控制器 在实际生产当中，很多时候我们需要切换相机的位置，以达到项目需求。 Three.js 也有很多相机控制插件，这个我们会在后面的课程当中讲解。

下面是我书写的一个小案例，能够通过鼠标左键拖拽界面让相机围绕模型周围查看模型。

首先，先上案例地址：[点击这里](https://johnson2heng.github.io/GitChat-Three.js/07%E7%AC%AC%E4%B8%83%E8%8A%82%20camera/control.html) 。

代码查看:

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>控制相机移动案例</title>
    <style type="text/css">
        html, body {
            margin: 0;
            height: 100%;
        }

        canvas {
            display: block;
        }

    </style>
</head>
<body onload="draw();">

</body>
<script src="../js/three.js"></script>
<script src="../js/stats.min.js"></script>
<script src="../js/dat.gui.min.js"></script>
<script>
    var renderer, camera, scene, gui, stats, ambientLight, directionalLight;

    function initRender() {
        renderer = new THREE.WebGLRenderer({antialias: true});
        renderer.setSize(window.innerWidth, window.innerHeight);
        //告诉渲染器需要阴影效果
        renderer.shadowMap.enabled = true;
        renderer.shadowMap.type = THREE.PCFSoftShadowMap; // 默认的是，没有设置的这个清晰 THREE.PCFShadowMap
        document.body.appendChild(renderer.domElement);
    }

    function initCamera() {
        camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(0, 100, 200);
        camera.lookAt(new THREE.Vector3(0, 0, 0));
    }

    function initScene() {
        scene = new THREE.Scene();
    }

    function initGui() {
        //声明一个保存需求修改的相关数据的对象
        gui = {};

        var datGui = new dat.GUI();
        //将设置属性添加到gui当中，gui.add(对象，属性，最小值，最大值）
    }

    function initLight() {
        ambientLight = new THREE.AmbientLight("#111111");
        scene.add(ambientLight);

        directionalLight = new THREE.DirectionalLight("#ffffff");
        directionalLight.position.set(40, 60, 10);

        directionalLight.shadow.camera.near = 1; //产生阴影的最近距离
        directionalLight.shadow.camera.far = 400; //产生阴影的最远距离
        directionalLight.shadow.camera.left = -50; //产生阴影距离位置的最左边位置
        directionalLight.shadow.camera.right = 50; //最右边
        directionalLight.shadow.camera.top = 50; //最上边
        directionalLight.shadow.camera.bottom = -50; //最下面

        //这两个值决定生成阴影密度 默认512
        directionalLight.shadow.mapSize.height = 1024;
        directionalLight.shadow.mapSize.width = 1024;

        //告诉平行光需要开启阴影投射
        directionalLight.castShadow = true;

        scene.add(directionalLight);
    }

    function initModel() {

        //球体
        var sphereGeometry = new THREE.SphereGeometry(5, 24, 16);
        var sphereMaterial = new THREE.MeshStandardMaterial({color: 0xff00ff});

        var sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);

        sphere.castShadow = true; //开启阴影

        directionalLight.target = sphere; //平行光的焦点到球

        scene.add(sphere);

        //立方体
        var cubeGeometry = new THREE.CubeGeometry(10, 10, 10);

        var cubeMaterial = new THREE.MeshPhongMaterial({color: 0x00ffff});

        var cube = new THREE.Mesh(cubeGeometry, cubeMaterial);
        cube.position.x = 30;
        cube.position.z = -5;

        cube.castShadow = true; //开启阴影

        scene.add(cube);

        //底部平面
        var planeGeometry = new THREE.PlaneGeometry(100, 100);
        var planeMaterial = new THREE.MeshLambertMaterial({color: 0xaaaaaa, side: THREE.DoubleSide});

        var plane = new THREE.Mesh(planeGeometry, planeMaterial);
        plane.rotation.x = -0.5 * Math.PI;
        plane.position.y = -5.1;

        plane.receiveShadow = true; //可以接收阴影

        scene.add(plane);

    }

    function initStats() {
        stats = new Stats();
        document.body.appendChild(stats.dom);
    }

    function initControl() {
        var dom = renderer.domElement;
        //鼠标按下时获取到当前相机的位置，并求出当前相机距离原点的位置
        var distance; //相机距离中心点的距离
        var pan, tilt; //相机的水平角度和垂直角度
        var downX, downY; //鼠标按下时的xy坐标
        var matrix = new THREE.Matrix4(); //声明一个旋转矩阵

        //绑定按下事件
        dom.addEventListener("mousedown", function (e) {
            distance = computeDistance(camera.position, new THREE.Vector3());
            computePanTilt(camera.position);

            downX = e.clientX;
            downY = e.clientY;

            //绑定移动事件
            document.addEventListener("mousemove", move);

            //绑定鼠标抬起事件
            document.addEventListener("mouseup", up);
        });

        //鼠标移动事件
        function move(e) {
            var moveX = e.clientX;
            var moveY = e.clientY;

            //计算鼠标的偏移量当前相机的pan和tilt
            var offsetX = moveX - downX;
            var offsetY = moveY - downY;

            var movePan = pan - offsetX / 3;
            var moveTilt = tilt - offsetY;

            //tilt的移动范围是90到-90度
            if (moveTilt >= 90) {
                moveTilt = 89.99;
            }

            if (moveTilt <= -90) {
                moveTilt = -89.99;
            }

            //根据pan和tilt计算出当前的相机应该所在的位置
            var p = computePosition(distance, movePan, moveTilt);
            camera.position.set(p.x, p.y, p.z);
            camera.lookAt(new THREE.Vector3());
        }

        //鼠标抬起事件
        function up() {
            //清楚绑定事件
            document.removeEventListener("mousemove", move);
            document.removeEventListener("mouseup", up);
        }

        //计算两点位置的距离
        function computeDistance(vec1, vec2) {
            return vec1.distanceTo(vec2);
        }

        //根据当前点到原点的线计算出到原点z轴正方向的pan和tilt的偏移量
        function computePanTilt(position) {
            //首先计算水平的偏移角度
            pan = new THREE.Vector3(position.x, 0, position.z).angleTo(new THREE.Vector3(0, 0, 1));

            pan = pan / Math.PI * 180;
            if (position.x < 0) {
                pan = 360 - pan;
            }

            //计算垂直的偏移角度
            tilt = new THREE.Vector3(position.x, 0, position.z).angleTo(position);

            tilt = tilt / Math.PI * 180;
            if (position.y > 0) {
                tilt = -tilt;
            }
        }

        //根据pan和tilt 相机到原点的距离计算出相机当前所在的位置
        function computePosition(distance, pan, tilt) {
            //重置旋转矩阵
            matrix.identity();

            var v = new THREE.Vector3(0, 0, distance);

            //根据pan和tilt修改旋转矩阵，注意顶点旋转计算需要按照xyz的顺序旋转

            matrix.makeRotationX(tilt / 180 * Math.PI);
            v.applyMatrix4(matrix);

            matrix.makeRotationY(pan / 180 * Math.PI);
            v.applyMatrix4(matrix);

            //计算出当前相机的位置
            return v;
        }
    }

    function render() {
        renderer.render(scene, camera);
    }

    function onWindowResize() {

        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);

    }

    function animate() {
        //更新控制器
        render();

        //更新性能插件
        stats.update();

        requestAnimationFrame(animate);
    }

    function draw() {
        initGui();
        initRender();
        initScene();
        initCamera();
        initLight();
        initModel();
        initStats();

        initControl();

        animate();
        window.onresize = onWindowResize;
    }
</script>
</html>
```

其实和以前的代码相比，我们也只是多出来了一个 initControl 方法，在这个方法里面绑定鼠标事件。实现的思路也很简单：

- 首先绑定鼠标按下事件，获取到按下时的相机位置和距离原点的距离，计算出来相对于 Z 轴正方向的偏移。绑定鼠标移动事件。
- 然后，在鼠标移动事件里面获取到距离鼠标按下的偏移，通过鼠标偏移数值计算出现在相机位置，并赋值。


---
收录时间: 2021-08-05

<Vssue :title="$title" />
