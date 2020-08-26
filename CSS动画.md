一个简单的例子：将 div 从做往右移动
·原理：每过一段时间（用 setlnterval 做到），将 div 移动到一小段距离，直到移动到目标地点
·注意性能：绿色表示重新绘制（repaint）了，CSS 渲染过程依次包含布局、绘制、合成，其中布局和绘制可能被忽略
![left做动画(2.png)](2.png)

前端高手不用 left 做动画
·transform（变形）
·原理：transform：translateX（0 >= 300px),直接修改会被合成，需要等一会修改，transition 过度属性可以自动脑补中间帧
·注意性能：并没有 repaint（ 重新绘制），比改 left 性能好
![left做动画(3.png)](3.png)

浏览器渲染原理
·根据 HTML 构建树（DOM）![left做动画(4.png)](4.png)
·根据 CSS 构建 CSS 树（CSSOM）
·将两棵树合并成一颗渲染书（render tree）
·Layout 布局（文档流、盒模型、计算机大小和位置）
·Paint 绘制（把边框颜色、文字颜色、阴影等画出来）
Compose 合成(根据层叠关系展示画面）)

如何更新样式
·一般我们用 JS 来更新样式
·比如 div.style.background='red'
·比如 div.style.display='none'
比如 div.classList.add('red')
比如 div.remove()直接删掉节点
·那么方法什么不同？三种不同的渲染方案
第一种：JS\CSS>样式>布局>绘制>合成
运行 JavaScrip、Style、Layout、Paint、Composite
第二种：JS\CSS>样式>绘制>合成
运行 JavaScrip、Style、Paint、Composite
第三种：JS\CSS>样式>合成
运行 JavaScrip、Style、Composite
http://csstriggers.com/ 不同属性对应不同渲染流程
https://developers.google.com/web/fundamentals/performance/

rendering/optimize-javascript-execution

### 优化 JavaScript 执行：

TL;DR（太长不读）
对于动画效果的实现，避免使用 setTimeout 或 setInterval，请使用 requestAnimationFrame。
将长时间运行的 JavaScript 从主线程移到 Web Worker。
使用微任务来执行对多个帧的 DOM 更改。
使用 Chrome DevTools 的 Timeline 和 JavaScript 分析器来评估 JavaScript 的影响。

### 缩小样式计算的范围并降低其复杂性：style

TL;DR
降低选择器的复杂性；使用以类为中心的方法，例如 BEM。
减少必须计算其样式的元素数量。

### 避免大型、复杂的布局和布局抖动：Layout

TL;DR
布局的作用范围一般为整个文档。
DOM 元素的数量将影响性能；应尽可能避免触发布局。
评估布局模型的性能；新版 Flexbox 一般比旧版 Flexbox 或基于浮动的布局模型更快。
避免强制同步布局和布局抖动；先读取样式值，然后进行样式更改。

### 简化绘制的复杂度、减小绘制区域：绘制

TL;DR
除 transform 或 opacity 属性之外，更改任何属性始终都会触发绘制。
绘制通常是像素管道中开销最大的部分；应尽可能避免绘制。
通过层的提升和动画的编排来减少绘制区域。
使用 Chrome DevTools 绘制分析器来评估绘制的复杂性和开销；应尽可能降低复杂性并减少开销。

### 坚持仅合成器的属性和管理层计数：合成

TL;DR
坚持使用 transform 和 opacity 属性更改来实现动画。
使用 will-change 或 translateZ 提升移动的元素。
避免过度使用提升规则；各层都需要内存和管理开销。

JS 优化：使用 requestAnimationFrame 代替 setTimeout 或 setlnterval
CSS 优化：使用 will-change 或 translate （上述：死记硬背）

https://developer.mozilla.org/zh-CN/docs/Web/CSS/transform MDN 讲解网址
![语法(5.png)](5.png)
![语法格式(6.png)](6.png)
transform：
四个常用功能：位移 translate 缩放 scale 旋转 rotate 倾斜 skew
经验：一般不需要 transition 过度 inline 元素不支持 transform，需要先变成 block

transform 之 translate
·translateX（<length-percentage>)
·translateY（<length-percentage>)
·translate（<length-percentage>),<length-percentage>
·translateZ(<length>)且父容器 perspective
translate3d(x,y,z)
·translateX（<length-percentage>)
经验：要学会看懂 MDN 的语法示例，translate(-50%，-50%)可做绝对定位元素的居中

transform 之 scale
常用写法：scaleX(<number>)\scaleY(<number>)\scale(<number>,<number>)
经验：用的少容易模糊

transform 之 rotate
常用写法 rotate([<angle>|<zero]),rotateZ([<angle>|<zero]),rotateX([<angle>|<zero]),rotateY([<angle>|<zero])
经验：一般用于 360 度旋转制作加载 loading 中，用到时再搜索 rotate MDN 看文档
transform 之 skew
常用写法 skewX([<angle|<zero>]>),skewY(<[angle|<zero>]>),skew([<angle|<zero>,]),[<angle|<zero>]?)
经验用的少，用到时看 MDN

transform 多重效果：组合使用
transform：scale(0.5)translate(-100%,-100%);
transform:none;取消所有
![transform(7.png)](7.png)

![动画(8.png)](8.png) CSS 动画（跳动的心）

transition:过度
作用：补充中间帧
语法：transition：属性名 时长 过度方式 延迟
transition：left200ms liner
可以用逗号分隔两个不同属性
transition：left200ms，top 400mx
可以用 all 代表所有属性
transition：all 200ms
过度方式有：liner、ease、ease-in、ease-out、ease-in-out、cubic-bezier、step-start、step-end、steps，具体含义靠数学知识

注意：不是所有属性都能过度
·display：none=>block 没法过渡，一般改成 visibility：hidden=>visible(没有为什么)
·display 和 visibility 的区别：display 动画就直接消失 visibility 淡出没有位置还在
background 颜色可以过度吗
opacity 透明度可以过度吗

其他过度方法：
使用两次 transform
·a===transform===>.b
·b===transform===>.c
如何知道中间点呢
用 setTimeout 或者坚挺 transitionend 时间

·使用 animation：声明关键帧、添加动画
![animation(9.png)](9.png)
缩写语法：animation：时长、过度方式、延迟、次数、方向、填充模式、是否暂停、动画名
·时长：1s 或者 1000ms
·过度方式：跟 transition 取值一样，如 linear
次数：3 或者 2.4 或者 infinite
方向：reverse、alternate、alternate-reverse
填充模式：none、forwards、backwards、both
是否暂停：paused、running
以上所有属性都有对用的单独属性

![跳动的心n(10.png)](10.png)
