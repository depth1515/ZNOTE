# setTimeout

写这篇文章是因为之前第一次面试吃了亏，当时面试官问了关于setTimeout的一些问题，没回答上来，所以记录一下。

## 问题一 参数

第一个参数可以是`function`也可以是字符串(在后面应用)，第二个参数为`delay`，第二个参数之后的参数会在定时器到期时作为参数传给`function`

### 栗子：

```js
setTimeout(function () {
  console.log(arguments)
}, 1000, 100, 200, 300)
```

一秒之后执行输出

> [Arguments] { '0': 100, '1': 200, '2': 300 }

还可以传入setTimeout

```js
setTimeout(function () {
  console.log(1)
}, 1000, setTimeout(() => {
  console.log(2)
}, 2000), setTimeout(() => {
  console.log(2)
}, 2000))
```

第一秒的时候输出1，第二秒的时候同时输出两个2

## 问题二 返回值

返回值是一个正整数，表示定时器的编号（number类型）。这个值可以传递给[`clearTimeout()`](https://developer.mozilla.org/zh-CN/docs/Web/API/clearTimeout)来取消该定时器。这个编号值跟`setInterval`的编号池同用。

## 问题三 delay为0

```setTimeout(() => {}, 0)```

```js
 console.log('a');
setTimeout(function(){
    console.log('b');
},0);
console.log('c');
console.log('d');
```

控制台输出：

a

c

d

b

理论上他延迟时间为0，应该是马上执行，但是不是这样的。因为setTImeout是宏任务，而console.log是微任务，js运行机制会把当前脚本的微任务和“任务队列中”已有的任务、全部处理完之后，才会执行setTimeout指定的任务，即setTimeout为异步执行，异步任务都会添加到任务队列中。

> 拓展：setTimeout(fn,0) 在浏览器中运行并不是真的0 MDN上说了>=4ms 也就是说至少大于等于4ms。

## 应用

### 计时器

```js
function ddd() {
    console.log(new Date().getSeconds())
    setTimeout('ddd()', 1000)
}
ddd()
```

### 面试题

```js
for (var i = 0; i < 10; i++) {
  setTimeout(() => {
    console.log(i)
  })
}

// 输出0 1 2 3 4 5 6 7 8 9 

for (var i = 0; i < 10; i++) {
    console.log(i)
}

// 输出 10个10
```

