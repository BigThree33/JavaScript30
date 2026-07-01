# Day 07 - Array Cardio Day 2

## 这一天主要练什么

Day 07 是第二组数组方法训练，目标不是做页面，而是在控制台里熟悉数组查询类 API：

- `some`：是否至少有一个元素满足条件。
- `every`：是否所有元素都满足条件。
- `find`：找到第一个满足条件的元素。
- `findIndex`：找到第一个满足条件的元素下标。
- `slice + spread`：用不可变方式删除数组元素。

这一天的重点是“如何从数组中判断、查找、定位和生成新数组”。它在真实项目中非常常见，例如权限判断、列表查找、删除评论、筛选状态、表单校验。

## HTML

HTML 几乎没有交互界面：

```html
<p><em>Psst: have a look at the JavaScript Console</em></p>
```

这句话提示你打开浏览器控制台，因为所有结果都通过 `console.log` 查看。

START 和 FINISHED 的页面结构没有实质变化，favicon 从火焰换成完成标识。这个挑战的主角是 JavaScript 数组方法。

## CSS

这个挑战没有 CSS。页面不承担展示任务，控制台才是输出区域。

这也说明 JavaScript30 里并不是每一天都在做 UI，有些天是专门训练语言能力和调试习惯。

## JavaScript

START 版本已经提供两组数据：

```js
const people = [
  { name: 'Wes', year: 1988 },
  { name: 'Kait', year: 1986 },
  { name: 'Irv', year: 1970 },
  { name: 'Lux', year: 2015 }
];
```

`people` 用于年龄判断。

```js
const comments = [
  { text: 'Love this!', id: 523423 },
  { text: 'Super good', id: 823423 },
  { text: 'You are the best', id: 2039842 },
  { text: 'Ramen is my fav food ever', id: 123523 },
  { text: 'Nice Nice Nice!', id: 542328 }
];
```

`comments` 用于按 id 查找和删除。

FINISHED 版本补全的是数组操作。

### 1. `some`：至少一个人年满 19 岁

```js
const isAdult = people.some(person => ((new Date()).getFullYear()) - person.year >= 19);

console.log({isAdult});
```

`some` 的语义是：只要数组里有一个元素让回调返回 `true`，整体结果就是 `true`。

这里判断的是：

```js
当前年份 - 出生年份 >= 19
```

适合场景：

- 是否存在一个未读消息。
- 是否有商品缺货。
- 是否有一个表单项校验失败。
- 是否至少选择了一个选项。

它比手写循环更表达意图：我不是要转换数组，也不是要找出全部元素，我只是要知道“有没有”。

### 2. `every`：是否所有人都年满 19 岁

```js
const allAdults = people.every(person => ((new Date()).getFullYear()) - person.year >= 19);
console.log({allAdults});
```

`every` 的语义是：数组里每个元素都让回调返回 `true`，整体才是 `true`。

适合场景：

- 所有表单字段都通过校验。
- 所有任务都已完成。
- 所有商品都有库存。
- 所有用户都拥有某个权限。

`some` 和 `every` 是一对非常实用的判断方法：

```js
items.some(condition)   // 有没有
items.every(condition)  // 是不是全都
```

### 3. `find`：按 id 找到某条评论

```js
const comment = comments.find(comment => comment.id === 823423);

console.log(comment);
```

`find` 返回第一个满足条件的元素本身。

它和 `filter` 的区别是：

- `filter` 返回数组，即使只有一个结果也是数组。
- `find` 返回单个元素，找不到时返回 `undefined`。

当你知道目标应该只有一个时，`find` 更合适。

典型场景：

```js
const user = users.find(user => user.id === targetId);
const product = products.find(product => product.slug === slug);
```

### 4. `findIndex`：找到目标元素的位置

```js
const index = comments.findIndex(comment => comment.id === 823423);
console.log(index);
```

`findIndex` 返回第一个满足条件的元素下标。找不到时返回 `-1`。

为什么需要下标？

因为删除、替换、插入某个元素时，常常需要知道它在数组中的位置。

例如：

```js
comments.splice(index, 1);
```

这会直接修改原数组。

### 5. 用不可变方式删除元素

FINISHED 版本没有启用 `splice`，而是写了：

```js
const newComments = [
  ...comments.slice(0, index),
  ...comments.slice(index + 1)
];
```

这是一个非常重要的思路：不修改原数组，而是创建一个新数组。

拆开看：

```js
comments.slice(0, index)
```

取目标元素之前的所有评论。

```js
comments.slice(index + 1)
```

取目标元素之后的所有评论。

再用展开语法合并：

```js
[
  ...before,
  ...after
]
```

这样就跳过了目标元素。

这个模式在 React、Redux、Vue 状态管理、不可变数据更新里非常常见。

## 主要实现过程

1. 用 `some` 判断是否存在至少一个成年人。
2. 用 `every` 判断是否所有人都是成年人。
3. 用 `find` 按 id 查出具体评论对象。
4. 用 `findIndex` 找到这条评论的位置。
5. 用 `slice + spread` 创建删除目标后的新数组。

## 值得记住

- `some` 和 `every` 处理布尔判断。
- `find` 和 `findIndex` 处理单项查找。
- `filter` 更适合保留所有匹配项，`find` 更适合只要第一个匹配项。
- `splice` 会改变原数组，`slice` 不会改变原数组。
- 前端状态更新时，优先考虑创建新数组，而不是直接修改旧数组。
