# Swiper 的种种踩坑和解决

> 由于各个项目关系，需要实现各种轮播，使用了强大的Swiper插件，由于功能太多，使用的时候踩了很多坑，下面记录踩的各种坑及其解决方案。
# 1. 翻页按钮和分页器在框外

默认情况下 Swiper 翻页按钮和分页器在轮播内容框区域内。

```html
<div class="swiper-container">
  <div class="swiper-wrapper">
    <div class="swiper-slide">
      <div class="slide-content">Slide 1</div>
    </div>
    <div class="swiper-slide">
      <div class="slide-content">Slide 2</div>
    </div>
    <div class="swiper-slide">
      <div class="slide-content">Slide 3</div>
    </div>
  </div>
  <div class="swiper-pagination"></div>
  <div class="swiper-button-prev"></div>
  <div class="swiper-button-next"></div>
</div>
```

```javascript
new Swiper('.swiper-container', {
  pagination: {
    el: '.swiper-pagination',
    clickable: true
  },
  navigation: {
    nextEl: '.swiper-button-next',
    prevEl: '.swiper-button-prev'
  },
});
```

![翻页按钮在内容区域内](/home/mirrory/Desktop/LearnMore/Others/img/swiper-1.jpeg)

想实现翻页按钮在其区域外，其实并不难。
**解决思路：** 在`.swiper-container`外层加一个相对定位的父标签，将翻页按钮和分页器移至`.swiper-container`外层。

```html
<div class="layout-container"><!-- 新增外层标签 -->
  <div class="swiper-container">
    <div class="swiper-wrapper">
      <div class="swiper-slide">
        <div class="slide-content">Slide 1</div>
      </div>
      <div class="swiper-slide">
        <div class="slide-content">Slide 2</div>
      </div>
      <div class="swiper-slide">
        <div class="slide-content">Slide 3</div>
      </div>
    </div>
  </div>
  <!-- 移至外层 -->
  <div class="swiper-pagination"></div>
  <div class="swiper-button-prev"></div>
  <div class="swiper-button-next"></div>
</div>
```

```css
.layout-container {
  padding-bottom: 0 50px 20px 50px;
  position: relative;
}
```

**实现效果：**

![翻页按钮在内容区域外](/home/mirrory/Desktop/LearnMore/Others/img/swiper-2.jpeg)

---

# 2. 禁用拖动时，分页器仍然可以拖动

Swiper 需要做两个步骤：

1. Swiper 对象上设置`noSwiping： true`

   ```javascript
   new Swiper('.swiper-container', {
     noSwiping: true,
     pagination: {
       el: '.swiper-pagination',
       clickable: true
     }
   });
   ```

2. HTML 中给禁用元素添加`swiper-no-swiping`，这个类名也可以通过`noSwipingClass`属性更改，默认为`swiper-no-swiping`

   ```html
   <div class="swiper-container">
     <div class="swiper-wrapper">
       <div class="swiper-slide swiper-no-swiping">Slide 1</div>
       <div class="swiper-slide swiper-no-swiping">Slide 2</div>
       <div class="swiper-slide swiper-no-swiping">Slide 3</div>
     </div>
     <div class="swiper-pagination"></div>
   </div>
   ```

但这个时候，鼠标在分页器上拖动仍然能实现滑动。

官方对`noSwiping`属性的说明是： **noSwiping 设为 true 时，可以在 slide 上（或其他元素）增加类名‘swiper-no-swiping’，使该元素无法拖动** 。

所以其实是，`noSwiping: ture`只对 class 为指定类名的元素有效。
So，想要分页器禁用拖动，也需要加上对应的禁止拖动的 class 。

---

# 3. 设置自动高度

要想每页高度自适应内容，可以设置`autoHeight: true`，这个时候`.swipe-container`会加上一个`swiper-container-autoheight`的 class，这个 class 下的`.swiper-slide`的高度是`auto`。

<font color=red>！注意：</font>这个时候，页面加载的时，会根据当前 slide 的高度来设置`.swiper-wrapper`的高度。

当这个 slide 中的内容完全载入需要一定时间，可能会导致`.swiper-wrapper`的初始高度计算错误。
比如，我页面上用的是 BrightCove 的视频，由于这个视频的完全载入晚于 swiper 的初始化，导致视频底部被遮住了一部分的内容。
解决这个问题，其实只需要设置初始的`.swiper-wrapper`高度为`auto`即可。

**解决方案：** 在 swiper 提供的初始化事件中，设置`.swiper-wrapper`高度为`auto`。

```javascript
new Swiper('.swiper-container', {
  on: {
    // 初始化后执行该事件函数
    init: function() {
      // 设置高度自动
      $('.swiper-wrapper').height('auto');
    }
  }
});
```