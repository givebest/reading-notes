# 《JavaScript语言精粹（修订版）》

## 2. 语法 Grammar

### 语句 Statements

下面值被当做假 (falsy):

```javascript
false
null
undefined
'' // 空字符串
0 // 数字
NaN // 数字
```

### 表达式 Expressions

#### 运算符优先级

| 运算符                    | 描述        |
| ---------------------- | --------- |
| .[]()                  | 提取属性与调用函数 |
| delete new typeof +- ! | 一元运算符     |
| * / %                  | 乘法、除法、求余  |
| +-                     | 加法、连接，减法  |
| >= <= > <              | 不等式运算符    |
| === !==                | 等式运算符     |
| &&                     | 逻辑与       |
| \|\|                   | 逻辑或       |
| ?:                     | 三元        |

## 3. 对象 Objects

### 原型 Prototype

创建一个使用原对象作为其原型的新对象

```javascript
Object.beget = function (o) {
  var F = function () {};
  F.prototype = o;
  return new F;
}

if (typeof Object.beget !== 'function') {
  Object.create = function (o) {
    var F = function () {};
    F.prototype = o;
    return new F();
  };
}

var another_stooge = Object.create(stooge);
```

原型连接在更新时是不起作用，当对某个对象做出改变时，不会触及该对象的原型，原型连接只有在检索值的时候才被用到。

```javascript
var obj1 = {1:111, 2:222};
var obj2 = Object.beget(obj1);

obj2[1]; // 111 obj2 prototype 上的属性

obj2[1] = 'aaa';  // 这里 obj2 添加了属性 1 ，覆盖了通过 prototype 从 obj1 上继承的属性 1
obj2[1]; // 'aaa' 1 是 obj2自己的属性
obj1[1]; // 111
```

原型链中的任何属性都会产生值 ：`function`    

```javascript
typeof flight.toString // 'function'
typeof flight.constructor // 'function'
```

### 枚举 Enumeration

`for in` 语句可用来遍历一个对象中的所有属性名，包括自身和原型链的属性。常用的过滤器是 `hasOwnProperty` 方法，以及使用 `typeof` 来排除函数。

```javascript
var name;
for (name in another_stooge) {
  if (typeof another_stooge[name] !== 'function') {
    document.writeIn(name + ':' + another_stooge[name]);
  }
}
```
`for in` 枚举出的属性名出现的顺序是不确实的，若要确保属性以特定顺序出现，最好完全避免使用 `for in`，创建一个数组。

```javascript
var i,
  properties = [
    'first-name',
    'middle-name',
    'last-name',
    'profession'
  ];
for (i = 0; i < properties.length; i += 1) {
  console.log(properties[i] + ': ' + another_stooge[properties[i]]);
}
```

去掉这些不需要的属性使用 `hasOwnProperty` 方法，如果对象拥有独有的属性，它将返回 `true` ，它不会检查原型链。详见：[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty)

```javascript
obj['a'] = 'aaa';
obj['b'] = 'bbb';
for (const name in obj2) {
  console.log(name);
}
// 1 b 2 a

for (const name in obj2) {
  if (obj2.hasOwnProperty(name)) {
    console.log(name);
  }
}
// 1 b  
// 1 、a 是原型链上拥有的属性
```

### 删除 Delete

`delete` 运算符可以用来删除对象的属性。如果包含该属性就会被移除。它不会触及`原型链中的任何对象`。删除对象的属性可能会让来自原型链中的属性透现出来。

## 4. 函数 Functions

一般来说，所谓编程，就是将一组需求分解成一组函数与数据结构的技能。

### Function Objects

对象是”名/值“对的集合并拥有`一个连到原型对象`的隐藏连接。对象字面量产生的对象连接到 `Object.prototype`。函数对象连接到 `Function.prototype`（该原型对象本身连接到 `Object.prototype`）。

### 调用 Invocation

JavaScript 中一共有4种调用模式：方法调用模式、函数调用模式、构造器调用模式、apply调用模式。

#### 方法调用模式 The Method Invocation Pattern

当一个方法被调用时，this 被绑定到该对象。如果调用表达式包含一个提取属性的动作，那么它就是被当做一个方法来调用。

```javascript
var myObject = {
  value: 0,
  fn: function (v) {
    return this.value + v;
  }
}
myObject.fn(1); // 1
myObject.fn('a'); // '0a'
```

#### 函数调用模式  The Function Invocation Pattern

this 被绑定到全局对象。

```javascript
function fn1 (v) {
  console.log('fn1', this);
  return v;
}

function fn2 (v) {
  console.log('fn2', this);
  console.log(fn1(v));
}
fn2(1);
// fn2 Window
// fn1 Window
// 1

function add (a, b) {
  return a + b;
}

var myObject = {
  value: 1,
  double: function () {
    var that = this; // 解决 this
    
    var helper = function () {
      that.value = add(that.value, that.value);
    };
    
    helper(); // 以函数的形式调用 helper
  }
}

myObject.double();
myObject.value;  // 2
```

#### 构造函数调用模式 The Constructor Invocation Pattern

函数前面带上new来调用，那么背地里将会创建一个连接到该函数的prototype成员的新对象，同时this会被绑定到那个新对象上。    

```javascript
// 创建一个名为 Quo 的构造器函数。
var Quo = function (string) {
  this.status = string;
}

// 给 Quo 的所有实例提供一个名为 get_status 的公共方法
Quo.prototype.get_status = function () {
  return this.status;
}

// 构造一个 Quo 实例
ƒ () {
  return this.status;
}

// 创建一个名为 Quo 的构造器函数。
var Quo = function (string) {
  this.status = string;
}

// 给 Quo 的所有实例提供一个名为 get_status 的公共方法
Quo.prototype.get_status = function () {
  return this.status;
}

// 构造一个 Quo 实例
var myQuo = new Quo('myQuo');
console.log(myQuo.get_status()); // myQuo
```

当代码 `new Quo(...)` 执行时：

1.  一个新对象被创建。它继承自 `Quo.prototype`。
2.  使用指定的参数调用构造函数 `Quo` ，并将 `this` 绑定到新创建的对象。不传递参数情况下：`new Quo` 等同于 `new Quo()`。
3.  如果构造函数返回了一个对象，那么这个对象会取代整个 `new` 出来的结果。如果构造函数没有返回对象，那么 `new` 出来的结果为步骤1创建的对象。
> 详见：[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new)

1. 创建（或者说构造）一个全新的对象。
2. 这个新对象会被执行[[原型]]连接。
3. 这个新对象会绑定到函数调用的 this 。
4. 如果函数没有返回其他对象，那么 new 表达式中的函数调用会自动返回这个新对象。
> 详见：《你不知道的JavaScript（上）》 —— 2.2.4 new绑定


按照约定，构造函数以大写格式命名在变量里。

#### Apply 调用模式 The Apply Invocation Pattern

JavaScript 是函数式的面向对象编程语言，所以函数可以拥有方法。     
`apply` 方法让我们构建一个参数数组传递给调用函数。它也允许我们选择 `this` 的值。`apply` 方法接收两个参数，第1个是要绑定给 `this` 的值，第2个就是一个参数数组。

```javascript
var statusObject = {
  status: 'A-OK'
}

var Quo = function (string) {
  this.status = string;
}

Quo.prototype.get_status = function () {
  return this.status;
}

Quo.prototype.get_status.apply(statusObject);

// 'A-OK'
```

### 参数 Arguments

函数可以通过此参数访问所有它被调用时传递给它的参数列表，包括那些没有被分配给函数声明时定义的形式参数的多余参数。

```javascript
var sum = function () {
  var i, sum = 0;
  for (i = 0; i < arguments.length; i+=1) {
    sum += arguments[i];
  }
  return sum;
};

console.log(sum(1, 2, 3, 4, 5)); // 15
```

### 返回 Return

return 语句可用来使函数提前返回，当 return 被执行时，函数立即被返回而不再执行余下的语句。    
一个函数总会返回一个值，如果没有指定则返回 `undefined` 。    
> 所以直接给变量赋值一个没有返回的函数就是 `undefined` 。（我想的）

如果函数调用时在前面加了 `new` 前缀，且返回值不是一个对象，则返回 `this` （该新对象）。

```javascript
// 构造函数
function Fn (name) {
  this.name = name;
}

var fn = Fn('pair'); // 没有指定返回值则返回 undefined，所以 fn 也是 undefined。
console.log(fn); // undefined 

var newFn = new Fn('pair');  //
console.log(newFn); // Fn {name: "pair"}
```

### 异常 Exceptions

`throw` 语句中断函数执行，抛出一个 `exception` 对象，该对象包含一个用来识别异常类型的 `name` 属性和一个描述性的 `message` 属性。

```javascript
var add = function (a, b) {
  if (typeof a !== 'number' || typeof b !== 'number') {
    throw {
      name: 'TypeError',
      message: 'add needs numbers'
    }
  }
  return a + b;
}

function try_it () {
      // @see https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/try...catch
      try {
        sum(1, 3, 'aa');
      } catch (e) {
        console.log(e);
      } finally {
        console.log('执行完成了');
      }
}
try_it(); // {name: "TypeError", message: "sum needs number"} 

try {
  if (a>1) {
    console.log('success')
  }
} catch (e) {
  console.log(e);
} finally {
  console.log('完成');
}
// ReferenceError: a is not defined
// 完成
```

### 扩充类型的功能 Augmenting Types

JavaScript 允许给语言的基本类型扩充功能。    
JavaScript 原型继承的动态本质，新的方法立即被赋值到所胡的对象实例上，哪儿怕对象实例是在方法被增加之前就创建好的。

```javascript
// 符合条件时才增加方法
Function.prototype.method = function (name, func) {
  if (!this.prototype[name]) {
    this.prototype[name] = func;
  }
  return this;
};

String.method('trim', function () {
  return this.replace(/^\s+|\s+$/g, '')
});
' str '.trim();  // 'str'

Number.method('integer', function () {
  return Math[this < 0 ? 'ceil' : 'floor'](this);
});
(-10 / 3).integer();  // -3
```

### 递归 Recursion

递归函数就是会直接或间接地调用自身的一种函数，它把一个问题分解为一组相似的子问题，每一个都用一个寻常解去解决。    
`尾递归`优化，一个函数返回自身递归调用的结果，那么调用的过程会被替换为一个循环，可以显著提高速度。

```javascript
// 从某个指定节点开始，按 HTML 源码中的顺序访问该树的每个节点。
var walk_the_DOM = function walk (node, func) {
  func(node);
  node = node.firstChild;
  while (node) {
    walk(node, func);
    node = node.nextSibling;
  }
};

// 属性名称字符串和可选匹配值参数，调用 walk_the_DOM，传递一个用来查找节点属性名的函数作为参数
var getElementsByAttribute = function (att, value) {
  var results = [];
  
  walk_the_DOM(document.body, function (node) {
    var actual = node.nodeType === 1 && node.getAttribute(att);
    if (typeof actual === 'string' && (actual === value || typeof value !== 'string')) {
      results.push(node);
    }
  });
  return results;
}

getElementsByAttribute('href', 'javascript:;'); // []
```

### 作用域 Scope

作用域控制着变量与参数的可见性及生命周期。    
JavaScript 缺少块级作用域，建议在函数体的顶部声明函数中可能用到的所有变量。

### 闭包 Closure

函数可以访问它被创建时所处的上下文环境，这被称为闭包。

```javascript
// 设置 DOM 节点为黄色，然后渐变为白色
var fade = function (node) {
  var level = 1;
  var step = function () {
    var hex = level.toString(16);
    node.style.backgroundColor = '#FFFF' + hex + hex;
    if (level < 15) {
      level += 1;
      setTimeout(step, 1000);
    }
  };
  setTimeout(step, 1000);
};
fade(document.body);

// 点击节点，弹出节点序号
var add_the_handlers = function (nodes) {
  var helper = function (i) {
    return function (e) {
      alert(i);
    }
  }
  for (var i = 0; i < nodes.length; i++) {
    nodes[i].onclick = helper(i)
  }
};
```

### 回调 Callbacks

网络上的同步请求会导致客户端进入假死状态，更好的方式是发起异步请求。

```javascript
request = prepare_the_request();
send_request_asynchronously(request, function (response) {
  display(response);
});
```

### 模块 Module

模块是一个提供接口却隐藏状态与实现的函数或对象，可以使用函数和闭包构造模块。

```javascript
String.method('deentityify', function () {
  var entity = {
    quot: '"',
    It: '<',
    gt: '>'
  };
  
  return function () {
    return this.replace(/&([^&;]+);/g, function (a, b) {
      var r = entity[b];
      return typeof r === 'string' ? r : a;
    })
  };
});
' &lt;&quot;&gt;'.deentityify()); / / <">
```

### 级联 Cascade

设置、修改对象某个状态返回 `this` 就可以启用级联。`jQuery` 的链式就是这样。

### 柯里化 Curry

把多参数函数转换为一系列单参数函数并进行调用。    
柯里化允许我们把函数与传递给它的参数相结合，产生出一个新的函数。

```javascript
Function.method('curry', function () {
  var slice = Array.prototype.slice,
    args = slice.apply(arguments), // arguments 是类数组，没有 concat 方法。
    that = this;
  return function () {
    return that.apply(null, args.concat(slice.apply(arguments)));
  };
});

var add1 = add.curry(1);
add1(6); // 7
```

### 记忆  Memoization

函数可以将先前操作的结果记录在某个对象里，从而避免无谓的重复运算，这种优化被称为记忆（memoization）。

```javascript
var fibonacci = function () {
  var memo = [0,1];
  var fib = function (n) {
    var result = memo[n];
    if (typeof result !== 'number') {
      result = fib(n -1) + fib(n - 2);
      memo[n] = result;
    }
    return result;
  };
  return fib;
}();

var memoizer = function (memo, formula) {
  var recur = function (n) {
    var result = memo[n];
    if (typeof result !== 'number') {
      result = formula(recur, n);
      memo[n] = result;
    }
    return result;
  };
  return recur;
};
var fibonacci = memoizer([0,1], function (recur, n) {
  return recur(n -1) + recur(n - 2);
});
```

## 5. 继承 Inheritance

### 伪类 Pseudoclassical  

* JavaScript 不直接让对象从其他对象继承，反而插入了一个多余的间接层：通过构造函数产生对象。
* 函数对象被创建时，`Function` 构造器产生的函数对象会运行类似这样的代码：
    ```javascript
    this.prototype = {construtor: this}
    ```
* 每个函数都会得到一个 `prototype` 对象。

#### 构造器调用模式

```javascript
Function.method('new', function () {
  // 创建一个新对象，它继承自构造函数的原型对象。
  var that = Object.create(this.prototype);
  // 调用构造函数，绑定 `this` 到新对象上。
  var other = this.apply(that, arguments);
  // 如果它的返回值不是一个对象，就返回该新对象。
  return (typeof other === 'object' && other) || that;
});
```

#### 定义构造器并扩充它的原型    

```javascript
var Mammal = function (name) {
  this.name = name;
};

Mammal.prototype.get_name = function () {
  return this.name;
};

Mammal.prototype.says = function () {
  return this.saying || '';
};

var myMammal = new Mammal('Herb the Mammal');
var name = myMammal.get_name(); // Herb the Mammal

// 构造另一个伪类来继承 Mammal，通过定义它的 constructor 函数并替换它的 prototype 为一个 Mammal 的实例来实现。
var Cat = function (name) {
  this.name = name;
  this.saying = 'meow';
};

// 替换 Cat.prototype 为一个新的 Mammal 实例。
Cat.prototype = new Mammal();

// 扩充新原型对象，增加 purr 和 get_name 方法
Cat.prototype.purr = function (n) {
  var i, s = '';
  for (i = 0; i < n; i++) {
    if (s) {
      s+='-';
    }
    s+='r'
  }
  return s;
};
Cat.prototype.get_name = function () {
  return this.says() + 'this.name' + this.says();
};

var myCat = new Cat('Henrietta');
var says = myCat.says(); // 'meow'
var purr = myCat.purr(5); // 'r-r-r-r-r-r'
var name = myCat.get_name(); // 'meow Henrietta meow'

// 隐藏细节
Function.method('inherits', function (Parent) {
  this.prototype = new Parent();
  return this;
});
```

### 对象说明符 Object Specifiers

让构造器接受一个简单的`对象说明符`，对象里面的多个参数可以任意排序不影响功能。    
可以简单地把 JSON 对象传递给构造器，而它将返回一个构造完全的对象。

### 原型  Prototypal

基于原型的继承相比基于类的继承在概念上更为简单：一个新对象可以继承一个旧对象的属性。

#### Object.create

> Object.create() 方法会使用指定的原型对象及其属性去创建一个新的对象。

```javascript
// Polyfill
if (typeof Object.create !== "function") {
    Object.create = function (proto, propertiesObject) {
        if (typeof o !== 'object' && typeof o !== 'function') {
            throw new TypeError('Object prototype may only be an Object: ' + o);
        } else if (o === null) {
            throw new Error("This browser's implementation of Object.create is a shim and doesn't support 'null' as the first argument.");
        }

        if (typeof properties != 'undefined') throw new Error("This browser's implementation of Object.create is a shim and doesn't support a second argument.");
        
        function F() {}
        F.prototype = proto;
        
        return new F();
    };
}
```
更多：[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)



```javascript
// 用对象字面量去构造一个有用的对象
var myMammal = {
  name :'Herb the Mammal',
  get_name: function () {
    return this.name;
  },
  says: function () {
    return this.saying || '';
  }
};

var myCat = Object.create(myMammal);

var block = function () {
  // 记住当前的作用域。构造一个包含了当前作用域中所有对象的新作用域。
  
  var oldScope = scope;
  scope = Object.create(scope);
  
  // 传递左花括号作为参数调用 advance
  advance('{');
  
  // 使用新的作用域进行解析
  parse(scope);
  
  // 传递右花括号作为参数调用 advance 并抛弃新作用域，恢复原来老的作域。
  advance('}');
  scope = oldScope;
};
```

### 函数化 Functional

> 应用模式模式

1. 创建一个新对象。
2. 有选择的定义私有实例变量和方法。
3. 给这个新对象扩充方法。这些方法拥有特权去访问参数以及第2步通过 var 定义的变量。
4. 返回那个新对象。

```javascript
var constructor = function (spec, my) {
  var that,  // 其它私有实例变量
    my = my || {};  // 把共享的变量和函数添加到 my 中
    
    that = {}; // 新对象
    
    // 添加给 that 的特权就方法
    return that;
};

var mammal = function (spec) {
  var that = {};
  that.get_name = function () {
    return spec.name;
  };
  that.says = function () {
    return spec.saying || '';
  };
  
  return that;
}
var myMammal = mammal({name: 'Herb'});

var cat = function (spec) {
  spec.saying = spec.saying || 'meow';
  var that = mammal(spec);
  that.purr = function () {
    console.log('purr')
  };

  that.get_name = function () {
    return that.says() + '' + spec.name + '' + that.says();
  }

  return that;
};

var myCat = cat({name: '在猫咪'});
myCat.get_name(); // meow在猫咪meow
myCat.says(); // meow
myCat.purr(); // purr

// 给 Object 添加 method
Object.prototype.method = function (name, func) {
  this.prototype[name] = func;
  return this;
}

// 处理父类方法的方法
Object.method('superior', function (name) {
  var that = this,
  method = that[name];
  
  return function () {
    return method.apply(that, arguments);
  };
});

// 在 coolcat 上试验一下
var coolcat = function (spec) {
  var that = cat(spec),
    super_get_name = that.superior('get_name');
  that.get_name = function (n) {
    return 'like ' + super_get_name() + ' baby';
  };
  return that;
};
myCoolCat.get_name(); // 'like meowBixmeow baby'
```

### 部件 Parts

我们可以从一套部件中把对象组装出来。    

```javascript
// 构造一个给任何对象添加简单事件处理特性的函数
var eventuality = function (that) {
  var registry = {};
  
  that.fire = function (event) {
    // 在一个对象上触发一个事件，该事件可以是一个包含事件名称的字符串
    // 或者是一个拥有包含事件名称的 type 属性的对象。
    // 通过 on 方法注册的事件处理程序中匹配事件名称的函数将被调用
    var array,
      func,
      handler,
      i,
      type = typeof event === 'string' ? event : event.type;
      
    // 如果这个事件存在一组事件处理程序，那么就遍历它们并按顺序依次执行。
    if (registry.hasOwnProperty(type)) {
      array = registry[type];
      for (i = 0; i < array.length; i++) {
        handler = array[i];
        
        // 每个处理程序包含一个方法和一组可选的参数。
        // 如果该方法是一个字符串形式的名字，那么寻找到该函数。
        func = handler.method;
        if (typeof func === 'string') {
          func = this[func];
        }
        
        // 调用一个处理程序，如果该条目包含参数，那么传递它们过去。否则，传递该 事件对象。
        func.apply(this, handler.parameters || [event]);
      }
    }
    return this;
  };
  
  that.on = function (type, method, parameters) {
    // 注册一个事件。构造一条处理程序条目。将它插入到处理程序数组中
    // 如果这种类型的事件还不存在，就构造一个。
    var handler = {
      method: method,
      parameters: parameters
    };
    if (registry.hasOwnProperty(type)) {
      registry[type].push(handler);
    } else {
      registry[type] = [handler];
    }
    return this;
  };
  return that;
};

eventuality(that);
```

## 8. 方法 Methods

### Array

#### array.sort(comparefn)

##### 数字排序

```javascript
var n = [4, 8, 15, 16, 23, 42];
n.sort(function (a, b) {
  return a- b;
});
// [4, 8, 15, 16, 23, 42]
```

##### 数字、字符串混合排序

```javascript
m = ['aa', 'bb', 'a', 14, 82, 215, 16, 23, 42];
m.sort(function (a, b) {
  if (a === b) {
    return 0;
  }
  
  if (typeof a === typeof b) {
    return a < b ? -1 : 1;
  }
  
  return typeof a < typeof b ? -1 : 1;
  // typeof 'a' < typeof 1 false
});
// [14, 16, 23, 42, 82, 215, "a", "aa", "bb"]
```

##### 对象数组排序

> sort 方法是不稳定的（会改变相等值的相对位置）

```javascript
// by 函数接受一个成员名字符串作为参数
// 并返回一个可以用来对包含该成员的对象数组进行排序的比较函数
var by = function (name, minor) {
  return function (o, p) {
    var a, b;
    if (o && p && typeof o === 'object' && typeof p === 'object') {
      a = o[name];
      b = p[name];
      
      if (a === b) {
        return typeof minor === 'function' ? minor(o, p) : 0;
      }
      
      if (typeof a === typeof b) {
        return a < b ? -1 : 1;
      }
      
      return typeof a < typeof b ? -1 : 1;
    } else {
      throw {
        name: 'Error',
        message: 'Expected an object when sorting by' + name
      };
    }
  };
};
s.sort(by('last', by('first')));
```

### Function

#### function.apply(thisArg, argArray)

```javascript
Function.method('bind', function (that) {
  var method = this,
    slice = Array.prototype.slice,
    args = slice.apply(arguments, [1]);
    
  return function () {
    return method.apply(that, args.concat(slice.apply(arguments, [0])));
  };
});

var x = function () {
  return this.value;
}.bind({value: 666});
// x(); // 666
```

### Number

#### number.toFixed(fractionDigits)

* `toFixed` 把 number 转换为一个十进制数形式的字符串。
* 可选参数 `fractionDigits` 控制小数点后的数字位数，值必须在0~20，默认为0。

```javascript
Math.PI.toFixed(2);
//'3.14'
```

#### number.toPrecision(precision)

* 转换为一个十进制数形式的字符串。
* 可选参数控制数字的精度，值必须在0~21。

```javascript
Math.PI.toPrecision(2);
// '3.1'
```

#### number.toString(radix)

* 转换为字符串。
* 可选参数 radix 控制基数，值必须在 2~36，默认值为10。
* `number.toString()` 可以更简单地写为 `String(number)`。

```javascript
Math.PI.toString();  // "11.001001000011111101101010100010001000010110100011"
String(Math.PI); // "3.141592653589793"
```

### Object

#### object.hasOwnProperty(name)

* object 包含名为 `name` 的属性，返回 `true` 。
* 原型链中的同名属性不会被检查。
* `name` 为 `hasOwnProperty` 时不起作用，返回 `false`。

```javascript
var a = {member: true};
var b = Object.create(a);

a.hasOwnProperty('member); // true
b.hasOwnProperty('member); // false
b.member; // true
```

### RegExp

#### regexp.test(string)

* test 方法是使用正则表达式的最简单和最快的方法。 
* `regexp` 匹配 `string` 返回 `true`；否则返回 `false` 。
* 不要对这个方法使用 g 标识。

```javascript
var b = /&.+;/.test('frank &amp; beans'); // true

// test 可以像这样实现
RegExp.method('test', function (string) {
  return this.exec(string) !== null;
});
```

### String

#### string.charAt(pos)

* `charAt` 方法返回在 `string` 中 `pos` 位置处的字符。
* 如果 pos 小于0或大于等于字符串的长度 `string.length`，它会返回空字符串。

```javascript
var name = 'Curly';
name.charAt(0); // 'C'
```

#### string.charCodeAt(pos)

* 同 `charAt` ，不过返回的是以整数形式表示的 `string` 中的 pos 位置处的字符的字符码位。
* 如果 pos 小于0或大于等于字符串的长度则返回 `NaN` 。

```javascript
name.charCodeAt(1); // 117
```

#### string.indexOf(searchString, position)

* `indexOf` 方法在 `string` 内查找字符串 `searchString` ，如果找到，返回第1个匹配字符的位置，否则返回 -1 。
* `position` 可设置从 `string` 的某个指定的位置开始查找。

#### string.lastIndexOf(searchString, position)

* 与 `indexOf` 类似，不过它是从末尾开始查找的。

#### string.match(regexp)

* match 方法让字符串和一个正则表达式进行匹配。
* `g` 标识来决定如何进行匹配，有 `g` 生成一个包含所有匹配的数组。

#### string.replace(searchValue, replaceValue)

* 参数 `searchValue` 可以是一个字符串或正则表达式对象。

* `replaceValue` 可以是字符串或函数。

  | 美元符号序列  | 替换对象    |
  | ------- | ------- |
  | $$      | $       |
  | $&      | 整个匹配文本  |
  | $number | 分组捕获的文本 |
  | $`      | 匹配之前的文本 |
  | $'      | 匹配之后的文本 |

```javascript
'(555)666-1212'.replace(/\((\d{3})\)/g, '$1-')
// "555-666-1212"
```

#### string.slice(start, end)

* 复制 `string` 的一部分构造一个新的字符串。
* start 参数为负，它将与 string.length 相加。
* end 参数可选，且默认值是 string.length 。
* end 参数为负，它将与 string.length 相加。
* end 参数等于你要取的最后一个字符的位置值加上1。

```javascript
var text = '123456789';
text.slice(0) // "123456789"
text.slice(8) // "9"
text.slice(0, 3) // "123"
```

#### string.split(separator, limit)

* 分割成片段来创建一个字符串数组。
* 可选参数 limit 可以限制被分割的片段数量。
* separator 参数可以是一个字符串或正则表达式。
* 如果 separator 是一个空字符，会返回一个单字符的数组。

```javascript
var digits = '0123456789';
digits.split('', 5); // ["0", "1", "2", "3", "4"]
digits.split(''); // ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"]

'192.168.1.1'.split('.'); //["192", "168", "1", "1"]
```






