# Day 09 - Dev Tools Domination

## 这一天主要练什么

Day 09 是浏览器开发者工具训练，重点是 `console` 的各种用法。它不是为了实现页面功能，而是为了让调试更高效：

- 普通日志、插值日志、样式日志。
- 警告、错误、信息输出。
- 条件断言。
- 查看 DOM 元素和对象结构。
- 折叠分组、计数、计时、表格输出。

这一天的价值在于：调试不是只会 `console.log`，控制台本身就是一个很强的观察工具。

## HTML

页面只有一个可点击段落：

```html
<p onClick="makeGreen()">×BREAK×DOWN×</p>
```

START 和 FINISHED 的 HTML 基本一致。点击段落会执行：

```js
makeGreen()
```

它用于演示如何在控制台观察 DOM 和触发调试。

## CSS

没有单独 CSS。样式变化直接在 JavaScript 中写到段落元素上：

```js
p.style.color = '#BADA55';
p.style.fontSize = '50px';
```

这不是推荐的常规页面样式组织方式，而是为了调试演示：点击页面元素后，能明显看到 DOM 样式变化。

## JavaScript

START 版本预留了很多注释标题，FINISHED 版本在每个标题下补充了对应的 console 技巧。

### 1. 准备演示数据和点击函数

```js
const dogs = [{ name: 'Snickers', age: 2 }, { name: 'hugo', age: 8 }];
```

这是一组用于控制台分组和表格展示的对象数据。

```js
function makeGreen() {
  const p = document.querySelector('p');
  p.style.color = '#BADA55';
  p.style.fontSize = '50px';
}
```

点击段落后，查询 DOM 并直接修改内联样式。

这个函数本身不是重点，重点是后面用 `console.log`、`console.dir` 等方法观察这个 DOM 元素。

### 2. 普通日志

```js
console.log('hello');
```

最常见的输出方式。适合临时查看变量、函数是否执行、流程是否走到某一步。

### 3. 字符串插值

```js
console.log('Hello I am a %s string!', '...');
```

`%s` 会被后面的字符串参数替换。控制台支持类似格式化占位：

- `%s`：字符串。
- `%d` 或 `%i`：整数。
- `%f`：浮点数。
- `%o` 或 `%O`：对象。

现代项目里也常用模板字符串：

```js
console.log(`Hello I am a ${value} string!`);
```

但了解控制台占位符有助于读旧代码和写调试输出。

### 4. 样式化日志

```js
// console.log('%c I am some great text', 'font-size:50px; background:red; text-shadow: 10px 10px 0 blue')
```

`%c` 可以给控制台输出加 CSS 样式。

它不影响页面，只影响控制台显示。适合给重要日志加醒目标记，例如模块启动、关键状态变化、调试分隔线。

这行在 FINISHED 中被注释掉了，不会执行。

### 5. 警告、错误、信息

```js
console.warn('OH NOOO');
console.error('Shit!');
console.info('Crocodiles eat 3-4 people per year');
```

这些方法会用不同视觉样式输出信息：

- `console.warn`：警告，通常黄色。
- `console.error`：错误，通常红色，并可能显示调用栈。
- `console.info`：信息提示。

实际项目里应根据问题严重程度选择输出方式，不要所有信息都用普通 `log`。

### 6. 条件断言

```js
const p = document.querySelector('p');

console.assert(p.classList.contains('ouch'), 'That is wrong!');
```

`console.assert(condition, message)` 的特点是：

- 条件为 `true` 时不输出。
- 条件为 `false` 时输出错误信息。

这里段落没有 `ouch` 类，所以会输出断言失败信息。

适合用于临时验证假设，例如：

```js
console.assert(items.length > 0, 'items should not be empty');
```

### 7. 清空控制台

```js
console.clear();
```

清空当前控制台输出。FINISHED 里执行了两次，是为了让不同演示之间更干净。

调试时可以用，但正式代码中通常不保留。

### 8. 查看 DOM 元素和对象结构

```js
console.log(p);
console.dir(p);
```

二者差异：

- `console.log(p)` 更偏向显示 DOM 节点本身，通常像 Elements 面板里的 HTML。
- `console.dir(p)` 更偏向显示对象属性结构，可以展开查看这个元素对象上的属性和方法。

当你想看元素长什么样，用 `log`。当你想看元素有哪些属性，用 `dir`。

### 9. 分组输出

```js
dogs.forEach(dog => {
  console.groupCollapsed(`${dog.name}`);
  console.log(`This is ${dog.name}`);
  console.log(`${dog.name} is ${dog.age} years old`);
  console.log(`${dog.name} is ${dog.age * 7} dog years old`);
  console.groupEnd(`${dog.name}`);
});
```

`console.groupCollapsed` 创建默认折叠的日志分组，`console.groupEnd` 结束分组。

适合在循环中输出复杂对象，避免控制台被大量日志刷屏。

常见用途：

- 按请求分组。
- 按组件分组。
- 按数据项分组。
- 按一次交互流程分组。

### 10. 计数

```js
console.count('Wes');
console.count('Steve');
```

`console.count(label)` 会统计同一个 label 被调用了多少次。

适合检查某段代码执行频率，例如：

- 函数是否被重复触发。
- 组件是否重复渲染。
- 事件监听是否绑定了多次。

### 11. 计时

```js
console.time('fetching data');
fetch('https://api.github.com/users/wesbos')
  .then(data => data.json())
  .then(data => {
    console.timeEnd('fetching data');
    console.log(data);
  });
```

`console.time(label)` 开始计时，`console.timeEnd(label)` 结束计时并输出耗时。

这里用于测量一次 `fetch` 请求加 JSON 解析的时间。

注意：`time` 和 `timeEnd` 必须使用同一个 label。

### 12. 表格输出

```js
console.table(dogs);
```

`console.table` 会把对象数组以表格形式展示。对于数组对象来说，比 `console.log` 更容易比较字段。

适合查看：

- API 返回的列表。
- 表格数据。
- 对象数组。
- 批量处理结果。

## 主要实现过程

1. 保留一个可点击 DOM，用于触发样式变化。
2. 使用不同 console API 展示日志级别和格式化方式。
3. 用 `console.assert` 验证条件。
4. 用 `log` 和 `dir` 对比 DOM 展示差异。
5. 用 `groupCollapsed` 管理循环日志。
6. 用 `count` 统计调用次数。
7. 用 `time/timeEnd` 测量异步请求耗时。
8. 用 `table` 更清楚地查看对象数组。

## 值得记住

- 控制台不仅是输出字符串，还能分组、计数、计时、断言和表格化数据。
- `console.dir` 对查看 DOM 对象属性很有用。
- `console.table` 是查看数组对象的高效方式。
- `console.time` 很适合快速定位慢操作。
- 调试日志应该有目的，用完及时清理，避免干扰真实错误。
