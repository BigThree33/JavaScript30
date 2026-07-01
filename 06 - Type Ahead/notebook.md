# Day 06 - Type Ahead

## 这一天主要练什么

Day 06 的核心是做一个“输入即搜索”的联想列表。它不是单纯练 DOM，而是把几个常见前端能力串起来：

- 从远程 JSON 接口拉取数据。
- 把数据缓存到本地数组。
- 根据输入内容实时过滤城市或州名。
- 用正则实现大小写不敏感匹配和关键词高亮。
- 把匹配结果渲染回列表。

这类模式常见于搜索框、地址选择器、命令面板、联系人筛选、后台管理系统的快速过滤。

## HTML

START 版本已经准备好了主要结构：

```html
<form class="search-form">
  <input type="text" class="search" placeholder="City or State">
  <ul class="suggestions">
    <li>Filter for a city</li>
    <li>or a state</li>
  </ul>
</form>
```

这个结构非常关键：

- `.search` 是用户输入源。
- `.suggestions` 是结果容器。
- 初始的两个 `<li>` 只是占位提示，后续会被 JavaScript 用 `innerHTML` 替换。

FINISHED 版本的 HTML 结构基本没变，只是 favicon 从火焰换成完成标识。真正的实现集中在 JavaScript。

## CSS

Day 06 的 CSS 在 `style.css` 中，START 和 FINISHED 共用同一份样式，因此它不是本次差异的重点。但它给 JavaScript 预留了几个重要展示能力：

```css
.suggestions li {
  display: flex;
  justify-content: space-between;
  text-transform: capitalize;
}
```

每条结果内部被分成左右两侧：左侧城市和州，右侧人口数。

```css
.hl {
  background: #ffc600;
}
```

`.hl` 是关键词高亮样式。FINISHED 版本在 JS 里把匹配到的字符包进：

```html
<span class="hl">...</span>
```

这就是典型的“CSS 负责样式，JS 负责生成状态/结构”。

另外，`.suggestions li:nth-child(even)` 和 `.suggestions li:nth-child(odd)` 让列表项有交错的 3D 倾斜效果，这和搜索逻辑无关，但让搜索结果更像一个展开的纸片列表。

## JavaScript

START 版本只给了远程数据地址：

```js
const endpoint = 'https://gist.githubusercontent.com/.../cities.json';
```

FINISHED 版本围绕这个地址完成了“拉取、过滤、渲染、绑定事件”四步。

### 1. 拉取远程数据

```js
const cities = [];
fetch(endpoint)
  .then(blob => blob.json())
  .then(data => cities.push(...data));
```

这里的重点不是 `fetch` 语法本身，而是数据流：

1. `fetch(endpoint)` 发起网络请求。
2. `blob.json()` 把响应体解析成 JavaScript 数据。
3. `cities.push(...data)` 把远程数组展开后放进本地 `cities` 数组。

为什么不用：

```js
cities = data;
```

因为 `cities` 是用 `const` 声明的，不能重新赋值。但 `const` 限制的是变量绑定，不限制数组内容变化。所以可以 `push`。

`...data` 是展开语法。如果 `data` 是一个城市数组：

```js
[{...}, {...}, {...}]
```

那么：

```js
cities.push(...data)
```

等价于把每个城市对象逐个 push 进去，而不是把整个数组作为一个元素 push 进去。

### 2. 根据输入查找匹配项

```js
function findMatches(wordToMatch, cities) {
  return cities.filter(place => {
    const regex = new RegExp(wordToMatch, 'gi');
    return place.city.match(regex) || place.state.match(regex)
  });
}
```

这个函数负责从全部城市里筛出匹配项。

关键点有三个：

- `filter` 返回所有符合条件的项，不改变原数组。
- `new RegExp(wordToMatch, 'gi')` 用用户输入动态创建正则。
- `g` 表示全局匹配，`i` 表示忽略大小写。

匹配逻辑是：

```js
place.city.match(regex) || place.state.match(regex)
```

也就是城市名匹配或州名匹配都可以进入结果。

这个写法适合简单联想搜索。如果用于正式产品，需要额外处理用户输入中的正则特殊字符，例如 `.`、`*`、`[`，否则输入可能被当成正则语法解释。

### 3. 格式化人口数字

```js
function numberWithCommas(x) {
  return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',');
}
```

这个函数把人口数格式化成带逗号的形式，例如：

```text
1234567 -> 1,234,567
```

这里用的是正则替换。项目里也可以用更现代的写法：

```js
Number(x).toLocaleString()
```

但这个挑战刻意展示了正则和 `replace` 的组合。

### 4. 渲染匹配结果

```js
function displayMatches() {
  const matchArray = findMatches(this.value, cities);
  const html = matchArray.map(place => {
    const regex = new RegExp(this.value, 'gi');
    const cityName = place.city.replace(regex, `<span class="hl">${this.value}</span>`);
    const stateName = place.state.replace(regex, `<span class="hl">${this.value}</span>`);
    return `
      <li>
        <span class="name">${cityName}, ${stateName}</span>
        <span class="population">${numberWithCommas(place.population)}</span>
      </li>
    `;
  }).join('');
  suggestions.innerHTML = html;
}
```

这是实现的中心。

`this.value` 来自触发事件的输入框，也就是用户当前输入的关键词。

渲染分三步：

1. `findMatches(this.value, cities)` 得到匹配数组。
2. `map` 把每个城市对象转换成一段 `<li>` 字符串。
3. `join('')` 把字符串数组拼成完整 HTML，再交给 `suggestions.innerHTML`。

高亮通过 `replace` 完成：

```js
place.city.replace(regex, `<span class="hl">${this.value}</span>`)
```

被正则匹配到的部分会被替换成带 `.hl` 的 span。

这里有一个实现细节：替换内容使用的是 `this.value`，不是实际匹配到的原文。如果用户输入大小写和原数据不同，高亮后的大小写会跟用户输入保持一致。更稳的写法可以使用 `replace` 回调保留原文：

```js
place.city.replace(regex, match => `<span class="hl">${match}</span>`)
```

### 5. 查询 DOM 并绑定事件

```js
const searchInput = document.querySelector('.search');
const suggestions = document.querySelector('.suggestions');

searchInput.addEventListener('change', displayMatches);
searchInput.addEventListener('keyup', displayMatches);
```

`.search` 是输入框，`.suggestions` 是列表容器。

这里绑定了两个事件：

- `keyup`：每次键盘输入后实时更新。
- `change`：输入值提交变化时也更新，例如某些浏览器自动填充或失焦后变化。

这两个事件都执行同一个 `displayMatches`，所以搜索逻辑集中在一个函数里。

## 主要实现过程

1. 准备静态搜索框和结果列表。
2. 页面加载后请求城市 JSON。
3. 把远程数据缓存到 `cities`。
4. 用户输入时，用正则过滤城市或州名。
5. 用 `map` 把数据转成 HTML 字符串。
6. 用 `.hl` 包裹匹配词，实现高亮。
7. 写入 `.suggestions.innerHTML` 更新页面。

## 值得记住

- `fetch + json + 数组缓存` 是前端加载远程数据的基础流程。
- `filter` 适合做多结果筛选，`map` 适合把数据转换成视图结构。
- 动态搜索常见组合是：输入事件 -> 读取 value -> 筛选数据 -> 重绘结果。
- 正则适合做模糊匹配和高亮，但真实项目要注意转义用户输入。
- `innerHTML` 写起来快，但如果渲染用户生成内容，要考虑 XSS 风险。
