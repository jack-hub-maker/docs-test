# 编程题

## 实现交通灯的逻辑：红灯亮 5s,绿灯亮 10s,黄灯亮 3s,红灯亮 5s...不停循环执行，亮的动作可以用一个伪函数来表示。

```JavaScript
      function sleep(wait) {
        return new Promise((reslove) => {
          setTimeout(reslove, wait);
        });
      }
      async function changeColor(color, wait) {
        // this.color = color;
        console.log(color);
        await sleep(wait);
      }
      async function go() {
        while (true) {
          await changeColor("red", 5000);
          await changeColor("green", 15000);
          await changeColor("yellow", 3000);
        }
      }
```

## 要求设计 LazyMan 类，实现以下功能

```JavaScript

LazyMan('Tony');
// Hi I am Tony

LazyMan('Tony').sleep(10).eat('lunch');
// Hi I am Tony
// 等待了10秒...
// I am eating lunch

LazyMan('Tony').eat('lunch').sleep(10).eat('dinner');
// Hi I am Tony
// I am eating lunch
// 等待了10秒...
// I am eating diner

LazyMan('Tony').eat('lunch').eat('dinner').sleepFirst(5).sleep(10).eat('junk food');
// Hi I am Tony
// 等待了5秒...
// I am eating lunch
// I am eating dinner
// 等待了10秒...
// I am eating junk food
```

代码

```JavaScript
      class LazyMans {
        constructor(name) {
          this.name = name;
          this.queen = [];
          setTimeout(() => {
            this._next();
          }, 0);
        }
        sleep(wait, isFiset = false) {
          const _sleep = () => {
            setTimeout(() => {
              console.log(`等待了${wait}秒`);
              this._next();
            }, wait * 1000);
          };
          isFiset ? this.queen.unshift(_sleep) : this.queen.push(_sleep);
          return this;
        }
        _next() {
          if (!this.queen.length) return;
          let shift = this.queen.shift();
          shift.call(this);
        }
        eat(food) {
          const _eat = () => {
            console.log(`I am eating ${food}`);
            this._next();
          };
          this.queen.push(_eat);
          return this;
        }
        sleepFirst(wait) {
          return this.sleep(wait, true);
        }
      }
      function LazyMan(name) {
        return new LazyMans(name);
      }
```

## 实现一个批量请求函数

要求：

1. 批量请求函数，最大并发数 maxNum
2. 每当有一个请求返回，就留下一个空位，可以增加新的请求
3. 所有请求完成后，结果按照 urls 里面的顺序依次发出。

```JavaScript
function handleFetchQueue(func, urls, max, callback) {
  const urlCount = urls.length;
  const requestsQueue = [];
  const results = [];
  const errList = [];
  let i = 0;
  const isCallBack = () => {
    const allLen = results.length + errList.length;
    if (allLen === urlCount) {
      "function" === typeof callback && callback(results, errList);
    }
  };
  const handleRequest = (url) => {
    const req = func(url)
      .then((res) => {
        console.log("当前并发： " + requestsQueue);
        results.push(res);
        const allLen = results.length + errList.length;
        if (allLen < urlCount && i + 1 < urlCount) {
          requestsQueue.shift();
          handleRequest(urls[++i]);
        } else {
          isCallBack();
        }
      })
      .catch((e) => {
        errList.push(e);
        isCallBack();
      });
    if (requestsQueue.push(req) < max) {
      handleRequest(urls[++i]);
    }
  };
  handleRequest(urls[i]);
}
```

## 实现 36 进制加法，不得直接使用 parselnt 转化十进制进行计算，如‘1b'+'0a'=57

## 实现 sortByDept()函数，被依赖最深的元素，输出到最前

```JavaScript
sortByDept([
    {name:'a',requires:['b','c']},
    {name:'b',requires:['c']},
    {name:'b',requires:['']},
])
// 输出：
[
  {name:'c',requires:['']},
  {name:'b',requires:['c']},
  {name:'a',requires:['b','c']},
]
```

## Promise

```JavaScript
async function async1 () {
  console.log('async1 start');
  setTimeout(() => {
    console.log('timer1 start');
  }, 500);
  Promise.resolve().then((res) => {
    console.log('promise1');
  })
  await async2();
  setTimeout(() => {
    console.log('timer1 end');
  }, 500);
  console.log('async1 end');
}

async function async2 () {
  setTimeout(() => {
    console.log('timer2');
  }, 1000);
  Promise.resolve().then((res) => {
    console.log('promise2');
  })
  console.log('async2');
}

async1();

console.log('start');
```

解：

```JavaScript
async1 start
async2
start
promise1
promise2
async1 end
timer1  start
timer1 end
timer2
```

## 简单说一下 MVC/MVVM/Flux 的区别。用 JS 实现一个 MVC

```JavaScript

/* 1. App 出始化时渲染出如下的用户列表 2. 并实现点击每个用户他对应的 priority 就自增
<ul id="userlist">
  <li>User Name: Peter, User Number: 123， User Priority: 1</li>
  <li>User Name: Glenn, User Number: 456, User Priority: 1</li>
  <li>User Name: Lucy, User Number: 789，User Priority: 1</li>
<ul> */
const users = [
  { name: 'Peter', num: 123, priority: 1 },
  { name: 'Glenn', num: 456, priority: 1 },
  { name: 'Lucy', num: 789, priority: 1 }
];

class UserModel {
  constructor() {
    this._data = users;
  }
  // todo
  addPriority (index) {
    this._data[index].priority++
  }
}

class UserView {
  constructor() {
    this.container = document.querySelector('#userlist');
  }

  render (users) {
    // todo

    while (this.container.firstChild) {
      this.container.removeChild(this.container.firstChild)
    }
    users.forEach((user, index) => {
      const li = document.createElement('li')
      li.textContent = `User Name: ${user.name}, User Number: ${user.name}， User Priority: ${user.priority}`
      li.id = index
      this.container.append(li)
    });
  }
  bindClick (handler) {
    this.container.addEventListener('click', event => {
      console.log(event.target.id)
      if (event.target) {
        handler(event.target)
      }
    })
  }
}

class UserController {
  constructor(model, view) {
    this.model = model;
    this.view = view;
    this.view.bindClick(this.bindEvent.bind(this))
  }

  init () {
    // todo
    this.view.render(this.model._data)
  }

  bindEvent (target) {
    // todo
    this.model.addPriority(target.id)
    this.init()
  }
}


const app = new UserController(new UserModel(), new UserView())
app.init();
```

[ github 地址 ](https://github.com/chelseachen007/writeCode/tree/master/MVC)

## 字符串的规则解析：

例子：a(b)<2>c 输出：abbc，a(b(c)<3>de)<2>f 输出 abcccdebcccdef；()代表重复内容，<>代表重复的次数

```JavaScript

```

## 实现一个函数将对象结构转成在树形结构

```JavaScript
// 原始 list 如下
let list =[
    {id:1,name:'部门A',parentId:0},
    {id:2,name:'部门B',parentId:0},
    {id:3,name:'部门C',parentId:1},
    {id:4,name:'部门D',parentId:1},
    {id:5,name:'部门E',parentId:2},
    {id:6,name:'部门F',parentId:3},
    {id:7,name:'部门G',parentId:2},
    {id:8,name:'部门H',parentId:4}
];
const result = convert(list, ...);
```

解：

```JavaScript
function convert () {
    let res = []
    let temp = {}
    for (let i = 0; i < list.length; i++) {
        temp[list[i].id] = list[i]; // 以id作为索引存储元素，可以无需遍历直接定位元素
    }
    for (let j = 0; j < list.length; j++) {
        const item = list[j]
        const current = temp[item.id]
        if (item.parentId === 0) {
            res.push(current)
        } else {
            if (!temp[item.parentId]['children']) temp[item.parentId].children = []
            temp[item.parentId].children.push(item)
        }
    }
    return res
}
```

## 实现一个函数将小驼峰转化成大驼峰

## 实现超大整数相加

```JavaScript
function bigNumAdd (num1, num2) {
    let res = '', carry = 0
    num1 = `${num1}`.split('')
    num2 = `${num2}`.split('')
    while (num1.length || num2.length || carry) {
        carry = ~~num1.pop() + ~~num2.pop()
        res = (carry % 10) + res
        carry = carry > 9
    }
    return res
}
```

## 将如下对象输出成指定数组

```JavaScript
var obj = {
    a: {
        b: {
            c: { f: "aa" }
        },
        d: {
            e: { g: "bb" },
            h: { i: "cc" }
        },
        j: {
            k: "dd"
        }
    }
}
//预期结果 [f,g,i,c,e,h,k,b,d,j,a]
```

解：用层序遍历的思想

```JavaScript
function Format (obj) {
    if (!obj) return []
    let res = []
    let queue = [obj]
    while (queue.length) {
        let arr = []
        let length = queue.length
        for (let i = 0; i < length; i++) {
            let curr = queue.shift()
            for (let s in curr) {
                if (typeof curr === 'object') {
                    queue.push(curr[s])
                    arr.push(s)
                }
            }
        }
        res = [...arr, ...res]
    }
    return res
}
```
