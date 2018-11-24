# 彻底搞懂Promise、Generator函数 yield、async await 的关系

## 说到底，这三者的出现，都是为了解决异步问题。

Promise是一个构造函数，自身有race、all、reject、resolve 方法，原型上有then、catch方法。
ES6将其写进语言标准，统一了用法，而在此之前，你也可以自己造一个Promise。

Generator 函数是 ES6 提供的一种异步编程解决方案，形式上，Generator 函数是一个普通函数，
整个Generator函数就是一个封装的异步任务，或者说是异步任务的容器，异步操作需要暂停的地方，都用yield语句。
Generator函数特征：
（1）function 关键字和函数之间有一个星号(*),且内部使用yield表达式，定义不同的内部状态。
（2）调用Generator函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象。
可以借助co模块（一个将Generator函数繁琐的next()过程简化的神器，实现方式源码其实很简单），通过 co(g()),将其最终返回结果变成Promise对象。

async await是ES7的新特性，也是Generator函数的语法糖。
可以让我们书写代码时更加流畅，当然也增强了代码的可读性。
简单来说：async-await 是建立在 promise机制之上的，并不能取代其地位。
执行一个async 函数，返回的是一个Promise对象，其实async await本身。

对比：
相较于 Generator，Async 函数的改进在于下面四点： 
* 内置执行器。Generator 函数的执行必须依靠执行器，而 Aysnc 函数自带执行器，调用方式跟普通函数的调用一样 
* 更好的语义。async 和 await 相较于 * 和 yield 更加语义化 
* 更广的适用性。co 模块约定，yield 命令后面只能是 Thunk 函数或 Promise对象。而 async 函数的 await 命令后面则可以是 Promise 或者 原始类型的值（Number，string，boolean，但这时等同于同步操作） 
* 返回值是 Promise。async 函数返回值是 Promise 对象，比 Generator 函数返回的 Iterator 对象方便，可以直接使用 then() 方法进行调用。

在网上找了个例子，
首先先定义一个 Fetch 方法用于获取 github user 的信息：
```javascript
function fetchUser() { 
    return new Promise((resolve, reject) => {
        fetch('https://api.github.com/users/superman66')
        .then((data) => {
            resolve(data.json());
        }, (error) => {
            reject(error);
        })
    });
}
```


【Promise 方式】
```javascript
function getUserByPromise() {
    fetchUser()
        .then((data) => {
            console.log(data);
        }, (error) => {
            console.log(error);
        })
}
getUserByPromise();
```

Promise 的方式虽然解决了 callback hell，但是这种方式充满了 Promise的 then() 方法，如果处理流程复杂的话，
整段代码将充满 then。语义化不明显，代码流程不能很好的表示执行流程。 



【Generator 方式】
```javascript
function* fetchUserByGenerator() {
    const user = yield fetchUser();
    return user;
}

const g = fetchUserByGenerator();
const result = g.next().value;
result.then((v) => {
    console.log(v);
}, (error) => {
    console.log(error);
})
```
Generator 的方式解决了 Promise 的一些问题，流程更加直观、语义化。
但是 Generator 的问题在于，函数的执行需要依靠执行器，每次都需要通过 g.next() 的方式去执行。 

【async 方式】
```javascript
async function getUserByAsync(){
     let user = await fetchUser();
     return user;
 }
getUserByAsync()
.then(v => console.log(v));
```
async 函数完美的解决了上面两种方式的问题。流程清晰，直观、语义明显。
操作异步流程就如同操作同步流程。同时 async 函数自带执行器，执行的时候无需手动加载。




