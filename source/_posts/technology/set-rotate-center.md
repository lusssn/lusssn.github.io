---
title: 旋转篇
header-img: images/comm-header/technology.jpg
date: 2017-03-14 17:19:42
tags: 
  - ThreeJs
---
在ThreeJs中，物体的旋转运动是绕世界中心旋转的，实际却有很多场景要求物体绕其他点旋转。
<!-- more -->
### 思路一
将物体移动至中心与世界中心重合的位置，绕世界中心旋转等于自己中心旋转。步骤：
- 加载模型
- 获取模型的边界框
- 获取模型的中心与-1做矩阵乘法（multiply scalar）
- 通过translate方法或者position.set()，移动模型至世界中心
- 旋转物体

```js
// loadObject
loader.load('./js/model/model.obj', onLoad);
var mesh, material = new THREE.MeshLambertMaterial({color: 0xffffff});

function onLoad(object) {
	mesh = object;
	var objBox = new THREE.Box3().setFromObject(mesh);
	// 通过模型边界框获取模型中心
	var boxCenter = objBox.getCenter();
	// 中心点矩阵*-1 得到中心点关于原点对称的新点
	boxCenter.multiplyScalar(-1);
	// 添加材质
	mesh.children.forEach(function (child) {
		if (child instanceof THREE.Mesh) {
			child.material = material;
		}
	});
	// 通过translate将模型向对称点方向移动至中心点
	mesh.traverse(function (child) {
		if (child instanceof THREE.Mesh) {
			child.geometry.translate(boxCenter.x, boxCenter.y, boxCenter.z);
		}
	});
	// mesh.position.set(boxCenter.x, boxCenter.y, boxCenter.z);
	scene.add(mesh);
	animation();
}

function animation() {
	requestAnimationFrame(animation);
	// 旋转物体
	mesh.rotation.y += 0.02;
	renderer.render(scene, camera);
	stats.update();
}
```

### 思路二
不移动物体，让物体在原地绕自己的中心旋转。

ThreeJs中有`Group`的概念，Group内的物体是绕着Group的中心旋转的。我`将Group看作一个新世界`来理解，转动Group就是在转动整个新世界。

新建Group时，其位置（position）即新世界的中心是与世界中心重合的，转动Group看上去就是Group内的所有物体都在绕着世界中心旋转。

想让物体在自己原来的位置（目标旋转点）旋转，步骤如下：
- 移动Group的位置到这个目标旋转点
	- 目标旋转点的获取与思路一获取模型的中心方式一样
	- 这个时候Group内的所有物体都会做相应的移动
- 再反方向移动物体
	- 此时物体相对于世界并没有移动
- 旋转新世界

```js
// loadObject
loader.load('./js/model/model.obj', onLoad);
var mesh, material = new THREE.MeshLambertMaterial({color: 0xffffff}),
	pivot = new THREE.Group();

function onLoad(object) {
	mesh = object;
	var objBbox = new THREE.Box3().setFromObject(mesh);
	// 通过模型边界框获取模型中心（目标旋转点）
	var bboxCenter = objBbox.clone().getCenter();
	// 添加材质
	mesh.children.forEach(function (child) {
		if (child instanceof THREE.Mesh) {
			child.material = material;
		}
	});
	// 反方向移动物体
	mesh.position.set(-bboxCenter.x, -bboxCenter.y, -bboxCenter.z);
	scene.add(pivot);
	pivot.add(mesh);
	// 移动Group的位置到目标旋转点
	pivot.position.set(bboxCenter.x, bboxCenter.y, bboxCenter.z);
	animation();
}

function animation() {
	requestAnimationFrame(animation);
	// 旋转新世界
	pivot.rotation.y += 0.02;
	renderer.render(scene, camera);
	stats.update();
}
```