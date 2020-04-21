
# 对象数组相关

## 1. 对象迭代行为

  ```javascript
  var obj = { x: 1, y: 2, z: 3};
  [...obj]; // TypeError
  ```
  > 以下两个方式可以使上诉语句使用展开运算而不导致类型错误

  解析：展开语法和for-if语句遍历iterabl对象定义要遍历的数据。Array和Map是具有默认迭代行为的内置迭代器。对象不是可迭代的，但是可以通过使用iterable和iterator协议使它们可迭代。

  在Mozilla文档中，如果一个对象实现了@iterator方法，那么他就是可迭代的，这意味着这个对象（或者它的原型链上的一个对象）必须有一个带有@iterator键的属性，这个键可以通过常量Symbol.iterator获得。

  ```javascript
  //  方法一
  var obj = { x: 1, y: 2, z: 3};
  obj[Symbol.iterator] = function(){
    // iterator 是一个具有 next 方法的对象
    // 他的返回至少有一个是对象
    // 返回对象具有两个个属性： value、done
    const self = this;
    const keys = Object.keys(self);
    const len = keys.length;
    let pointer = 0;
    return {
      next() {
        const done = pointer >= len;
        const value = !done ? self[keys[pointer++]]: undefined;
        return {
          done,
          value
        }
      }
    }
  }
  [...obj]
  ```

  ```javascript
  //  方法二
  //  使用 generator 函数来定制对象的迭代行为
  var obj = { x: 1, y: 2, z: 3};
  obj[Symbol.iterator] = function*() {
    yield 1;
    yield 2;
    yield 3;
  }
  [...obj]
  ```

## 2. 数组去重

# 执行环境相关

## 1. 变量赋值行为

  ```javascript
  
  ```