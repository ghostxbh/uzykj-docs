---
title: Threejs学习和总结基础篇 - 粒子Points
date: 2021-08-06
tags:
    - Threejs
    - 3D
    - WebGL
    - Points
author: aigisss
location: blog
summary: 我们将学习到 Sprite 精灵和 Points 粒子，这两种对象共同点就是我们通过相机查看它们时，始终看到的是它们的正面，它们总朝向相机。通过它们的这种特性，我们可以实现广告牌的效果，或实现更多的比如雨雪、烟雾等更加绚丽的特效。
---

# 粒子Points
我们将学习到 Sprite 精灵和 Points 粒子，这两种对象共同点就是我们通过相机查看它们时，始终看到的是它们的正面，它们总朝向相机。
通过它们的这种特性，我们可以实现广告牌的效果，或实现更多的比如雨雪、烟雾等更加绚丽的特效。

## Sprite 精灵
精灵由于一直正对着相机的特性，一般使用在模型的提示信息当中。通过 THREE.Sprite 创建生成，
由于 `THREE.Sprite` 和 `THREE.Mesh` 都属于 `THREE.Object3D` 的子类，所以，我们操作模型网格的相关属性和方法大部分对于精灵都适用。
和精灵一起使用的还有一个 `THREE.SpriteMaterial` 对象，它专门配合精灵的材质。注意，精灵没有阴影效果。

下面首先创建一个最简单的精灵：
```js
var spriteMaterial = new THREE.SpriteMaterial( { color: 0xffffff } );
var sprite = new THREE.Sprite( spriteMaterial );
scene.add( sprite );
```
创建一个带有纹理图片的精灵：
```js
var spriteMap = new THREE.TextureLoader().load( "sprite.png" );
var spriteMaterial = new THREE.SpriteMaterial( { map: spriteMap, color: 0xffffff } );
var sprite = new THREE.Sprite( spriteMaterial );
scene.add( sprite );
```
直接使用 canvas 创建精灵：
```js
var spriteMap = new THREE.Texture(canvas);
var spriteMaterial = new THREE.SpriteMaterial( { map: spriteMap, color: 0xffffff } );
var sprite = new THREE.Sprite( spriteMaterial );
scene.add( sprite );
```

### 相关属性
精灵特有的属性就一个，即 center 属性，值是一个二维向量 THREE.Vector2，这个属性意义就是当前设置的精灵位置点的位置处于精灵图中的位置。
如果 center 的值是 0,0旋转的位置就在左下角，如果值为 1,1 的话，那旋转的位置则在精灵图的右上角，默认值是 0.5,0.5：
```js
sprite.center.set(0.5, 0); //设置位置点处于精灵的最下方中间位置
```

接下来，我们查看一下精灵的案例：[点击这里](https://johnson2heng.github.io/GitChat-Three.js/08%E7%AC%AC%E5%85%AB%E8%8A%82%20points/sprite.html) 。

案例代码:
```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>精灵案例</title>
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

        //创建一个最普通的精灵
        var spriteMaterialNormal = new THREE.SpriteMaterial({color: 0x00ffff});
        var spriteNormal = new THREE.Sprite(spriteMaterialNormal);
        spriteNormal.position.set(-30, 10, 0); //设置位置
        spriteNormal.scale.set(5, 5, 1); //设置scale进行大小设置
        scene.add(spriteNormal);

        //球体
        var sphereGeometry = new THREE.SphereGeometry(5, 24, 16);
        var sphereMaterial = new THREE.MeshStandardMaterial({color: 0xff00ff});
        var sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);
        sphere.castShadow = true; //开启阴影
        directionalLight.target = sphere; //平行光的焦点到球
        scene.add(sphere);

        //添加一个精灵 使用了将canvas生成img的src导入的方式
        var spriteMap = new THREE.TextureLoader().load(drawCanvas({text: "球", width: 64, height: 64}).toDataURL());
        var spriteMaterial = new THREE.SpriteMaterial({map: spriteMap, color: 0xffffff});
        var sprite = new THREE.Sprite(spriteMaterial);
        sprite.position.set(0, 10, 0); //设置位置
        sprite.scale.set(5, 5, 1); //设置scale进行大小设置
        scene.add(sprite);

        //立方体
        var cubeGeometry = new THREE.CubeGeometry(10, 10, 10);
        var cubeMaterial = new THREE.MeshPhongMaterial({color: 0x00ffff});
        var cube = new THREE.Mesh(cubeGeometry, cubeMaterial);
        cube.position.x = 30;
        cube.position.z = -5;
        cube.castShadow = true; //开启阴影
        scene.add(cube);

        //添加一个精灵
        var canvas = drawCanvas({text: "立方体", width: 256, height: 64});
        var spriteMapCube = new THREE.Texture(canvas);
        spriteMapCube.wrapS = THREE.RepeatWrapping;
        spriteMapCube.wrapT = THREE.RepeatWrapping;
        spriteMapCube.needsUpdate = true;

        var spriteCube = new THREE.Sprite(new THREE.SpriteMaterial({map: spriteMapCube, color: 0xffffff}));
        spriteCube.position.set(30, 10, -5); //设置位置
        spriteCube.scale.set(20, 5, 1); //设置scale进行大小设置
        spriteCube.center.set(0.5, 0); //设置位置点处于精灵的最下方中间位置
        scene.add(spriteCube);

        //底部平面
        var planeGeometry = new THREE.PlaneGeometry(100, 100);
        var planeMaterial = new THREE.MeshLambertMaterial({color: 0xaaaaaa, side: THREE.DoubleSide});
        var plane = new THREE.Mesh(planeGeometry, planeMaterial);
        plane.rotation.x = -0.5 * Math.PI;
        plane.position.y = -5.1;
        plane.receiveShadow = true; //可以接收阴影
        scene.add(plane);

    }

    //创建canvas对象
    function drawCanvas(options) {
        let canvas = document.createElement("canvas");
        canvas.width = options.width;
        canvas.height = options.height;

        let ctx = canvas.getContext("2d");
        ctx.fillStyle = "rgba(0, 0, 0, 0)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.font = "60px Verdana";
        ctx.fillStyle = "#fff";
        ctx.fillText(options.text, 0, 56, options.width);
        return canvas;
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

案例效果从左到右依次是，普通的精灵，贴图纹理的精灵和 `canvas` 创建的精灵。

## points 粒子
粒子和精灵的效果是一样的，它们之间的区别是，如果当前场景内的精灵过多的话，就会出现性能问题。
粒子的作用就是为解决很多精灵而出现的，我们可以使用粒子去模型数量很多的效果，比如下雨，下雪等，数量很多的时候就适合使用粒子来创建，相应的，
提高性能的损失就是失去了对单个精灵的操作，所有的粒子的效果都是一样。总的来说，粒子就是提高性能减少的一些自由度，而精灵就是为了自由度而损失了一些性能。

### 粒子的创建
粒子 `THREE.Points` 和精灵 `THREE.Sprite` 还有网格 `THREE.Mesh` 都属于 `THREE.Object3D` 的一个扩展，
但是粒子有一些特殊的情况就是 `THREE.Points` 是它们粒子个体的父元素，它的位置设置也是基于 `THREE.Points` 位置而定位，
而修改 `THREE.Points` 的 `scale` 属性只会修改掉粒子个体的位置。下面我们看下一个粒子的创建方法。创建一个粒子，需要一个含有顶点的几何体，
和粒子纹理 `THREE.PointsMaterial` 的创建：
```js
//球体
var sphereGeometry = new THREE.SphereGeometry(5, 24, 16);
var sphereMaterial = new THREE.PointsMaterial({color: 0xff00ff});
var sphere = new THREE.Points(sphereGeometry, sphereMaterial);
scene.add(sphere);
```
上面是通过球体几何体创建的一个最简单的粒子特效。

使用任何几何体都可以，甚至自己生成的几何体都可以，比如创建星空的案例：
```js
var starsGeometry = new THREE.Geometry();
//生成一万个点的位置
for (var i = 0; i < 10000; i++) {
    var star = new THREE.Vector3();
    //THREE.Math.randFloatSpread 在区间内随机浮动* - 范围 / 2 *到* 范围 / 2 *内随机取值。
    star.x = THREE.Math.randFloatSpread(2000);
    star.y = THREE.Math.randFloatSpread(2000);
    star.z = THREE.Math.randFloatSpread(2000);
    starsGeometry.vertices.push(star);
}
var starsMaterial = new THREE.PointsMaterial({color: 0x888888});
var starField = new THREE.Points(starsGeometry, starsMaterial);
scene.add(starField);
```
使用一个空的几何体，将自己创建的顶点坐标放入，也可以实现一组粒子的创建。如果我们需要单独设置每一个粒子的颜色，
可以给 geometry 的 colors 数组添加相应数量的颜色：
```js
for (var i = 0; i < 10000; i++) {
    var star = new THREE.Vector3();
    //THREE.Math.randFloatSpread 在区间内随机浮动* - 范围 / 2 *到* 范围 / 2 *内随机取值。
    star.x = THREE.Math.randFloatSpread(2000);
    star.y = THREE.Math.randFloatSpread(2000);
    star.z = THREE.Math.randFloatSpread(2000);
    starsGeometry.vertices.push(star);

    starsGeometry.colors.push(new THREE.Color("rgb("+Math.random()*255+", "+Math.random()*255+", "+Math.random()*255+")")); //添加一个随机颜色
}
```

### THREE.PointsMaterial 粒子的纹理
如果我们需要设置粒子的样式，还是需要通过设置 `THREE.PointsMaterial` 属性实现：
```js
var pointsMaterial = new THREE.PointsMaterial({color: 0xff00ff}); //设置了粒子纹理的颜色
```
我们还可以通过 PointsMaterial 的 size 属性设置粒子的大小：
```js
var pointsMaterial = new THREE.PointsMaterial({color: 0xff00ff, size:4}); //粒子的尺寸改为原来的四倍
//或者直接设置属性
pointsMaterial.size = 4;
```
我们也可以给粒子设置纹理：
```js
var pointsMaterial = new THREE.PointsMaterial({color: 0xff00ff, map:texture}); //添加纹理
```
默认粒子是不受光照影响的，我们可以设置 lights 属性为 true，让粒子受光照影响：
```js
var pointsMaterial = new THREE.PointsMaterial({color: 0xff00ff, lights:true}); 
//或者
pointsMaterial.lights = true; //开启受光照影响
```
我们也可以设置粒子不受到距离的影响产生近大远小的效果：
```js
var pointsMaterial = new THREE.PointsMaterial({color: 0xff00ff, sizeAttenuation: false}); 
//或者
pointsMaterial.sizeAttenuation = false; //关闭粒子的显示效果受距离影响
```

粒子的效果就介绍到这里，希望大家熟练了以后能够做出来各种各样的花哨效果。

接下来展示下我给大家准备的粒子案例：[点击这里](https://johnson2heng.github.io/GitChat-Three.js/08%E7%AC%AC%E5%85%AB%E8%8A%82%20points/points.html) 。

代码查看:
```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>粒子案例</title>
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

        //创建一个最普通的精灵
        var spriteMaterialNormal = new THREE.SpriteMaterial({color: 0x00ffff});
        var spriteNormal = new THREE.Sprite(spriteMaterialNormal);
        spriteNormal.position.set(-30, 10, 0); //设置位置
        spriteNormal.scale.set(5, 5, 1); //设置scale进行大小设置
        scene.add(spriteNormal);

        //球体
        var sphereGeometry = new THREE.SphereGeometry(5, 24, 16);
        var sphereMaterial = new THREE.PointsMaterial({color: 0xff00ff});
        var sphere = new THREE.Points(sphereGeometry, sphereMaterial);
        scene.add(sphere);

        //添加一个精灵 使用了将canvas生成img的src导入的方式
        var spriteMap = new THREE.TextureLoader().load(drawCanvas({text:"球", width:64, height:64}).toDataURL());
        var spriteMaterial = new THREE.SpriteMaterial({map: spriteMap, color: 0xffffff});
        var sprite = new THREE.Sprite(spriteMaterial);
        sprite.position.set(0, 10, 0); //设置位置
        sprite.scale.set(5, 5, 1); //设置scale进行大小设置
        scene.add(sprite);

        //立方体
        var texture = new THREE.Texture(createPoint());
        texture.needsUpdate = true;
        var cubeGeometry = new THREE.CubeGeometry(10, 10, 10);
        var cubeMaterial = new THREE.PointsMaterial({map: texture, size:5, transparent : true});
        var cube = new THREE.Points(cubeGeometry, cubeMaterial);
        cube.position.x = 30;
        cube.position.z = -5;
        scene.add(cube);

        //添加一个精灵
        var canvas = drawCanvas({text:"立方体", width:256, height:64});
        var spriteMapCube = new THREE.Texture(canvas);
        spriteMapCube.wrapS = THREE.RepeatWrapping;
        spriteMapCube.wrapT = THREE.RepeatWrapping;
        spriteMapCube.needsUpdate = true;

        var spriteCube = new THREE.Sprite(new THREE.SpriteMaterial({map: spriteMapCube, color: 0xffffff}));
        spriteCube.position.set(30, 10, -5); //设置位置
        spriteCube.scale.set(20, 5, 1); //设置scale进行大小设置
        spriteCube.center.set(0.5, 0); //设置位置点处于精灵的最下方中间位置
        scene.add(spriteCube);

        //底部平面
        var planeGeometry = new THREE.PlaneGeometry(100, 100);
        var planeMaterial = new THREE.MeshLambertMaterial({color: 0xaaaaaa, side: THREE.DoubleSide});
        var plane = new THREE.Mesh(planeGeometry, planeMaterial);
        plane.rotation.x = -0.5 * Math.PI;
        plane.position.y = -6.1;
        plane.receiveShadow = true; //可以接收阴影
        scene.add(plane);

        //创建一个星空的效果
        var starsGeometry = new THREE.Geometry();
        //生成一万个点的位置
        for (var i = 0; i < 10000; i++) {

            var star = new THREE.Vector3();
            //THREE.Math.randFloatSpread 在区间内随机浮动* - 范围 / 2 *到* 范围 / 2 *内随机取值。
            star.x = THREE.Math.randFloatSpread(2000);
            star.y = THREE.Math.randFloatSpread(2000);
            star.z = THREE.Math.randFloatSpread(2000);

            starsGeometry.vertices.push(star);

        }
        var starsMaterial = new THREE.PointsMaterial({color: 0x888888});
        var starField = new THREE.Points(starsGeometry, starsMaterial);
        scene.add(starField);
    }

    //创建canvas对象
    function drawCanvas(options) {
        let canvas = document.createElement("canvas");
        canvas.width = options.width;
        canvas.height = options.height;

        let ctx = canvas.getContext("2d");
        ctx.fillStyle = "rgba(0, 0, 0, 0)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.font = "60px Verdana";
        ctx.fillStyle = "#fff";
        ctx.fillText(options.text, 0, 56, options.width);
        return canvas;
    }

    //使用canvas绘制一个亮点
    function createPoint() {
        var canvas = document.createElement('canvas');
        canvas.width = 512;
        canvas.height = 512;
        var context = canvas.getContext('2d');
        var gradient = context.createRadialGradient(canvas.width / 2, canvas.height / 2, 0, canvas.width / 2, canvas.height / 2, canvas.width / 2);
        gradient.addColorStop(0, 'rgba(255,255,255,1)');
        gradient.addColorStop(0.2, 'rgba(0,255,255,1)');
        gradient.addColorStop(0.4, 'rgba(0,0,64,1)');
        gradient.addColorStop(1, 'rgba(0,0,0,0)');
        context.fillStyle = gradient;
        context.fillRect(0, 0, canvas.width, canvas.height);
        return canvas;
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

这个案例和精灵的案例区别就是，将球体改成了粒子，然后将立方体修改成了带有 canvas 纹理的粒子，并且在背景里面添加了一万个粒子

---
收录时间: 2021-08-06

<Vssue :title="$title" />
