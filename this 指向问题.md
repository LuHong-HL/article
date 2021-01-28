# this 指向问题

### this是什么
this 是在运行时进行绑定的，并不是在编写时绑定，它的上下文取决于函数调用时的各种条件。this 的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式。当一个函数被调用时，会创建一个活动记录（有时候也称为执行上下文）。这个记录会包含函数在哪里被调用（调用栈）、函数的调用方法、传入的参数等信息。this 就是记录的其中一个属性，会在函数执行的过程中用到。即this是执行上下文的一个属性。

## this的绑定方式
- 默认绑定 
不带任何修饰的函数引用直接调用，this会绑定到全局作用域;用严格模式（strict mode），那么全局对象将无法使用默认绑定，因此 this 会绑定到 undefined

``` javascript 
    function foo() {
      console.log( this.a );
    }
    var a = 2;
    foo(); // 2
```

- 隐式绑定 
当函数引用有上下文对象时，隐式绑定规则会把函数调用中的 this 绑定到这个上下文对象

``` javascript 
    function foo() {
      console.log( this.a );
    }
    var obj = {
      a: 2,
      foo: foo
    };
    obj.foo(); // 2
```

- 隐式丢失 
一个最常见的 this 绑定问题就是被隐式绑定的函数会丢失绑定对象，也就是说它会应用默认绑定，从而把 this 绑定到全局对象或者 undefined 上，取决于是否是严格模式。

```javascript
    function foo() {
      console.log( this.a );
    }
    function doFoo(fn) {
      // fn 其实引用的是 foo
      fn(); // <-- 调用位置！
    }
    var obj = {
      a: 2,
      foo: foo
    };
    var a = "oops, global"; // a 是全局对象的属性
    doFoo( obj.foo ); // "oops, global"
```

- 显式绑定 
具体来说可以使用函数的 call() 、apply() 来显示绑定

``` javascript 
    function foo() {
      console.log( this.a );
    }
    var obj = {
      a:2
    };
      foo.call( obj ); // 2
```

- 硬绑定 
具体来说可以使用函数的 bind() 来硬绑定

``` javascript 
    function foo() {
      console.log( this.a );
    }
    var obj = {
      a:2
    };
    var bar = function() {
      foo.call( obj );
    };
    bar(); // 2
    setTimeout( bar, 100 ); // 2
    // 硬绑定的 bar 不可能再修改它的 this
    bar.call( window ); // 2
```

- new绑定
在 JavaScript 中，构造函数只是一些使用 new 操作符时被调用的函数。它们并不会属于某个类，也不会实例化一个类。实际上，它们甚至都不能说是一种特殊的函数类型，它们只是被 new 操作符调用的普通函数而已。

使用 new 来调用函数，或者说发生构造函数调用时，会自动执行下面的操作。
> 1. 创建（或者说构造）一个全新的对象。
> 2. 这个新对象会被执行 [[ 原型 ]] 连接。
> 3. 这个新对象会绑定到函数调用的 this。
> 4. 如果函数没有返回其他对象，那么 new 表达式中的函数调用会自动返回这个新对象。

``` javascript
    function foo(a) {
      this.a = a;
    }
    var bar = new foo(2);
    console.log( bar.a ); // 2
```

### 优先级 
根据优先级来判断函数在某个调用位置应用的是哪条规则。可以按照下面的顺序来进行判断：
> - 函数是否在 new 中调用（new 绑定）？如果是的话 this 绑定的是新创建的对象。`var bar = new foo()`
> - 函数是否通过 call、apply（显式绑定）或者硬绑定调用？如果是的话，this 绑定的是指定的对象。`var bar = foo.call(obj2)` 
> - 函数是否在某个上下文对象中调用（隐式绑定）？如果是的话，this 绑定的是那个上下文对象。`var bar = obj1.foo()` 
> - 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到 undefined，否则绑定到全局对象。`var bar = foo()` 

### 绑定的例外

- 被忽略的this 
如果你把 null 或者 undefined 作为 this 的绑定对象传入 call、apply 或者 bind，这些值
在调用时会被忽略，实际应用的是默认绑定规则：

``` javascript 
    function foo() {
      console.log( this.a );
    }
    var a = 2;
    foo.call( null ); // 2
```

### 软绑定 
之前我们已经看到过，硬绑定这种方式可以把 this 强制绑定到指定的对象（除了使用 new时），防止函数调用应用默认绑定规则。问题在于，硬绑定会大大降低函数的灵活性，使用硬绑定之后就无法使用隐式绑定或者显式绑定来修改 this。如果可以给默认绑定指定一个全局对象和 undefined 以外的值，那就可以实现和硬绑定相同的效果，同时保留隐式绑定或者显式绑定修改 this 的能力。可以通过一种被称为软绑定的方法来实现我们想要的效果：

``` javascript 
    if (!Function.prototype.softBind) {
        Function.prototype.softBind = function(obj) {
            var fn = this;
            // 捕获所有 curried 参数
            var curried = [].slice.call( arguments, 1 );
            var bound = function() {
            return fn.apply(
                (!this || this === (window || global)) ? obj : this,
                curried.concat.apply( curried, arguments )
            );
            };
            bound.prototype = Object.create( fn.prototype );
            return bound;
        };
    }

    function foo() {
        console.log("name: " + this.name);
    }
    var obj = { name: "obj" },
    obj2 = { name: "obj2" },
    obj3 = { name: "obj3" };
    var fooOBJ = foo.softBind( obj );
    fooOBJ(); // name: obj
    obj2.foo = foo.softBind(obj);
    obj2.foo(); // name: obj2 <---- 看！！！
    fooOBJ.call( obj3 ); // name: obj3 <---- 看！
    setTimeout( obj2.foo, 10 ); // name: obj <---- 应用了软绑定
```

### this词法（箭头函数）

ES6 中的箭头函数并不会使用四条标准的绑定规则，而是根据当前的词法作用域来决定this，具体来说，箭头函数会继承外层函数调用的 this 绑定（无论 this 绑定到什么）。这其实和 ES6 之前代码中的 self = this 机制一样。

- 看箭头函数的词法作用域：

``` javascript 
    function foo() {
      // 返回一个箭头函数
      return (a) => {
        //this 继承自 foo()
        console.log( this.a );
      };
    }
    var obj1 = {
      a:2
    };
    var obj2 = {
      a:3
    };
    var bar = foo.call( obj1 );
    bar.call( obj2 ); // 2, 不是 3 ！
```

- 箭头函数最常用于回调函数中，例如事件处理器或者定时器：

``` javascript 
    function foo() {
      setTimeout(() => {
        // 这里的 this 在此法上继承自 foo()
        console.log( this.a );
      },100);
    }
    var obj = {
      a:2
    };
    foo.call( obj ); // 2
```

- 在 ES6 之前我们就已经在使用一种几乎和箭头函数完全一样的模式

``` javascript 
    function foo() {
      var self = this; // lexical capture of this
      setTimeout( function(){
        console.log( self.a );
      }, 100 );
    }
    var obj = {
      a: 2
    };
    foo.call( obj ); // 2
```

### 引用
- 你不知道的 JavaScript 上卷 [美] KYLE SIMPSON 著 赵望野 梁杰译