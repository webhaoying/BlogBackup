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
function Human() {
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
function Family() {
  Human.call(this)
}
var male = new Family()
console.log(male.getName) //  this is name prop 人类, this is family prop father
console.log(male.getNameInHuman()) //  this is name prop 人类, this is family prop father
male.name = '男性'
male.family.father = 'sonFather'
var female = new Family()
console.log(male.getNameInHuman()) // this is name prop 男性, this is family prop sonFather
console.log(female.getNameInHuman()) // this is name prop 人类, this is family prop sonFather
```
```shell
 git diff --cached --name-only --diff-filter=ACM -- '*.jsx'  '*.js' '*.ts'  
```
