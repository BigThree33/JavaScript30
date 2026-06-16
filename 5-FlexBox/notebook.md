# JavaScript30 Day 5: Flex Panels Image Gallery

## 1. 这个练习在做什么

这个练习把 5 张图片做成一组横向铺满屏幕的面板。每个面板里有 3 行文字：

- 默认状态：5 个面板平均分配宽度，第一行和第三行文字藏在面板外。
- 点击某个面板：该面板变宽，字号变大。
- 面板变宽动画结束后：第一行和第三行文字滑入可见区域。
- 再次点击同一个面板：面板收回，文字再滑出去。

核心知识点有两个：

- CSS Flexbox：用弹性布局分配面板宽度，并让文字在面板中垂直、水平居中。
- JavaScript class 切换：点击时切换 CSS 类，监听 CSS 过渡结束，再触发下一段动画。

起始文件是 `index-START.html`，完成文件是 `index-FINISHED.html`。下面重点解释完成版相对起始版新增或修改的代码。

## 2. HTML 中的变化

```html
<link rel="icon" href="https://fav.farm/✅" />
```

起始版里 favicon 是火焰图标，完成版改成了勾号图标。

这行不影响 Flexbox 或 JavaScript 逻辑，只是页面标签页的小图标。它表示这是完成版页面。实际项目里，`rel="icon"` 常用于设置网站 favicon，例如品牌 logo、状态图标或环境标识。

## 3. 外层容器 `.panels`

完成版在 `.panels` 中新增了：

```css
display: flex;
```

### 这行是什么意思

`.panels` 是 5 个 `.panel` 的父容器。设置 `display: flex;` 后，`.panels` 成为一个 Flex 容器，里面的直接子元素 `.panel` 自动成为 Flex items。

默认情况下，Flex 容器的主轴方向是横向：

```css
flex-direction: row;
```

所以 5 个面板会从左到右排列，而不是像普通 `div` 一样从上到下堆叠。

### 这行解决了什么问题

如果没有这行：

- 每个 `.panel` 仍然是普通块级元素。
- 5 个面板会垂直排列。
- 后面的 `flex: 1`、`flex: 5` 无法对面板宽度产生预期效果。

### 如何运用

只要你想让一组子元素按一行或一列弹性排列，就可以先在父元素上写：

```css
.container {
  display: flex;
}
```

常见场景：

- 导航栏横向排列。
- 卡片列表平均分配宽度。
- 左右布局，比如侧边栏加主内容。
- 工具栏按钮排列。

## 4. 单个面板 `.panel`

完成版在 `.panel` 中新增了 4 行：

```css
flex: 1;
justify-content: center;
display: flex;
flex-direction: column;
```

这 4 行非常关键，因为 `.panel` 在这个练习中有双重身份：

- 它是 `.panels` 的子元素，所以它是外层 Flex 容器里的 Flex item。
- 它自己又要管理内部 3 个 `<p>`，所以它也被设置成了 Flex container。

## 4.1 `flex: 1;`

```css
flex: 1;
```

### 这行是什么意思

`flex` 是一个简写属性，常见完整形式是：

```css
flex: flex-grow flex-shrink flex-basis;
```

这里写 `flex: 1;`，可以理解为“每个面板都愿意平均瓜分父容器里的剩余空间”。

5 个 `.panel` 都有 `flex: 1`，所以默认状态下 5 个面板宽度相等。

### 这行解决了什么问题

如果没有 `flex: 1`：

- 面板宽度会根据内容、背景和默认块级行为来决定。
- 外层虽然已经 `display: flex`，但每个面板不一定会平均撑满整行。
- 后面的 `.panel.open { flex: 5; }` 就没有“从 1 变到 5”的动画基础。

### 如何运用

平均分栏时非常常用：

```css
.item {
  flex: 1;
}
```

如果有三个子元素都设置 `flex: 1`，它们会大致各占三分之一。

如果其中一个设置 `flex: 2`，空间比例就会变成 `1 : 2 : 1`。

## 4.2 `justify-content: center;`

```css
justify-content: center;
```

### 这行是什么意思

`justify-content` 控制 Flex 容器中子元素沿主轴的对齐方式。

但要注意：这行写在 `.panel` 上，所以它控制的是 `.panel` 内部的 3 个 `<p>`，不是控制 5 个面板之间的排列。

由于 `.panel` 后面又写了：

```css
flex-direction: column;
```

所以 `.panel` 内部主轴变成了垂直方向。于是：

```css
justify-content: center;
```

表示把 3 个 `<p>` 在垂直方向上居中排列。

### 这行解决了什么问题

如果没有它：

- `<p>` 可能从面板顶部开始排列。
- 面板中间的主文字不一定自然位于视觉中心。

### 如何运用

Flexbox 中要先判断主轴方向：

```css
.row {
  display: flex;
  flex-direction: row;
  justify-content: center; /* 横向居中 */
}

.column {
  display: flex;
  flex-direction: column;
  justify-content: center; /* 纵向居中 */
}
```

## 4.3 `display: flex;`

```css
display: flex;
```

### 这行是什么意思

这行让单个 `.panel` 也变成 Flex 容器。

外层 `.panels` 使用 Flex，是为了排列 5 个面板。

内层 `.panel` 使用 Flex，是为了排列它自己的 3 个文字段落。

这就是嵌套 Flex 布局：

```text
.panels                 Flex container
  .panel                Flex item，同时也是 Flex container
    p                   Flex item
    p                   Flex item
    p                   Flex item
```

### 这行解决了什么问题

没有这行时，`.panel` 里的 `justify-content`、`align-items`、`flex-direction` 不会按 Flexbox 规则生效。

起始版里已经有：

```css
align-items: center;
```

但在 `.panel` 没有 `display: flex` 时，这行对 `.panel` 内部 `<p>` 的 Flex 对齐没有效果。完成版补上 `display: flex` 后，`align-items: center` 才真正参与布局。

### 如何运用

当你发现自己写了这些属性却没效果时：

```css
justify-content: center;
align-items: center;
flex-direction: column;
```

先检查对应元素本身有没有：

```css
display: flex;
```

这是 Flexbox 新手最常见的坑之一。

## 4.4 `flex-direction: column;`

```css
flex-direction: column;
```

### 这行是什么意思

这行把 `.panel` 内部的主轴从默认横向改成纵向。

也就是说，3 个 `<p>` 会按从上到下排列：

```text
Hey
Let's
Dance
```

如果不写这行，默认是：

```css
flex-direction: row;
```

3 个 `<p>` 会横向排成一行。

### 这行解决了什么问题

这个练习的设计是一个面板里竖向展示 3 行文字，中间文字最大，上下文字通过滑动动画出现。如果没有 `flex-direction: column`，整个视觉结构就变了。

### 如何运用

当你需要纵向布局但又想用 Flexbox 的居中、伸缩和空间分配能力时，可以写：

```css
.stack {
  display: flex;
  flex-direction: column;
}
```

常见场景：

- 页面主布局：顶部、主体、底部。
- 卡片内部：标题、内容、按钮。
- 弹窗内部：头部、正文、操作区。

## 5. `.panel > *`：面板内所有直接子元素

起始版注释是：

```css
/* Flex Children */
```

完成版改成：

```css
/* Flex Items */
```

这个改动虽然只是注释，但概念更准确。

在 Flexbox 里，父元素叫 Flex container，直接子元素叫 Flex items。`.panel > *` 选中的正是 `.panel` 的所有直接子元素，也就是 3 个 `<p>`。

完成版在 `.panel > *` 中新增了 4 行：

```css
flex: 1 0 auto;
display: flex;
justify-content: center;
align-items: center;
```

## 5.1 `flex: 1 0 auto;`

```css
flex: 1 0 auto;
```

### 这行是什么意思

这是 `flex-grow`、`flex-shrink`、`flex-basis` 的简写：

```css
flex-grow: 1;
flex-shrink: 0;
flex-basis: auto;
```

分别表示：

- `flex-grow: 1`：可以增长，占用可用空间。
- `flex-shrink: 0`：空间不足时不要压缩到更小。
- `flex-basis: auto`：基础尺寸先按内容或自身尺寸计算。

因为 `.panel` 是纵向 Flex 容器，3 个 `<p>` 是竖向排列的 Flex items。所以这行让每个 `<p>` 在垂直方向上分配空间。

### 这行解决了什么问题

每个面板里有 3 行文字。通过 `flex: 1 0 auto`，三行文字所在区域可以平均撑开，使它们分别占据上、中、下三个区域：

```text
上方文字区域
中间文字区域
下方文字区域
```

这样中间那行大字才能稳定地位于视觉中心，上下两行也有足够空间做滑入滑出。

### 如何运用

在纵向布局中，如果你想让几个子块均分高度，可以使用：

```css
.parent {
  display: flex;
  flex-direction: column;
}

.parent > * {
  flex: 1;
}
```

如果你还想避免子项被压缩，可以更明确地写：

```css
.parent > * {
  flex: 1 0 auto;
}
```

## 5.2 `display: flex;`

```css
display: flex;
```

### 这行是什么意思

这里又出现了一层 Flex。

每个 `<p>` 不只是 `.panel` 的 Flex item，它自己也被设置成 Flex container。

这样做的目的不是为了排列多个子元素，因为 `<p>` 里只有文字；它的目的是借助 Flexbox 非常方便地让文字在 `<p>` 这个区域内居中。

### 这行解决了什么问题

上一行 `flex: 1 0 auto` 让每个 `<p>` 获得一个较大的区域。现在需要让文字位于这个区域的正中央，而不是贴着区域顶部或默认基线位置。

`display: flex` 是后面两行居中属性生效的前提。

## 5.3 `justify-content: center;`

```css
justify-content: center;
```

### 这行是什么意思

写在 `<p>` 上时，它控制 `<p>` 内部内容沿主轴居中。

`<p>` 的默认 Flex 主轴是横向，所以这行让文字在 `<p>` 盒子的横向方向居中。

不过这个文件里 `.panel` 本身已经有：

```css
text-align: center;
```

所以从纯文字水平居中的角度看，`text-align: center` 已经能做到一部分。但这里用 Flex 居中可以和下一行 `align-items: center` 配合，让文字在自己的区域内完整居中。

### 如何运用

让按钮、标签、单元格里的内容水平居中：

```css
.button {
  display: flex;
  justify-content: center;
}
```

## 5.4 `align-items: center;`

```css
align-items: center;
```

### 这行是什么意思

写在 `<p>` 上时，它控制 `<p>` 内部内容沿交叉轴居中。

`<p>` 默认主轴是横向，交叉轴就是纵向。所以这行让文字在 `<p>` 盒子的垂直方向居中。

### 这行解决了什么问题

每个 `<p>` 都被分配了一块区域。没有 `align-items: center` 时，文字可能不会正好位于该区域的垂直中心。加上后，三行文字的视觉位置更稳。

### 如何运用

最常见的水平垂直居中写法：

```css
.center {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

## 6. 上下文字的滑入滑出动画

完成版新增了 4 个选择器：

```css
.panel > *:first-child { transform: translateY(-100%); }
.panel.open-active > *:first-child { transform: translateY(0); }
.panel > *:last-child { transform: translateY(100%); }
.panel.open-active > *:last-child { transform: translateY(0); }
```

它们控制每个面板的第一行和第三行文字。

## 6.1 第一行默认向上藏起来

```css
.panel > *:first-child { transform: translateY(-100%); }
```

### 这行是什么意思

`.panel > *:first-child` 选中每个 `.panel` 里的第一个直接子元素，也就是第一行 `<p>`。

`transform: translateY(-100%)` 表示让它沿 Y 轴向上移动自身高度的 100%。

结果就是第一行文字默认藏到面板上方。

### 知识点

`translateY()` 是 CSS transform 的一种，用于垂直平移元素。

- `translateY(-100%)`：向上移动自身高度。
- `translateY(100%)`：向下移动自身高度。
- `translateY(0)`：回到原位置。

这里的百分比是相对于元素自身尺寸，不是父元素尺寸。

## 6.2 面板激活后第一行回到原位

```css
.panel.open-active > *:first-child { transform: translateY(0); }
```

### 这行是什么意思

当 `.panel` 同时拥有 `open-active` 类时，第一行文字移动回原位。

由于 `.panel > *` 里已经写了：

```css
transition: transform 0.5s;
```

所以从 `translateY(-100%)` 到 `translateY(0)` 会有一个 0.5 秒的滑动动画。

### 这行解决了什么问题

它定义的是“显示状态”。默认状态藏起来，激活状态滑进来。

状态切换由 JavaScript 完成：

```js
this.classList.toggle('open-active');
```

## 6.3 第三行默认向下藏起来

```css
.panel > *:last-child { transform: translateY(100%); }
```

### 这行是什么意思

`.panel > *:last-child` 选中每个 `.panel` 里的最后一个直接子元素，也就是第三行 `<p>`。

`translateY(100%)` 让它向下移动自身高度的 100%，默认藏到下方。

### 为什么第一行用 `-100%`，第三行用 `100%`

因为它们要从不同方向进入：

- 第一行从上方滑入，所以默认向上藏。
- 第三行从下方滑入，所以默认向下藏。

这样动画会更有层次。

## 6.4 面板激活后第三行回到原位

```css
.panel.open-active > *:last-child { transform: translateY(0); }
```

### 这行是什么意思

当 `.panel` 拥有 `open-active` 类时，最后一行文字也回到原位。

它和第一行的激活规则对应：

```css
.panel.open-active > *:first-child { transform: translateY(0); }
.panel.open-active > *:last-child { transform: translateY(0); }
```

这两行共同完成上下文字的滑入。

## 7. `.panel.open`：点击后面板变大

起始版中 `.panel.open` 只有：

```css
font-size: 40px;
```

完成版新增了：

```css
flex: 5;
```

完整规则是：

```css
.panel.open {
  flex: 5;
  font-size: 40px;
}
```

### 这行是什么意思

默认每个面板是：

```css
flex: 1;
```

点击后当前面板加上 `open` 类，变成：

```css
flex: 5;
```

也就是当前面板比其他面板更愿意占用空间。

如果 5 个面板中只有一个打开，那么比例大致是：

```text
1 : 1 : 5 : 1 : 1
```

打开的面板会明显变宽。

### 为什么它有动画

`.panel` 里已经定义了过渡：

```css
transition:
  font-size 0.7s cubic-bezier(0.61,-0.19, 0.7,-0.11),
  flex 0.7s cubic-bezier(0.61,-0.19, 0.7,-0.11),
  background 0.2s;
```

所以当 `flex` 从 `1` 变成 `5` 时，浏览器会播放 0.7 秒的动画。

### 如何运用

这种写法适合做“当前项展开”的交互：

```css
.item {
  flex: 1;
  transition: flex 0.3s ease;
}

.item.active {
  flex: 3;
}
```

应用场景：

- 图片手风琴。
- 横向菜单展开。
- 产品展示区。
- 可展开侧栏。

## 8. 移动端字号适配

完成版新增：

```css
@media only screen and (max-width: 600px) {
  .panel p {
    font-size: 1em;
  }
}
```

## 8.1 `@media only screen and (max-width: 600px)`

```css
@media only screen and (max-width: 600px) {
```

### 这行是什么意思

这是媒体查询，表示：当设备是屏幕，并且视口宽度不超过 `600px` 时，应用里面的 CSS。

简单理解：这是给手机或窄屏设备准备的样式。

### 如何运用

响应式设计中常用媒体查询根据屏幕宽度调整布局：

```css
@media (max-width: 768px) {
  .layout {
    flex-direction: column;
  }
}
```

## 8.2 移动端缩小段落字号

```css
.panel p {
  font-size: 1em;
}
```

### 这行是什么意思

在窄屏下，把 `.panel p` 的字号改成 `1em`。

起始的普通段落是：

```css
.panel p {
  font-size: 2em;
}
```

中间段落是：

```css
.panel p:nth-child(2) {
  font-size: 4em;
}
```

移动端空间有限，如果还保持大字号，文字容易挤压、换行或溢出。完成版用媒体查询降低基础字号，让内容更适合小屏。

### 注意

这条规则只改 `.panel p` 的普通字号。由于 `.panel p:nth-child(2)` 的选择器更具体，并且仍然存在，中间文字可能仍然保持更大的视觉权重。

如果实际项目里手机端仍然太大，可以额外写：

```css
@media (max-width: 600px) {
  .panel p:nth-child(2) {
    font-size: 2em;
  }
}
```

## 9. JavaScript：选中所有面板

完成版在 `<script>` 中新增：

```js
const panels = document.querySelectorAll('.panel');
```

### 这行是什么意思

`document.querySelectorAll('.panel')` 会找到页面中所有 class 包含 `panel` 的元素，并返回一个 `NodeList`。

在这个页面里，返回的是 5 个面板：

```text
.panel.panel1
.panel.panel2
.panel.panel3
.panel.panel4
.panel.panel5
```

`const panels = ...` 把这个列表保存起来，方便后面统一绑定事件。

### 知识点

`querySelectorAll` 接收 CSS 选择器：

```js
document.querySelectorAll('p');          // 所有 p
document.querySelectorAll('.panel');     // 所有 class 为 panel 的元素
document.querySelectorAll('#app');       // id 为 app 的元素
document.querySelectorAll('.panel > p'); // panel 里的直接 p 子元素
```

### 如何运用

当你要对一组元素做同样操作时，先选中它们：

```js
const buttons = document.querySelectorAll('.button');
```

然后再遍历绑定事件或修改样式。

## 10. JavaScript：点击时切换打开状态

完成版新增函数：

```js
function toggleOpen() {
  console.log('Hello');
  this.classList.toggle('open');
}
```

## 10.1 函数声明

```js
function toggleOpen() {
```

### 这行是什么意思

定义一个名为 `toggleOpen` 的函数。后面会把它作为点击事件的处理函数：

```js
panel.addEventListener('click', toggleOpen)
```

当某个面板被点击时，浏览器会调用这个函数。

### 如何运用

事件处理函数通常单独定义出来，这样代码更清晰：

```js
function handleClick() {
  // 点击后要做的事
}
```

## 10.2 调试输出

```js
console.log('Hello');
```

### 这行是什么意思

在浏览器控制台输出 `Hello`。

这行主要用于调试：点击面板后，如果控制台出现 `Hello`，说明点击事件确实触发了。

### 实际项目中怎么处理

调试完成后，这行通常可以删除。它不影响页面功能，但生产代码里一般不保留无意义的日志。

## 10.3 切换 `open` 类

```js
this.classList.toggle('open');
```

### 这行是什么意思

`this` 指向当前触发事件的面板元素。

`classList.toggle('open')` 的作用是：

- 如果当前元素没有 `open` 类，就添加它。
- 如果当前元素已经有 `open` 类，就移除它。

所以点击一次打开，再点击一次关闭。

### 为什么这里不能随便换成箭头函数

这里的事件回调使用的是普通函数：

```js
function toggleOpen() {}
```

普通函数作为事件处理函数时，`this` 通常指向绑定事件的元素。

如果写成箭头函数：

```js
const toggleOpen = () => {
  this.classList.toggle('open');
};
```

箭头函数不会创建自己的 `this`，这里的 `this` 就不再自动指向被点击的 `.panel`，代码可能失效。

### 这行如何联动 CSS

添加 `open` 类后，CSS 规则生效：

```css
.panel.open {
  flex: 5;
  font-size: 40px;
}
```

所以面板变宽，字体变大。

JavaScript 只负责改状态，动画和视觉效果交给 CSS。这是一个很好的前端交互分工方式。

## 11. JavaScript：监听过渡结束

完成版新增函数：

```js
function toggleActive(e) {
  console.log(e.propertyName);
  if (e.propertyName.includes('flex')) {
    this.classList.toggle('open-active');
  }
}
```

这个函数用于在面板变宽动画结束后，再让上下文字滑入。

## 11.1 接收事件对象

```js
function toggleActive(e) {
```

### 这行是什么意思

`e` 是事件对象。对于 `transitionend` 事件，它里面会包含这次结束的是哪个 CSS 过渡属性。

比如可能是：

```text
font-size
flex-grow
background
```

因为 `.panel` 同时 transition 了多个属性：

```css
font-size
flex
background
```

所以 `transitionend` 可能触发多次。必须判断是哪一个属性结束了。

## 11.2 打印结束的属性名

```js
console.log(e.propertyName);
```

### 这行是什么意思

把刚刚结束过渡的 CSS 属性名打印出来。

这同样是调试用代码。它能帮助你看到不同浏览器返回的属性名。

文件里也有注释提醒：

```css
/* Safari transitionend event.propertyName === flex */
/* Chrome + FF transitionend event.propertyName === flex-grow */
```

也就是说：

- Safari 可能返回 `flex`。
- Chrome 和 Firefox 可能返回 `flex-grow`。

## 11.3 判断是否是 Flex 相关动画结束

```js
if (e.propertyName.includes('flex')) {
```

### 这行是什么意思

`includes('flex')` 判断 `e.propertyName` 里是否包含字符串 `flex`。

这样无论浏览器返回的是：

```text
flex
```

还是：

```text
flex-grow
```

都能通过判断。

### 为什么不直接写 `e.propertyName === 'flex'`

因为不同浏览器返回的属性名不完全一致。

如果只写：

```js
e.propertyName === 'flex'
```

在 Chrome 或 Firefox 中可能无法触发 `open-active` 的切换。

使用：

```js
e.propertyName.includes('flex')
```

兼容性更好。

## 11.4 切换 `open-active` 类

```js
this.classList.toggle('open-active');
```

### 这行是什么意思

当 Flex 相关的过渡结束后，给当前面板切换 `open-active` 类。

添加 `open-active` 后，这些 CSS 生效：

```css
.panel.open-active > *:first-child { transform: translateY(0); }
.panel.open-active > *:last-child { transform: translateY(0); }
```

上下文字从隐藏位置滑回原位。

### 为什么不在点击时同时加 `open` 和 `open-active`

如果点击时立刻同时加：

```js
open
open-active
```

面板变宽和文字滑入会同时发生。

这个练习想要的是两段式动画：

1. 面板先变宽、字号变大。
2. 变宽动画结束后，上下文字再滑入。

所以 `open-active` 放在 `transitionend` 之后切换。

## 12. 给每个面板绑定点击事件

完成版新增：

```js
panels.forEach(panel => panel.addEventListener('click', toggleOpen));
```

### 这行是什么意思

`panels` 是前面选中的所有面板。`forEach` 会逐个遍历它们。

对每一个 `panel`，执行：

```js
panel.addEventListener('click', toggleOpen)
```

也就是给每个面板绑定点击事件。点击时执行 `toggleOpen`。

### 拆开来看

这行等价于：

```js
panels.forEach(function(panel) {
  panel.addEventListener('click', toggleOpen);
});
```

箭头函数 `panel => ...` 是更简洁的写法。

### 知识点

`addEventListener` 的基本格式：

```js
element.addEventListener(eventName, handler);
```

例如：

```js
button.addEventListener('click', handleClick);
input.addEventListener('input', handleInput);
window.addEventListener('resize', handleResize);
```

注意这里传入的是函数名：

```js
toggleOpen
```

不是函数调用：

```js
toggleOpen()
```

如果写成 `toggleOpen()`，页面加载时就会立刻执行函数，而不是等点击时执行。

## 13. 给每个面板绑定过渡结束事件

完成版新增：

```js
panels.forEach(panel => panel.addEventListener('transitionend', toggleActive));
```

### 这行是什么意思

给每个面板绑定 `transitionend` 事件。

当某个面板上的 CSS transition 完成时，浏览器触发 `transitionend`，然后执行 `toggleActive`。

### 为什么监听的是 `.panel`

面板变宽的动画发生在 `.panel` 自己身上：

```css
.panel.open {
  flex: 5;
  font-size: 40px;
}
```

所以监听 `.panel` 的 `transitionend` 是合理的。

### 与点击事件的配合

完整流程是：

1. 用户点击面板。
2. `click` 事件触发 `toggleOpen`。
3. `toggleOpen` 切换 `open` 类。
4. `.panel.open` 的 `flex` 和 `font-size` 发生变化。
5. CSS transition 播放。
6. Flex 相关动画结束后触发 `transitionend`。
7. `toggleActive` 检查 `e.propertyName` 是否包含 `flex`。
8. 如果是，切换 `open-active` 类。
9. 上下两行文字通过 `transform` 动画滑入或滑出。

## 14. 完整交互状态图

默认状态：

```text
.panel
flex: 1
font-size: 20px
first-child: translateY(-100%)
last-child: translateY(100%)
```

点击后：

```text
.panel.open
flex: 5
font-size: 40px
```

Flex 过渡结束后：

```text
.panel.open.open-active
first-child: translateY(0)
last-child: translateY(0)
```

再次点击后：

```text
移除 open
flex 从 5 回到 1
font-size 从 40px 回到 20px
transitionend 后移除 open-active
上下文字滑出
```

## 15. 这个练习真正要掌握的知识点

## 15.1 Flex 容器和 Flex item 是相对关系

同一个元素可以同时是两种身份。

在这个练习里：

```text
.panels 是 Flex container
.panel 是 .panels 的 Flex item

.panel 又是 Flex container
p 是 .panel 的 Flex item

p 又是 Flex container
p 里的文字被居中
```

判断 Flexbox 时，不要只问“这个元素是不是 flex”，而要问：

- 它相对于谁是 Flex item？
- 它自己有没有成为 Flex container？
- 当前主轴方向是 row 还是 column？

## 15.2 `justify-content` 和 `align-items` 要看主轴

默认：

```css
flex-direction: row;
```

此时：

- `justify-content` 控制横向。
- `align-items` 控制纵向。

如果改成：

```css
flex-direction: column;
```

此时：

- `justify-content` 控制纵向。
- `align-items` 控制横向。

这个练习中 `.panel` 是 column，所以 `.panel` 上的 `justify-content: center` 是纵向居中。

## 15.3 用 class 表示状态

这个练习没有在 JavaScript 里直接写：

```js
panel.style.flex = 5;
panel.style.fontSize = '40px';
```

而是写：

```js
this.classList.toggle('open');
```

这是更推荐的方式。

原因：

- JavaScript 负责状态变化。
- CSS 负责视觉表现。
- 样式集中在 CSS 中，更容易维护。
- 同一个状态可以同时触发多个样式变化。

实际项目里也常见：

```js
menu.classList.toggle('is-open');
modal.classList.add('visible');
tab.classList.remove('active');
```

## 15.4 `transitionend` 可以串联动画

这个练习不是所有动画同时开始，而是用 `transitionend` 做顺序控制。

先打开面板：

```js
this.classList.toggle('open');
```

等打开动画结束：

```js
this.classList.toggle('open-active');
```

这就是“用 CSS transition + JS 事件串联动画”的基础模式。

常见应用：

- 弹窗先淡入，再播放内部内容动画。
- 卡片先展开，再显示详细信息。
- 页面切换时先收起旧内容，再显示新内容。

## 15.5 注意 `transitionend` 会触发多次

一个元素如果 transition 了多个属性：

```css
transition:
  font-size 0.7s,
  flex 0.7s,
  background 0.2s;
```

那么这些属性结束时都可能触发 `transitionend`。

所以代码中要过滤：

```js
if (e.propertyName.includes('flex')) {
  this.classList.toggle('open-active');
}
```

否则 `open-active` 可能被反复 toggle，导致状态错乱。

## 15.6 `transform` 适合做动画

上下文字滑动使用的是：

```css
transform: translateY(...);
```

而不是改：

```css
top
margin
height
```

原因是 `transform` 通常更适合做动画，性能更好，也不容易影响其他元素的布局。

常见动画属性：

```css
transform: translateX(100px);
transform: translateY(-100%);
transform: scale(1.2);
transform: rotate(10deg);
```

## 16. 可以迁移到哪些场景

### 图片手风琴

这个练习本身就是一个图片手风琴。可以用在：

- 摄影作品展示。
- 旅行地点展示。
- 产品卖点展示。
- 团队成员展示。

### 可展开导航

用 `flex` 改变当前项比例，可以做展开式导航：

```css
.nav-item {
  flex: 1;
}

.nav-item.active {
  flex: 2;
}
```

### 卡片详情展开

点击卡片后加一个类：

```js
card.classList.toggle('active');
```

CSS 中让它变大、显示详情：

```css
.card.active {
  flex: 3;
}

.card.active .details {
  transform: translateY(0);
}
```

### 分步动画

用 `transitionend` 让动画按顺序发生：

```js
element.addEventListener('transitionend', event => {
  if (event.propertyName === 'opacity') {
    element.classList.add('next-step');
  }
});
```

## 17. 常见问题

### 为什么 `align-items: center` 一开始没效果？

因为起始版 `.panel` 没有：

```css
display: flex;
```

Flexbox 的对齐属性必须在 Flex 容器上才会按 Flex 规则生效。

### 为什么 `.panel.open` 要改 `flex`，不是改 `width`？

因为父容器 `.panels` 是 Flex 布局。在 Flex 布局中，子项空间分配应该优先使用 `flex`。这样其他面板会自动响应当前面板的变大，不需要手动计算宽度。

### 为什么要用 `includes('flex')`？

因为不同浏览器在 `transitionend` 事件中返回的属性名可能不同：

- Safari 可能是 `flex`。
- Chrome 和 Firefox 可能是 `flex-grow`。

`includes('flex')` 能同时兼容这两类情况。

### 为什么第一行和第三行能滑动？

因为默认状态通过 `transform` 把它们移出：

```css
translateY(-100%)
translateY(100%)
```

激活状态再回到：

```css
translateY(0)
```

再加上：

```css
transition: transform 0.5s;
```

所以产生滑动动画。

### 为什么使用普通函数而不是箭头函数定义 `toggleOpen`？

因为代码里用到了：

```js
this.classList.toggle('open');
```

普通函数作为事件处理函数时，`this` 指向触发事件的元素。箭头函数没有自己的 `this`，不适合这里直接替换。

## 18. 复习清单

学完这个练习后，可以检查自己是否掌握这些点：

- 我知道父元素设置 `display: flex` 后，直接子元素才成为 Flex items。
- 我知道同一个元素可以既是 Flex item，又是 Flex container。
- 我能解释 `flex: 1` 和 `flex: 5` 为什么能让面板宽度变化。
- 我能根据 `flex-direction` 判断 `justify-content` 控制的是横向还是纵向。
- 我知道 `classList.toggle()` 是在切换元素状态。
- 我知道 `transitionend` 可能因为多个属性触发多次。
- 我知道为什么要用 `e.propertyName.includes('flex')` 做兼容判断。
- 我知道 `transform: translateY()` 可以让元素滑入滑出。
- 我知道 CSS 负责动画表现，JavaScript 负责状态切换。

## 19. 最小核心代码回顾

CSS 状态：

```css
.panels {
  display: flex;
}

.panel {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.panel.open {
  flex: 5;
}

.panel > *:first-child {
  transform: translateY(-100%);
}

.panel > *:last-child {
  transform: translateY(100%);
}

.panel.open-active > * {
  transform: translateY(0);
}
```

JavaScript 状态切换：

```js
const panels = document.querySelectorAll('.panel');

function toggleOpen() {
  this.classList.toggle('open');
}

function toggleActive(e) {
  if (e.propertyName.includes('flex')) {
    this.classList.toggle('open-active');
  }
}

panels.forEach(panel => panel.addEventListener('click', toggleOpen));
panels.forEach(panel => panel.addEventListener('transitionend', toggleActive));
```

这套模式可以总结成一句话：

> 用 Flexbox 负责布局，用 CSS transition 负责动画，用 JavaScript 切换 class 来驱动状态变化。
