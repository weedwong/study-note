# 深拷贝

因为对象是应用类型数据，所以在将对象赋值给某个变量的时候，该变量拿到的只是这个对象在内存中的引用地址。这样其他变量也引用了这个地址并且修改了该对象，就会造成数据混乱难以管理，程序也容易出现bug。

用循环实现的深拷贝，以防栈溢出
```js
  function cloneDeep(obj) {
    // 初始化一个空对象
    var root = {};
    // 模拟一个栈，并给一个初始数据
    var loop = [
      {
        parent: root, // 循环执行时拷贝数据的目标节点
        key: undefined, // 当前节点对应的key，根节点为undefined
        data: obj, // 循环执行时需要拷贝的数据
      }
    ]; 

    while(loop.length > 0) {
      // 取出上一轮存在栈中的数据
      var node = loop.pop();
      var parent = node.parent;
      var key = node.key;
      var data = node.data;

      var temp = parent;
      // 如果可以为undefined，说明此时正在拷贝的目标是根节点，
      // 否则在下一轮循环中将data拷贝到key对应的子节点
      if (typeof key !== 'undefined') {
        temp = parent[key] = {};
      }

      for(var k in data) {
        if (data.hasOwnProperty(k)) {
          if (typeof data[k] == 'object') {
            // 存储下一轮循环的数据
            loop.push({
              parent: temp,
              key: k,
              data:data[k]
            })
          } else {
            parent[k] = data[k];
          }
        }
      }
    }
  }
```

解决循环引用，循环引用本质上是把某个节点的引用赋值给改节点的子节点，利用上面的循环，把该引用缓存起来即可
```js
  function cloneDeep(obj) {
    // 缓存被引用的节点
    var refList = [];
    // 初始化一个空对象
    var root = {};
    // 模拟一个栈，并给一个初始数据
    var loop = [
      {
        parent: root, // 循环执行时拷贝数据的目标节点
        key: undefined, // 当前节点对应的key，根节点为undefined
        data: obj, // 循环执行时需要拷贝的数据
      }
    ]; 

    while(loop.length > 0) {
      // 取出上一轮存在栈中的数据
      var node = loop.pop();
      var parent = node.parent;
      var key = node.key;
      var data = node.data;

      var temp = parent;
      // 如果可以为undefined，说明此时正在拷贝的目标是根节点，
      // 否则在下一轮循环中将data拷贝到key对应的子节点
      if (typeof key !== 'undefined') {
        temp = parent[key] = {};
      }

      // 如果缓存列表中有当前节点，取出并建立引用关系
      var ref = hasCache(refList, data);
      if (ref) {
        parent[key] = ref.target;
        continue; // 跳过本次循环
      }

      // 如果缓存列表中没有当前节点的引用，保存当前节点到缓存列表中
      refList.push({
        source: data,
        target: temp
      })

      for(var k in data) {
        if (data.hasOwnProperty(k)) {
          if (typeof data[k] == 'object') {
            // 存储下一轮循环的数据
            loop.push({
              parent: temp,
              key: k,
              data:data[k]
            })
          } else {
            parent[k] = data[k];
          }
        }
      }
    }
  }

  function hasCache(list, data) {
    for(var i; i < list.length; i++;) {
      if (list[i].source === data) {
        return list[i]
      }
    }
    return null
  }
```

利用`JSON.stringify`和`JSON.parse`实现克隆, 此方法无法解决循环引用以及`RegExp`、`Date`等特殊类型的克隆
```js
  cloneObject: function (obj) {
    let objStr = JSON.stringify(obj, function(key, val) {
      if (typeof val === 'function') {
        return val + '';
      }
      return val;
    })
    return JSON.parse(objStr,function(k,v){
      if(v.indexOf&&v.indexOf('function')>-1){
        return eval("(function(){return "+v+" })()")
      }
      return v;
    });
  }
```

处理特殊类型

```js
function deepClone(target, hash = new WeakMap()) {
  // 判断数组类型
  if (Array.isArray(target)) return [...target];
  // 利用hash缓存复制过的对象，实现循环引用
  if (hash.get(target)) return hash.get(target);

  const types = [Date, RegExp, Map, Set, WeakMap, WeakSet];

  if (types.includes(target.constructor)) {return new target.constructor(target)};

  const allDesc = Object.getOwnPropertyDescriptor(target);

  // 处理继承关系
  const newObj = Object.create(target.getPrototypeOf, allDesc);
  // Reflect.ownKeys 可以拷贝不可枚举属性和符号类型
  for (const key of Reflect.ownKeys(target)) {
    const value = target[key];
    newObj[key] = (typeof value === 'object' && value !== null) ? deepClone(value, hash) : value;
  }
  
  hash.set(target, newObj);

  return newObj;
}
```