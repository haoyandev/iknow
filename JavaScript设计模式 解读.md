# JavaScript设计模式 解读

## 第一章 灵活的语言

**前提：项目开发新功能**

1. 将函数写在全局上

   - 分析：团队合作，定义同名函数容易产生互相覆盖现象，产生的bug不好发现

2. 用对象收编变量

   - 分析：把新功能的函数写在对象里面，成为对象中的方法，使用的时候通过点语法调用，**对象名.方法名**，一定程度上减少覆盖显现，一旦出现覆盖，则所有方法不可用

   - ```js
     const checkObject = {
     	checkName () {
         //验证姓名
       }
     }
     ```

3. 用函数收编变量

   - 分析：函数是一种特殊的对象，所以新功能的函数可以添加到函数对象中，其调用方式也是通过点语法调用，但是不能很好的复用，也就是说改对象类用new关键字创建新对象时，新创建的对象不能继承这些方法

   - ```js
     const checkObject = function () {}
     checkObject.checkName = function () {
       // 验证姓名
     }
     ```

4. 假对象

   - 分析：把新功能函数写在对象中，通过一个函数返回该对象，只要调用这个函数就能获得一个新的对象，可以通过点语法调用，但是因为每次调用函数返回一个新对象，创建出来的新对象和checkObject函数没有任何关系，并不是一个真正意义上类的创建

   - ```js
     const checkObject = function () {
       return {
         checkName () {
           // 验证姓名
         }
     	}
     }
     ```

5. 类

   - 分析：把新功能函数保存在构造函数中(类)，每次创建新对象时，新创建的对象就可以调用这些方法，但是每个实例身上都会有一套方法，所以内存花销会比较浪费

   - ```js
     const checkObject = function () {
       this.checkName = function () {
         // 验证姓名
     	}
     }
     ```

6. 通过原型

   - 把新功能函数写在类的原型对象中，节约了内存开销

   - ```js
     const checkObjct = function () {}
     checkObject.prototype.checkName = function () {
      // 验证姓名
     }
     ```

7. 函数的祖先

   - 把新功能函数写在Function的prototype中，这样每个函数实例都能调用这个新功能，缺点是污染了原生的Function对象

8. 在Function抽象出一个统一添加方法的功能方法

   - ```js
     Function.prototype.addMethod = function (name, fun) {
       this.prototype[name] = fun
     } 
     ```

   - 分析： 函数是一种特殊的对象，如果作为普通对象使用，其__ proto __属性指向Function的原型对象，当作为构造函数使用，通过new调用，其prototype属性是由其创建出来的实例所共享的原型对象



## 第二章 面向对象

## 继承

- 类式继承

  - 分析： 把子类的原型对象指向父类的实例，父类的实例对象继承了父类构造器里面的属性(前提是属性是写死的不是通过传参动态赋值)和父类原型对象的方法，由子类创建的实例可以通过原型链访问到父类的属性和方法

  - ```js
    function Father() {
      this.name = 'hhy'
    }
    Father.prototype.intr = function () {
      // 自我介绍
    }
    
    function Son() {}
    Son.prototype = new Father()
    ```

  - instanceof 是根据对象的prototype链判断的

  - 缺点： 如果父类的固定属性是引用类型，那么由子类创建的实例修改该属性会互相影响

- 构造函数式继承

  - 分析： 在子类构造函数中通过call方法调用父类的构造函数，可以继承父类构造函数中的属性，这样每个由子类构造函数创建出来的实例都拥有属于自己的属性，修改属性不影响其他实例

  - ```js
    function Father() {
      this.books = ['js', 'java']
    }
    Father.prototype.intr = function () {}
    
    function Son(id) {
      Father.call(this)
      this.id = id
    }
    ```

  - 无法继承父类原型中的方法，只能把方法写在构造函数中，浪费了内存

- 组合继承

  - 分析：在子类构造函数中调用父类的构造函数，把子类的原型对象指向父类的实例对象，由子类构造函数创建的实例既继承了父类构造函数的实例(而且属性是属于自己),还可以通过原型链掉用父类原型中的方法

  - ```js
    function Father() {
      this.books = ['js', 'java']
    }
    Father.prototype.intr = function () {}
    
    function Son () {
      Father.call(this)
    }
    Son.prototype = new Father()
    ```

  - 缺点：总共执行了两次父类构造函数

- 原型式继承

  - 分析： 借助原型对象，根据已有的对象创建一个新的对象，用户不必创建新的自定义对象类型，(不涉及构造函数的定义，纯原型上的继承)，类似类型继承，是对类式继承的一种封装，只不过子类构造函数作为一个过渡对象出现，Object.create()就是通过这种思想发展出来的

  - 实现：创建一个函数，函数内部

  - ```js
    function inherit(obj) {
      // 声明一个过渡构造函数
      function F () {}
      F.prototype = obj
      return new F()
    }
    ```

- 寄生式继承

  - 分析：对原型继承的二次封装

  - ```js
    const book = {
      books: ['js', 'java']
    }
    function createBook(obj) {
      const o = new inherit(obj)
      o.getName = function () {}
      return o
    }
    ```

- 寄生组合式继承

  - 分析： 

## 第三章 简单工厂模式

- 分析： 通过一个函数创建同一种类型的对象，函数内部封装了多个构造函数，调用者通过参数的不同会获得不同的实例对象

- ```js
  const Basketball = function () {
    this.intr = '篮球'
  }
  Basketball.prototype.play = function () {
    console.log('打篮球')
  }
  
  const sportFactory = function (type) {
    switch (type) {
      case 'basketball':
        return new Basketball()
    }
  }
  ```

- ```js
  const sportFactory = function (intr, type) {
    const o = {}
    o.intr = intr
    if (type === 'basketball') {
      o.pototype.show = function () {
        console.log('打篮球')
      }
    }
    return o
  }
  ```

## 第四章 工厂方法模式

- 分析： 在工厂函数中不直接保存构造函数，把构造函数存放在原型对象中，工厂函数里实现一个统一的接口，实现通过不同的参数去原型中找到不同的构造函数然后调用

- ```js
  var factory = function (type) {
    if (type instanceof factory) {
      
    }
  }
  ```

##  第五章 抽象工厂模式

- 分析：通过工厂函数实现对某一种类型的继承，一般抽象类保存在工厂函数的原型对象中，父类定义各种必须实现的方法，如果子类继承父类后就必须重写父类的方法(前提：该对象总是应该具备一些必要的方法，比如car类，对象必须有run方法)，不然调用方法的时候会报错

  - ```js
    const Car = function () {}
    Car.prototype.run = {
      return new Error()
    }
    
    const AbstractorFactory (subClass, superClass) {
      const fun = AbstractorFactory[superClass]
      if (typeof fun === 'function') {
        function F () {}
        F.prototype = fun.prototype
        subClass.prototype = new F()
    	}
    }
    
    const BWM
    
    ```

##  第六章 建造者模式

- 分析： 建造者模式是一种注重创建过程和细节的创建对象方式，把对象中的某个属性抽象出一个构造函数，通过构造函数根据不同情况返回不通知，然后通过组装，返回一整个对象

- ```js
  const Human = function (params) {
    this.skill = params && params.skill || '保密'
  }
  Human.prototype.getKill = function () {
    console.log(this.skill)
  }
  
  const Name = function (name) {
  	var that = this;
    (function (name, that) {
      if (name.indexOf(' ') > -1) {
        that.FirstName = name.slice(0, name.indexOf(' '))
        that.secondName = name.slice(name.indexOf(' '))
      }
    })(name, that)
  }
  
  const Person = function (name) {
    const p = new Human()
    p.name = new Name(name)
    return p
  }
  
  const p = new Person('haoyan huang')
  console.log(p)
  ```



## 第七章 原型模式

- 分析：原型对象是一个可以用来保存共享属性和方法的对象，通过继承，子类可以通过原型链继承到父类的属性和方法，一般把方法(相对于属性来说，创建函数的内存花销会比较大)，集中在原型对象中创建方法，因为子类是通过指向引用来继承父类的属性和方法，所以只要对父类的原型对象中进行扩展，子类都能使用到新的属性和方法



## 第八章 单例模式

- 分析：单列模式主要的用途有三种，1、作为命名空间使用，把方法或者属性写在一个对象中，调用的时候通过 对象名.方法() 或者 对象名.属性 ，这样暴露出全局就只有一个变量值，可以更好的管理自己的代码模块 ，2、惰性单例，是一个类只能创建一个实例 ，3、静态属性的实现，感觉像闭包



## 第九章 外观模式

- 分析：通常用于兼容不同浏览器问题，提供一个高级接口，高级接口中实现对底层接口的分装

- 优点： 封装子系统api，提供一个高级接口让用户调用，对于用户来说，使用起来比较简单，不需要考虑底层接口
- 缺点： 性能问题，每次调用都要检测功能是否可用

