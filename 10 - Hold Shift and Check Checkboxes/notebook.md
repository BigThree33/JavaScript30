# Day 10 - Hold Shift and Check Checkboxes

## 这一天主要练什么

Day 10 实现的是邮箱里常见的 Shift 批量勾选：

1. 先勾选一个 checkbox。
2. 按住 Shift。
3. 再勾选另一个 checkbox。
4. 两者之间的 checkbox 自动全部勾选。

这个挑战主要练的是事件对象、状态记录和区间选择算法：

- 查询一组 checkbox。
- 记录上一次点击的 checkbox。
- 判断事件发生时 Shift 是否按下。
- 遍历 checkbox 列表，找出两个端点之间的元素。
- 用布尔开关标记“当前是否处于区间内”。

这种思路可以迁移到邮件列表、文件管理器、多选表格、任务列表批量操作。

## HTML

START 版本已经准备好了完整列表：

```html
<div class="inbox">
  <div class="item">
    <input type="checkbox">
    <p>This is an inbox layout.</p>
  </div>
  ...
</div>
```

结构特点：

- `.inbox` 是整个列表容器。
- 每个 `.item` 是一行。
- 每行都有一个 checkbox 和一个文本段落。

FINISHED 版本 HTML 基本没变，只调整了少量文案和注释措辞。主要逻辑都在 JavaScript。

## CSS

CSS 在 START 版本里已经基本写好，FINISHED 版本没有本质新增。这里需要关注的是两处和交互有关的样式。

### 1. 行布局

```css
.item {
  display: flex;
  align-items: center;
  border-bottom: 1px solid #F1F1F1;
}
```

每一行用 Flexbox 排列 checkbox 和文字。`align-items: center` 让它们垂直居中。

```css
p {
  flex: 1;
}
```

文字区域占据剩余空间，让整行看起来像邮箱列表。

### 2. 勾选后的视觉状态

```css
input:checked + p {
  background: #F9F9F9;
  text-decoration: line-through;
}
```

这是本挑战中最重要的 CSS 选择器。

`input:checked + p` 表示：

- 选中处于 checked 状态的 input。
- 再选中它后面紧挨着的那个 `p`。

所以 JavaScript 只需要改变：

```js
checkbox.checked = true;
```

CSS 就会自动让对应文本变灰并加删除线。

这也是一个很好的分工：JS 改状态，CSS 根据状态展示样式。

## JavaScript

START 版本的脚本为空。FINISHED 版本补全了多选逻辑。

### 1. 查询全部 checkbox

```js
const checkboxes = document.querySelectorAll('.inbox input[type="checkbox"]');
```

这里没有直接写：

```js
document.querySelectorAll('input')
```

而是限定在 `.inbox` 里，并且限定 `type="checkbox"`。这样选择范围更准确，不会误选页面上其他输入框。

返回值是一个 `NodeList`，可以用 `forEach` 遍历。

### 2. 记录上一次点击项

```js
let lastChecked;
```

Shift 批量勾选需要两个端点：

- 第一个端点：上一次点击的 checkbox。
- 第二个端点：这一次 Shift 点击的 checkbox。

`lastChecked` 就是用来保存第一个端点。

它不能写成函数内部变量，因为每次点击都会重新调用函数。必须放在外层作用域，才能跨事件保留上一次点击结果。

### 3. 点击处理函数

```js
function handleCheck(e) {
  let inBetween = false;
  if (e.shiftKey && this.checked) {
    checkboxes.forEach(checkbox => {
      if (checkbox === this || checkbox === lastChecked) {
        inBetween = !inBetween;
      }

      if (inBetween) {
        checkbox.checked = true;
      }
    });
  }

  lastChecked = this;
}
```

这是本挑战的核心。

### 4. 判断 Shift 是否按下

```js
if (e.shiftKey && this.checked) {
```

`e.shiftKey` 来自鼠标事件对象，表示点击发生时 Shift 键是否处于按下状态。

`this.checked` 表示当前 checkbox 点击后是否是选中状态。

两个条件一起使用，意味着只有“按住 Shift 并且是在勾选”时，才执行批量选中逻辑。

如果用户按住 Shift 取消勾选，当前代码不会批量取消。这是这个挑战的设计取舍。

### 5. 用 `inBetween` 标记区间

```js
let inBetween = false;
```

`inBetween` 表示遍历过程中是否已经进入两个端点之间。

遍历所有 checkbox 时，遇到第一个端点就切换为 `true`，开始勾选中间项；遇到第二个端点再切换为 `false`。

端点判断是：

```js
if (checkbox === this || checkbox === lastChecked) {
  inBetween = !inBetween;
}
```

这行非常巧妙，因为它不关心用户是从上往下选，还是从下往上选。

例如从第 2 项 Shift 选到第 6 项：

```text
第 2 项：遇到端点，inBetween -> true
第 3-5 项：inBetween 为 true，全部勾选
第 6 项：遇到端点，inBetween -> false
```

反过来从第 6 项 Shift 选到第 2 项也一样，因为遍历顺序固定从上到下，两个端点总会先后出现。

### 6. 勾选区间内的 checkbox

```js
if (inBetween) {
  checkbox.checked = true;
}
```

只要遍历时处于两个端点之间，就把当前 checkbox 勾上。

当前点击的 checkbox 本身已经由浏览器默认行为勾选了；代码主要负责补齐中间项。

注意：由于 `inBetween` 在遇到第二个端点时会立刻切回 `false`，第二个端点不靠这行勾选，而是靠用户点击本身勾选。

### 7. 更新最后点击项

```js
lastChecked = this;
```

无论是否按 Shift，最后都要更新 `lastChecked`。这样下一次 Shift 点击才知道从哪里开始算区间。

如果不更新它，批量选择的起点就会丢失。

### 8. 绑定事件

```js
checkboxes.forEach(checkbox => checkbox.addEventListener('click', handleCheck));
```

给每个 checkbox 绑定点击事件。

这里使用 `click` 而不是 `change`，因为需要读取点击事件上的 `shiftKey`。`click` 事件天然包含键盘修饰键状态。

## 主要实现过程

1. 查询 `.inbox` 内所有 checkbox。
2. 声明 `lastChecked` 保存上一次点击项。
3. 每次点击时进入 `handleCheck`。
4. 如果本次点击按住了 Shift，且当前项被勾选，就开始批量逻辑。
5. 遍历所有 checkbox。
6. 遇到当前项或上次项时，翻转 `inBetween`。
7. `inBetween` 为 true 的项全部设置 `checked = true`。
8. 最后把当前项保存为新的 `lastChecked`。

## 值得记住

- 批量选择的关键是保存“上一次点击”。
- `event.shiftKey` 可以判断点击时是否按住 Shift。
- `this` 在普通事件处理函数中指向当前触发事件的元素。
- 用布尔开关寻找两个端点之间的范围，是一种很轻量的区间算法。
- CSS 的 `input:checked + p` 让 JS 不必手动改文本样式。

## 可以改进的方向

- 支持 Shift 批量取消勾选。
- 用数组下标计算区间，而不是用 `inBetween` 开关。
- 增加键盘可访问性提示。
- 把事件绑定到 `.inbox` 上做事件委托，适合动态新增列表项。
