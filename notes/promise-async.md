### Promise
+ 基本用法
```
let p = new Promise((resolve,reject) => {
let status = 200 // 请求状态
if (status === 200) {
resolve('success')
}else {
reject('fail')
}
});
p.then(result => {
    	console.log(result);//success
}).catch(err => {
console.log(err) // fail
})
```
+ p.then() 链式调用 p.catch() 错误捕捉
+ Promise.resolve() 返回一个 成功状态的 Promise 对象
    - 示例
    1. 传一个普通对象(数字、字符串也可，但是意义不大)
    ```
    let p = Promise.resolve({a: 1, b: 2})
    p.then(result => console.log(result))
    ```
    2. 如果传Promise对象，直接返回
    ```
    let p1 = new Promise((resolve, reject) => {
        resolve('OK')
    })
    let p2 = Promise.resolve(p1)
    console.log(p1 === p2) // true
    ```
+ Promise.reject()  返回一个拒绝状态的 Promise 对象. 用法同 Promise.resolve()

+ Promise.all([p1, p2, p3...])
- 说明： 用于将多个 Promise 实例，包装成一个新的 Promise 实例，若都为成功状态则返回 一个数组
- 示例
    ```
    let p1 = Promise.resolve(123);
	let p2 = Promise.resolve('hello');
	let p3 = Promise.resolve('success');
	Promise.all([p1,p2,p3]).then(result => {
   	 console.log(result); // 按实例的 顺序返回数组 [123, 'hello', 'success']
	})
    ```
    成功之后就是数组类型，当所有状态都是成功状态才返回 数组，若有一个实例的状态返回 reject,就返回reject状态值
    ```
    let p1 = Promise.resolve(123);
    let p2 = Promise.resolve('hello');
    let p3 = Promise.reject('error 500');
    Promise.all([p1,p2,p3]).then(result => {
            console.log(result); // 不执行
    }).catch(result => {
            console.log(result); // 执行 返回 error 500
    });
    ```

+ Promise.race([p1, p2...])
- 说明： 也是用于将多个 Promise 实例，包装成一个新的 Promise 实例，只返回一个值.和all同样接受多个对象，不同的是，race()接受的对象中，哪个对象返回的快就返回哪个对象，就如race直译的赛跑这样。如果对象其中有reject状态的，必须catch捕捉到，如果返回的够快，就返回这个状态。race最终返回的只有一个值。
- 示例
    用sleep来模仿浏览器的AJAX请求
    ```
	function sleep(wait) {
    		return new Promise((res,rej) => {
        		setTimeout(() => {
           			 res(wait);
       			 },wait);
    		});
	}

	let p1 = sleep(500);
	let p0 = sleep(2000);

	Promise.race([p1,p0]).then(result => {
	       console.log(result); // 500
	});

	let p2 = new Promise((resolve,reject) => {
    		setTimeout(()=>{
        		reject('error');
    		},1000);
	});

	Promise.race([p0,p2]).then(result => {
        	console.log(result); // 若 p0 更快，则返回p0状态的值 2000
	}).catch(result => {
	        console.log(result); // error 
	});
    ```
+ 注意： 在 Promise 链的尾部用 .catch() 接着

###  Async-Await
+ 说明： async和await在干什么，async用于申明一个function是异步的，而await可以认为是async wait的简写，等待一个异步方法执行完成。
+ 规则
    - async 表示这是一个async函数，await只能用在这个函数里面
    - await 表示在这里等待Promise返回结果后，再继续往下执行
    - await 后面一般跟着一个 Promise 对象
+ 示例
- 基本使用
```
async function demo() {
    let result = await Promise.resolve(123);
    console.log(result);
}
demo();
```
- 模拟接口使用
```
function sleep(wait) {
    return new Promise((res,rej) => {
        setTimeout(() => {
            res(wait);
        },wait);
    });
}
async function demo() {
    let result01 = await sleep(100);
    //上一个await执行之后才会执行下一句
    let result02 = await sleep(result01 + 100);
    let result03 = await sleep(result02 + 100);
    // console.log(result03);
    return result03;
}
demo().then(result => {
    console.log(result); // 300
});
```
- 错误处理   如果是reject状态 尽量使用try-catch（也可在每个Promise 实例内部进行 .catch() 来捕捉）
```
let p = new Promise((resolve,reject) => {
    setTimeout(() => {
        resolve('OK')
        // reject('error')
    },1000)
})
async function test () {
    try{
        let result = await p
        console.log(result) // OK
    }catch(err) {
        console.log(err) // error
    }   
}
test()
```

