---
title: 法向贴图
tags: JS插件
date: 2017-04-22 14:34:13
---
皮肤的皱纹，砖墙表面的凹凸，衣服的褶皱，树皮的纹路...
这些丰富的细节在计算机的3D世界中，最常用的展现方式就是法相贴图了。
<!-- more -->
对于高模来说，如果精度足够高，近似认为一个面就是一个点，那么将其贴图揭下就是一张图片。
这张高模贴图上的**每个点拥有一个法向量**，反应出高模上的细节。
对于低模来说，一个面对应高模中的很多个面，每个面看作是一面镜子，将其贴图揭下来就是一张很多三角形镜子拼接的图，**每个三角形内的所有点拥有相同的法向量**。
![低模贴图示意图](low_texture.jpg)

### 为什么需要法向贴图
因为面数量越大，需要计算的量和内存需求就越大，CPU的计算能力是有限的。要让低模也能体现出很多的光照细节，就有人想出赋予低模上的点对应高模上的的法向量，不就可以让低模看起来逼真生动了吗。
将这些法向量存在图片上每个点的rgb值中，让计算机在解析贴图的时候就可以读到每个点的法向量，这样的图片就叫作法向贴图。

### 高模贴图的法线信息如何映射到低模贴图中
不用解释，高模贴图总面积肯定是比低模贴图的总面积大的，在做映射的时候，高模贴图就是一张凹凸不平的图了。
![法向贴图示意图](normals.png)
想象一下，将高模贴图包住低模贴图，然后用针垂直于低模贴图插下去，针连接的高低贴图上的点就是对应点，将高模贴图上的点的x值存入r中，y值存入g中，z值存入b中，就完成了一个点的映射。
颜色的分量取值范围为0到1，而向量的分量取值范围是-1到1；可以建立从纹素到法线的简单映射：
```
normal = (2*color)-1 // on each component
```
由于法线基本都是指向”曲面外侧”的（按照惯例，X轴朝右，Y轴朝上），因此得到的法相贴图看上去就是蓝紫色调。
![法向贴图](normal.jpg)

### 没有高模贴图怎么得到法向贴图
理论上讲，法向贴图离不开高模贴图和低模贴图，缺其一就不能得到一个正确的法向贴图。
但是，很多人都知道Photoshop的nVidia插件就可以生成diffuse贴图，并没有用到高模贴图，这是怎么回事呢？
其实，Photoshop的做法，是分析图片的亮度，越黑表示越深，越白表示越浅。这样的打开方式是不正确的。
更好的方式是根据特定规则将物体实际高低手动画成一张黑白的图片，再结合原图来制作diffuse贴图。
![制作diffuse映射图](create_height.png)
- 50%的灰色意思是高度不变（水平参考线）。
- 白色意思是最高的突起部分。
- 黑色表示最低的凹下部分。

### js代码怎么使用法向贴图
```js
var mesh,
    scene = new THREE.Scene(),
    textureLoader = new THREE.TextureLoader();

textureLoader.load( "./texture/cloud.png", function( texture ){
    // 加载法向贴图
    textureLoader.load("./texture/normal.png", function( normalTexture ){
        var geometry = new THREE.BoxGeometry( 50, 50, 50 );
        var material = new THREE.MeshPhongMaterial({
            map: texture, 
            normalMap: normalTexture // 只要将法向贴图赋给材质的normalMap属性即可
        });
        mesh = new THREE.Mesh( geometry, material );
        scene.add( mesh );
    } );
});
```

[更多关于法线贴图的内容](http://www.opengl-tutorial.org/cn/intermediate-tutorials/tutorial-13-normal-mapping/)
