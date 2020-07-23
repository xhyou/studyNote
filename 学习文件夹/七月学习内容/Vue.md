# 背景

**什么是vue**

> vue就是一个js库，并且无依赖别的js库，跟jquery差不多。vue采用mvvm模型，支持数据的双向绑定

##  模板语法

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>02_模板语法</title>
</head>
<body>
<!--
1. 模板的理解:
  动态的html页面
  包含了一些JS语法代码
    大括号表达式
    指令(以v-开头的自定义标签属性)
2. 双大括号表达式
  语法: {{exp}} 或 {{{exp}}}
  功能: 向页面输出数据
  可以调用对象的方法
3. 指令一: 强制数据绑定
  功能: 指定变化的属性值
  完整写法:
    v-bind:xxx='yyy'  //yyy会作为表达式解析执行
  简洁写法:
    :xxx='yyy'
4. 指令二: 绑定事件监听
  功能: 绑定指定事件名的回调函数
  完整写法:
    v-on:click='xxx'
  简洁写法:
    @click='xxx'
-->

<div id="app">
  <h2>1. 双大括号表达式</h2>
  <p>{{content}}</p>
  <p>{{content.toUpperCase()}}</p>

  <h2>2. 指令一: 强制数据绑定</h2>
  <a href="url">访问指定站点</a><br>
  <a v-bind:href="url">访问指定站点2</a><br>
  <a :href="url">访问指定站点2</a><br>

  <h2>3. 指令二: 绑定事件监听</h2>
  <button v-on:click="test">点我</button>
  <button @click="test">点我</button>

</div>


<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
  new Vue({
    el: '#app',
    data: {
      content: 'NBA I Love This Game',
      url: 'http://www.baidu.com'
    },
    methods: {
      test () {
        alert('好啊!!!')
      }
    }
  })
</script>
</body>
</html>
```

## 计算属性和监听

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>03_计算属性和监视</title>
</head>
<body>
<!--
1. 计算属性
  在computed属性对象中定义计算属性的方法
  在页面中使用{{方法名}}来显示计算的结果
2. 监视属性:
  通过通过vm对象的$watch()或watch配置来监视指定的属性
  当属性变化时, 回调函数自动调用, 在函数内部进行计算
3. 计算属性高级:
  通过getter/setter实现对属性数据的显示和监视
  计算属性存在缓存, 多次读取只执行一次getter计算
-->
<div id="demo">
  姓: <input type="text" placeholder="First Name" v-model="firstName"><br>
  名: <input type="text" placeholder="Last Name" v-model="lastName"><br>
  <!--fullName1是根据fistName和lastName计算产生-->
  姓名1(单向): <input type="text" placeholder="Full Name1" v-model="fullName1"><br>
  姓名2(单向): <input type="text" placeholder="Full Name2" v-model="fullName2"><br>
  姓名3(双向): <input type="text" placeholder="Full Name3" v-model="fullName3"><br>

  <p>{{fullName1}}</p>
  <p>{{fullName1}}</p>
</div>

<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
  const vm = new Vue({
    el: '#demo',
    data: {
      firstName: 'A',
      lastName: 'B',
       fullName2: 'A-B'
    },

    // 计算属性配置: 值为对象
    computed: {
      fullName1 () { // 属性的get()
        console.log('fullName1()', this)
        return this.firstName + '-' + this.lastName
      },

      fullName3: {
        // 当获取当前属性值时自动调用, 将返回值(根据相关的其它属性数据)作为属性值
        get () {
          console.log('fullName3 get()')
          return this.firstName + '-' + this.lastName
        },
        // 当属性值发生了改变时自动调用, 监视当前属性值变化, 同步更新相关的其它属性值
        set (value) {// fullName3的最新value值  A-B23
          console.log('fullName3 set()', value)
          // 更新firstName和lastName
          const names = value.split('-')
          this.firstName = names[0]
          this.lastName = names[1]
        }
      }
    },

    watch: {
      // 配置监视firstName
      firstName: function (value) { // 相当于属性的set
        console.log('watch firstName', value)
        // 更新fullName2
        this.fullName2 = value + '-' + this.lastName
      }
    }
  })

  // 监视lastName
  vm.$watch('lastName', function (value) {
    console.log('$watch lastName', value)
    // 更新fullName2
    this.fullName2 = this.firstName + '-' + value
  })

</script>
</body>
</html>
```

## class与style绑定

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>04_class与style绑定</title>
  <style>
    .classA {
      color: red;
    }
    .classB {
      background: blue;
    }
    .classC {
      font-size: 20px;
    }
  </style>
</head>
<body>

<!--
1. 理解
  在应用界面中, 某个(些)元素的样式是变化的
  class/style绑定就是专门用来实现动态样式效果的技术
2. class绑定: :class='xxx'
  xxx是字符串
  xxx是对象
  xxx是数组
3. style绑定
  :style="{ color: activeColor, fontSize: fontSize + 'px' }"
  其中activeColor/fontSize是data属性
-->

<div id="demo">
  <h2>1. class绑定: :class='xxx'</h2>
  <p :class="myClass">xxx是字符串</p>
  <p :class="{classA: hasClassA, classB: hasClassB}">xxx是对象</p>
  <p :class="['classA', 'classB']">xxx是数组</p>

  <h2>2. style绑定</h2>
  <p :style="{color:activeColor, fontSize}">:style="{ color: activeColor, fontSize: fontSize + 'px' }"</p>

  <button @click="update">更新</button>

</div>

<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
  new Vue({
    el: '#demo',
    data: {
      myClass: 'classA',
      hasClassA: true,
      hasClassB: false,
      activeColor: 'red',
      fontSize: '20px'
    },

    methods: {
      update () {
        this.myClass = 'classB'
        this.hasClassA = !this.hasClassA
        this.hasClassB = !this.hasClassB
        this.activeColor = 'yellow'
        this.fontSize = '30px'
      }
    }
  })
</script>
</body>
</html>
```

## 条件渲染

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>08_条件渲染</title>
</head>
<body>
<!--
1. 条件渲染指令
  v-if
  v-else
  v-show
2. 比较v-if与v-show
  如果需要频繁切换 v-show 较好
-->

<div id="demo">
  <p v-if="ok">表白成功</p>
  <p v-else>表白失败</p>

  <hr>
  <p v-show="ok">求婚成功</p>
  <p v-show="!ok">求婚失败</p>

  <button @click="ok=!ok">切换</button>
</div>

<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
  new Vue({
    el: '#demo',
    data: {
      ok: true,
    }
  })
</script>
</body>
</html>
```

## 列表渲染

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>06_列表渲染</title>
</head>
<body>

<!--
1. 列表显示
  数组: v-for / index
  对象: v-for / key
2. 列表的更新显示
  删除item
  替换item
-->

<div id="demo">
  <h2>测试: v-for 遍历数组</h2>
  <ul>
    <li v-for="(p, index) in persons" :key="index">
      {{index}}--{{p.name}}--{{p.age}}
      --<button @click="deleteP(index)">删除</button>
      --<button @click="updateP(index, {name:'Cat', age: 16})">更新</button>
    </li>
  </ul>
  <button @click="addP({name: 'xfzhang', age: 18})">添加</button>

  <h2>测试: v-for 遍历对象</h2>

  <ul>
    <li v-for="(item, key) in persons[1]" :key="key">{{key}}={{item}}</li>
  </ul>

</div>

<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
  new Vue({
    el: '#demo',
    data: {
      persons: [
        {name: 'Tom', age:18},
        {name: 'Jack', age:17},
        {name: 'Bob', age:19},
        {name: 'Mary', age:16}
      ]
    },

    methods: {
      deleteP (index) {
        this.persons.splice(index, 1) // 调用了不是原生数组的splice(), 而是一个变异(重写)方法
              // 1. 调用原生的数组的对应方法
              // 2. 更新界面
      },

      updateP (index, newP) {
        console.log('updateP', index, newP)
        // this.persons[index] = newP  // vue根本就不知道
        this.persons.splice(index, 1, newP)
        // this.persons = []
      },

      addP (newP) {
        this.persons.push(newP)
      }
    }
  })
</script>
</body>
</html>
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>06_列表渲染_过滤与排序</title>
</head>
<body>
<!--
1. 列表过滤
2. 列表排序
-->

<div id="demo">
  <input type="text" v-model="searchName">
  <ul>
    <li v-for="(p, index) in filterPersons" :key="index">
      {{index}}--{{p.name}}--{{p.age}}
    </li>
  </ul>
  <div>
    <button @click="setOrderType(2)">年龄升序</button>
    <button @click="setOrderType(1)">年龄降序</button>
    <button @click="setOrderType(0)">原本顺序</button>
  </div>
</div>

<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
  new Vue({
    el: '#demo',
    data: {
      searchName: '',
      orderType: 0, // 0代表不排序, 1代表降序, 2代表升序
      persons: [
        {name: 'Tom', age:18},
        {name: 'Jack', age:17},
        {name: 'Bob', age:19},
        {name: 'Mary', age:16}
      ]
    },

    computed: {
      filterPersons () {
//        debugger
        // 取出相关数据
        const {searchName, persons, orderType} = this
        let arr = [...persons]
        // 过滤数组
        if(searchName.trim()) {
          arr = persons.filter(p => p.name.indexOf(searchName)!==-1)
        }
        // 排序
        if(orderType) {
          arr.sort(function (p1, p2) {
            if(orderType===1) { // 降序
              return p2.age-p1.age
            } else { // 升序
              return p1.age-p2.age
            }

          })
        }
        return arr
      }
    },

    methods: {
      setOrderType (orderType) {
        this.orderType = orderType
      }
    }
  })
</script>
</body>
</html>
```

## 事件处理

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>07_事件处理</title>
</head>
<body>
<!--
1. 绑定监听:
  v-on:xxx="fun"
  @xxx="fun"
  @xxx="fun(参数)"
  默认事件形参: event
  隐含属性对象: $event
2. 事件修饰符:
  .prevent : 阻止事件的默认行为 event.preventDefault()
  .stop : 停止事件冒泡 event.stopPropagation()
3. 按键修饰符
  .keycode : 操作的是某个keycode值的健
  .enter : 操作的是enter键
-->

<div id="example">

  <h2>1. 绑定监听</h2>
  <button @click="test1">test1</button>
  <button @click="test2('abc')">test2</button>
  <button @click="test3('abcd', $event)">test3</button>

  <h2>2. 事件修饰符</h2>
  <a href="http://www.baidu.com" @click.prevent="test4">百度一下</a>
  <div style="width: 200px;height: 200px;background: red" @click="test5">
    <div style="width: 100px;height: 100px;background: blue" @click.stop="test6"></div>
  </div>

  <h2>3. 按键修饰符</h2>
  <input type="text" @keyup.13="test7">
  <input type="text" @keyup.enter="test7">

</div>

<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
  new Vue({
    el: '#example',
    data: {

    },
    methods: {
      test1(event) {
        alert(event.target.innerHTML)
      },
      test2 (msg) {
        alert(msg)
      },
      test3 (msg, event) {
        alert(msg+'---'+event.target.textContent)
      },

      test4 () {
        alert('点击了链接')
      },

      test5 () {
        alert('out')
      },
      test6 () {
        alert('inner')
      },

      test7 (event) {
        console.log(event.keyCode)
        alert(event.target.value)
      }
    }
  })
</script>
</body>
</html>
```

## 表单数据绑定

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>08_表单输入绑定</title>
</head>
<body>
<!--
1. 使用v-model(双向数据绑定)自动收集数据
  text/textarea
  checkbox
  radio
  select
-->
<div id="demo">
  <form action="/xxx" @submit.prevent="handleSubmit">
    <span>用户名: </span>
    <input type="text" v-model="username"><br>

    <span>密码: </span>
    <input type="password" v-model="pwd"><br>

    <span>性别: </span>
    <input type="radio" id="female" value="女" v-model="sex">
    <label for="female">女</label>
    <input type="radio" id="male" value="男" v-model="sex">
    <label for="male">男</label><br>

    <span>爱好: </span>
    <input type="checkbox" id="basket" value="basket" v-model="likes">
    <label for="basket">篮球</label>
    <input type="checkbox" id="foot" value="foot" v-model="likes">
    <label for="foot">足球</label>
    <input type="checkbox" id="pingpang" value="pingpang" v-model="likes">
    <label for="pingpang">乒乓</label><br>

    <span>城市: </span>
    <select v-model="cityId">
      <option value="">未选择</option>
      <option :value="city.id" v-for="(city, index) in allCitys" :key="city.id">{{city.name}}</option>
    </select><br>
    <span>介绍: </span>
    <textarea rows="10" v-model="info"></textarea><br><br>

    <input type="submit" value="注册">
  </form>
</div>

<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
  new Vue({
    el: '#demo',
    data: {
      username: '',
      pwd: '',
      sex: '男',
      likes: ['foot'],
      allCitys: [{id: 1, name: 'BJ'}, {id: 2, name: 'SS'}, {id: 3, name: 'SZ'}],
      cityId: '2',
      info: ''
    },
    methods: {
      handleSubmit () {
        console.log(this.username, this.pwd, this.sex, this.likes, this.cityId, this.info)
        alert('提交注册的ajax请求')
      }
    }
  })
</script>
</body>
</html>
```

## vue的生命周期

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>09_Vue实例_生命周期</title>
</head>
<body>
<!--
1. vue对象的生命周期
  1). 初始化显示
    * beforeCreate()
    * created()
    * beforeMount()
    * mounted()
  2). 更新状态
    * beforeUpdate()
    * updated()
  3). 销毁vue实例: vm.$destory()
    * beforeDestory()
    * destoryed()
2. 常用的生命周期方法
  created()/mounted(): 发送ajax请求, 启动定时器等异步任务
  beforeDestory(): 做收尾工作, 如: 清除定时器
-->

<div id="test">
  <button @click="destroyVue">destory vue</button>
  <p v-if="isShow">初学Vue入门,希望自己保持本心</p>
</div>

<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
  new Vue({
    el: '#test',
    data: {
      isShow: true
    },

    mounted () {
      // 执行异步任务
      this.intervalId = setInterval(() => {
        console.log('-----')
        this.isShow = !this.isShow
      }, 1000)
    },

    beforeDestroy() {
      console.log('beforeDestroy()')
      // 执行收尾的工作
      clearInterval(this.intervalId)
    },

    methods: {
      destroyVue () {
        this.$destroy()
      }
    }
  })


</script>
</body>
</html>
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>09_Vue实例_生命周期</title>
</head>
<body>
<!--
1. vue对象的生命周期
  1). 初始化显示
    * beforeCreate()
    * created()
    * beforeMount()
    * mounted()
  2). 更新状态
    * beforeUpdate()
    * updated()
  3). 销毁vue实例: vm.$destory()
    * beforeDestory()
    * destoryed()
2. 常用的生命周期方法
  created()/mounted(): 发送ajax请求, 启动定时器等异步任务
  beforeDestory(): 做收尾工作, 如: 清除定时器
-->

<div id="test">
  <button @click="destroyVue">destory vue</button>
  <p v-if="isShow">初学Vue入门,希望自己保持本心</p>
</div>

<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
  new Vue({
    el: '#test',
    data: {
      isShow: true
    },

    beforeCreate() {
      console.log('beforeCreate()')
    },

    created() {
      console.log('created()')
    },

    beforeMount() {
      console.log('beforeMount()')
    },

    mounted () {
      console.log('mounted()')
      // 执行异步任务
      this.intervalId = setInterval(() => {
        console.log('-----')
        this.isShow = !this.isShow
      }, 1000)
    },


    beforeUpdate() {
      console.log('beforeUpdate()')
    },
    updated () {
      console.log('updated()')
    },


    beforeDestroy() {
      console.log('beforeDestroy()')
      // 执行收尾的工作
      clearInterval(this.intervalId)
    },

    destroyed() {
      console.log('destroyed()')
    },

    methods: {
      destroyVue () {
        this.$destroy()
      }
    }
  })


</script>
</body>
</html>
```

## 动画效果

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>10_过渡&动画1</title>
  <style>
    /*指定过渡样式*/
    .xxx-enter-active, .xxx-leave-active {
      transition: opacity 1s
    }
    /*指定隐藏时的样式*/
    .xxx-enter, .xxx-leave-to {
      opacity: 0;
    }


    .move-enter-active {
      transition: all 1s
    }

    .move-leave-active {
      transition: all 3s
    }

    .move-enter, .move-leave-to {
      opacity: 0;
      transform: translateX(20px)
    }
  </style>
</head>
<body>
<!--
1. vue动画的理解
  操作css的trasition或animation
  vue会给目标元素添加/移除特定的class
2. 基本过渡动画的编码
  1). 在目标元素外包裹<transition name="xxx">
  2). 定义class样式
    1>. 指定过渡样式: transition
    2>. 指定隐藏时的样式: opacity/其它
3. 过渡的类名
  xxx-enter-active: 指定显示的transition
  xxx-leave-active: 指定隐藏的transition
  xxx-enter: 指定隐藏时的样式
-->



<div id="demo">
  <button @click="show = !show">Toggle</button>
  <transition name="xxx">
    <p v-show="show">hello</p>
  </transition>
</div>

<hr>
<div id="demo2">
  <button @click="show = !show">Toggle2</button>
  <transition name="move">
    <p v-show="show">hello</p>
  </transition>
</div>


<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
  new Vue({
    el: '#demo',
    data: {
      show: true
    }
  })

  new Vue({
    el: '#demo2',
    data: {
      show: true
    }
  })

</script>
</body>
</html>
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>10_过渡&动画2</title>
  <style>
    .bounce-enter-active {
      animation: bounce-in .5s;
    }
    .bounce-leave-active {
      animation: bounce-in .5s reverse;
    }
    @keyframes bounce-in {
      0% {
        transform: scale(0);
      }
      50% {
        transform: scale(1.5);
      }
      100% {
        transform: scale(1);
      }
    }
  </style>
</head>
<body>

<div id="example-2">
  <button @click="show = !show">Toggle show</button>
  <br>
  <transition name="bounce">
    <p v-if="show" style="display: inline-block">Lorem</p>
  </transition>
</div>

<script type="text/javascript" src="../js/vue.js"></script>
<script>
  new Vue({
    el: '#example-2',
    data: {
      show: true
    }
  })
</script>
</body>
</html>
```

## 过滤器

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>11_过滤器</title>
</head>
<body>
<!--
1. 理解过滤器
  功能: 对要显示的数据进行特定格式化后再显示
  注意: 并没有改变原本的数据, 可是产生新的对应的数据
2. 编码
  1). 定义过滤器
    Vue.filter(filterName, function(value[,arg1,arg2,...]){
      // 进行一定的数据处理
      return newValue
    })
  2). 使用过滤器
    <div>{{myData | filterName}}</div>
    <div>{{myData | filterName(arg)}}</div>
-->
<!--需求: 对当前时间进行指定格式显示-->
<div id="test">
  <h2>显示格式化的日期时间</h2>
  <p>{{time}}</p>
  <p>最完整的: {{time | dateString}}</p>
  <p>年月日: {{time | dateString('YYYY-MM-DD')}}</p>
</div>

<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript" src="https://cdn.bootcss.com/moment.js/2.22.1/moment.js"></script>
<script>
  // 定义过滤器
  Vue.filter('dateString', function (value, format='YYYY-MM-DD HH:mm:ss') {

    return moment(value).format(format);
  })


  new Vue({
    el: '#test',
    data: {
      time: new Date()
    },
    mounted () {
      setInterval(() => {
        this.time = new Date()
      }, 1000)
    }
  })
</script>
</body>
</html>
```

## Vue指令

### 内置对象

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>12_指令_内置指令</title>
  <style>
    [v-cloak] { display: none }
  </style>
</head>
<body>
<!--
常用内置指令
  v:text : 更新元素的 textContent
  v-html : 更新元素的 innerHTML
  v-if : 如果为true, 当前标签才会输出到页面
  v-else: 如果为false, 当前标签才会输出到页面
  v-show : 通过控制display样式来控制显示/隐藏
  v-for : 遍历数组/对象
  v-on : 绑定事件监听, 一般简写为@
  v-bind : 强制绑定解析表达式, 可以省略v-bind
  v-model : 双向数据绑定
  ref : 为某个元素注册一个唯一标识, vue对象通过$refs属性访问这个元素对象
  v-cloak : 使用它防止闪现表达式, 与css配合: [v-cloak] { display: none }
-->
<div id="example">
  <p v-cloak>{{content}}</p>
  <p v-text="content"></p>   <!--p.textContent = content-->
  <p v-html="content"></p>  <!--p.innerHTML = content-->
  <p ref="msg">abcd</p>
  <button @click="hint">提示</button>
</div>

<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
  new Vue({
    el: '#example',
    data: {
      content: '<a href="http://www.baidu.com">百度一下</a>'
    },
    methods: {
      hint () {
        alert(this.$refs.msg.innerHTML)
      }
    }
  })
</script>
</body>
</html>
```

### 全局指令

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>12_指令_自定义指令</title>
</head>
<body>

<!--
1. 注册全局指令
  Vue.directive('my-directive', function(el, binding){
    el.innerHTML = binding.value.toupperCase()
  })
2. 注册局部指令
  directives : {
    'my-directive' : {
        bind (el, binding) {
          el.innerHTML = binding.value.toupperCase()
        }
    }
  }
3. 使用指令
  v-my-directive='xxx'
-->
<!--
需求: 自定义2个指令
  1. 功能类型于v-text, 但转换为全大写
  2. 功能类型于v-text, 但转换为全小写
-->

<div id="test">
  <p v-upper-text="msg"></p>
  <p v-lower-text="msg"></p>
</div>

<div id="test2">
  <p v-upper-text="msg"></p>
  <p v-lower-text="msg"></p>
</div>

<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript">
  // 注册一个全局指令
  // el: 指令所在的标签对象
  // binding: 包含指令相关数据的容器对象
  Vue.directive('upper-text', function (el, binding) {
    console.log(el, binding)
    el.textContent = binding.value.toUpperCase()
  })
  new Vue({
    el: '#test',
    data: {
      msg: "I Like You"
    },
    // 注册局部指令
    directives: {
      'lower-text'(el, binding) {
        console.log(el, binding)
        el.textContent = binding.value.toLowerCase()
      }
    }

  })

  new Vue({
    el: '#test2',
    data: {
      msg: "I Like You Too"
    }
  })
</script>
</body>
</html>
```

## Vue插件

```js
(function (window) {
  const MyPlugin = {}
  MyPlugin.install = function (Vue, options) {
    // 1. 添加全局方法或属性
    Vue.myGlobalMethod = function () {
      console.log('Vue函数对象的myGlobalMethod()')
    }

    // 2. 添加全局资源
    Vue.directive('my-directive',function (el, binding) {
      el.textContent = 'my-directive----'+binding.value
    })

    // 4. 添加实例方法
    Vue.prototype.$myMethod = function () {
      console.log('vm $myMethod()')
    }
  }
  window.MyPlugin = MyPlugin
})(window)

```

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>17_插件</title>
</head>
<body>

<div id="test">
  <p v-my-directive="msg"></p>
</div>

<script type="text/javascript" src="../js/vue.js"></script>
<script type="text/javascript" src="vue-myPlugin.js"></script>
<script type="text/javascript">
  // 声明使用插件(安装插件: 调用插件的install())
  Vue.use(MyPlugin) // 内部会调用插件对象的install()

  const vm = new Vue({
    el: '#test',
    data: {
      msg: 'HaHa'
    }
  })
  Vue.myGlobalMethod()
  vm.$myMethod()

  new Object()
</script>
</body>
</html>
```

## 基于脚手架的运行

> 1.执行命令
>
> vue init webpack 项目名称
>
> 2.进入到项目名称中执行
>
> npm install 

## 项目的打包与发布

> 1.打包
>
> npm run build
>
> 2.使用静态服务器工具包
>
> npm install -g serve //全局安装server服务
>
> serve dist

