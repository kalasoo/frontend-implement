大家好，我是新人kalasoo，现在还处在“试用期”，作为一个自学出来的前端新手，能够加入前端观察实在是异常兴奋。既然要一起来维护这个关于前端的博客，我一定会努力争取我所写的内容可以追上这里文章的质量水平。作为开始，我会做一个小小的系列来分析那些有名、特别、设计感十足或是交互体验出众的网站。同时我还会认真阅读其前端代码，为大家重现那些神奇的效果是如何实现的，当然也会尽我所知引用更多的资源来丰富内容。希望这个系列可以让大家更加了解前端技术，同时也可以锻炼我自己。

### 我们就从[Medium](http://medium.com)开始

决定第一个来做这个网站的原因很简单，那就是好看啊！由于Medium的出现严重影响改变了博客、发布平台的风格以及编辑器等前端组建的设计，我们会分多期来分析各种细节是如何实现的。我们尤整体走向局部，所以我们先从整体布局来分析：

* 首页布局以及大背景图

### 网站背景

Medium是由Twitter的联合创始人：Evan Williams和Biz Stone创办于2012年8月创办的一个文章写作、阅读平台。注意，我这里并没有用很多网站上援引的博客平台是因为Medium的出现塑造了一种新的社会化自我营销渠道。在首页引入的[Welcome to Medium](https://medium.com/about/9e53ca408c48)里，我们看到它的初衷是为了让人们更好地写作，但是作为Twitter的一个延伸，它依旧搭载在一个社交性很强的平台之上。这也让在Medium中写作的人更愿意去分享、营销、推广自己的写作内容，甚至成为一些知名Developers, Designers and even Managers的发布渠道。例如：Facebook的Product Design Director Julie Zhuo就曾经通过在Medium上发表[文章](https://medium.com/art-of-product-design/ed75a0ee7641)来解释Facebook为何revert掉在视觉层面上非常突出的新版界面。而更让人觉得特别的是，每当我看到好的文章分享到twitter上时，都会有作者亲自来favorite你的tweet，这简直就是自我运营啊，有木有！

好啦，闲话说完，我们进入主题。

### 首页布局
![Medium Home](http://imagizer.imageshack.us/v2/1024x768q90/843/mcoa.png)

大背景图或是视频，已经成为累当今服务性网站设计的一个趋势，随着网速越来越快、屏幕越来越大，一张巨幅首页图片无论从视觉冲击力还是从性能上都已经不再是不可能实现的功能。那我们来看一下这个首页的布局是如何实现的：

HTML:

```html
<div class="container surface-container" id="container">
  <div class="surface" id="..." style="display: block; visibility: visible;">
    ::before
    <div class="screen-content surface-content">
      <div class="image-fill layout-fill promo-fill" style="background-image: url('xxx.jpeg')">
        <div class="layout-fill align-middle">
          ::before
          <div class="align-block layout-fill-width">
            <div class="layout-single-column">...</div>
            <div class="align-center">...</div>
          </div>
          <footer class="footer ..."></footer>
        </div>
      </div>
    </div>
    ::after
  </div>
</div>
```

CSS:

为了实现全window的覆盖，第一个要做的就是设置`html, body`的`height`为100%，宽度的话，应为是自动全覆盖，就无需多做设置。`.container`只是作为一个过度，同样设置到100%的高度（这里我会暂时忽略除了layout之外的属性设置）。
再深一层是便是`.surface`，除了再次继承了满高度、满宽度外，还定义了`box-sizing: border-box;`，请移位[MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/box-sizing)来理解。这里要注意它加了`.suface:before, .surface:after`的属性，是为了自动生成clearfix的效果来阻止float溢出，但其实在这个首页之中并没有用到。

```css
html, body, .container {
  height: 100%;
}
.surface {...}
.surface:before, .surface:after {
  display: table;
  content: " ";
}
.surface:after {
  clear: both;
}
```

在往里面的几个`.screen-content, .surface-content`只是继续控制高度，和一些默认背景颜色。直到下面这一层才定义了覆盖整个背景大图的属性，而具体的背景图片是只额姐加到了DOM的这个`div`里面（我想原因应该是有一些数据binding在里面，这样好做更新）：

```css
.image-fill, {
  background-position: center;
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover;
}
.image-fill {
  background-color: rgba(0, 0, 0, 0.9);
}
.promo-fill {
  background-color: #000;
}
```

### 大背景图

在这里，为了实现图片可以满背景覆盖，最主要的一个元素就是`background-size: cover;`，这个属性一共有以下几个可能的值：
```
background-size: <length> | <percentage> | auto | cover | contain;
```

它们分别代表的意思是：

1. `background-size: 50% | 120px | auto;` 设置了背景图片的宽度，高度则自动计算。默认的`auto`维持了背景图原有的大小。
2. `background-size: 50% auto;` 同时强制定义了宽度和高度。
3. `background-size: auto, auto, [...];` 定义给多个背景图片（注意被一个定义会用逗号隔开，与`auto auto`并不是一个意思）。
4. `background-size: cover;` 这样定义的背景图片会被修改图片大小（长宽比例不变），以确保刚刚好覆盖整个Element。
5. `background-size: contain;` 背景图片会被修改大小（长宽比例不变），以确保刚刚好被这个Element包裹在里面。

下面几张图分别展示了集中不同的情况：

当`background-size`设置为`cover`时，即使屏幕被拉窄拉宽，图片依旧很好地覆盖着整个屏幕（更多的比对还可以访问[MDN](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Scaling_background_images)）：

![home cover](http://imagizer.imageshack.us/v2/640x480q90/845/2iix.png)

`contain`时，图片长宽比并不发生变化，却会被包在window里面：

![home contain](http://imagizer.imageshack.us/v2/320x240q90/836/d76h.png)

`50% 100%`时，图片长宽比按照设置的数值被拉伸：

![home 50% 100%](http://imagizer.imageshack.us/v2/320x240q90/845/w6l5.png)

注意，这里的背景图片都是按照默认的会重复铺排在x和y轴上，所以当背景图不能覆盖element的时候，便会出现重复的样式。此外，`background-position: center;`也定义了图片按照所属element的中心位置来调整大小。

#### 兼容性

这样图片全页面覆盖的实现方法就说明完了，`contain & cover`属性的兼容性如下:

1. Chrome 3.0+
2. Firefox 3.6 (1.9.2)+
3. IE 9.0+
4. Opera 10.0+
5. Safari 4.1+

如果为了实现同样的效果，却要面对恐怖的IE7/8，肿么破，请尝试如下：

```css
.background-cover {
  filter: progid:DXImageTransform.Microsoft.AlphaImageLoader(
          src='PATH_TO_BACKGROUND_IMAGE',
          sizingMethod='scale');
  -ms-filter: progid:DXImageTransform.Microsoft.AlphaImageLoader(
              src='PATH_TO_BACKGROUND_IMAGE',
              sizingMethod='scale');
}
```

但是要小心，这个实现方法会使得起内部的链接点击失效（请给我大MS 32个赞），而且这个实现的是`background-size: 100% 100%`的效果。或者可以通过jQuery ([jquery.backgroundSize.js](https://github.com/louisremi/jquery.backgroundSize.js))来实现。

#### 其他示例

这里面是一些好看的大图覆盖的例子，但是并不是所有的页面都是通过`background-size`来实现的：

* http://www.whitmansnyc.com/
* http://quinntonharris.strikingly.com/
  * 其实现方式其实是背景图很大，大小设置为auto，而覆盖效果则是通过设置外面一层的div大小来实现的。
* http://www.lattrapereve.fr/
  * 背景视频大小是根据window大小的变化，用javascript来修改的。

### 下期预告

下一期，我们会深入探讨Medium上的侧边推进栏sidebar的实现，而这个推进效果其实有好多种不一样的方法，有的可以在mobile上实现滑动效果，有的会有更好地兼容性。敬请大家期待。


