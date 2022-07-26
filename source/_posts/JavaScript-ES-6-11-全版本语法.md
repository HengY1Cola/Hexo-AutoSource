---
title: JavaScript ES(6-11)全版本语法
categories: 分享
keywords: JavaScript
top_img: /img/1555172.webp
cover: /img/1555154.png
abbrlink: 3dee90a5
date: 2022-07-26 22:54:36
---

慕课链接：https://coding.imooc.com/class/444.html

继续系统加固前端知识，我发现先看文档，视频辅助效率最高；Fighting

电子书网址：http://es.xiecheng.live

Babel 编译器：https://www.babeljs.cn/

为什么我要写这个笔记？ 万一哪天域名失效了呢，是吧，当个文档字典来查吧

## ECMAScript2015

### 新的声明方式：let

```javascript
// 1. 不属于顶层对应window 弥补了var的全局污染问题
let a = 5
console.log(a)
console.log(window.a)

// 2.不允许重复声明

// 3. 不存在变量提升
console.log(a)
var a = 5 // 使用var会在下面

// 4. 暂时性死区
var a = 5
if (true) {
    a = 6
    let a
}

// 5. 块级作用域
for (let i = 0; i < 3; i++) {
    console.log('循环内:' + i)
}
console.log('循环外:' + i) //出不来了

// 闭包
for (var i = 0; i < 3; i++) {
    (function(j){
        setTimeout(function(){
            console.log(j)
        })
    })(i)
}
// 是相同的效果
for (let i = 0; i < 3; i++) {
    setTimeout(function () {
        console.log(i)
    })
}
```

### 新的声明方式：const

```javascript
// 定义一个常量其余与let相似
// 不变的是内存地址
const a = 5

// 只能浅层冻结
const obj = {
    name: 'xiecheng',
    age: 34,
    skill: {
        name: 'code',
        year: 11
    }
}
Object.freeze(obj)
console.log(obj)
obj.school = 'imooc'
obj.skill.year = 12
console.log(obj)
```

### 解构赋值

```js
// 左边右边能够对应就好了
let [a, b, [c, d]] = [1, 2, [3, 4]]
console.log(a, b, c, d)

let [a, b, [c]] = [1, 2, [3, 4]]
console.log(a, b, c) // c = 3

let [a, b, c] = [1, 2, [3, 4]]
console.log(a, b, c) // c = [3, 4]

let [a, b, c, d = 5] = [1, 2, [3, 4], 6]
console.log(a, b, c, d) // d = 6 惰性的

function foo([a, b, c]) {
    console.log(a, b, c)
}
let arr = [1, 2, 3]
foo(arr)

let json = '{"a": "hello", "b": "world"}'
let {a, b}= JSON.parse(json)
console.log(a, b)
```

###  数组相关

```javascript
// forEach 不支持break和continue
arr.forEach(function(elem, index, array){
    console.log(elem, index)
})

// map 回调处理、原来不变
let result = arr.map(function(value){
    value += 1
    return value
})
console.log(arr, result)

// filter 找到符合的然后组成新的
let result = arr.filter(function (value) {
    return value == 2
})
console.log(arr, result)

// some 存在符合条件的结果返回true
let result = arr.some(function (value) {
    return value == 4
})
console.log(arr, result)

// every 是否都符合
let result = arr.every(function(value) {
    return value == 2
})
console.log(arr, result)

// reduce
let sum = arr.reduce(function(prev, cur, index, array) {
    return prev + cur
}, 0) // 0 就是初始值
console.log(sum)

// 遍历 for...of是支持 break、continue、return的
for (let val of [1, 2, 3]) {
    console.log(val);
}

// ----------------------------------------------

// from形成数组
let imgs = Array.from(document.querySelectorAll('img')); // 来形成数组形式
Array.from({
    length: 5
}, function() {
    return 1
}) // 来形成[1,1,1,1,1]

// fill fill(换成哪个, 从第几个开始，到第几个结束(不包括))
let array = [1, 2, 3, 4]
array.fill(0, 1, 2)
// [1,0,3,4]

// find
let array = [5, 12, 8, 130, 44];
let found = array.find(function(element) {
    return element > 10;
});
console.log(found);

// findIndex 找不到为-1
let array = [5, 12, 8, 130, 44];
let found = array.find(function(element) {
    return element > 10;
});
console.log(found);

// copyWithin 会修改当前数组
let arr = [1, 2, 3, 4, 5]
console.log(arr.copyWithin(1, 3)) // [1, 4, 5, 4, 5]
```

###  函数相关

```javascript
// 可设置默认值
function foo(x, y = 'world') {
    console.log(x, y)
}
foo('hello', 0)

// 判断函数有几个参数 Function.length 是统计第一个默认参数前面的变量数
function foo(a, b = 1, c) {
    console.log(foo.length)
}
foo('a', 'b') // 1

// 不是很确定参数有多少个 Rest Parameter 是数组
function sum(...nums) {
    let num = 0
    nums.forEach(function(item) {
        num += item * 1
    })
    return num
}

// Spread Operator 是把固定的数组内容“打散”到对应的参数
function sum(x = 1, y = 2, z = 3) {
    return x + y + z
}
console.log(sum(...[4])) // 9

// 返回没有指定默认值的参数个
function foo(x = 1, y = 2, z = 3) {
    console.log(x, y)
}
console.log(foo.length)

// 匿名函数
let hello = (name) => {
    console.log('say hello', name)
}
// 如果返回值是字面量对象，一定要用小括号包起来
let person = (name) => ({
      age: 20,
      addr: 'Beijing City'
 })

// 箭头函数中的this
let foo = {
    name: 'es',
    say: function() {
        console.log(this.name)
    }
} // this 指向的是调用 say 方法的对象
console.log(foo.say()) // es

let foo = {
    name: 'es',
    say: () => {
        console.log(this.name, this)
    }
} // this 的指向也就是 foo 外层的所指向的 window
console.log(foo.say()) // undefined
```

###  对象相关

```javascript
// Object.is() 判断两个对象是否相等
let obj1 = { // new Object()
    name: 'xiecheng',
    age: 34
}

let obj2 = { // new Object()
    name: 'xiecheng',
    age: 34
}
console.log(obj1 == obj2) // false
console.log(Object.is(obj1, obj2)) // false
let obj2 = obj1 // 即使值相同但是地址不同
console.log(Object.is(obj1, obj2)) // true

// Object.assign(target, ...sources)
const target = {
    a: 1,
    b: 2
}
const source = {
    b: 4,
    c: 5
}
const returnedTarget = Object.assign(target, source)
console.log(target) //  Object { a: 1, b: 4, c: 5 }
console.log(returnedTarget) // Object { a: 1, b: 4, c: 5 }

// 对象的遍历方式
for (let key in obj) {
    console.log(key, obj[key])
}
Object.keys(obj).forEach(key => {
    console.log(key, obj[key])
})
```

### Class相关

```javascript
class Animal {
    constructor(type) {
        this.type = type
    }
    walk() {
        console.log( `I am walking` )
    }
}
let dog = new Animal('dog')
let monkey = new Animal('monkey')
console.log(dog.hasOwnProperty('type')) //true

// set/get
class Animal {
    constructor(type, age) {
        this.type = type
        this._age = age
    }
    get age() {
        return this._age
    }
    set age(val) {
        this._age = val
    }
}

// 静态方法
class Animal {
    constructor(type) {
        this.type = type
    }
    walk() {
        console.log( `I am walking` )
    }
    static eat() {
        console.log( `I am eating` )
    }
}

// 继承（原）
let Dog = function() { // 定义子类
    // 初始化父类 
    Animal.call(this, 'dog')
    this.run = function() {
        console.log('I can run')
    }
}
Dog.prototype = Animal.prototype

// 继承（现）
class Animal {
    constructor(type) {
        this.type = type
    }
    walk() {
        console.log( `I am walking` )
    }
    static eat() {
        console.log( `I am eating` )
    }
}

class Dog extends Animal {
  constructor () {
    super('dog')
  }
  run () {
    console.log('I can run')
  }
}
```

###  Symbol相关

数据类型 `Symbol` ，表示独一无二的值，可以保证不会与其他属性名产生冲突

```javascript
let s1 = Symbol('foo')
let s2 = Symbol('foo')
console.log(s1)
console.log(s2)
console.log(s1 === s2) // false

// Symbol.for()
// 如果有，就返回这个 Symbol 值，否则就新建一个以该字符串为名称的 Symbol 值
let s1 = Symbol.for('foo')
let s2 = Symbol.for('foo')
console.log(s1 === s2) // true

// Symbol.keyFor()
// 返回一个已登记的 Symbol 类型值的key
const s1 = Symbol('foo')
console.log(Symbol.keyFor(s1)) // undefined
const s2 = Symbol.for('foo')
console.log(Symbol.keyFor(s2)) // foo

// 同名的学生信息就不会被覆盖
const stu1 = Symbol('李四')
const stu2 = Symbol('李四')
const grade = {
    [stu1]: {
        address: 'yyy',
        tel: '222'
    },
    [stu2]: {
        address: 'zzz',
        tel: '333'
    },
}
console.log(grade)
console.log(grade[stu1])
console.log(grade[stu2])

// Symbol 解决“强耦合”
const shapeType = {
    triangle: Symbol(),
    circle: Symbol()
}
function getArea(shape) {
    let area = 0
    switch (shape) {
        case shapeType.triangle:
            area = 1
            break
        case shapeType.circle:
            area = 2
            break
    }
    return area
}
console.log(getArea(shapeType.triangle))
```

###  Set相关

```javascript
// Set类似于数组，但是成员的值都是唯一的

// 声明
let s = new Set([1, 2, 3, 4])
// 添加
s.add('hello').add('goodbye')

// 删除指定数据
s.delete('hello') // true
s.clear()    // 删除全部数据

// 判断是否包含数据项
s.has('hello') // true

// 计算数据项总数
s.size // 2

// 数组合并并去重
let arr1 = [1, 2, 3, 4]
let arr2 = [2, 3, 4, 5, 6]
let s = new Set([...arr1, ...arr2])

// 交集
let s1 = new Set(arr1)
let s2 = new Set(arr2)
let result = new Set(arr1.filter(item => s2.has(item)))
console.log(Array.from(result))

// 差集
let arr3 = new Set(arr1.filter(item => !s2.has(item)))
let arr4 = new Set(arr2.filter(item => !s1.has(item)))
console.log(arr3)
console.log(arr4)
console.log([...arr3, ...arr4])

// 遍历方式
console.log(s.keys()) // SetIterator {"hello", "goodbye"}
console.log(s.values()) // SetIterator {"hello", "goodbye"}
console.log(s.entries()) // SetIterator {"hello" => "hello", "goodbye" => "goodbye"}
s.forEach(item => {
    console.log(item) // hello // goodbye
})
for (let item of s) {
    console.log(item)
}
```

###  String相关

```javascript
`string text ${expression} string text`  // 来表示长的以及带参数

// String.prototype.fromCodePoint() 用于从 Unicode 码点返回对应字符
console.log(String.fromCodePoint(0x20BB7))

// String.prototype.indexOf() 返回出现的下标位置
console.log(str.indexOf('mo'))

// String.prototype.includes() 判断一个字符串是否包含在另一个字符串中
console.log(str.includes('mo'))
```

###  Number相关

```javascript
// 二进制 用前缀0b
const a = 0B0101
console.log(a)

// 八进制 用前缀0o
const b = 0O777
console.log(b)

// Number.isFinite() 检查一个数值是否为有限的
Number.isFinite(NaN) // false

// Number.isNaN() 检查一个值是否为NaN
Number.isNaN(NaN) // true

// Number.parseInt() 

// Number.isInteger() 判断一个数值是否为整数
Number.isInteger(25.1) // false
 
// Number.MAX_SAFE_INTEGER Number.MIN_SAFE_INTEGER

// Math.trunc() 用于去除一个数的小数部分，返回整数部分
// Math.sign() 判断一个数到底是正数、负数、还是零
// Math.cbrt() 计算一个数的立方根
```

###  Proxy相关

```javascript
let o = {
    name: 'xiaoming',
    age: 20
}
// 原来
console.log(o.from || '')
// 现在
let handler = {
    get(obj, key) {
        return Reflect.has(obj, key) ? obj[key] : ''
    }
}
let p = new Proxy(o, handler)
console.log(p.from)

// 使用proxy做校验
// Validator.js
export default (obj, key, value) => {
    if (Reflect.has(key) && value > 20) {
        obj[key] = value
    }
}

import Validator from './Validator'
let data = new Proxy(response.data, {
    set: Validator // 在进行set的时候进行校验
})

// 常规拦截操作get
let dict = {
    'hello': '你好',
    'world': '世界'
}
dict = new Proxy(dict, {
    get(target, prop) {
        return prop in target ? target[prop] : prop
    }
})
console.log(dict['world'])
console.log(dict['imooc'])

// 常规拦截操作set
let arr = []
arr = new Proxy(arr, {
    set(target, prop, val) {
        if (typeof val === 'number') {
            target[prop] = val
            return true
        } else {
            return false
        }
    }
})
arr.push(5)
arr.push(6)
console.log(arr[0], arr[1], arr.length)

// 常规拦截操作has
let range = {
    start: 1,
    end: 5
}

range = new Proxy(range, {
    has(target, prop) {
        return prop >= target.start && prop <= target.end
    }
})
console.log(2 in range)
console.log(9 in range)
```

###  *Promise相关

```javascript
const promise = new Promise(function(resolve, reject) {
    if ( /* 异步操作成功 */ ) {
        resolve(value)
    } else {
        reject(error)
    }
})

// Promise.prototype.then()
promise.then(function(value) {
    console.log(value)
}, function(error) {
    console.error(error)
})

//  Promise.prototype.catch()
// reject(new Error()) 的方式来触发异常
.catch((e) => {
    console.log(e.message) // es
})

// Promise.resolve() 认为是以下代码的语法糖
new Promise(function(resolve) {
    resolve(42)
})
// Promise.reject()
Promise.reject(new Error("出错了")) === new Promise(function(resolve, reject) {
    reject(new Error('出错了'))
})

// Promise.all()
// Promise.all 生成并返回一个新的 Promise 对象 参数传递promise数组中所有的 Promise 对象都变为resolve的时候，该方法才会返回
var p1 = Promise.resolve(1)
var p2 = Promise.resolve(2)
var p3 = Promise.resolve(3)
Promise.all([p1, p2, p3]).then(function(results) {
    console.log(results) // [1, 2, 3]
})

// Promise.race()
// 任何一个 Promise 对象如果变为 resolve 或者 reject 的话， 该函数就会返回
var p1 = Promise.resolve(1)
var p2 = Promise.resolve(2)
var p3 = Promise.resolve(3)
Promise.race([p1, p2, p3]).then(function(value) {
    console.log(value) // 1
})
```

###  Generator相关

```javascript
//  Generators 是可以用来控制迭代器的函数。它们可以暂停，然后在任何时候恢复
// 比普通函数多一个 *
// 函数内部用 yield 来控制程序的执行的“暂停”
// 函数的返回值通过调用 next 来“恢复”程序执行

function* generatorForLoop() {
    for (let i = 0; i < 5; i += 1) {
        yield console.log(i)
    }
}
const genForLoop = generatorForLoop()
console.log(genForLoop.next()) // first console.log - 0
console.log(genForLoop.next()) // 1

// yield 表达式的返回值是 undefined，但是遍历器对象的 next 方法可以修改这个默认值
// yeild * 是委托给另一个遍历器对象或者可遍历对象

function* gen() {
      let val
      val = yield 1
      console.log( `1:${val}` ) // 1:undefined
      val = yield 2
      console.log( `2:${val}` ) // 2:undefined
      val = yield 3
      console.log( `3:${val}` ) // 3:undefined
  }
var g = gen()
console.log(g.next()) // {value: 1, done: false}
console.log(g.next()) // {value: 2, done: false}
console.log(g.next()) // {value: 3, done: false}
console.log(g.next()) // {value: undefined, done: true}

// next[value] 是可以接受参数的，这个参数可以让你在 Generator 外部给内部传递数据，而这个参数就是作为 yield 的返回值
// return 方法可以让 Generator 遍历终止
// 可以通过 throw 方法在 Generator 外部控制内部执行的“终断”。
function* gen() {
      var val = 100
      while (true) {
          console.log( `before ${val}` )
          val = yield val
          console.log( `return ${val}` )
      }
  }

var g = gen()
console.log(g.next(20).value)
  // before 100
  // 100
console.log(g.next(30).value)
  // return 30
  // before 30
  // 30
console.log(g.return()) // {value: undefined, done: true}
g.throw(new Error('break'))
```

###  Iterator相关

```javascript
// 一种机制来自定义for...of循环的行为
// 自定义的数据结构进行遍历

// 想最后获取到每个作者
// next	返回一个对象的无参函数，被返回对象拥有两个属性：done 和 value
let authors = {
    allAuthors: {
        fiction: [
            'Agatha Christie',
            'J. K. Rowling',
            'Dr. Seuss'
        ],
        scienceFiction: [
            'Neal Stephenson',
            'Arthur Clarke',
            'Isaac Asimov',
            'Robert Heinlein'
        ],
        fantasy: [
            'J. R. R. Tolkien',
            'J. K. Rowling',
            'Terry Pratchett'
        ]
    }
}

authors[Symbol.iterator] = function() {
    let allAuthors = this.allAuthors
    let keys = Reflect.ownKeys(allAuthors)
    let values = []
    return {
        next() {
            if (!values.length) {
                if (keys.length) {
                    values = allAuthors[keys[0]]
                    keys.shift()
                }
            }
            return {
                done: !values.length,
                value: values.shift()
            }
        }
    }
}

for (let value of authors) {
    console.log( `${value}` )
}

// 换一种Generator实现
authors[Symbol.iterator] = function*() {
    let allAuthors = this.allAuthors
    let keys = Reflect.ownKeys(allAuthors)
    let values = []
    while (1) {
        if (!values.length) {
            if (keys.length) {
                values = allAuthors[keys[0]]
                keys.shift()
                yield values.shift()
            } else {
                return false
            }
        } else {
            yield values.shift()
        }
    }
}
```

###  Module模块

将每个js文件看作是一个模块，每个模块通过固定的方式引入，并且通过固定的方式向外暴露指定的内容

按照js模块化的设想，一个个模块按照其依赖关系组合，最终插入到主程序中

无模块化-->CommonJS规范-->AMD规范-->CMD规范-->ES6模块化

在ES6中，可以使用 import 关键字引入模块，通过 exprot 关键字导出模块，但是由于ES6目前无法在浏览器中执行，所以，只能通过babel将不被支持的import编译为当前受到广泛支持的 require

```javascript
// 暴露变量/常量
export const name = 'hello'
export let addr = 'BeiJing City'

// 暴露函数
export function say(content) {
    console.log(content)
}

// as 重新取名字
const name = 'hello'
let addr = 'BeiJing City'
var list = [1, 2, 3]
export {
    name as cname,
    addr as caddr,
    list
}

// import 导入
import list, {
      cname,
      caddr
} from A
```

##  **ECMAScript2016(ES7)**

###  Array.prototype.includes()

ES7引入的Array.prototype.includes() 方法用来判断一个数组是否包含一个指定的值

```javascript
const arr = ['es6', 'es7', 'es8']
console.log(arr.includes('es6')) // true

const arr = ['es6', 'es7', 'es8']
console.log(arr.includes('es7', 1)) // true
console.log(arr.includes('es7', 2)) // false

const demo = [1, NaN, 2, 3]
demo.indexOf(NaN) //-1
demo.includes(NaN) //true
```

## **ECMAScript2017(ES8)**

###  async / await

async 是让我们写起 Promise 像同步操作， 前面添加了async的函数在执行后都会自动返回一个Promise对象

在async函数中使用await，那么await这里的代码就会变成同步的了

意思就是说只有等await后面的Promise执行完成得到结果才会继续下去，await就是等待。

await 只能在 async 标记的函数内部使用，单独使用会触发 Syntax error

```javascript
async function foo() {
    return 'imooc' // Promise.resolve('imooc')
}
console.log(foo()) // Promise

function timeout() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            // resolve('success')
            reject('error')
        }, 1000)
    })
}
async function foo() {
    return await timeout()
}
foo().then(res => {
    console.log(res)
}).catch(err => {
    console.log(err)
})
```

###  Object

ES8中对象扩展补充了两个静态方法，用于遍历对象：Object.values()，Object.entries()

```javascript
const obj = {
    name: 'imooc',
    web: 'www.imooc.com',
    course: 'es'
}

console.log(Object.values(obj)) // ["imooc", "www.imooc.com", "es"]
for (let [k, v] of Object.entries(grade)) {
    console.log(k, v)
}

// getOwnPropertyDescriptors() 不想让 Lima 这个属性和值被枚举
const data = {
    Portland: '78/50',
    Dublin: '88/52',
    Lima: '58/40'
}
Object.defineProperty(data, 'Lima', {
    enumerable: false
})
Object.entries(data).map(([city, temp]) => {
    console.log( `City: ${city.padEnd(16)} Weather: ${temp}` )
    // City: Portland         Weather: 78/50
    // City: Dublin           Weather: 88/52
})
console.log(Object.getOwnPropertyDescriptor(data, 'Lima')) // 会发现 enumerable 为flase
// {value: "58/40", writable: true, enumerable: false, configurable: true}
```

###  String

新增了两个实例函数 String.prototype.padStart 和 String.prototype.padEnd

允许将空字符串或其他字符串添加到原始字符串的开头或结尾

```javascript
const tel = '13012345678'
const newTel = tel.slice(-4).padStart(tel.length, '*') // 需要为原来的长度但是现在只有后面4位
console.log(newTel) // *******5678
```

## **ECMAScript2018(ES9)**

###  for await of

循环等待每个Promise对象变为resolved状态才进入下一步，更加优雅了

```javascript
function Gen(time) {
    return new Promise(function(resolve, reject) {
        setTimeout(function() {
            resolve(time)
        }, time)
    })
}

async function test() {
    let arr = [Gen(2000), Gen(100), Gen(3000)]
    for await (let item of arr) {
        console.log(Date.now(), item)
    }
}

// 想定义适合 for...await...of 的异步遍历器 Symbol.asyncIterator

let obj = {
  count: 0,
    
  Gen (time) {
    return new Promise(function (resolve, reject) {
      setTimeout(function () {
        resolve({ done: false, value: time })
      }, time)
    })
  }, 
    
  [Symbol.asyncIterator] () {
    let self = this
    return {
      next () {
        self.count++
        if (self.count < 4) {
          return self.Gen(Math.random() * 1000)
        } else {
          return Promise.resolve({
            done: true,
            value: ''
          })
        }
      }
    }

  }
}
async function test () {
  for await (let item of obj) {
    console.log(Date.now(), item)
  }
}
```

###  RegExp

```javascript
// 点（.）是一个特殊字符，代表任意的单个字符，但是有两个例外。一个是四个字节的 UTF-16 字符，这个可以用u修饰符解决；另一个是行终止符
// dotAll 模式：它让 . 名副其实

const re = new RegExp('foo.bar', 's')
console.log(re.test('foo\nbar')) // true
console.log(re.test('foo\nbar')) // true

// index [匹配的结果的开始位置]
// input [搜索的字符串]
// groups [一个捕获组数组 或 undefined（如果没有定义命名捕获组）]
console.log('2020-05-01'.match(/(\d{4})-(\d{2})-(\d{2})/))
// ["2020-05-01", "2020", "05", "01", index: 0, input: "2020-05-01", groups: undefined]


let test = 'hello world'
console.log(test.match(/hello(?=\sworld)/)) // ["hello", index: 0, input: "hello world", groups: undefined]
console.log(test.match(/(?<=world\s)hello/)) // ["hello", index: 6, input: "world hello", groups: undefined]
// (?<...)是后行断言的符号，(?...)是先行断言的符号，然后结合 =(等于)、!(不等)、\1(捕获匹配)

```

###  Rest & Spread

```javascript
const input = {
  a: 1,
  b: 2
}
const output = {
  ...input,
  c: 3
}

const input = {
  a: 1,
  b: 2,
  c: 3
}
let { a, ...rest } = input
```

###  Promise.prototype.finally()

指定不管最后状态如何都会执行的回调函数。Promise.prototype.finally() 方法返回一个Promise，在promise执行结束时，无论结果是fulfilled或者是rejected，在执行then()和catch()后，都会执行finally指定的回调函数。

```javascript
let connection
db.open()
    .then(conn => {
        connection = conn
        return connection.select({
            name: 'Jane'
        })
    })
    .then(result => {
        // Process result
        // Use `connection` to make more queries
    })···
    .catch(error => {
        // handle errors
    })
    .finally(() => {
        connection.close()
    })
```

## **ECMAScript2019(ES10)**

###  Object.fromEntries()

```javascript
// 可以将map先entries后filter再fromEntries转回来实现过滤
const obj = {
    name: 'imooc',
    course: 'es'
}
const entries = Object.entries(obj)
const fromEntries = Object.fromEntries(entries) // 转回来
```

###  Array

```javascript
// Array.prototype.flat() 所有元素与遍历到的子数组中的元素合并为一个新数组返回
const numbers = [1, 2, [3, 4, [5, 6]]]
console.log(numbers.flat())  // [1, 2, 3, 4, [5, 6]]
console.log(numbers.flat(2)) // [1, 2, 3, 4, 5, 6]

// flatMap()  首先使用映射函数映射每个元素，然后将结果压缩成一个新数组
const numbers = [1, 2, 3]
numbers.map(x => [x * 2]) // [[2], [4], [6]]
numbers.flatMap(x => [x * 2]) // [2, 4, 6]
```

###  Symbol

```javascript
const name = Symbol('es')
console.log(name.description) // es
console.log(name.description === 'es') // true 来判断
```

###  **ECMAScript2020(ES11)**

###  Dynamic Import

```javascript
const oBtn = document.querySelector('#btn')
oBtn.addEventListener('click', () => {
    import('./ajax').then(mod => {
        // console.log(mod)
        mod.default('static/a.json', res => {
            console.log(res)
        })
    })
})
```

###  Promise.allSettled()

并发任务中，无论一个任务正常或者异常，都会返回对应的的状态

```javascript
Promise.allSettled([
    Promise.reject({
        code: 500,
        msg: '服务异常'
    }),
    Promise.resolve({
        code: 200,
        data: ['1', '2', '3']
    }),
    Promise.resolve({
        code: 200,
        data: ['4', '5', '6']
    })
]).then(res => {
    console.log(res)
    const data = res.filter(item => item.status === 'fulfilled')
    console.log(data)
}).catch(err => {
    console.log(err)
    console.log('失败')
})
```

###  Optional chaining

```javascript
const user = {
    address: {
        street: 'xx街道',
        getNum() {
            return '80号'
        }
    }
}

const street2 = user?.address?.street
const num2 = user?.address?.getNum?.()
console.log(street2, num2)
```
