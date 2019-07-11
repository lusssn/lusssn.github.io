---
title: 无缝轮播的实现
tags: 实用Demo
headerimg: images/comm-header/technology.jpg
date: 2016-09-27 17:48:19
---

通常看到的轮播（例如从右到左）的最后一页到第一页会看到页面朝右回滚的过程。
现在我希望能实现圆环式无限轮播的效果。
<!-- more -->
### 两种轮播效果对比

通常，轮播效果实现逻辑是在视框内，隔一段时间将需要展示的内容向左移动一页的宽度，移到最后一页时再将内容左边移动距离重置为0。
这样会看到最后一张跳到第一张的过程，效果像酱紫👇

<div class="normalCarousel"><div class="carousel"><div class="content"><div class="contentItem" style="background-color: #A18E58">1</div><div class="contentItem" style="background-color: #FFCEA6">2</div><div class="contentItem" style="background-color: #DCDCDC">3</div></div></div></div>

<script type="text/javascript">
	setTimeout(function() {
        const itemLength = $('.normalCarousel .contentItem').length;
	    let currentItem = 0;
        setInterval(function() {
            currentItem++;
            if (currentItem === itemLength) {
                currentItem = 0;
                $('.normalCarousel .content').css ({
                    'transform' : 'translateX(0)'
                });
            } else {
              $('.normalCarousel .content').css ({
                  'transform': `translateX(-${100 * currentItem}px)`,
                  'transition': 'transform 0.5s'
              });
            }
    	}, 2000 );
	}, 500);
</script>

如果想做到无限循环的效果，只需要稍微改动一点点逻辑，就可以实现如下的效果：

<div class="circleCarousel"><div class="carousel"><div class="content"><div class="contentItem" style="background-color: #A18E58">1</div><div class="contentItem" style="background-color: #FFCEA6">2</div><div class="contentItem" style="background-color: #DCDCDC">3</div></div></div></div>

<script type="text/javascript">
    setTimeout(function() {
        const itemLength = $('.circleCarousel .contentItem').length;
        // 1. 末尾追加第一张卡片
        const $content = $('.circleCarousel .content');
        $content.append($content.find('.contentItem:first').clone());
        let currentItem = 0;
        setInterval(function () {
            currentItem++;
            $('.circleCarousel .content').css({
                'transform': `translateX(-${100 * currentItem}px)`,
                'transition': 'transform 0.5s ease',
            });
            if (currentItem === itemLength) {
                currentItem = 0;
                setTimeout(function () {
                    // 2. 去掉transition效果，换到第一张卡片位置
                    $('.circleCarousel .content').css({
                        'transform': 'translateX(0)',
                        'transition': 'unset',
                    });
                }, 1000);
            }
        }, 2000);
    }, 500);
</script>

### 代码剖析

HTML和CSS很简单

```html
<style type='text/css'>
    .carousel {
        font-size: 0;
        overflow: hidden;
        width: 100px;
        height: 50px;
        margin: 0 auto;
        white-space: nowrap;
    }
    .carousel .content {
        display: inline-block;
        white-space: nowrap;
    }
    .carousel .contentItem {
        display: inline-block;
        width: 100px;
        height: 50px;
        line-height: 50px;
        text-align: center;
        font-size: 14px;
    }
</style>
<div class='circleCarousel'>
    <div class='carousel'>
        <div class='content'>
            <div class='contentItem' style='background-color: #A18E58'>1</div>
            <div class='contentItem' style='background-color: #FFCEA6'>2</div>
            <div class='contentItem' style='background-color: #DCDCDC'>3</div>
        </div>
    </div>
</div>
```

主要是js的逻辑

```js
const itemLength = $('.circleCarousel .contentItem').length;

// 1. 末尾追加第一张卡片
const $content = $('.circleCarousel .content');
$content.append($content.find('.contentItem:first').clone());

let currentItem = 0;
setInterval(function () {
    currentItem++;
    $('.circleCarousel .content').css({
        'transform': `translateX(-${100 * currentItem}px)`,
        'transition': 'transform 0.5s ease',
    });
    if (currentItem === itemLength) {
        currentItem = 0;
        setTimeout(function () {
            // 2. 去掉transition效果，换到第一张卡片位置
            $('.circleCarousel .content').css({
                'transform': 'translateX(0)',
                'transition': 'unset',
            });
        }, 1000);
    }
}, 2000);
```

如果去掉 3-4行 和 20行 的代码，就是普通的轮播效果了。是不是十分简单！💅

<!-- 样式 -->
<style type="text/css">.carousel {width: 100px; height: 50px; overflow: hidden; white-space: nowrap; margin: 0 auto;} .carousel .content {display: inline-block; white-space: nowrap; } .carousel .contentItem {display: inline-block; width: 100px; height: 50px; text-align: center; line-height: 50px;}</style>
