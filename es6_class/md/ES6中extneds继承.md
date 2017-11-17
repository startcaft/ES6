### extends继承简介
Class 可以通过 **extends** 关键字实现继承，这比 ES5 的通过修改原型链实现继承，要清晰和方便很多。
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
class ColorPoint extends Point {
    constructor(x,y,color){
        super();            // 调用父类的 constructor(x,y)
        this.color = color; 
    }
    toString() {
        return this.color + ' ' + super.toString();     //调用父类的 toString()
    }
}
```
子类 ColorPoint 继承了 Point 类的**所有属性和方法**。
注意：**子类必须在 constructor 方法中调用 supert 方法**，否则新建实例时会报错。这是因为*子类没有自己的this对象，而是继承父类的this对象，然后对其加工*

ES6 的继承机制是先创建父类的实例对象 this，然后再用子类的构造函数修改 this。

最后，父类的静态方法，也会被子类继承。
```
class A {
    static Hello(){
        console.log('hello');
    }
}
class B extends A {}
B.Hello();
```

---

### Object.getPrototypeOf()
Object.getPrototypeOf 方法可以用来从子类上获取父类。
```
console.log(Object.getPrototypeOf(ColorPoint) === Point); // true
```
因此，可以使用这个方法判断，一个类是否继承了另外一个类。

---


### super 关键字
注意：是**super关键字**，而不是 super() 方法。
super作为对象，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。
```
class C {
    p(){
        return 2;
    }
}
class D extends C {
    constructor(){
        super();
        console.log(super.p());
    }
}
let d = new D();    // 2
```
上述代码中，super对象指向 C.prototype，所以 super.p() 相当于 C.prototype.p();

需要注意的是：由于 super 指向父类的原型对象，所以在父类实例上的方法和属性，是无法通过 super 调用的。
```
class C {
    constructor(){
        this.hello = 3; // 父类实例属性
    }
    p(){
        return 2;
    }
}
class D extends C {
    constructor(){
        super();
        console.log(super.p());
    }

    get m() {
        return super.hello; // 无法方法父类实例上的属性和方法
    }
}
let d = new D();
console.log(d.m);   // undefined
```
总结：**super 只能获取定义在父类对象上的属性和方法**。


ES6规定，**通过 supert 调用父类的方法时，方法内部的 this 指向子类**
```
class One {
    constructor(){
        this.x = 1;
    }
    print(){
        console.log(this.x);
    }
}
class Two extends One {
    constructor() {
        super();
        this.x = 2;
    }
    m() {
    super.print();
    }
}
let two = new Two();
console.log(two.m());
```

---


### extends 的继承目标
extends 关键字后面可以跟很多类型的值。

1. 子类继承 Object 类
```
class A ectends Object{

}
A.__proto__ === Object; // true
A.prototype.__proto__ === Object.prototype; // true
```
A 其实就是构造函数 Object 的复制，A的实例就是Object的实例。


2. 不存在任何继承
```
class A {

}
A.__proto__ === Function.prototype // true
A.prototype.__proto__ === Object.prototype // true
```
A就是一个普通函数。


3. 继承null
```
class A extends null {

}
A.__proto__ === Function.prototype // true
A.prototype.__proto__ === undefined // true
```
A也是一个普通函数，所以直接继承 Function.prototype。但是，A调用后返回的对象不继承任何方法，所以它的\__proto__指向 Function.porptotype,
即实质上执行了下面的代码：
```
class C extends null {
  constructor() { return Object.create(null); }
}
```

---


### 原生构造函数的继承
原生构造函数是值语言内置的构造函数，通常用来生成数据结构。ECMAScript 的原生构造函数大致有下面这些：
* Boolean()
* Number()
* String()
* Array()
* Date()
* Function()
* RegExp()
* Error()
* Object()

ES6 允许继承原生构造函数定义子类，因为 ES6 是先创建父类的实例对象 this，然后再用子类的构造函数修饰 this，这使得父类的所有行为都可以继承。

---

### Mixin 模式的实现
Mixin 指的是多个对象合成一个新的对象，新对象具有各个组成成员的接口。它的最简单实现如下：
```
const a = {
  a: 'a'
};
const b = {
  b: 'b'
};
const c = {...a, ...b}; // {a: 'a', b: 'b'}
```

下面是一个更完备的实现：
```
function mix(...mixins) {
  class Mix {}

  for (let mixin of mixins) {
    copyProperties(Mix, mixin); // 拷贝实例属性
    copyProperties(Mix.prototype, mixin.prototype); // 拷贝原型属性
  }

  return Mix;
}

function copyProperties(target, source) {
  for (let key of Reflect.ownKeys(source)) {
    if ( key !== "constructor"
      && key !== "prototype"
      && key !== "name"
    ) {
      let desc = Object.getOwnPropertyDescriptor(source, key);
      Object.defineProperty(target, key, desc);
    }
  }
}
```
使用的时候，只要继承这个类即可。
```
class DistributedEdit extends mix(Loggable, Serializable) {
  // ...
}
```
