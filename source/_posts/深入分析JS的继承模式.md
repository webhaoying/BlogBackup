---
itle: 深入分析JS的继承模式
date: 2019-12-27 11:36:31
tags: 
        - konwledge
        - js 

categories: 
        - konwledge
        - js
---
# 深入分析JS的继承模式

[JS原型链与继承别再被问倒了]https://juejin.im/post/58f94c9bb123db411953691b (主要)

[从prototype与__proto__窥探JS继承之源]https://juejin.im/post/58f9d0290ce46300611ada65(主要)
  ````
      JavaScript是一门完全面向对象的语言，如果想要更好地使用JavaScript的面向对象系统，原型和原型链就是个绕不开的话题。 
      原型及其原型链通过学习借鉴，自己加以汇总总结于此。
      原型链本身就是一种继承，讨论一下本身的
      
  ````
### 原型链的问题
  原型链继承的时候，存在两个问题
  - 1、当原型链中存在引用类型的值的时候，由于该值被所有的实例继承，因此该值会被多有的实例共享，即‘一个实例改变了原型链中的引用类型值，则别的实例也就跟着变了’
  - 2、在创建实例（子类型）的时候，不能像构造函数（父类型）传递参数
  问题2显而易见，下边举例子说明问题1
  ```javascript
  function Human() {
    this.name = '人类'
    this.family = { 
      father:'father',
      mother:'mother',
    }
  }
  Human.prototype.getName = function() {
    console.log('this is name prop', this.name)
    console.log('this is family prop', this.family.father)
  }
  function Family() {}
  Family.prototype = new Human()
  var male = new Family()
  console.log(male.getName()) //  this is name prop 人类, this is family prop father
  male.name = '男性'
  male.family.father = 'sonFather'
  var female = new Family()
  console.log(male.getName()) // this is name prop 男性, this is family prop sonFather
  console.log(female.getName()) // this is name prop 人类, this is family prop sonFather

  ```
由上, 可以发现，我们利用Human构造函数创建了实例male，修改male实例中的name属性后，不会影响在male之后创建的实例female，但是修改实例的引用类型属性family的时候， 新创建的实例female的family属性值也会跟着变化。这个大多数场景并不是我们想要的结果，因为不可控。
因此，实际中很少单独使用原型链进行继承。
### 借用构造函数
为了解决上述的两个问题，我们可以使用一种叫做借用构造函数的技术，又叫经典继承
```
  基本思想： 在子类型构造函数内部调用父类型构造函数
```
举个例子
```javascript
function Human(age) {
    this.age = age || '18'
    this.name = '人类'
    this.family = { 
      father:'father',
      mother:'mother',
    }
    this.getNameInHuman = function() {
      console.log('this is name prop', this.name)
      console.log('this is family prop', this.family.father)
    }
  }
Human.prototype.getName = function() {
  console.log('this is name prop', this.name)
  console.log('this is family prop', this.family.father)
}
function Family(age) {
  Human.call(this, age)
}
var male = new Family()
console.log(male.getName) // undefined
console.log(male.getNameInHuman()) //  this is name prop 人类, this is family prop father
console.log(male.age) //  18
male.name = '男性'
male.family.father = 'sonFather'
var female = new Family(28)
console.log('this is male.age', male.age) // 18
console.log(male.getNameInHuman()) // this is name prop 男性, this is family prop sonFather
console.log(female.getNameInHuman()) // this is name prop 人类, this is family prop father
console.log(female.getName) // undefined
console.log('this is female.age', female.age) // 28
```
由上获得，解决了上述简单的单独使用原型链继承的两大问题：
+ 保证了原型链中引用类型值的独立，不在被多有实例共享，即不会一个实例改变，影响以后新创建的实例的属性
+ 子类型创建时候也能够向父类型传值 
```javascript
  var female = new Family(28) //  将age参数，通过创建子类型的实例的时候，传递过去············································································
```
随之出现的问题是：如果仅仅使用构造函数，那么无法避免构造函数模式存在的问题，
+ 所有的方法都在构造函数中定义，每个实例的创建，都会拷贝一份构造函数中的方法，作为实例自己的方法；这样的话，内存占用会变大，尤其是方法过多的时候，因此这里就没有了函数复用的意义，但是我们利用继承本身就是为了能够高效复用。
+ 构造函数中的方法，最终实际上都成了实例的方法，当需求改变的时候，要改动其中的一个方法的时候，之前所有的实例否不能即使作出更新。只有在更改了构造函数方法，之后新创建的实例才能访问新的方法。
+ 即便父级构造函数的原型对象中，已经定义了方法，但是在子级的构造函数的实例中，无法使用，因为只是借用了构造函数创造的实例，也就是借用构造函数实现继承的话，都需要在构造函数中声明想要复用的方法和属性，是没有办法沿着原型链进行继承。

因此单独的使用原型链继承、借用构造函数继承都有无法接受的缺点。最好的办法是两者结合在一起
### 组合继承
组合继承，指的是将原项链继承和借用构造函数继承的技术结合到一起，从而发挥两者之长的一种继承模式
```javascript
 基本的思路： 使用原型链实现对原型属性和方法的继承，通过借用构造函数来实现对视力属性的继承
```
这样节能在原型上实现函数的复用，又能保证每个实例都有自己的属性。举例：
```javascript
function Father(name) {
  //定义Father这个大类中共有的属性 name  color
  this.name = name 
  this.colors = ['red', 'blue', 'green']
}
Father.prototype.sayName = function() {
  console.log(this.name) //利用原型链继承，定义复用的方法
}
function Son(name, age) {
  this.age = age //  定义Son子类 私有的属性
  Father.call(this, name) //  借用父级构造函数，实现继承
}
Son.prototype = new Father()
Son.prototype.sayAge = function() {
  console.log(this, 11111)
  console.log(this.age)
}
var instance1 =  new Son('louis', 5)
instance1.colors.push('black')
console.log(instance1.colors)// ["red", "blue", "green", "black"]
instance1.sayName() //  louis
instance1.sayAge() // 5

var instance2 = new Son('zhai', 10)
console.log(instance2.colors) //  ["red", "blue", "green"]
instance2.sayName() // zhai
instance2.sayAge() // 10
```
