### 手淘设计师和前端开发的适配写作基本思路

- 选择一种尺寸作为设计和开发基准

- 定义一套适配规则，自动适配剩下的两种尺寸（其实不仅这两种）

- 特殊适配效果给出设计效果

  ![1552481076960](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1552481076960.png)

  在手淘设计师和前端开发协作过程中：手淘设计师常选择iphone6作为基准设计尺寸，交付给前端的设计尺寸是按750px*1334px为准（高度会随着内容多少而改变）。前端开发人员通过一套适配规则自动适配到其他的尺寸。

  ### 前端开发完成终端适配方案

  #### 一些基本概念

  ##### 视窗 viewport

  简单的理解，viewport是严格等于浏览器的窗口。在桌面浏览器中，viewport就是浏览器窗口的宽度高度。但在移动端设备上就有点复杂。

  移动端的viewport太窄，为了能更好的为css布局服务，所以提供了两个viewport：虚拟的viewportvisualviewport和布局的viewportlayoutviewport。

  ##### 物理像素（physical pixel）

  物理像素又被称为设备像素，他是显示设备中一个最微小的物理部件。每个像素可以根据操作系统设置自己的颜色和亮度。正是这些设备像素的微小距离欺骗了我们肉眼看到的图像效果。

  ##### 设备独立像素（density-independent pixel)

  设备独立像素也称为密度无关像素，可以认为是计算机坐标系统中的一个点，这个点代表一个可以由程序使用的虚拟像素（比如css像素），然后由相关系统转换为物理像素。

  ##### css像素

  css像素是一个抽象的单位，主要使用在浏览器上，用来精确度量web页面上的内容。一般情况之下，css像素称为与设备无关的像素，简称DIPs。

  ##### 屏幕密度

  屏幕密度是指一个设备表面上存在的像素数量，它通常以每英寸有多少像素来计算（PPI）。

  ##### 设备像素比（device pixel ratio）

  这杯像素比简称为dpr，其定义了物理像素和设备独立像素的对应关系。

  ```
  设备像素比 = 物理像素 / 设备独立像素
  ```

  在JavaScript中，可以通过window.devicePixelRatio获取到当前设备的dpr。而在css中，可以通过-webkit-device-pixel-ratio，-webkit-min-device-pixel-ratio和-webkit-max-device-pixel-ratio进行媒体查询，对不同dpr的设备，做一些样式适配（这里只针对webkit内核的浏览器和webview）。

  dip或dp（设备独立像素）与屏幕密度有关。dip可以用来辅助区分视网膜设备还是非视网膜设备。

  ##### meta标签

  这里要着重说的是viewport的meta标签，其主要用来告诉浏览器如何规范的渲染web页面，而你则需要告诉它视窗有多大。在开发移动端页面，我们需要设置meta标签如下：

  ```HTML
  <meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1">
  ```

  代码以显示网页的屏幕宽度定义了视窗宽度。网页的比例和最大比例被设置为100%。

  ##### css单位rem

  简单的理解，rem就是相对于根元素<html>的font-size来做计算。而我们的方案中使用rem单位，是能轻易的根据<html>的font-size计算出元素的盒模型大小。而这个特色对我们来说是特别的有益处。

  ### 前端实现方案

  在整个手淘团队，我们有一个名叫lib-flexible的库，而这个库就是用来解决H5页面终端适配的。

  ##### lib-flexible是什么？

  是一个制作H5适配的开源库。

  ##### 使用方法

  第一种方法是将文件下载到项目中，然后通过相对路径添加。

  或者直接加载阿里CDN的文件。

  另外强烈建议对JS做内联处理，在所有资源加载之前执行这个JS。执行这个JS后，会在<html>元素上增加一个data-dpr属性，以及一个font-size样式。JS会根据不同的设备添加不同的data-dpr值，比如说2或者3，同时会给html加上对应的font-size的值，比如75px。

  除此之外，在引入lib-flexible需要执行的js之前，可以手动设置meta来控制dpr值，如：

  ```html
  <meta name="flexible" content="initial-dpr=2" />
  ```

  其中initial-dpr会把dpr强制设置为给定的值。如果手动设置了dpr之后，不管设备是多少的dpr，都会强制认为其dpr是你设置的值。在此不建议手动强制设置dpr，因为在Flexible中，只对iOS设备进行dpr的判断，对于Android系列，始终认为其dpr为1。

  ##### flexible的实质

  flexible实际上就是能通过js来动态改写meta标签。

  ##### 把视觉稿中的px转换成rem

  首先，目前日常工作当中，视觉设计师给到前端开发人员手中的视觉稿尺寸一般是基于640px、750px、以及1125px宽度为准。（考虑Retina屏）

  目前Flexible会将视觉稿分为100份（主要是为了以后能更好的兼容vh和vw），而每一份被称为一个单位a。同时1rem单位被认定为10a。针对我们这份视觉稿可以计算出：

  ```
  1a = 7.5px
  1rem = 75px
  ```

  ##### 如何快速计算

  CSSREM

  ​       CSSREM是一个css的px值转rem值得Sublime Text3自动完成插件。

  CSS处理器

  ​        Sass、PostCSS(px2rem)

  ##### 文本字号不建议使用rem

  我们不希望文本在Retina屏幕下变小，另外，我们希望在大屏手机上看到更多文本，以及，现在绝大多数的字体文件都自带一些点阵尺寸，通常是16px和24px，所以我们不希望出现13px和15px这样的奇葩尺寸。

  如此一来，就决定了在制作H5的页面中，rem并不适合用到段落文本上。所以在Flexible整个适配方案中，考虑文本还是使用px作为单位。只不过使用[data-dpr]属性来区分不同dpr下的文本字号大小。

  ```css
  div{
      width: 1rem;
      height: 0.4rem;
      font-size: 12px; // 默认写上dpr为1的fontsize
  }
  [data-dpr="2"] div {
      font-size: 24px;
  }
  [data-dpr="3"] div {
      font-size: 36px;
  }
  ```

  为了能更好的利于开发，在实际开发中，我们可以定制一个font-dpr()这样的Sass混合宏：

  ```Sass
  @mixin font-dpr($font-size){
      font-size: $font-size;
      [data-dpr="2"] & {
          font-size: $font-size * 2;
      }
      [data-dpr="3"] & {
          font-size: $font-size * 3;
      }
  }
  ```

  有了这样的混合宏之后，在开发中可以直接这样使用：

  ```sass
  @include font-dpr(16px);
  ```

  当然这只是针对于描述性的文本，比如说段落文本。但有的时候文本的字号也需要分场景的，比如在项目中有一个slogan，业务方希望这个slogan能根据不同的终端适配。针对这样的场景，完全可以使用rem给slogan做计量单位。