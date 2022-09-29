---
title: js手写题
permalink: /pages/c68880/
categories:
  - js手写题
summary: 手写题
tags:
  - 手写专题
abbrlink: 53796
date: 2021-03-12 18:56:10
---


# JS手写题

## 一、JavaScript基础

### 1.手写Object.create

> 思路 ： 传入一个对象 作为返回对象的原型
> 目的 ： 为一个对象指定原型

```js
function create(obj) {
    function F() { };
    F.prototype = obj;
    return new F();
}
let obj = {a: 1 , b: 2};
let obj1 = create(obj);
console.log(obj1.__proto__);//{a: 1 , b: 2}
//思路 ： 传入一个对象 作为返回对象的原型
//目的 ： 为一个对象指定原型
```

### 2.手写instanceof方法

> 目的：判断构造函数的原型是否出现在对象的原型链上
>
> 思路：①获取构造函数原型
>
> ​            ②获取对象的原型
>
> ​			③循环判断对象的原型是否等于构造函数的原型，知道对象原型为null，原型链最终为null

具体实现：

```js
function myinstanceof(obj, F) {
    let objProto = Object.getPrototypeOf(obj);//获取对象obj的原型
    let fProto = F.prototype;//获取构造函数的原型
    while (true) {
        if (!objProto) return false;//如果遍历到原型对象为null 则表示找不到
        if (objProto === fProto) return true;//当前对象原型===构造函数原型 则表示找到
        objProto = Object.getPrototypeOf(objProto);//获取对象原型的原型，循环向上寻找
    }
}
//测试
function F() {
}
let obj = new F();
console.log(myinstanceof(obj, F));//true
```

### 3.手写new操作符

> 思路：①创建一个空对象
>
> ​			②将实例对象的原型指向构造函数的prototype对象
>
> ​			③构造函数的this指向实例对象，并调用构造函数
>
> ​			④判断函数的返回值，如果是值类型，返回创建的对象。如果是引用类型，就返回这个引用类型的对象。

```js
function _new(Fn, ...args) {
    let obj = {};//1、创建一个空对象
    obj.__proto__ = Fn.prototype;//2、实例对象的原型指向构造函数Fn的prototype对象
    let res = Fn.call(obj, ...args);//3、构造函数Fn绑定this指向obj实例，并调用构造函数 返回对象res
    return typeof res === 'object' ? res : obj;//判断res的类型 如果是对象则返回res 不是则返回新创建的obj对象实例
}
function Person(name, age) {
    this.name = name;
    this.age = age;
    // return {
    //     name: name,
    //     age: age
    // }
}
//测试
let personImpl = _new(Person, 'guang', '28');
console.log(personImpl);
//return没注释     打印{ name: 'guang', age: '28' }
//return语句注释   打印Person { name: 'guang', age: '28' }
```

### 4.手写Promise.all()

> 思路：Promise.all( )方法接收一个promise可迭代对象，并只返回一个新的Promise实例，所有输入的promise的resolve回调的结果是一个数组，返回值数组与参数顺序一致。只要有一个promise的回调结果是reject，直接返回第一个reject。简单来说就是多个promise实例包装成一个新的promise实例
>
> 修改======接收一个promise可迭代对象，并返回一个新的Promise实例(一个数组)。当所有promise都为resolve成功时 才返回成功，但是只要有一个失败那么就返回第一个失败的

**应用场景**：`Promise.all( ).then( )`适用于处理多个异步任务，且所有的异步任务都得到结果时的情况。

如：用户点击按钮，会弹出一个弹出对话框，对话框中有两部分数据呈现，这两部分数据分别是不同的后端接口获取的数据。

弹框弹出后的初始情况下，就让这个弹出框处于数据加载的状态，当这两部分数据都从接口获取到的时候，才让这个数据加载状态消失。让用户看到这两部分的数据。

那么此时，我们就需求这两个异步接口请求任务都完成的时候做处理，所以此时，使用Promise.all方法，就可以轻松的实现，我们来看一下代码写法

```js
function myPromiseAll(promises) {// 接收一个Promise的可迭代对象Array，Map，Set
    return new Promise((resolve, reject) => {// 返回一个Promise实例
        if (!Array.isArray(promises)) {
            return new TypeError('arguments must be an array');// 判断是否为一个数组 不是则抛出类型异常
        }
        let count = 0;// 设置计数器 计算被解决的期约Promise个数
        let result = [];// 定义一个数组保存所有被解决的期约Promise
        let n = promises.length;// 获取可迭代对象的长度（算是一种小优化吧）
        for (let i = 0; i < n; i++) {// 遍历所有参数，并发执行每一个promise
            Promise.resolve(promises[i]).then(res => {
                result[i] = res;// 将每一个promise的resolve回调结果丢进result数组
                count++;// 每循环一次计数器加一
                if (count === promises.length) {// 计数器等于数组长度 则返回结果数组result
                    resolve(result);
                }
            }).catch(err => reject(err));// 捕获reject,catch不仅可以捕获reject时的状态 同时能够捕获then回调中的错误
        }
    })
}
// 测试
let p1 = Promise.resolve(1);
let p2 = Promise.resolve(2);
let p3 = Promise.resolve(3);
myPromiseAll([p3, p1, p2]).then(res => {
    console.log(res) // [3, 1, 2]
})
Promise.all([p3, p1, p2]).then(res => {
    console.log(res);// [3, 1, 2]
})
```

> 对顺序的一个解惑，then里的 函数内部引用了外部的产量i 因此外部的变量i 不会被释放。当异步onfullfilled函数执行时，内部引用的总是之前的那个i ，因此保证了顺序的准确性。

### 5.手写Promise.race()

> Promise.race() 接收一个可迭代对象，返回一个包装期约，是一组集合中最先解决或拒绝的期约的镜像。

```js
Promise.race = function (args) {//传入可迭代对象args
  return new Promise((resolve, reject) => {//返回一个promise实例（包装期约）
    for (let i = 0, len = args.length; i < len; i++) {
      args[i].then(resolve, reject)// Promise.then()函数最多接收两个参数，分别进入兑现和拒绝状态时执行，执行某一个就直接返回当前状态的期约
    }
  })
}
```

**4，5 总结**

- Promise.all接收的是数组，得到的结果也是数组，并且一一对应，也可以理解为Promise.all照顾跑的最慢的，最慢的跑完才结束。
- Promise.race接收的也是数组，不过，得到的却是数组中跑的最快的那个，当最快的一跑完就立马结束。

🎨用一句话总结防抖和节流的区别：**防抖是将规定时间内多次执行变为最后一次执行，节流是规定时间内将多次执行变为每隔一段时间执行**

### 6.手写函数防抖

> 函数防抖是指在事件被触发 n 秒后再执行回调，如果在这 n 秒内事件又被触发，则**重新计时**，也就是说事件在规定事件内重新触发只执行最后一次。主要是利用闭包保存一个timer计时器，这可以使用在一些点击请求的事件上，避免因为用户的多次点击向后端发送多次请求。

**场景：input事件模糊查询**

> input输入框的input事件会在输入框内容发生改变的时候执行，那么就存在一个问题：每次输入，都会触发input事件，执行函数，或者接口请求，而这并不是我们想要的：比如，你想要模糊查询 "liu" 相关的所有数据，而当你在input框中输入 "l" 的时候就已经触发了input事件，去请求了接口，而这并不是我们想要的。所以，我们的防抖函数就登场了；

```js
// 事件在规定时间内重复触发只执行一次
function debounce(fn, time, immediate) {//fn为回调函数(需要防抖的函数)，time是自定义间隔时间 debounce是对需要防抖的函数的封装
    let timer = null;
    return function (...args) {
        // 添加一个立即执行功能
        if (immediate && !timer) {
            fn.apply(this, args);
        }
        timer && clearTimeout(timer);//timer为true则表示有新的触发 需要把之前的setTimeout计时器清除再重新计时
        // let args = arguments;//不同函数可能会有不同的参数传入 这里使用args接收arguments是为了避免下面直接使用了setTimeout函数的arguments
        timer = setTimeout(() => {
            fn.apply(this, args);//需要apply绑定this并调用传进来的原函数 ？？疑问：为什么要绑定this----this指向容器
        }, time)
    }
}
let addBtn = document.getElementById('add')
function addOne() {
    console.log('增加一个')
}
addBtn.addEventListener('click', debounce(addOne))//添加click点击事件，对addOne实现防抖
// 实现思路：
// 1利用闭包保存一个timer，然后返回一个函数（该函数就是后续频繁触发操作时调用的函数）
// 2根据标识位判断是否第一次需要执行（有些时候需要首次调用函数立即执行的）
// 3当有新的事件触发，若存在定时器，则清空定时器
// 4设置一个新的定时器重新计时
```

### 7.手写函数节流

> 函数节流是指单位时间内，只能有一次触发事件的回调函数执行，节流可以使用在 scroll 函数的事件监听上，通过事件节流来降低事件调用的频率。如；实现一个鼠标滚动打印事件，**想让它在3s内只执行一次**

**场景：点击按钮重复发送请求：(节流)**

项目中点击创建、编辑、删除等按钮都会发送http请求，网络卡顿的情况下点击按钮之后不能快速的响应，一般情况下用户会重复点击按钮，所以会造成**重复发送请求**问题，一定量造成卡顿延迟问题，这个时候便可以采用**节流**

```js
// 时间戳版本
// 主要是利用闭包保存上一次的时间preTime
function throttle(fn, time) {
    let preTime = 0;//第一次是记录初始时间 接下来记录上一次时间
    return function (...args) {
        let curTime = Date.now()//获取当前时间
        if (curTime - preTime >= time) {//如果两次时间间隔超出指定时间间隔time 则可以执行函数fn
            fn.apply(this, args);
            preTime = curTime;//更新上一次的时间为当前时间
        }
    }
  }
function scrollTest(){
    console.log('现在我触发了')
  }
document.addEventListener('scroll',throttle(scrollTest,time)) //对鼠标滚动事件进行函数节流

// 定时器版本
// 利用闭包保存定时器timer，事件触发后经过time时间则执行函数
function throttle(fn, time) { 
  let timer = null;// 创建定时器
  return function (... args) {
    if (!timer) {// 定时器不存在
      timer = setTimeout(() => {
        fn.apply(this, args);
        timer = null;
      }, time)
    }
  }
}
```

### 8.函数柯里化实现

> 函数柯里化指的是将接收**多个参数**的**一个函数**转换成一系列接收**一个参数**的**函数**。主要使用闭包和递归实现
>
> 以下两种方式实现柯里化

#### 8.1普通递归实现

```js
function toCurry(fn, args) {// fn为需要柯里化的函数
    var length = fn.length;// 原函数参数个数
    var args = args || []; 
    return function(){
        // 
        newArgs = args.concat(Array.prototype.slice.call(arguments));
        if (newArgs.length < length) {// 收集的参数比原函数所需参数少 就继续柯里化
            return toCurry.call(this,fn,newArgs);
        }else{
            return fn.apply(this,newArgs);
        }
    }
}
// 测试
function multiFn(a, b, c) {
    return a * b * c;
}
var multi = toCurry(multiFn);
console.log(multi(2)(3)(4));
console.log(multi(2, 3, 4));
console.log(multi(2)(3, 4));
console.log(multi(2, 3)(4));
```

#### 8.2 ES6箭头函数递归实现

```js
function toCurry(fn, ...args) {
    if (args.length >= fn.length) {
        return fn(...args);
    } else {
        return (...args2) => toCurry(fn, ...args, ...args2);
    }
}
// 测试
function multiFn(a, b, c) {
    return a * b * c;
}
var multi = toCurry(multiFn);
console.log(multi(2)(3)(4));
console.log(multi(2, 3, 4));
console.log(multi(2)(3, 4));
console.log(multi(2, 3)(4));
```



### 深浅拷贝的前置知识

前置知识：

> **基本数据类型**：存在栈中，**变量名+值**；复制时系统会自动为新的变量在栈内存中分配一个新值，旧值不会被改变

```js
let number1 = 1;
let number2 = 2;
console.log(number1, number2); // 1 2
number2 = number1;
number2 = 3
console.log(number1, number2); // 1 3
```

> 引用数据类型：存在堆中，栈中存储**变量名+地址**，地址指向堆中的数据；复制时系统同样为新的变量分配一个值，只是这个值是指向队中对象的一个地址，因此对其进行修改会导致同一个堆中对象的修改

```js
let obj1 = {
    name: 'guang',
};
let obj2 = obj1;
console.log(obj1); // { name: 'guang' }
console.log(obj2); // { name: 'guang' }
obj2.name = 'me';
console.log(obj1); // { name: 'me' }
console.log(obj2); // { name: 'me' }
// 复制引用数据类型，并进行修改，会改变原本的值
```

### 9.手写浅拷贝

**浅拷贝**：指的是将一个**对象的属性值**复制到另一个对象，如果有的属性的值为引用类型的话，那么会将这个引用的地址复制给对象，因此两个对象会有同一个引用类型的引用。浅拷贝可以使用  Object.assign 和展开运算符，还有数组API(concat、slice方法等)来实现。

> `Object.assign `  用于拷贝对象，它对于第一层来说，是完全拷贝；对于第二层及以上来说，是简单复制。
>
> `Array.prototype.concat(target)`：`concat()` 是数组的一个内置方法，用于合并两个或者多个数组。这个方法不会改变现有数组，而是返回一个新数组。`const b = [].concat(a)`
>
> `Array.prototype.slice(start, end)`：`slice()` 也是数组的一个内置方法，该方法会返回一个新的对象。`slice()` 不会改变原数组。`const b = a.slice()`
>
> **展开运算符**：`[...arr]` 可以得到一个浅拷贝新数组。

```js
// 浅拷贝的实现
function shallowCopy(obj) {
    if (!obj || typeof obj !== "object") return;// 空对象和非对象类型 直接返回
    let newObj = Array.isArray(obj) ? [] : {};// 根据 obj 的类型判断是新建一个数组还是对象
    //for...in语句以任意顺序迭代对象的可枚举属性(包括它的原型链上的可枚举属性)。----for...of 语句遍历可迭代对象定义要迭代的数据。
    for (let key in obj) {// 遍历 obj，并且判断是 obj 自身的属性才拷贝
        if (obj.hasOwnProperty(key)) {
            newObj[key] = obj[key];
        }
    }
    return newObj;
}//修改赋值后的对象B的非对象属性，不会影响原对象A的非对象属性；修改赋值后的对象B的对象属性，却会影响原对象A的对象属性。
```

**直接赋值和浅拷贝的区别**：非对象属性意思是属性对应的值不是一个对象

- 直接赋值：无论修改赋值后的对象的**非对象属性**还是**对象属性**，都会影响原对象A的对应的属性

- 浅拷贝：修改**拷贝后**的对象的**非对象属性**，不会影响原对象的**非对象属性**；修改**拷贝后**的对象的**对象属性**，却会影响原对象的**对象属性**。因为对象属性拷贝的是引用地址

### 10.手写深拷贝

深拷贝会另外在堆空间中拷贝一份一模一样的对象，新对象跟原对象不共享内存，修改拷贝后的对象不会改到原对象。二者指向不同的对象，地址空间完全不一样

**方法一：**利用 JSON 对象中的 parse 和 stringify；

```js
var obj1 = {
    name: "cat",
    show:function(){
        console.log(this.name);
    }
};
var obj2 = JSON.parse(JSON.stringify(obj1));
obj2.name = "pig";
console.log(obj1.name); //cat
console.log(obj2.name); //pig
obj1.show(); //cat
obj2.show(); //函数被丢失，报错
```

> JSON.stringify()会导致一系列的问题，因为要严格遵守JSON序列化规则：原对象中如果含有**Date对象**，JSON.stringify()会将其变为字符串，之后并不会将其还原为日期对象。或是含有RegExp对象，JSON.stringify()会将其变为空对象，属性中含有NaN、Infinity和-Infinity，则序列化的结果会变成null，如果属性中**有函数**,undefined,symbol则经过JSON.stringify()序列化后的JSON字符串中**这个键值对会消失**，因为不支持。

**方法二：**函数库lodash的_.cloneDeep方法

> lodash是前端开发常用的工具库，是对各种方法、函数的封装，项目中直接安装依赖包，引入并使用里面的函数，常用函数中就有`_.cloneDeep (value)`深拷贝函数

```js
var _ = require('lodash');
var obj1 = {
    a: 1,
    b: { f: { g: 1 } },
    c: [1, 2, 3]
};
var obj2 = _.cloneDeep(obj1);
console.log(obj1.b.f === obj2.b.f);// false
```

**💎方法三：**手写深拷贝

> 进一步对浅拷贝的认识：
>
> 1. 明白浅拷贝的局限性: 只能拷贝一层对象。 如果存在对象的嵌套, 那么浅拷贝将无能为力
> 2. 对于基础数据类型做一个最基本的拷贝
> 3. 对引用类型开辟一个新的存储, 并拷贝一层对象属性，第二层只拷贝引用地址

- 解决循环引用问题，我们可以额外用一个存储空间Map，来存储当前对象和拷贝对象的对应关系，当需要拷贝当前对象时，先去存储空间中找，有没有拷贝过这个对象，如果有的话直接返回，如果没有的话继续拷贝，这样就可以解决的循环引用的问题。
- 这个存储空间，需要可以存储 key-value 形式的数据，且 key 可以是一个引用类型，我们可以选择 Map 这种数据结构：检查 map 中有无克隆过的对象,如果有则直接返回，如果没有则将当前对象作为 key，克隆对象作为 value 进行存储，继续克隆
- 可以使用 WeakMap 进一步优化。WeakMap 对象是一组键/值对的集合，其中的键是弱引用的。其键必须是对象，而值可以是任意的。
- 我们默认创建一个对象：const obj = {}，就默认创建了一个强引用的对象，我们只有手动将 obj = null，它才会被垃圾回收机制进行回收，如果是弱引用对象，垃圾回收机制会自动帮我们回收。

```js
//方法三：手写深拷贝
// 与Map不同，WeakMap的键key只能是object类型  key不能被遍历(弱引用)，不使用了可以被垃圾回收机制处理，释放内存，而Map不用了js还是会保持每个key和value的引用(强引用)。
// 使用WeakMap来解决循环引用问题
function deepClone(target, map = new WeakMap()) {
    if(target instanceof RegExp) return new RegExp(target)// 如果是正则对象直接new并返回，传入对象实例
    if(target instanceof Date) return new Date(target)// 如果是日期对象直接new并返回
    if(typeof target === 'object' && target !== null && target !== undefined){
        if(map.get(target)) return cache// 深拷贝递归之前，判断WeakMap中的key存在，则证明之前已经拷贝过该对象，有拷贝过直接返回
        const result = new target.constructor// 使用对象实例target的构造方法创建一个新对象
        map.set(target, result)// 使用WeakMap来存储对象对应关系，key和value都是对象，以便之后判断对象是否被拷贝过
        if(Array.isArray(target)){// 判断对象target是否是数组
            target.forEach((item, index) =>{// 使用forEach进行遍历，并对每一层进行深拷贝
                result[index] = deepClone(item, map)
            })
        } else {
            Object.keys(target).forEach(key => {
                result[key] = deepClone(target[key], map)
            })
        }
        return result
    } else {// 如果是基本类型或者null undefined 直接返回
        return target
    }
}
let obj5 = deepClone(obj3);
console.log(obj5)// { a: 1, b: { f: { g: 1 } }, c: [ 1, 2, 3 ] }
```



### 11.手写数组扁平化

#### 	11.1 递归实现

> 一项一项遍历，如果某一项是一个数组，则继续递归往下遍历，直到当前项为一个数再添加进数组

```js
// 方法一： 递归实现
let arr = [1, [2, [3, 4, 5]]];
function flatten(arr) {
    let res = [];// 定义一个数组存储结果
    for (let i = 0; i < arr.length; i++) {//遍历数组
        if (Array.isArray(arr[i])) {// 如果当前数组元素是一个数组 则进入递归调用--》扁平化
            res = res.concat(flatten(arr[i]));// 每一层扁平化都会返回一个数组 和上一层的数组res作一个拼接 
        } else {
            res.push(arr[i]);// 非数组则直接添加到res中
        }
    }
    return res;
}
console.log(flatten(arr)); //  [1, 2, 3, 4，5]
```

#### 	11.2 ES6中的flat([depth])方法

> flat()方法会按照一个可指定的深度递归遍历数组，并将所有元素与遍历到的子数组中的元素合并为一个新数组返回，depth是数组展开深度。

```js
function flatten(arr) {
    return arr.flat(Infinity);// 数组的嵌套层数不确定，最好直接使用 Infinity
}
```

#### 	11.3 数组的归并方法 reduce

> reduce方法接受两个参数，第一个是每一项都会运行的归并函数(回调函数)reducer，第二项是归并起点的初始值，传进reduce中的回调函数接收四个参数，1：上一个归并值(函数返回值)pre，2：当前项cur，3：当前项索引值index，4：数组本身arr，一般只用到前面两个。如果reduce没有传入可选的第二个参数，则第一次迭代从数组的第二项（当前项）开始，pre为第二项，例如求和。如果传入了参数，像下面传入了[]空数组，则传给回调函数的第一个参数pre是[]空数组

```js
function flatten(arr) {
    return arr.reduce((pre, next) => {
        return pre.concat(Array.isArray(next) ? flatten(next) : next)
    }, [])// 第二个参数[]空数组作为初始值pre
}
```

#### 	11.4 扩展运算符

#### 	11.5 正则

### 12.手写数组去重

#### 	12.1 使用Set集合

```js
// 方法一：使用Set集合
let arr = [1, 2, 3, 5, 1, 5, 9, 1, 2, 8];
// Array.from() 方法对一个类似数组或可迭代对象创建一个新的，浅拷贝的数组实例。
let res = Array.from(new Set(arr));
console.log(res);// [ 1, 2, 3, 5, 9, 8 ]
```

#### 	12.2 for循环

```js
// 方法二：for循环 
function removeDuplicates(arr) {
    let map = {};// 定义个map对象,使用map存储不重复的数字
    let res = [];// 定义数组保存结果
    for (let i = 0; i < arr.length; i++) {// 遍历数组
        if (!map.hasOwnProperty([arr[i]])) {// 判断map对象自身是否有arr[i]属性
            map[arr[i]] = 1;// 没有的话 就把arr[i]作为属性 1为值添加到对象中
            res.push(arr[i]);// 因为没有map中没有arr[i] 所以arr[i]不是重复项 则添加进res数组中
        }
    }
    return res;// 返回res
}
let a = removeDuplicates(arr);
console.log(a);// [ 1, 2, 3, 5, 9, 8 ]
```

### 13.手写数组的flat方法

```js
// flat()方法会按照一个可指定的深度递归遍历数组，并将所有元素与遍历到的子数组中的元素合并为一个新数组返回
// flat()方法 传入数组和depth数组深度
let arr = [1, [2, [3, 4, 5]]];
function _flat(arr, depth) {
    if (!Array.isArray(arr) || depth <= 0) {
        return arr;// 如果arr不是数组 或者 传入depth深度为小于0 则直接返回arr
    }
    return arr.reduce((pre, next) => {
        return pre.concat(Array.isArray(next) ? _flat(next) : next)
    }, [])
}
//测试
let res = _flat(arr, 3);
console.log(res);
```

### 14.树状结构的递归遍历

```js
let dt = [{
    id: '2',
    title: '浙江省',
    children: [{
        id: '2-1',
        title: '杭州市',
            children: [{
                id: '2-1-2',
                title: 'ad区',
                children: [{
                    id: '2-1-1-1',
                    children: []
                }]
            }],
            children: [{
                id: '2-1-1',
                title: '滨江区',
                children: [{
                    id: '2-1-1-1',
                    children: []
                }]
            }]
        },
        {
            id: '2-2',
            children: [{
                id: '2-2-1',
                children: [{
                    id: '2-2-1-1',
                    children: []
                }]
            }]
        }
    ]
}]

//查找所有上级方法
function treeFindPath (tree, path = []) {
  if (!tree) return []
  for (const data of tree) {
    // 这里按照你的需求来存放最后返回的内容吧
    path.push(data.title)
    // 满足条件返回path
    if (data.id=='2-1-1') return path
    if (data.children) {
      // 返回的path意味着找到了
      const findChildren = treeFindPath(data.children, [...path])
      // 递归返回
      if (findChildren.length) return findChildren
    }
    path.pop()
  }
  return []
}
//调用
let arr = treeFindPath(dt)
console.log('/'+arr.join('/'))  // /浙江省/杭州市/滨江区
```



## 二、设计模式

### 发布订阅模式和观察者模式的区别

两者所需要的角色数量不一样，观察者模式需要两个角色(观察者和消费者)便可成型，发布订阅需要三个角色(发布者，订阅者，第三方发布订阅中心)。不同应用场景的设计模式实现方法不尽相同。

<img src="../images/image-20220601141406185.png" alt="image-20220601141406185" style="zoom: 67%;" />

### 1.手写事件订阅发布模式(设计模式)

> 对象间一种一对多的依赖关系，当目标对象指定的动作发生改变时，订阅该动作的依赖对象会收到相应的通知。订阅者在发布订阅中心EventSubPub订阅/注册事件和回调函数，发布者通过中心发布/触发某一事件，会调用回调函数通知订阅/注册该事件的人

```js
// 用ES6的方式来写
class EventSubPub {
    constructor() {
        this.events = {};// 定义一个事件对象，存储订阅事件和回调函数(方法)
        // 如： {click : [handler1, handler2, ...], sleep : [handler1, handler2]...} 每一个事件有各种方法,形成方法数组
    }
    // 订阅事件/注册事件以及事件的回调函数
    on(eventName, handler) {// 参数为参数名和回调函数(就是触发该事件会执行某些方法)，这些方法可以是多个
        if (!this.events[eventName]) {// 如果本来没有注册事件
            this.events[eventName] = [handler]// 则创建数组并添加第一个回调函数
        } else {
            this.events[eventName].push(handler);// 如果本来注册过事件，则存入接下来的回调函数
            // this.event[eventName] 这是一个数组 是上面所创建的
        }
    }
    // 发布事件/触发事件回调 参数：事件名和事件回调函数参数
    emit(eventName, args) {// 需要传入事件名和参数
        if (!this.events[eventName]) {// 事件未注册 直接抛出错误
            return new Error('该事件未注册')
        }
        // 遍历对应事件里数组中的方法(回调函数) 并 执行所有回调函数(需要传入参数)
        this.events[eventName].forEach(handler => { handler(args) })
    }
    // 事件移除 参数 事件名和回调函数
    off(eventName, handler) {
        if(this.events[eventName]) {// 判断是否注册过该事件
            if(!handler) {// 判断是否传入了事件对应的回调函数
              delete this.events[eventName];// 没有传入直接删除事件
            } else {
                let index = this.events[eventName].indexOf(handler);// 在数组中找到第一个出现该回调函数handler的下标
                this.events[eventName].splice(index, 1);// 
                // slice 可以传入两个参数 若只传入一个 则返回该索引到数组末尾所有元素，若传入两个(a,b) 则切割索引a到b 不包括b的数组元素(左闭右开)
                // splice可以传入三个参数，第一个为开始位置，第二个为删除个数，第三个(还可以任意多个)为插入元素
            }
        }
    }
}
// 测试
const events = new EventSubPub()
function fn1() {
    console.log("fn1");
}
function fn2() {
    console.log("fn2");
}
function fn3() {
    console.log("fn3 ");
}
events.on("event1", fn1)
events.on("event1", fn2)
events.on("event2", fn2)
events.on("event3", fn3)// 注册事件和回调函数
events.emit("event1")// fn1 fn2
events.off("event1", fn1);// 关闭订阅 移除事件及对应的回调函数
events.emit("event1")// fn2
events.emit("event2")// fn2
events.emit("event3")// fn3
events.off("event2")// 关闭、移除整个事件event2
events.off("event3")// 关闭、移除整个事件event3
events.emit("event2")// 无
events.emit("event3")// 无
```

### 2.手写观察者模式

> 对象间一种一对多的依赖关系，当目标对象 Subject 的状态发生改变时，所有依赖它的对象 Observer 都会得到通知。被观察者Subject将观察者添加到自己的观察者列表，

```js
// Subject 被观察对象
function Subject() {
    this.observers = [];// 创建数组用于存储观察者
}
Subject.prototype = {// 在被观察者的原型上添加方法
    add(observer) {  // 添加观察者
        this.observers.push(observer);
    },
    notify() {  // 添加通知方法
        var observers = this.observers;// 获取所用添加进来的观察者
        for (var i = 0; i < observers.length; i++) {// 遍历调用观察者的更新方法 通知观察者们更新
            observers[i].update();
        }
    },
    remove(observer) {  // 删除观察者
        var observers = this.observers;
        for (var i = 0; i < observers.length; i++) {// 遍历所有观察者
            if (observers[i] === observer) {// 如果当前观察者和传进来的一样
                observers.splice(i, 1);// 则删除传进来的观察者
            }
        }
    },
}
// Observer 观察者对象
function Observer(name) {
    this.name = name;
}
Observer.prototype = {// 在观察者的原型上添加更新方法
    update() {  // 更新
        console.log('my name is ' + this.name);
    }
}
var sub = new Subject();
var obs1 = new Observer('guang1');
var obs2 = new Observer('guang2');
sub.add(obs1);
sub.add(obs2);
// 通知所有观察者
sub.notify();  //my name is guang1 my name is guang2
```

