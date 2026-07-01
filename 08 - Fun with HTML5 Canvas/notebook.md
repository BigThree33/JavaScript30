# Day 08 - Fun with HTML5 Canvas

## 这一天主要练什么

Day 08 是 Canvas 鼠标绘图练习。它的重点不是 DOM 操作，而是理解 Canvas 的绘图上下文和鼠标事件流：

- 获取 `<canvas>` 的 2D 绘图上下文。
- 根据窗口尺寸设置画布大小。
- 监听鼠标按下、移动、松开、离开。
- 用上一帧坐标和当前坐标连线，形成连续笔迹。
- 动态改变颜色和线宽，制造彩色画笔效果。

这类能力可以迁移到签名板、涂鸦板、标注工具、截图批注、简单白板应用。

## HTML

START 版本已经放好了画布：

```html
<canvas id="draw" width="800" height="800"></canvas>
```

这是本挑战的唯一主要元素。FINISHED 版本没有改变结构，只是在脚本里接管这个 canvas。

注意：HTML 上写的 `width="800"` 和 `height="800"` 只是初始尺寸。FINISHED 版本会在 JavaScript 里把它改成浏览器窗口大小。

## CSS

CSS 只有一个目的：去掉页面默认边距。

```css
html, body {
  margin: 0;
}
```

如果不去掉默认 `margin`，canvas 即使设置成窗口宽高，也会因为 body 默认边距产生空白边。

这个挑战的视觉主体完全由 Canvas 绘制，不靠 CSS 布局。

## JavaScript

START 版本的 `<script>` 是空的。FINISHED 版本从初始化 canvas 开始。

### 1. 获取 canvas 和绘图上下文

```js
const canvas = document.querySelector('#draw');
const ctx = canvas.getContext('2d');
```

`canvas` 是 DOM 元素，`ctx` 是 2D 绘图上下文。真正的绘图 API 都在 `ctx` 上，例如：

```js
ctx.beginPath();
ctx.moveTo();
ctx.lineTo();
ctx.stroke();
```

可以把 `canvas` 理解成纸，把 `ctx` 理解成笔和绘图规则。

### 2. 让画布铺满窗口

```js
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;
```

这里设置的是 canvas 的绘图尺寸，不只是 CSS 显示尺寸。

Canvas 有两个尺寸概念：

- 元素显示尺寸：CSS 控制。
- 绘图缓冲区尺寸：`canvas.width` 和 `canvas.height` 控制。

如果只用 CSS 拉伸 canvas，绘图可能变模糊；直接设置 `canvas.width/height` 更符合这个练习的需求。

### 3. 设置画笔样式

```js
ctx.strokeStyle = '#BADA55';
ctx.lineJoin = 'round';
ctx.lineCap = 'round';
ctx.lineWidth = 100;
```

这些是绘制线条的默认配置：

- `strokeStyle`：线条颜色。
- `lineJoin = 'round'`：线条转角圆滑。
- `lineCap = 'round'`：线条端点圆滑。
- `lineWidth`：线宽。

FINISHED 版本后面会在绘制过程中动态修改 `strokeStyle` 和 `lineWidth`。

这行被注释掉了：

```js
// ctx.globalCompositeOperation = 'multiply';
```

`globalCompositeOperation` 控制新绘制内容和已有像素的混合方式。`multiply` 会让颜色叠加时产生类似正片叠底的效果，是 Canvas 绘图中很有用的扩展点。

### 4. 保存绘图状态

```js
let isDrawing = false;
let lastX = 0;
let lastY = 0;
let hue = 0;
let direction = true;
```

这些变量共同描述当前画笔状态：

- `isDrawing`：鼠标是否处于按下绘制状态。
- `lastX` / `lastY`：上一段线的起点。
- `hue`：当前颜色的色相。
- `direction`：线宽当前是在增加还是减少。

Canvas 本身不会替你记住“上一点在哪里”，连续笔迹必须自己维护上一次坐标。

### 5. 绘图函数

```js
function draw(e) {
  if (!isDrawing) return;
  ctx.strokeStyle = `hsl(${hue}, 100%, 50%)`;
  ctx.beginPath();
  ctx.moveTo(lastX, lastY);
  ctx.lineTo(e.offsetX, e.offsetY);
  ctx.stroke();
  [lastX, lastY] = [e.offsetX, e.offsetY];
  ...
}
```

这个函数只在鼠标移动时被调用，但不代表每次移动都要画线。

```js
if (!isDrawing) return;
```

这行保证只有鼠标按下后移动才绘制。否则鼠标从画布上经过也会留下线条。

绘制一段线的流程是：

1. `beginPath()` 开启新路径。
2. `moveTo(lastX, lastY)` 移动到上一点。
3. `lineTo(e.offsetX, e.offsetY)` 连到当前鼠标位置。
4. `stroke()` 真正描边。
5. 更新 `lastX/lastY`，让下一次移动从当前点继续。

`e.offsetX` 和 `e.offsetY` 是鼠标相对 canvas 元素左上角的位置，正好适合绘图坐标。

### 6. 让颜色循环变化

```js
hue++;
if (hue >= 360) {
  hue = 0;
}
```

绘图颜色使用的是：

```js
ctx.strokeStyle = `hsl(${hue}, 100%, 50%)`;
```

HSL 中的 hue 范围通常是 `0-360`。每画一小段线就让 hue 加 1，颜色就会沿色相环变化。超过 360 后回到 0，形成循环。

这比手写很多颜色值更适合做连续彩虹效果。

### 7. 让线宽来回变化

```js
if (ctx.lineWidth >= 100 || ctx.lineWidth <= 1) {
  direction = !direction;
}

if(direction) {
  ctx.lineWidth++;
} else {
  ctx.lineWidth--;
}
```

这里实现的是“碰到边界就反向”。

- 线宽到 100 时，开始变细。
- 线宽到 1 时，开始变粗。

`direction` 是一个布尔开关，用它决定当前应该加还是减。

这是很多动画都会用到的模式：状态值在一个范围内往返变化。

### 8. 鼠标事件绑定

```js
canvas.addEventListener('mousedown', (e) => {
  isDrawing = true;
  [lastX, lastY] = [e.offsetX, e.offsetY];
});
```

鼠标按下时进入绘制状态，并把起点设为当前鼠标位置。

这里必须更新 `lastX/lastY`。否则新一笔可能会从上一笔结束的位置突然连过来。

```js
canvas.addEventListener('mousemove', draw);
canvas.addEventListener('mouseup', () => isDrawing = false);
canvas.addEventListener('mouseout', () => isDrawing = false);
```

事件分工：

- `mousemove`：尝试绘制。
- `mouseup`：松开鼠标，停止绘制。
- `mouseout`：鼠标离开画布，也停止绘制，避免重新进入时继续连线。

## 主要实现过程

1. 获取 canvas 和 2D context。
2. 把 canvas 尺寸设置为窗口宽高。
3. 配置线条样式。
4. 用 `isDrawing` 控制是否允许绘图。
5. 鼠标按下时记录起点。
6. 鼠标移动时从上一点连到当前点。
7. 每次绘制后更新上一点坐标。
8. 绘制过程中动态改变 hue 和 lineWidth。
9. 鼠标松开或离开时停止绘制。

## 值得记住

- Canvas 是命令式绘图：画上去的像素不会自动对应 DOM 节点。
- 连续线条的关键是保存上一帧坐标。
- `mousedown + mousemove + mouseup` 是拖拽类交互的基本事件组合。
- `hsl()` 很适合做连续色彩变化。
- 动态绘图中，很多效果都来自“状态变量 + 每帧更新”。
