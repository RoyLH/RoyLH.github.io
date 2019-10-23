---
title: 自行实现一个Promise
tags:
  - JavaScript
  - Web前端
categories:
  - Web前端
  - JavaScript
abbrlink: 61442
date: 2018-10-10 17:38:59
---
![](https://pic2.zhimg.com/v2-09b5dffffe94bf7c2f9c564d90769101_b.jpg)

## 实现
```
// Promise.js

const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

function Promise(executor) {
  const self = this;
  self.state = PENDING;
  self.value = null;
  self.reason = null;
  self.onFulfilledCallbacks = [];
  self.onRejectedCallbacks = [];

  function resolve(value) {
    if (self.state === PENDING) {
      self.state = FULFILLED;
      self.value = value;

      self.onFulfilledCallbacks.forEach((fulfilledCallback) => {
        fulfilledCallback();
      });;
    }
  }

  function reject(reason) {
    if (self.state = PENDING) {
      self.state = REJECTED;
      self.reason = reason;
      
      self.onRejectedCallbacks.forEach((rejectedCallback) => {
        rejectedCallback();
      });
    }
  }

  try {
    executor(resolve, reject);
  } catch (reason) {
    reject(reason);
  }
}

function resolvePromise(promise, x, resolve, reject) {
  const self = this;
  let called = false; // called 防止多次调用

  // 如果 promise 和 x 指向同一对象，说明是循环引用的情况 以 TypeError 为 reason 拒绝执行 promise  
  // 循环引用例子
  // let promise = p.then(()=>{
  //   return promise;//Chaining cycle detected for promise，会报错循环引用。
  // });
  if (promise === x) {
    return reject(new TypeError('循环引用'));
  }
  
  // 判断 x 是否为对象或者函数，
      // 是：判断 x.then 是否是函数
          // 是：则说明x是一个promise， 则使 promise 接受 x 的状态
          // 否：x是对象或者函数，但没有thenable
      // 否：不为对象或者函数，说明x是一个普通值，以x为参数执行 promise（resolve和reject参数携带promise的作用域，方便在x状态变更后去更改promise的状态）
  if ((Object.prototype.toString.call(x) === '[object Object]' || Object.prototype.toString.call(x) === '[object Function]')) {
    // x是对象或者函数，因为typeof null 是 'object'，所以这里要排除null
    try {
      let then = x.then;

      if (typeof then === 'function') { // 说明是thenable函数，符合Promise要求
        then.call(x, (y) => { // 返回值y有可能还是一个Promise，也有可能是一个普通值，所以这里继续递归进行 resolvePromise
          // 别人的Promise的then方法可能设置了getter等，使用called防止多次调用then方法
          if (called) return ;
          called = true;
          resolvePromise(promise, y, resolve, reject);
        }, (reason) => {
          if (called) return;
          called = true;
          reject(reason);
        });
      } else { // x是对象或者函数，但没有thenable，直接返回
        if (called) return ;
        called = true;
        resolve(x);
      }
    } catch (reason) {
      if (called) return;
      called = true;
      reject(reason);
    } 
  } else { // x是普通值，直接返回
    resolve(x);
  }
}

Promise.prototype.then = function (onFulfilled, onRejected) {
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => { return value; };
  onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason; };

  const self = this;
  let promise = null;

  // then的返回值也应该是一个promise
  promise = new Promise((resolve, reject) => {
    if (self.state === PENDING) {
      // console.log('PENDING');
      self.onFulfilledCallbacks.push(() => {
        setTimeout(() => {
          // 所有的回调函数的执行 都要放在 try catch 中，因为使用者的回掉函数可能出错
          try {
            const x = onFulfilled(self.value);
            resolvePromise(promise, x, resolve, reject);
          } catch (reason) {
            reject(reason);
          }
        }, 0);
      });
      self.onRejectedCallbacks.push(() => {
        setTimeout(() => {
          try {
            const x = onRejected(self.reason);
            resolvePromise(promise, x, resolve, reject);
          } catch(reason) {
            reject(reason);
          }
        }, 0);
      })
    } else if (self.state === FULFILLED) {
      // console.log('FULFILLED');
      setTimeout(() => {
        try {
          const x = onFulfilled(self.value);
          resolvePromise(promise, x, resolve, reject);
        } catch (reason) {
          reject(reason);
        }
      }, 0);
    } else if (self.state === REJECTED) {
      // console.log('REJECTED');
      setTimeout(() => {
        try {
          const x = onRejected(self.reason);
          resolvePromise(promise, x, resolve, reject);
        } catch (reason) {
          console.log('err');
          reject(reason);
        }
      }, 0);
    }
  });

  return promise;
};

Promise.prototype.catch = function (onRejected) {
  return this.then(null, onRejected);
};

Promise.prototype.finally = function (fn) {
  return this.then((value) => {
    fn();
    return value;
  }, (reason) => {
    fn();
    throw reason;
  });
};

Promise.prototype.done = function () {
  return this.catch((reason) => {
    throw reason;
  });
};

Promise.all = function (promises) {
  return new Promise((resolve, reject) => {
    const result = [];

    promises.forEach((promise, index) => {
      promise.then((value) => {
        result[index] = value;

        if (result.length === promises.length) {
          resolve(result);
        }
      }, reject);
    });
  });
};

Promise.race = function (promises) {
  return new Promise((resolve, reject) => {
    promises.forEach((promise, index) => {
      promise.then((value) => {
        resolve(value);
      }, reject);
    });
  });
};

Promise.resolve = function (value) {
  let promise;

  promise = new Promise((resolve, reject) => {
    resolvePromise(promise, value, resolve, reject);
  });

  return promise;
};

Promise.reject = function (reason) {
  return new Promise((resolve, reject) => {
    reject(reason);
  });
};

Promise.defer = Promise.deferred = function () {
  const dfd = {};
  dfd.promise = new Promise((resolve, reject) => {
    dfd.resolve = resolve;
    dfd.reject = reject;
  });
  return dfd;
};

module.exports = Promise;

```

## 测试

```
// Test.js

const Promise = require('./Promise');

// 测试1 - 同步触发resolve
{
  new Promise((resolve, reject) => {
    resolve(1);
  }).then((data) => {
    console.log(data); // 1
  }, (error) => {
    console.log('error: ', error);
  });
}

// 测试2 - 异步触发resolve
{
  new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve(2);
    }, 1000);
  }).then((data) => {
    console.log(data); // 2
  }, (error) => {
    console.log('error: ', error);
  });
}

// 测试3 - 注册多个then函数
{
  var promise3 = new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(3);
    }, 2000);
  });

  promise3.then((data) => {
    console.log(data); // 3
    return data + 'c';
  }, (error) => {
    console.log('error: ', error);
  });

  promise3.then((data) => {
    console.log(data); // 3
  }, (error) => {
    console.log('error: ', error);
  });
}

// 测试4 - executor执行报错
{
  new Promise((resolve, reject) => {
    a.b = 10;
    resolve(4);
  }).then((data) => {
    console.log(data); // error:  ReferenceError: a is not defined
  }, (error) => {
    console.log('error: ', error);
  });
}

// 测试5 - 主动执行reject
{
  new Promise((resolve, reject) => {
    setTimeout(() => {
      reject('5 error');
    }, 1000);
  }).then((data) => {
    console.log(data);
  }, (error) => {
    console.log('error: ', error); // error:  5 error
  });
}

// 测试6 - 链式调用，then函数返回值是一个新Promise
{
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(6);
    }, 3000);
  }).then((data) => {
    console.log(data); // 6
    return new Promise((resolve, reject) => {
      resolve(new Promise((resolve, reject) => {
        resolve('new 6');
      }));
    })
  }, (error) => {
    console.log('error: ', error);
  }).then((data) => {
    console.log('then返回新Promise的数据', data); // then返回新Promise的数据 new 6
  }, (error) => {
    console.log('then返回新Promise的error', error);
  })
}

// 测试7 - 就算马上调用resolve，Promise也始终是异步调用，微队列micro task，也叫jobs，宏队列macro task，又叫tasks
{
  console.log('执行1');
  var promise7 = new Promise((resolve, reject) => {
    resolve(7);
  });
  promise7.then((data) => {
    console.log('执行3') // 执行3
    console.log(data); // 7
  }, (error) => {
    console.log('error: ', error);
  });
  promise7.then((data) => {
    console.log('执行4') // 执行4
    console.log(data); // 7
  }, (error) => {
    console.log('error: ', error);
  });
  console.log('执行2')
}

// 测试8 - Promise.all 和 Promise.race
{
  let promise8_1 = new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('8-1');
    }, 1000);
  });
  
  let promise8_2 = new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('8-2');
    }, 2000);
  });
  
  let promise8_3 = new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('8-3');
    }, 3000);
  });

  Promise.all([promise8_1, promise8_2, promise8_3])
  .then((data) => {
    console.log(data); // [ '8-1', '8-2', '8-3' ]
  }, (error) => {
    console.log('error: ', error);
  });

  Promise.race([promise8_1, promise8_2, promise8_3])
  .then((data) => {
    console.log(data); // 8-1
  }, (error) => {
    console.log('error: ', error);
  });
}

// 测试9 - Promise穿透
{
  new Promise((resolve, reject) => {
    resolve(9);
  })
  .then()
  .then((data) => {
    console.log(data); // 9
  }, (error) => {
    console.log('error: ', error);
  });
}

// 测试10 - catch()
{
  new Promise((resolve, reject) => {
    reject(10);
  }).then((data) => {
    console.log(data);
  }).catch((error) => {
    // 这里catch其实是一个新的Promise的then
    // 上一个Promise有默认的onRejected函数，做的是 throw reason，所以then返回的Promise就捕获到了错误
    // 执行reject(reason)，然后返回的这个Promise就进入错误处理函数，把第一个Promise的错误reason打印出来
    // 原理就是第一个Promise的错误reason被throw出来，被下一个Promise捕获到
    console.log('error: ', error); // error:  10
  });
}
```