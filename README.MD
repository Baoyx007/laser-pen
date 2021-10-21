# 简介

Laser Pen 是一套用来在 web canvas 上绘制鼠标尾迹的工具集。

![demo](./docs/assets/demo.gif)

众所周知，产品经理的需求往往千奇百怪天马行空，所以 Laser Pen 并不仅仅是提供一个绘制鼠标尾迹的方法，同时也将内部的各种工具方法暴露出来，方便开发同学能在此工具集的基础上实现各种特殊需求，减少脱发。

# 示例

[在线演示](https://silenttiger.github.io/laser-pen/)

# 如何使用

首先在你的项目中添加对 Laser Pen 的依赖：

```bash
yarn add laser-pen
```

然后监听鼠标的 move 事件，获取鼠标移动轨迹的坐标，并同时记录 move 事件触发时的时间戳。

```javascript
// 获取 canvas 元素的坐标
const canvasPos = document.querySelector('canvas').getBoundingClientRect()
// 用来存储所有鼠标轨迹数据
let mouseTrack = []
// 监听鼠标事件
document.addEventListener('mousemove', (event) => {
  mouseTrack.push({
    x: event.clientX - canvasPos.x,
    y: event.clientY - canvasPos.y,
    time: Date.now(),
  })
})
```

最后在需要的时候调用 Laser Pen 的方法绘制鼠标尾迹

```javascript
import { drainPoints, drawLaserPen } from 'laser-pen'

function draw(ctx) {
  mouseTrack = drainPoints(mouseTrack)
  if (mouseTrack.length < 3) {
    return
  }
  drawLaserPen(ctx, mouseTrack)
}
```

强烈建议通过 requestAnimationFrame 来管理动画帧。具体用法可参考代码仓库中 example 目录下的代码。

# 详细说明

## 动机

为什么会有这个项目？在公司的某次周会上，我吐槽了某产品中一个显示鼠标尾迹的效果实现得比较抽象，当时我觉得这么简单一个效果完全应该做的完善一点，也花不了多少时间。

紧接着的一个周末的中午，我正在洗碗，突然脑子里灵光一闪，我意识到，在 web canvas 上要实现一个「完美」的鼠标尾迹效果似乎并没有我在周会上想象的那么简单。于是我决定自己尝试一下，就有了这个项目。

## 问题

所谓没有想象的那么简单，主要是有一下几个方面的问题。

- 通过鼠标移动的事件获取的鼠标轨迹是离散的点坐标，而不是真实的尾迹曲线，如何通过离散坐标拟合出一个平滑曲线？
- 鼠标尾迹的透明度应该是渐变的，web canvas 上并没有提供在一个 path 上做线性渐变的接口，这个效果如何实现？
- 鼠标尾迹的粗细也应该是渐变的，web canvas 上的单一 path 也没有提供画笔粗细渐变的接口，这个效果又如何实现？

## 原理

### 如何通过离散坐标点拟合出平滑曲线？

如果你用过 Photoshop 中的钢笔工具，答案其实就很简单，用贝塞尔曲线。Photoshop 中的钢笔工具其实就是一个贝塞尔曲线编辑器，通过起点、终点以及两个控制点，就可以在起点和终点间建立一条曲线。

![photoshop pen](./docs/assets/pen.png)

而如果一个中间点上的两个控制点满足一定的规律，就可以实现曲线的连续，也就是视觉效果上的平滑。感兴趣的话可以阅读「[用钢笔工具绘图](https://helpx.adobe.com/cn/photoshop/using/drawing-pen-tools.html)」中的内容。

那么中间点上的两个控制点满足什么样的规律就可以实现曲线的连续呢？其实也很简单，就是中间点和两个控制点在同一直线上即可。

### 如何实现线宽和透明度的渐变

## 性能

![flame chart](./docs/assets/performance_1.png)
从上面的火焰图可以看出，持续绘制鼠标尾迹的性能还是比较不错的，帧率始终都位置在 60fps 左右。

![flame chart](./docs/assets/performance_2.png)
每一次绘制的时间也基本上不到 0.5ms。

根据个人以往的经验，类似 Laser Pen 这样完全不依赖 DOM 和 BOM 的 JavaScript 库往往都有着不错的性能指标。JavaScript 在纯数学和逻辑处理上效率还是很高的。

## 总结

没事多洗洗碗。说不定就会有意外的收获 😂

# License

MIT