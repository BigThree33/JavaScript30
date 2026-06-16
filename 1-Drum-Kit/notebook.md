# Drum Kit 学习笔记

这份笔记整理 `index.html` 中涉及到的核心知识点。这个小项目的目标是：按下键盘上的指定按键时，页面上对应的鼓点按钮产生动画效果，同时播放对应的音频。

## 1. 页面结构

页面主要由两类元素组成：

```html
<div data-key="65" class="key">
  <kbd>A</kbd>
  <span class="sound">clap</span>
</div>

<audio data-key="65" src="sounds/clap.wav"></audio>
```

第一类是可视化按键：

```html
<div data-key="65" class="key">...</div>
```

第二类是音频元素：

```html
<audio data-key="65" src="sounds/clap.wav"></audio>
```

这两类元素通过相同的 `data-key` 连接起来。例如：

- `div[data-key="65"]` 表示页面上按键 `A` 对应的可视化按钮。
- `audio[data-key="65"]` 表示按键 `A` 对应的音频文件。

当用户按下键盘上的 `A` 时，浏览器会产生一个键盘事件，事件中的按键码可以和页面上的 `data-key="65"` 匹配，从而找到对应的按钮和音频。

## 2. `data-key` 自定义属性

`data-key` 是 HTML5 的自定义数据属性。

HTML 允许我们使用 `data-*` 的形式给元素保存额外信息：

```html
<div data-key="65"></div>
```

这里的 `data-key` 不会影响页面默认样式或行为，它的作用是给 JavaScript 提供一个可以查询和匹配的标记。

在这个项目中，`data-key` 的值通常对应键盘按键的编码，例如：

| 按键 | keyCode | 页面元素 |
| --- | ---: | --- |
| A | 65 | `data-key="65"` |
| S | 83 | `data-key="83"` |
| D | 68 | `data-key="68"` |
| F | 70 | `data-key="70"` |
| G | 71 | `data-key="71"` |
| H | 72 | `data-key="72"` |
| J | 74 | `data-key="74"` |
| K | 75 | `data-key="75"` |
| L | 76 | `data-key="76"` |

> 说明：`keyCode` 在现代 Web 标准中已经不再推荐作为新项目首选，实际开发中更推荐使用 `event.key` 或 `event.code`。不过 JavaScript30 这个练习使用 `keyCode`，是为了让 DOM 查询和属性匹配的思路更直观。

## 3. 键盘事件

项目通过监听 `keydown` 事件来捕获用户按下键盘的行为：

```js
window.addEventListener('keydown', playSound);
```

含义是：当用户按下键盘时，浏览器会触发 `keydown` 事件，然后执行 `playSound` 函数。

事件监听的一般形式是：

```js
目标对象.addEventListener('事件名', 回调函数);
```

在这里：

- `window`：监听整个浏览器窗口。
- `'keydown'`：键盘按下事件。
- `playSound`：事件触发后执行的函数。

当事件触发时，浏览器会自动把事件对象传给回调函数：

```js
function playSound(e) {
  console.log(e);
}
```

这个 `e` 中包含了按键相关信息，例如 `e.keyCode`。

## 4. DOM 查询

DOM 是 Document Object Model 的缩写，即文档对象模型。浏览器会把 HTML 页面解析成一棵 DOM 树，JavaScript 可以通过 DOM API 查找和操作页面元素。

这个项目中常用的是 `document.querySelector` 和 `document.querySelectorAll`。

### 4.1 `querySelector`

`querySelector` 用于查找匹配选择器的第一个元素：

```js
const audio = document.querySelector(`audio[data-key="${e.keyCode}"]`);
const key = document.querySelector(`.key[data-key="${e.keyCode}"]`);
```

这里使用的是 CSS 属性选择器：

```css
audio[data-key="65"]
```

它表示：查找一个 `audio` 标签，并且这个标签的 `data-key` 属性值是 `65`。

当用户按下 `A` 键时，`e.keyCode` 是 `65`，所以模板字符串会拼出：

```js
document.querySelector('audio[data-key="65"]');
document.querySelector('.key[data-key="65"]');
```

这样就可以同时找到：

- 要播放的音频元素。
- 要添加动画样式的页面按钮。

### 4.2 `querySelectorAll`

`querySelectorAll` 用于查找所有匹配的元素：

```js
const keys = document.querySelectorAll('.key');
```

这行代码会找到页面上所有 class 为 `key` 的元素，并返回一个类似数组的 `NodeList`。

之后可以遍历这些元素，为每个按键绑定 transition 结束事件：

```js
keys.forEach(key => key.addEventListener('transitionend', removeTransition));
```

## 5. 模板字符串

代码中常见这一类写法：

```js
`audio[data-key="${e.keyCode}"]`
```

这是 JavaScript 的模板字符串。

模板字符串使用反引号包裹：

```js
`hello`
```

它可以通过 `${}` 插入变量或表达式：

```js
const name = 'Tom';
console.log(`Hello, ${name}`);
```

在 Drum Kit 中，模板字符串用来动态生成 CSS 选择器：

```js
const selector = `audio[data-key="${e.keyCode}"]`;
```

如果 `e.keyCode` 是 `65`，结果就是：

```js
audio[data-key="65"]
```

## 6. 音频播放

HTML 中的 `<audio>` 标签用于加载和播放音频：

```html
<audio data-key="65" src="sounds/clap.wav"></audio>
```

JavaScript 可以通过 DOM 找到这个音频元素，并调用它的播放方法：

```js
audio.play();
```

完整逻辑通常是：

```js
function playSound(e) {
  const audio = document.querySelector(`audio[data-key="${e.keyCode}"]`);

  if (!audio) return;

  audio.currentTime = 0;
  audio.play();
}
```

### 6.1 为什么要判断 `if (!audio) return`

并不是键盘上的每一个按键都对应一个鼓点音频。

例如用户按下空格、回车、方向键时，页面中可能没有对应的：

```html
<audio data-key="..."></audio>
```

如果没有找到音频，`audio` 的值会是 `null`。这时继续执行：

```js
audio.play();
```

会报错，因为 `null` 没有 `play` 方法。

所以要先判断：

```js
if (!audio) return;
```

这行代码表示：如果没找到对应音频，就直接结束函数。

### 6.2 `audio.currentTime = 0`

这行代码很关键：

```js
audio.currentTime = 0;
```

`currentTime` 表示音频当前播放到第几秒。

把它设置为 `0` 的意思是：每次按键时，都让音频从头开始播放。

如果不加这行代码，当你快速连续按同一个键时，音频可能还没播放完，新的播放不会立刻从头触发，鼓点反馈会变得迟钝。

加上之后，连续敲击同一个键也能立即重新播放音效。

## 7. class 操作和动画效果

当按下某个按键时，页面上的对应按钮会添加一个 `playing` 类：

```js
key.classList.add('playing');
```

`classList` 是 DOM 元素上的一个属性，用于操作元素的 class。

常见方法包括：

```js
element.classList.add('active');
element.classList.remove('active');
element.classList.toggle('active');
element.classList.contains('active');
```

在这个项目中，CSS 会提前定义 `.playing` 的样式，例如：

```css
.playing {
  transform: scale(1.1);
  border-color: #ffc600;
  box-shadow: 0 0 1rem #ffc600;
}
```

当 JavaScript 执行：

```js
key.classList.add('playing');
```

这个元素就会应用 `.playing` 的样式，看起来像被按下或高亮。

## 8. CSS transition 和 `transitionend`

按键动画通常依赖 CSS transition：

```css
.key {
  transition: all 0.07s ease;
}
```

含义是：当 `.key` 元素的某些样式发生变化时，不要瞬间切换，而是在 `0.07s` 内平滑过渡。

当 `playing` 类被添加后，元素的 `transform`、`border-color`、`box-shadow` 等样式发生变化，于是 transition 动画开始。

动画结束时，浏览器会触发 `transitionend` 事件。

项目可以监听这个事件，并在动画结束后移除 `playing` 类：

```js
function removeTransition(e) {
  if (e.propertyName !== 'transform') return;
  this.classList.remove('playing');
}

const keys = document.querySelectorAll('.key');
keys.forEach(key => key.addEventListener('transitionend', removeTransition));
```

## 9. `removeTransition` 函数

这个函数的作用是：在 CSS 过渡动画结束后，把 `playing` 类移除，让按钮恢复原状。

```js
function removeTransition(e) {
  if (e.propertyName !== 'transform') return;
  this.classList.remove('playing');
}
```

### 9.1 `e.propertyName`

一次样式变化可能包含多个 transition 属性，例如：

- `transform`
- `border-color`
- `box-shadow`

这些属性的 transition 结束时，都可能触发 `transitionend`。

为了避免函数被重复执行，代码只关心 `transform` 的结束：

```js
if (e.propertyName !== 'transform') return;
```

如果结束的不是 `transform`，就直接返回。

### 9.2 `this`

在下面这种事件监听写法中：

```js
key.addEventListener('transitionend', removeTransition);
```

当 `removeTransition` 被触发时，函数内部的 `this` 通常指向绑定事件的那个元素，也就是当前动画结束的 `.key` 元素。

所以：

```js
this.classList.remove('playing');
```

表示从当前这个按键元素上移除 `playing` 类。

注意：如果这里使用箭头函数直接定义 `removeTransition`，`this` 的行为会不同。箭头函数没有自己的 `this`，它会继承外层作用域的 `this`。

## 10. 完整执行流程

一次按键触发的完整流程如下：

1. 用户按下键盘，例如 `A`。
2. 浏览器触发 `keydown` 事件。
3. `playSound(e)` 被执行。
4. 通过 `e.keyCode` 得到按键编码，例如 `65`。
5. 使用 `document.querySelector` 找到 `audio[data-key="65"]`。
6. 使用 `document.querySelector` 找到 `.key[data-key="65"]`。
7. 如果没有找到对应音频，函数直接结束。
8. 如果找到了音频，把 `audio.currentTime` 设置为 `0`。
9. 调用 `audio.play()` 播放音频。
10. 给对应的 `.key` 元素添加 `playing` 类。
11. CSS transition 让按键产生缩放、高亮、阴影等动画。
12. transition 结束后触发 `transitionend`。
13. `removeTransition(e)` 执行。
14. 从按键元素上移除 `playing` 类。
15. 按键恢复原样，等待下一次输入。

## 11. 关键代码拆解

下面是这个项目中最核心的 JavaScript 逻辑：

```js
function playSound(e) {
  const audio = document.querySelector(`audio[data-key="${e.keyCode}"]`);
  const key = document.querySelector(`.key[data-key="${e.keyCode}"]`);

  if (!audio) return;

  audio.currentTime = 0;
  audio.play();
  key.classList.add('playing');
}

function removeTransition(e) {
  if (e.propertyName !== 'transform') return;
  this.classList.remove('playing');
}

const keys = document.querySelectorAll('.key');
keys.forEach(key => key.addEventListener('transitionend', removeTransition));
window.addEventListener('keydown', playSound);
```

可以把它拆成三部分理解：

| 代码 | 作用 |
| --- | --- |
| `window.addEventListener('keydown', playSound)` | 监听键盘按下 |
| `document.querySelector(...)` | 根据按键编号查找 DOM 元素 |
| `audio.play()` | 播放对应音频 |
| `key.classList.add('playing')` | 添加按键动画样式 |
| `transitionend` | 监听动画结束 |
| `classList.remove('playing')` | 移除动画样式 |

## 12. 常见问题

### 12.1 为什么按其他键没有反应

因为页面中只给部分按键配置了 `data-key` 和音频。

当按下没有配置的键时：

```js
document.querySelector(`audio[data-key="${e.keyCode}"]`);
```

会返回 `null`，然后这句代码会提前结束函数：

```js
if (!audio) return;
```

### 12.2 为什么要同时给 `div` 和 `audio` 设置相同的 `data-key`

因为按键触发时需要同时完成两件事：

- 找到要播放的音频。
- 找到要添加动画效果的页面按钮。

相同的 `data-key` 就像一个共同的编号，让这两个元素可以被同一次键盘事件关联起来。

### 12.3 为什么要监听 `transitionend`

如果只添加 `playing` 类但不移除，按钮会一直停留在高亮状态。

监听 `transitionend` 可以在动画结束后自动清理样式，让下一次按键还能重新触发动画。

### 12.4 为什么快速按键也可以连续播放

因为每次播放前都会执行：

```js
audio.currentTime = 0;
```

它会把音频播放进度重置到开头。

## 13. 可以尝试的改进

### 13.1 使用 `event.code` 替代 `keyCode`

现代写法可以使用：

```js
window.addEventListener('keydown', function (e) {
  console.log(e.code);
});
```

例如按下 `A` 时，`e.code` 通常是：

```js
KeyA
```

如果使用这种方式，HTML 可以改成：

```html
<div data-key="KeyA" class="key">...</div>
<audio data-key="KeyA" src="sounds/clap.wav"></audio>
```

查询时：

```js
const audio = document.querySelector(`audio[data-key="${e.code}"]`);
```

这种写法比 `keyCode` 更符合现代标准。

### 13.2 增加鼠标点击播放

目前项目主要响应键盘。如果想让鼠标点击也能播放，可以给 `.key` 元素添加点击事件。

思路是：点击某个 `.key` 元素时，读取它自己的 `data-key`，再找到对应音频。

```js
function playSoundByKeyCode(keyCode) {
  const audio = document.querySelector(`audio[data-key="${keyCode}"]`);
  const key = document.querySelector(`.key[data-key="${keyCode}"]`);

  if (!audio || !key) return;

  audio.currentTime = 0;
  audio.play();
  key.classList.add('playing');
}

keys.forEach(key => {
  key.addEventListener('click', function () {
    playSoundByKeyCode(this.dataset.key);
  });
});
```

这里的：

```js
this.dataset.key
```

可以读取元素上的：

```html
data-key="65"
```

## 14. 本项目知识点总结

这个 Drum Kit 小项目虽然代码不多，但串联了很多前端基础能力：

| 知识点 | 在项目中的作用 |
| --- | --- |
| HTML 结构 | 创建按键和音频元素 |
| `data-*` 自定义属性 | 建立按键、DOM 元素、音频之间的对应关系 |
| 键盘事件 | 捕获用户按键行为 |
| 事件对象 | 获取被按下的键 |
| DOM 查询 | 找到对应的按钮和音频 |
| 模板字符串 | 动态拼接选择器 |
| `<audio>` 标签 | 加载音频文件 |
| `audio.play()` | 播放音效 |
| `audio.currentTime` | 重置播放进度 |
| `classList` | 添加和移除动画类名 |
| CSS transition | 实现按键动画 |
| `transitionend` | 在动画结束后清理状态 |
| `forEach` | 给多个元素批量绑定事件 |

掌握这个项目后，你会对“事件驱动的页面交互”有一个很清晰的理解：用户操作触发事件，JavaScript 根据事件找到对应元素，修改状态或播放媒体，CSS 负责把状态变化表现成可见动画。
