---
title: 材质篇
header-img: images/comm-header/technology.jpg
date: 2017-03-18 15:38:13
tags: 
  - ThreeJs
---
材质=材料+质感。光照射到物体上折射出不同的颜色、光强等，人眼才能感知物体的材质。构建材质的过程可以看作是操作改造反射光的过程。
<!-- more -->
将入射光比作原材料，物体比作工厂，反射光比作产品，人眼比作消费者。
那么材质就是工厂里的装配机器。
机器对入射光进行装配雕琢，改变其颜色、方向、强度等属性之后以反射光出厂，人眼消费后感知反射光反应的信息——物体材质。

WebGL将大自然中的材质抽象出来，ThreeJs做了划分，下面就来看看阿3是怎么划分材质的。

### 0. 材质
![](material_2.jpg)
- *Material*
	材质基类，抽象了材质的基本属性，不列举全部属性。
	感光（`lights`）：物体什么颜色就反射什么颜色的光。
	雾化（`fog`）：是否受雾化影响。
	透明（`transparent`）：是否透明。
	可视（`visible`）：是否可见。
	透明度（`opacity`）：透明度。
	阿尔法值（`alphaTest`）：若opacity小于alpha值则不会渲染材质。
	混淆、混淆方程式、混淆目标等：不明白。
	平面剪裁：不明白。
	颜色（`colorWrite`）：是否进行颜色渲染。
	材质更新（`needsUpdate`）：是否实时更新材质变化。
	理论上基类的属性子类都会继承，但实际中并不是每个属性在具体子类上都会有效果。比如雾化对ShadowMaterial来说就无意义。

### 1. 点材质
- *PointsMaterial*
	一个点的属性：颜色（`color`）、感光（`lights`）、大小（`size`），另外在阿3中还可以设置尺寸衰减（`sizeAttenuation`，近大远小）。

### 2. 线材质
两种线条：虚线 和 实线。
基本属性：颜色（`color`）、感光（`lights`）、粗细（`linewidth`）。
- *LineBasicMaterial*
	一根普通线条的属性：基本属性、末端结束方式（`linecap`）、连接方式（`linejoin`）。
- *LineDashedMaterial*
	一根虚线条的属性：基本属性、实部长度（`dashSize`）、虚部长度（`gapSize`）、虚线范围（`scale`）。

### 3. 面材质
从点到线，再到面，属性就更加丰富了。
面的基本属性：感光（`lights`）。
- *SpriteMaterial*
	**sprite**是一个总是面向相机的平面。
	其材质属性：基本属性、颜色（`color`）、雾化（`fog`）、纹理映射（`map`）、旋转角度（`rotation`）。
	![](sprite.png)
- *ShadowMaterial*
	继承自ShaderMaterial，可以接受物体投影呈现出阴影。
	属性：基本属性、透明（`transparent`）。

### 4. 物体材质
- *MeshBasicMaterial*
	用在绘制简单轮廓的时候。比如物体扁平化（flat）或则仅线框。
	默认不感光，无质感。
	属性列举不全：线框模式（`wireframe`）、反射光模式（`reflectivity`、`combine`）。
- *MeshDepthMaterial*
	根据参数parameters创建基于相机远近裁切面自动变换亮度（明暗度）的材质类型,离相机越近,材质越亮（白）,离相机越远,材质越暗（黑）。
	默认不感光，无质感。体现的是远近，白色表示最近，黑色表示最远。
- *MeshLambertMaterial*
	表面均匀，粗糙，比如一张纸。
	其特点是能产生漫反射，有光泽，表面凹凸呈现出不同的阴暗效果。
- *MeshPhongMaterial*
	表面光亮圆滑的材质，比如塑料、金属。
	比Lambert更光滑，有高光，明暗更为明显。
	![](material_1.png)
- *MeshToonMaterial*
	使用了Toon Shader的MeshPhongMaterial的扩展，或叫做物体拥有高光。
	Toon Shader又被叫做Cel Shader，这种效果能让物体看上去有卡通的感觉，很神奇吧！
	![](toon.png)
- *MeshNormalMaterial*
	将普通的向量映射成RGB值，呈现出相应色彩。
- *MeshStandardMaterial*
	标准物体材质，可设置其粗糙度、颜色、纹理、光感等属性，理论上讲，Lambert和Phong材质效果可以通过该API构建出来。
- *MeshPhysicalMaterial*
	MeshStandardMaterial的扩展，更好地控制反射率（高光）。
- *MultiMaterial*
	给同一个物体设置多个材质，比如一个立方体的六个面设置六种材质。
	参数接受一个material数组。
	![](multi.png)
- *RawShaderMaterial*
	继承自ShaderMaterial，自定义的uniforms和attribute属性不会自动追加到GLSL着色器代码中
- *ShaderMaterial*
	以着色器的方式自定义材质，用户可借此扩充材质类型。

持续完善中...