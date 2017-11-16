### es6中class简介
```
function Point(x,y){
    this.x = x;
    this.y = y;
}
Point.prototype.toString = function(){
    return '(' + this.x + ', ' + this.y + ')';
};

var p = new Point(2,3);
console.log(p.toString());
```
跟传统的面向对象语言（C++，Java）差异很大，导致许多转过来学习JavaScript的人感到困惑。

ES6 提供了更接近传统语言的写法，引入了 类 这个概念，作为对象的模板。通过 **class** 关键字，可以定义类。
ES6中的 class 只是一个*语法糖*。这样做只是让 *对象原型* 的写法更加清晰，更加面向对象化。
```
class Point {
    constructor(x,y){
        this.x = x;
        this.y = y;
    }

    toString(){
        return '(' + this.x + ', ' + this.y + ')';
    }
}
var p = new Point(2,3);
console.log(p.toString());
```
类中有一个 **constructor方法，它就是构造方法**，而 this 关键字则指向实例对象。也就是说，ES5中的构造函数，，对应ES6中的类中的构造方法。

注意：类中的函数是 **不需要加function关键字的**，并且 **函数定义与函数定义之间不需要用逗号分隔，否则报错**。

---

```
class Point{
    //...
}
typeof Point // "function"
Point === Point.prototype.constructor // true
```
上面代码说明：**类的数据类型就是函数**，**类本身就指向构造函数**。
**构造函数的 prototype 属性**，在ES6的“类”上继续存在。事实上，类的所有方法都定义在 prototype 属性上面。
也可以理解为**prototype 对象的 constructor 属性，直接指向"类"的本身**
```
class Point {
  constructor() {
    // ...
  }
  toString() {
    // ...
  }
  toValue() {
    // ...
  }
}

// 等同于

Point.prototype = {
  constructor() {},
  toString() {},
  toValue() {},
};
```
**在类的实例上面调用方法，其实就是调用原型上的方法**。
```
var p = new Point(2,3);
console.log(p.constructor === Point.prototype.constructor);//true
```

由于类的方法都定义在 prototype 对象上面，所以类的新方法可以添加在 prototype 对象上面。**Object.assign 方法可以很方便地一次向类添加多个方法**。
```
Object.assign(Point.prototype,{
    sayHello(){console.log('hello es6')},
    sayGoodBye(){console.log('goodbye')}
})
var p1 = new Point(1,2);
p1.sayHello();
```

---

**类的内部所有定义的方法，都是不可枚举的**
```
Object.keys(Point.prototype)
// []
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString","sayHello","sayGoodBye"]
```
这一点与 ES5 的行为不一致。
```
var Point = function (x, y) {
  // ...
};

Point.prototype.toString = function() {
  // ...
};

Object.keys(Point.prototype)
// ["toString"]
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString"]
```
在ES5中，toString方法就是可枚举的。

---

ES6中类的属性名，可以采用表达式。
```
let methodName = 'getArea';
class Square {
    constructor() {
    }

    [methodName]() {
        console.log('123');
    }
}
var s = new Square();
s.getArea();
```
---


### 严格模式
类和模块内部，默认就是严格模式，所以不需要使用 **use strict** 指定运行模式。
考虑到未来所有的代码，其实都是运行在模块之中的，所以 ES6 实际上把整个语言升级到了严格模式。

---


### constructor方法
它是类的默认方法，使用 new 命令生成对象实例时，自动调用该方法，一个类必须有 constructor 方法，如果没有显示定义，一个无参的 constructor 方法都被默认添加。这倒是和传统的面向对象语言一样。

constructor 方法默认返回实例对象，即 this，我们完全可以返回另外一个对象。
```
class Foo {
    constructor(){
        return Object.create(null);
    }
}
console.log(new Foo() instanceof Foo);//false
```

**类必须使用 new 调用**，否则会报错。这是它跟传统的构造函数的一个主要区别。传统的构造函数不使用 new 也可以执行。

---


### 类的实例对象
与 ES5 一样，实例的属性除非显示定义在其本身（即定义在 this 对象上），否则都是定义在原型上。
与 ES5 一样，类的所有实例共享一个原型对象。
```
var p1 = new Point(2,3);
var p2 = new Point(3,2);

p1.__proto__ === p2.__proto__
//true
```
p1和p2都是Point的实例，它们的原型都是 Point.prototype所以\__proto__属性是相等。
注意：\__proto__并不是语言本身的特性，这是各大厂商实现时添加的私有属性，虽然很多现代浏览器的JS引擎中都提供了这个私有属性，但是依旧不建议在生产中使用该属性，避免对环境产生依赖。可以使用 **Object.getPrototypeOf** 方法来获取实例对象的原型，然后再来为原型添加方法/属性。
```
class Square {
    constructor() {
    }
}
var s = new Square();

let sProto = Object.getPrototypeOf(s);
sProto.sayhello = function(){
    console.log("234");
}
s = new Square();
s.sayhello();
```

---


### Class 表达式
与函数一致，类也可以使用表达式的形式定义（类的本身就是函数）。
```
const MyClass = class Me {
    getclassName(){
        return Me.name;
    }
};

let inst = new MyClass();
console.log(inst.getclassName());//Me
console.log(Me.name);// Me is not defined
```
需要注意的是：**类的名字是MyClass而不是Me**，Me只在Class的内部代码可用，值当前类。
如果类的内部没有用到的话，可以省略 Me，也就是可以写成下面的形式。
```
const MyClass = class {};
```
采用 Class表达式，可以写出立即执行的 Class。
```
let person = new class {
    constructor(name) {
        this.name = name;
    }
    
    sayName() {
        console.log(this.name);
    }
}('张三');
console.log(person.sayName());
```
person是一个立即执行的类的实例，但是上面的例子貌似有点问题。

---


## 不存在变量提升
类不存在变量提升，与 ES5 完全不同。
所以类必须先定义，然后在进行实例化，子类也必须定义在父类后面。
```
new Foo(); // ReferenceError
class Foo {}
```

---


### 私有方法
私有方法是常见需求，但是 ES6 并不提供，只能通过变通方法模拟实现。
一种做法是在命名上加以区别，但是这只是一种心理安慰，在类的外部照样可以调用。

另一种方法就是索性将私有方法移出模块，因为模块内部的所有方法都是对外可见的。
```
class Widget {
    foo(baz){
        bar.call(this,baz);
    }
}

function bar(baz){
    return this.snaf = baz;
}
```
foo 是类中的共有方法，内部调用了 bar.call(this,baz)。这使得 bar 实际上成为了当前模块的私有方法。

还有一个方法是利用 Symbol 值的唯一性，将私有方法的名字命名为一个 Symbol 值。
```
const bar = Symbol('bar');
const snaf = Symbol('snaf');

export default class myClass {
    //共有方法
    foo(baz){
        this[bar](baz);
    }

    //私有方法
    [bar](baz){
        return this[snaf] = baz;
    }
}
```
上面代码中，bar 和 baz 都是 Symbol 值，导致第三方无法获取到它们，因此*达到了私有方法和私有属性的效果*。

---


### 私有属性
与私有方法一样，ES6 不支持私有属性。目前，有一个提案，为 class 加了私有属性。办法是在属性名之前，使用 *#* 表示。
```
class Point {
    #x;
    constructor(x=0){
        #x = +x; // 写成 this.#x 亦可。
    }

    get x() {return #x}
    set x(value) {#x = value}
}
```
目前来说，只是一个提案，之规定了私有属性的写法。

---


### this指向
类的方法内部如果含有 this，它默认指向类的实例。
```
class Logger {
    printName(name = 'there') {
        this.print(`Hello ${name}`);
    }

    print(text) {
        console.log(text);
    }
}

const logger = new Logger();
const { printName } = logger;
printName(); // TypeError: Cannot read property 'print' of undefined
```
printName 方法中的 this，默认指向 Logger 类的实例。但是如果将 printName 单独提取出来使用，this 会指向该方法运行时所在的环境，因为找不到 print 方法而报错。
这是因为，**this 是在函数运行时确定的**

一种比较简单的解决办法是：在构造函数中绑定 this 上下文。
```
class Logger {
  constructor() {
    this.printName = this.printName.bind(this);
  }
}
```

还有一种是使用箭头函数。
```
class Logger {
  constructor() {
    this.printName = (name = 'there') => {
      this.print(`Hello ${name}`);
    };
  }
}
```
还有就是使用 Proxy。

---

### name属性
本质上，ES6的类还是函数，所以函数的许多特性都被 class 继承，包括 name 属性。
```
class Point { 
}
Point.name;// "Point"
```

---


### class 的取值函数和存储函数(getter/setter)
与 ES5 一样，在类的内部可以使用 get 和 set 关键字，对某个属性设置存储函数和取值函数，拦截该属性的存取行为。
```
class MyClassOne {
    constructor(){}

    get prop(){
        return 'getter'
    }
    set prop(value){
        console.log('setter:' + value);
    }
}
let myClassOne = new MyClassOne();
inst.prop = 123;
inst.prop;
```
上述代码中，prop 属性有对应的getter函数和setter函数，因此赋值和读取行为都被自定义了。

---


### class的静态方法
类相当于实例的原型，所有在类中定义的方法，都会被实例继承。如果**在一个方法前，加上 static 关键字**，就表示该方法不会被实例继承，而是直接通过类来调用，者就成为"静态方法"。
```
class MyClassTwo {
    static classMethodA(){
        return 'hello'
    }
}
console.log(MyClassTwo.classMethodA()); // 'hello'

var myClassTwo = new MyClassTwo();
// TypeError: foo.classMethodA is not a function
myClassTwo.classMethodA();
```

如果静态方法中包含 this 关键字，这个 this 值的是类，而不是实例。并且**静态方法可以与非静态方法重名**。
父类的静态方法，可以被子类继承。
```
class MyClassThree extends MyClassTwo {
}
MyClassThree.classMethodA();// "hello"
```

---

### class的静态属性和实例属性
静态属性指的是 class 本身的属性，即 **Class.propName**，而不是定义在实例对象（this）上的属性。
```
class Foo {}
Foo.prop = 1;
Foo.prop;// 1
```
为 Foo 类定义一个静态属性 prop。

目前。只有这种写法可行，因为 ES6 明确规定，class内部只有静态方法，没有静态属性。
```
// 以下两种写法都无效
class Foo {
    prop:2
    stati prop:2
}
```

---


### new.target属性
new 是从构造函数生成实例对象的命令。ES6为new命令引入了一个 new.target属性，该属性一般用在构造函数之中，返回呢我命令作用于的那个构造函数。如果构造函数不是通过 new 命令调用的，new.target 会返回 undefined。因此*这个属性可以用来确定构造函数是怎么调用的*。
```
function MyPerson(name){
    if(new.target !== undefined){
        this.name = name;
    }
    else {
        throw new Error("必须使用new命令生成实例");
    }

    //另一种写法
    if(new.target === MyPerson){
        this.name = name;
    }
    else {
        throw new Error("必须使用new命令生成实例");
    }
}
```
上面的代码确保构造函数只能通过 new 命令调用。

Class 内部调用 new.target，返回当前 Class
```
class Rectangle {
  constructor(length, width) {
    console.log(new.target === Rectangle);
    this.length = length;
    this.width = width;
  }
}

var obj = new Rectangle(3, 4); // 输出 true
```

注意：**子类继承父类时，new.target 会返回子类**
```
class Rectangle {
  constructor(length, width) {
    console.log(new.target === Rectangle);
  }
}
class Square extends Rectangle {
  constructor(length) {
    super(length, length);
  }
}
var obj = new Square(3); // 输出 false
```

利用这个特点，可以写出不能独立使用，必须继承后才能使用的类
```
class Shape {
    constructor(){
        if(new.target === Shape){
            throw new Error('本类不允许实例化');
        }
    }
}

class Rectangle extends Shape {
    constructor(length,width){
        super();
    }
}
var x = new Shape();// 报错
var y = new Rectangle(); // 正确调用
```
上面的代码中，Shape类不能被实例化，只能用于继承
