---
title: 设置物体的旋转中心
header-img: images/comm-header/technology.jpg
date: 2017-03-14 17:19:42
tags: 
  - ThreeJs
---
在ThreeJs中，物体的旋转运动是绕世界中心旋转的，实际却有很多场景要求物体绕其他点旋转。
<!-- more -->
### 思路一
将物体移动至中心与世界中心重合的位置。步骤：
- 加载模型
- 获取模型的边界框
- 获取模型的中心与-1做矩阵乘法（multiply scalar）
- 通过translate方法移动模型至世界中心

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

待续...