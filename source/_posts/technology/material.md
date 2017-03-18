---
title: ThreeJs的材质API
header-img: images/comm-header/technology.jpg
date: 2017-03-18 15:38:13
tags: 
  - ThreeJs
---
世界因为材质才显得五彩缤纷，要在计算机里构建一个五彩缤纷的世界，必须要了解构建工具里提供的材质API。
材质反射光。
<!-- more -->
LineBasicMaterial
	画一条实线时，指定其颜色，感光，宽度，线两端样式，连接点样式。

LineDashedMaterial
	画一条虚线时，指定虚线段间距、长度、宽度、颜色。

MeshBasicMaterial
	没有质感，可指定其颜色，显示线框，感光，纹理等。

MeshDepthMaterial
	没有质感，体现的是远近，白色表示最近，黑色表示最远。

MeshLambertMaterial
	表面粗糙，均匀，比如一张纸，其特点是能产生漫反射，让物体因为表面凹凸呈现出不同的阴暗效果。

MeshNormalMaterial
	将普通的向量映射成RGB值，呈现出相应色彩。

MeshPhongMaterial
	表面光滑的材质，物体呈现出的明暗明显。

MeshPhysicalMaterial


MeshStandardMaterial
	标准物体材质，可设置其粗糙度、颜色、纹理、光感等属性，理论上讲，Lambert和Phong材质效果可以通过该API构建出来。

MeshToonMaterial
MultiMaterial
	给同一个物体设置多个材质，比如一个立方体的六个面设置六种材质。参数接受一个material数组。

PointsMaterial
	画一个点时，指定其颜色，感光，大小等。

RawShaderMaterial
ShaderMaterial
	shader 着色器。

ShadowMaterial
	

SpriteMaterial：
	sprite 子图形，将其作为材质，可以改变其颜色，感光。