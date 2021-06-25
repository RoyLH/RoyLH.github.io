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

'use strict';

const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

const isFunction = function (target) {
  return target && typeof target === 'function';
};

const isObject = function (target) {
  return target && typeof target === 'object';
};

class Promise {
  constructor (executor) {
    if (!this || this.constructor !== Promise) {
      throw new TypeError('Promise must be called with new');
    }

    if (!isFunction(executor)) {
      throw new TypeError(`Promise constructor's argument must be a function`);
    }

    this.state = PENDING;
    this.value = null;
    this.onFulfilledCallbacks = [];
    this.onRejectedCallbacks = [];

    const resolve = value => {
      resolutionProcedure(this, value);
    };
  
    const reject = error => {
      if (this.state === PENDING) {
        this.state = REJECTED;
        this.value = error;
  
        this.onRejectedCallbacks.forEach(callback => callback());
      }
    };

    const resolutionProcedure1 = (promise, target) => {
      if (promise === target) return reject(new TypeError('Promise can not resolved with it self')); // 循环引用

      if (target instanceof Promise) return target.then(resolve, reject);

      if (isObject(target) || isFunction(target)) {
        let called = false;
        try {
          let then = target.then;

          if (isFunction(then)) {
            then.call(
              target,
              value => {
                if (called) return;
                called = true;
                resolutionProcedure(promise, value);
              },
              error => {
                if (called) return;
                called = true;
                reject(error);
              }
            );
            return;
          }
        } catch (error) {
          if (called) return;
          called = true;
          reject(error);
        }
      }

      if (promise.state === PENDING) {
        promise.state = FULFILLED;
        promise.value = target;

        promise.onFulfilledCallbacks.forEach((callback) => callback());
      }
    };

    const resolutionProcedure = (promise, target) => {
      if (promise === target) return reject(new TypeError('Promise can not resolved with it self')); // 循环引用

      if (target instanceof Promise) return target.then(resolve, reject);
    
      if (isObject(target) || isFunction(target)) {
        let called = false;

        try {
          let then = target.then;
    
          if (isFunction(then)) {
            then.call(
              target, 
              value => {
                if (called) return;
                called = true;
                resolutionProcedure(promise, value);
              },
              error => {
                if (called) return;
                called = true;
                reject(error);
              }
            );
            return;
          }
        } catch (error) {
          if (called) return;
          called = true;
          reject(error);
        }
        
      }

      if (promise.state === PENDING) {
        promise.state = FULFILLED;
        promise.value = target;
        
        promise.onFulfilledCallbacks.forEach(callback => callback());
      }
    };
  
    try {
      executor(resolve, reject);
    } catch (error) {
      reject(error);
    }
  }

  then(onFulfilled, onRejected) {
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason; };
  
    const newPromise = new Promise((resolve, reject) => {
      const wrapOnFulfilledCallback = () => {
        setTimeout(() => {
          try {
            resolve(onFulfilled(this.value));
          } catch (error) {
            reject(error);
          }
        }, 0);
      };

      const wrapOnRejectedCallback = () => {
        setTimeout(() => {
          try {
            resolve(onRejected(this.value));
          } catch (error) {
            reject(error);
          }
        }, 0);
      };

      if (this.state === PENDING) {
        this.onFulfilledCallbacks.push(wrapOnFulfilledCallback);
        this.onRejectedCallbacks.push(wrapOnRejectedCallback);
      } else if (this.state === FULFILLED) {
        wrapOnFulfilledCallback();
      } else if (this.state === REJECTED) {
        wrapOnRejectedCallback();
      }
    });

    return newPromise;
  }

  catch(onRejected) {
    return this.then(null, onRejected);
  }
    
  finally(callback) {
    return this.then(
      value => {
        callback();
        return value;
      },
      error => {
        callback();
        throw error;
      }
    );
  }

  done() {
    return this.catch(error => {
      throw error;
    });
  }
  
  static resolve(value) {
    return value instanceof Promise
      ? value
      : new Promise(resolve => resolve(value));
  }
  
  static reject(reason) {
    return new Promise((resolve, reject) => reject(reason));
  }

  static race(promises) {
    return new Promise ((resolve, reject) => {
      promises.forEach(promise => Promise.resolve(promise).then(resolve, reject));
    });
  }
  
  static all(promises) {
    return new Promise((resolve, reject) => {
      let result = [];
      let resolveCount = 0;
      const len = promises.length;
      if (!len) {
        resolve([]);
      }

      for (let index = 0; index < len; index++) {
        Promise.resolve(promises[index]).then(
          (data) => {
            result[index] = data;
            if (++resolveCount === len) {
              resolve(result);
            }
          },
          (error) => {
            reject(error);
          }
        );
      }
    });
  }
  
  static allSettled(promises) {
    return new Promsie((resolve, reject) => {
      let result = [];
      let resolveCount = 0;
      const len = promises.length;
      if (!len) {
        resolve([]);
      }

      for (let index = 0; index < len; index++) {
        Promsie.resolve(promises[index]).then(
          data => {
            result[index] = {
              status: FULFILLED,
              value: data,
            };
            if (++resolveCount === len) {
              resolve(result);
            }
          },
          error => {
            result[index] = {
              status: REJECTED,
              value: error,
            };
            if (++resolveCount === len) {
              resolve(result);
            }
          }
        );
      }
    });
  }

  static defer() {
    const dfd = {};
  
    dfd.promise = new Promise((resolve, reject) => {
      dfd.resolve = resolve;
      dfd.reject = reject;
    });
    
    return dfd;
  }

  static deferred() {
    const dfd = {};
  
    dfd.promise = new Promise((resolve, reject) => {
      dfd.resolve = resolve;
      dfd.reject = reject;
    });
    
    return dfd;
  }
}

module.exports = Promise;

```

## 测试

```
// test.js

'use strict';

const Promise = require('./Promise');

// 测试1 - 同步触发resolve
{
  new Promise((resolve, reject) => {
    resolve(1);
  }).then(
    data => {
      console.log(data); // 1
    },
    error => {
      console.log('error: ', error);
    }
  );
}

// 测试2 - 异步触发resolve
{
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(2);
    }, 1000);
  }).then(
    data => {
      console.log(data); // 2
    }, 
    error => {
      console.log('error: ', error);
    }
  );
}

// 测试3 - 注册多个then函数
{
  var promise3 = new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(3);
    }, 2000);
  });

  promise3.then(
    data => {
      console.log(data); // 3
      return data + 'c';
    },
     error => {
      console.log('error: ', error);
    }
  );

  promise3.then(
    data => {
      console.log(data); // 3
    }, 
    error => {
      console.log('error: ', error);
    }
  );
}

// 测试4 - executor执行报错
{
  new Promise((resolve, reject) => {
    a.b = 10;
    resolve(4);
  }).then(
    data => {
      console.log(data); // error:  ReferenceError: a is not defined
    }, 
    error => {
      console.log('error: ', error);
    }
  );
}

// 测试5 - 主动执行reject
{
  new Promise((resolve, reject) => {
    setTimeout(() => {
      reject('5 error');
    }, 1000);
  }).then(
    data => {
      console.log(data);
    },
    error => {
      console.log('error: ', error); // error:  5 error
    }
  );
}

// 测试6 - 链式调用，then函数返回值是一个新Promise
{
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(6);
    }, 3000);
  }).then(
    data => {
      console.log(data); // 6
      return new Promise((resolve, reject) => {
        resolve(new Promise((resolve, reject) => {
          resolve('new 6');
        }));
      })
    }, 
    error => {
      console.log('error: ', error);
    }
  ).then(
    data => {
      console.log('then返回新Promise的数据', data); // then返回新Promise的数据 new 6
    }, 
    error => {
      console.log('then返回新Promise的error', error);
    }
  );
}

// 测试7 - 就算马上调用resolve，Promise也始终是异步调用，微队列micro task，也叫jobs，宏队列macro task，又叫tasks
{
  console.log('执行1');
  var promise7 = new Promise((resolve, reject) => {
    resolve(7);
  });
  promise7.then(
    data => {
      console.log('执行3') // 执行3
      console.log(data); // 7
    }, 
    error => {
      console.log('error: ', error);
    }
  );
  promise7.then(
    data => {
      console.log('执行4') // 执行4
      console.log(data); // 7
    }, 
    error => {
      console.log('error: ', error);
    }
  );
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
  .then(
    data => {
      console.log(data); // [ '8-1', '8-2', '8-3' ]
    }, 
    error => {
      console.log('error: ', error);
    }
  );

  Promise.race([promise8_1, promise8_2, promise8_3])
  .then(
    data => {
      console.log(data); // 8-1
    },
    error => {
      console.log('error: ', error);
    }
  );
}

// 测试9 - Promise穿透
{
  new Promise((resolve, reject) => {
    resolve(9);
  })
  .then()
  .then(
    data => {
      console.log(data); // 9
    }, 
    error => {
      console.log('error: ', error);
    }
  );
}

// 测试10 - catch()
{
  new Promise((resolve, reject) => {
    reject(10);
  }).then(data => {
    console.log(data);
  }).catch(error => {
    // 这里catch其实是一个新的Promise的then
    // 上一个Promise有默认的onRejected函数，做的是 throw reason，所以then返回的Promise就捕获到了错误
    // 执行reject(reason)，然后返回的这个Promise就进入错误处理函数，把第一个Promise的错误reason打印出来
    // 原理就是第一个Promise的错误reason被throw出来，被下一个Promise捕获到
    console.log('error: ', error); // error:  10
  });
}
```