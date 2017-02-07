---
title: 无缝轮播的实现
tags:
  - Technology
header-img: images/comm-header/technology.jpg
date: 2016-09-27 17:48:19
---

通常看到的轮播（例如从右到左）的最后一页到第一页会看到页面朝右回滚的过程。
现在我希望能实现圆环式无限轮播的效果。
<!-- more -->
<style type="text/css">.carousel {width: 100px; height: 50px; overflow: hidden; white-space: nowrap; margin: 0 auto;} .carousel .content {display: inline-block; white-space: nowrap; } .carousel .content_item {display: inline-block; width: 100px; height: 50px; text-align: center; line-height: 50px;}</style>
一般的轮播效果实现逻辑是在视框内，隔一段时间将需要展示的内容向左移动一页的宽度，移到最后一页时再将内容左边移动距离重置为0。
<div class="normal_carousel"><div class="carousel"><div class="content"><div class="content_item" style="background-color: #A18E58">1</div><div class="content_item" style="background-color: #FFCEA6">2</div><div class="content_item" style="background-color: #DCDCDC">3</div></div></div></div>

<script type="text/javascript">
	var cur_scroll_index = 0;
	setInterval (function() {
		cur_scroll_index ++;
		$('.normal_carousel .content').css ({
			'transform': 'translateX(-' + 100 * cur_scroll_index + 'px)',
			'transition': 'transform 0.5s'
		});
		if (cur_scroll_index >= $('.normal_carousel .content_item').length - 1) {
			cur_scroll_index = 0;
			setTimeout (function() {
				$('.normal_carousel .content').css ({
					'transform' : 'translateX(0)',
				});
			}, 1000 );
		}
	}, 2000 );
</script>

圆环式无限轮播的效果：
<div class="circle_carousel"><div class="carousel"><div class="content"><div class="content_item" style="background-color: #A18E58">1</div><div class="content_item" style="background-color: #FFCEA6">2</div><div class="content_item" style="background-color: #DCDCDC">3</div></div></div></div>

<script type="text/javascript">
	setTimeout(function() {
		var carousel_index = 1, show_next_width,
			$show_first = $('.circle_carousel .content'),
			$wrapper = $('.circle_carousel .carousel');

			$wrapper.append( $show_first.find('.content_item:first').clone() );
		// carousel
		setInterval (function() {
			if ($wrapper.scrollLeft() >= $show_first.width()) {
				carousel_index = 1;
				$wrapper.scrollLeft(0);
			}
			show_next_width = $wrapper.width()*carousel_index;
			slide_timer = setInterval(_slide, 5);

			carousel_index++;
		}, 1600);

		function _slide() {
			if ($wrapper.scrollLeft() < show_next_width) {
				$wrapper.scrollLeft($wrapper.scrollLeft() + 2);
			} else {
				clearInterval(slide_timer);
			}
		}
	}, 500);
</script>

而圆环式的逻辑是在原来的展示内容尾部添加第一页的克隆页，运动时，scrollLeft每次增加一个单位，内容和克隆内容一起向左滑动，完成最后一页过渡到第一页的效果，即`状态-4`。之后将scrollLeft置为0个单位，由`状态-5`回到`状态-1`，再继续下一个循环。
<pre>
状态-1
                       __ 
视框：                |__|
                       __ __ __
内容：                |__|__|__|           scrollLeft = 0个单位;
                                __ 
克隆内容：                     |__|
</pre>
<pre>
状态-2
                       __ 
视框：                |__|
                    __ __ __
内容：             |__|__|__|              scrollLeft = 1个单位;
                             __ 
克隆内容：                  |__|
</pre>
<pre>
状态-3
                       __ 
视框：                |__|
                 __ __ __
内容：          |__|__|__|                 scrollLeft = 2个单位;
                          __ 
克隆内容：               |__|
</pre>
<pre>
状态-4
                       __ 
视框：                |__|
               __ __ __
内容：        |__|__|__|           
                        __ 
克隆内容：             |__|
</pre>
<pre>
状态-5
                       __ 
视框：                |__|
              __ __ __
内容：       |__|__|__|                    scrollLeft = 3个单位;
                       __ 
克隆内容：            |__|
</pre>

上面的圆环式无限轮播demo实现代码如下：

```html
<style type="text/css">
    .carousel {
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
    .carousel .content_item {
        display: inline-block;
        width: 100px;
        height: 50px;
        line-height: 50px;
        text-align: center;
    }
</style>
<div class="carousel">
    <div class="content">
        <div class="content_item" style="background-color: #A18E58">1</div>
        <div class="content_item" style="background-color: #FFCEA6">2</div>
        <div class="content_item" style="background-color: #DCDCDC">3</div>
    </div>
</div>
```

```javascript
var carousel_index = 1, show_next_width,
    $show_first = $('.content'),
    $wrapper = $('.carousel');

    // 克隆展示内容
   $wrapper.append( $show_first.find('.content_item:first').clone() );

setInterval (function() {
    if ($wrapper.scrollLeft() >= $show_first.width()) {
        carousel_index = 1;
        $wrapper.scrollLeft(0);
    }
    show_next_width = $wrapper.width()*carousel_index;

    slide_timer = setInterval(_slide, 5); 
    carousel_index++;
}, 1600);

function _slide() {
    if ($wrapper.scrollLeft() < show_next_width) {
        $wrapper.scrollLeft($wrapper.scrollLeft() + 2);
    } else {
        clearInterval(slide_timer); 
    }
}
```

**关键点有：**

1. `_slide`不可以写成`_slide()`。
2. `slide_timer = setInterval(_slide, 5); `执行时间如果比较大，下一次执行到这里时即一页还没有滑动完，则外层设置的间隔时间（这里是1600ms）就会看不到效果。
3. `_slide()`中如果不清除定时器`slide_timer`，会呈现开始滑动慢后面变快的效果。
4. 递增`$wrapper.scrollLeft()`才能让内容和克隆内容一起运动，改用transform则不能达到该效果。