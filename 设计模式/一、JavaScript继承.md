# JavaScript继承

## 基础前提

1. `__proto__`:
   - 对象特有
   - 指向上层(创建自己的那个构造函数)的原型对象(*prototype*)
   - 对象从prototype继承属性和方法
   - 原型也是对象，所以也有`__proto__`属性，指向上一层的原型对象，这种现象形成了原型链。访问实例的某个属性，如: obj.name,会先查询自身是否有该属性，如果没有则查找上一层的`__proto__`，这样一层一层往上找直到最顶层，若找不到属性则返回undefined
2. `prototype`:
   - 函数特有，指向专属的原型对象
   - 用于存储由构造函数实例化出来的所有实例共享的属性和方法
3. `constructor`:
   - 原型特有，指向专属的构造函数
   - 通过new调用构造函数创建实例时，实例的`__proto__` 自动指向constructor指向的原型对象(即prototype)

- 结论
  1. 原型链最顶层是Object.prototype, `Object.prototype.__proto__== null`
  2. 函数比较特殊，内部有`__proto__`和`prototype` 两个属性，当函数作为构造函数使用时，由其创建出来的实例共享prototype里面的属性和方法，当函数作为对象使用时(点语法，如: Object.create())，能访问到`__proto__` 里面的属性和方法



## 继承方式

1. 原型链继承

   ```js
   // 父级构造器
   function Father() {
     this.name = 'father'
     this.hobby = ['football', 'basketball']
   }
   
   // 父级prototype
   Father.prototype.intr = function () {
     console.log('父级的自我介绍方法', this.name)
   }
   
   // 子类构造器
   function Son() {}
   
   // 继承父类-关键点 : 让子类构造器的原型对象指向父类的实例 
   Son.prototype = new Father()
   ```

   解析

   1. 因为父类的实例继承了父类构造函数的属性和父类原型的方法， 让子类构造器的原型指向该实例，子类构造器创建出来的实例可通过原型链访问到父类原型对象的方法

   缺点

   1. 若父类的属性为引用类型，则所有子类实例共享同一份数据，会互相影响

      ```js
      const s1 = new Son()
      const s2 = new Son()
      s1.hobby.push('computer game')
      console.log(s1.hobby) // ['football', 'basketball', 'computer game']
      console.log(s2.hobby) // ['football', 'basketball', 'computer game']
      ```

   2. 无法给父类构造器传参

   3. 父类实例对象中的属性(继承父类构造函数的属性)无意义，全为undefined

2. 借用构造函数继承

   ```js
   // 父类构造器
   function Father(name) {
     this.name = name
   }
   
   // 父类prototype
   Father.prototype.intr = function () {
     console.log('父级的自我介绍方法', this.name)
   }
   
   // 子类构造器
   function Son(nam, age) {
     Father.call(this, name)
     this.age = age
   }
   ```

   解析

   1. 在子类构造器内通过call调用父类构造器，可以继承父类构造器里定义的属性

   缺点

   1. 无法继承父类的原型上的方法
   2. 若要给子类添加方法需要在子类构造器里面定义，这样子创建出来的实例都有各自的方法造成内存上不必要的浪费

3. 组合继承

   ```js
   // 父类构造器
   function Father(name) {
     this.name = name
   }
   
   // 父类prototype
   Father.prototype.intr = function () {
     console.log('父级的自我介绍方法', this.name)
   }
   
   // 子类构造器
   function Son(nam, age) {
     Father.call(this, name)
     this.age = age
   }
   Son.prototype = new Father()
   ```

   解析

   1. 这种模式避免了原型链和构造函数继承的缺陷，融合了他们的优点

   缺点

   1. 为了继承父类构造函数的属性和父类原型的方法，调用了两次父类的构造函数
   2. 子类的constructor属性指向父类构造函数，需要手动设置子类的constructor属性
   3. 当使用`const instance = new Son('dingding', 12)`的时候，会产生两组name属性，一组在Son实例上，一组在Son原型上，只不过实例上的屏蔽了原型上的。

4. 原型式继承

   ```js
   function inheritObject(o) {
     // 定义一个临时构造函数
     function F() {}
     // 制定临时构造函数的prototype
     F.prototype = o
     return o
   }
   ```

   解析

   1. 定义一个函数封装对原型式继承的方法，属于工厂模式
   
缺点
   
1. 若父类的上有引用类型的属性，多个子类共享这一份属性，会互相影响
   
ES5
   
1. **Object.create()方法**规范了原型式继承
   
   ```js
      const father = {
        name: 'father',
        hobby: ['football', 'basketball', 'computer game']
      }
      
      const son = Object.create(father)
      ```
   
5. 寄生式继承

   ```js
   function inheritObj(obj) {
     // 创建临时构造函数
     function F() {}
     F.prototype = obj
     return new F()
   }
   
   // 父类构造器
   function Father(name) {
     this.name = name
   }
   Father.prototype.intr = function () {
     console.log(this.name)
   }
   
   // 创建父类实例
   const father = new Father()
   
   // 给原型式继承再套一层函数传递参数
   function createSub(obj) {
     const son = inheritObj(obj)
     son.age = 12
     return son
   }
   
   const son = createSub(father)
   ```

   解析

   1. 给原型式继承套一层函数

   缺点

   1. 没有创建专属的构造函数(类型)
   2. 没有用到原型

6. 寄生组合式继承

   ```js
   function inheritObj(son, father) {
     // 创建son原型对象 该对象继承了father的原型对象
     const prototype = Object.create(father.prototype)
     // Object.create 返回一个对象 该对象没有constructor属性
     prototype.constructor = son
     // 指定son的原型对象
     son.prototype = son
   }
   
   // 父类构造器
   function Father(name) {
     this.name = name
   }
   Father.prototype.intr = function () {
     console.log(this.name)
   }
   
   // 子类构造器
   function Son(name, age) {
     // 继承父类构造函数的属性
     Father.call(this, name)
     this.age = age
   }
   
   inheritObject(Son, Father)
   ```

   ```js
   // 使用Object.create
   
   // 父类构造器
   function Father(name) {
     this.name = name
   }
   Father.prototype.intr = function () {
     console.log(this.name)
   }
   
   function Son(name, age) {
     Father.call(this, name)
   	this.age = age
   }
   
   // 继承
   Son.prototype = Object.create(Father.prototype)
   Son.prototype.constructor = Son
   
   // es6
   // Object.setPrototypeOf，可以直接创建关联，而且不用手动添加constructor属性
   // Object.setPrototypeOf(Son.prototype, Father.prototype) 
   ```

   