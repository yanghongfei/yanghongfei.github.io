---
layout: wiki
title: Vue
categories: Vue
description: 最初学习vue中的一些记录。
keywords: Vue
---


**目录**

* TOC
{:toc}



### Vue  

> 最初学习vue中的一些记录

#### Hello Vue
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Hello Vue</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <script src="https://unpkg.com/vue"></script>

</head>
<body>
    <div id="app">
        {{ message }}
    </div>
    <script>
      var app = new Vue({
          el: "#app",
          data: {
              message: 'Hello, Vue!'
          }
      })  
        
    </script>
</body>
</html>
```

#### Vue-bind
- 数据绑定
- 示例

```
#数据：website: "https://yanghongfei.github.io",


<a v-bind:href="website">web开发</a>
```


#### Vue-html
- 解析html
- 示例
```
websitetag: "<a href='https://yanghongfei.github.io'>yanghongfei.github.io</a>"

<p v-html="websitetag"></p>

```


#### Vue-methods
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Vue Input加减计算</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <script src="https://unpkg.com/vue"></script>
</head>
<body>
    <div id="app">
        <input type="text" v-model="one">
        <input type="text" v-model="two">
        <p>相加的结果:{{ add() }}</p>
        <p>相减的结果:{{ jian() }}</p>
    </div>

    <script>
        var obj = new Vue({
            el:'#app',
            data: {
                one: 0,  //默认0
                two: 0   //默认0
            },
            methods: {
                add() {
                    return this.one*1 + this.two*1
                },
                jian() {
                    return this.one*1 - this.two*1
                }
            }
        })
    </script>

</body>
</html>
```

#### Vue-computed
```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>vue-computed-watch</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <script src="https://unpkg.com/vue"></script>
</head>

<body>
    <div id="app">
        <input type="text" v-model="one">
        <input type="text" v-model="two">
        <input type="text" v-model="three">
        <p>相加的结果:{{ add }}</p>
        <p>相减的结果:{{ jian }}</p>
    </div>

    <script>
        var obj = new Vue({
            el: '#app',
            data: {
                one: 0,  //默认0
                two: 0,   //默认0
                three: 0
            },
            // methods: {
            //     add() {
            //         return this.one * 1 + this.two * 1
            //     },
            //     jian() {
            //         return this.one * 1 - this.two * 1
            //     }
            // }
            //使用computed，只有操作one和two的时候才会执行computed下面的函数
            computed: {
                add(){
                    console.log('add')
                    return this.one*1 + this.two*1
                },
                jian(){
                    console.log('jian')
                    return this.one*1 - this.two*1
                }
            }

        })
    </script>

</body>

</html>
```

#### Vue-watch
- 监控数据的变化

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>vue-computed-watch</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <script src="https://unpkg.com/vue"></script>
</head>

<body>
    <div id="app">
        <input type="text" v-model="one">
        <input type="text" v-model="two">
    </div>

    <script>
        var obj = new Vue({
            el: '#app',
            data: {
                one: 0,  //默认0
                two: 0,   //默认0
            },
            watch:{
                one(var1,var2){
                    console.log('one')
                    console.log('之前的值是:',var2)
                    console.log('改变的值是:',var1)

                },
                two(var1,var2){
                    console.log('two')
                    console.log('之前的值是:', var2)
                    console.log('改变的值是:', var1)

                }

            }
        })
    </script>

</body>

</html>
```

#### Vue-v-for
```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Vue Input加减计算</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <script src="https://unpkg.com/vue"></script>
</head>

<body>
    <div id="app" style=width:300px; margin:auto>
        <input type="text" v-model="val" v-on:keyup.13="down()">

        <ul>
            <li v-for="item in arr">
                {{ item }}
            </li>
        </ul>
    </div>

    <script>
        new Vue({
            el: '#app',
            data: {
                val: "",  
                arr: []   
            },
            methods:{
                down(){
                    // console.log(this.val)
                    this.arr.push(this.val);  //添加到数组
                    console.log(this.arr);
                    this.val = "";   
                }
            }
        })
    </script>

</body>

</html>
```

#### Vue-component
- 定义模板

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Page Title</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <script src="https://unpkg.com/vue"></script>
</head>
<body>
    <div class="app" style="width:300px; margin:auto">
    <!-- <todo></todo>   定义的模板 -->
    <todo></todo>
    <todo></todo>
    <todo></todo>
        
    </div>
    <script>
        Vue.component("todo", {
            template: `
            <div>
                <input type="text" v-model="val" @keyup.13="up">
                <ul>
                    <li v-for="item in arr">
                        {{ item }}
                    </li>
                </ul>
            </div>
            `,
            data(){
                return {
                    val:"",
                    arr:[]
                }
            },
            methods:{
                up(){
                    this.arr.push(this.val);
                    console.log(this.arr);
                    this.val="";
                }

            }
        })
    
        new Vue({
            el: '.app',
        })

    </script>

</body>
</html>
```

#### Vue-props

- 变量取值，单向数据流

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Page Title</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <script src="https://unpkg.com/vue"></script>
</head>
<body>
    <div class="app" style="width:300px; margin:auto">
    <!-- <todo></todo>   定义的模板 -->
    <todo text="请输入内容111"></todo>
    <todo text="请输入内容222"></todo>
    <todo :text="text"></todo>    
        
    </div>
    <script>
          //:placeholder 会当作JS的代码进行执行变量
        Vue.component("todo", {
            props: ["text"],  //变量取值
            template: `
            <div>
                <input type="text" v-model="val" @keyup.13="up" :placeholder="text">   
                <ul>
                    <li v-for="item in arr">
                        {{ item }}
                    </li>
                </ul>
            </div>
            `,
            data(){
                return {
                    val:"",
                    arr:[]
                }
            },
            methods:{
                up(){
                    this.arr.push(this.val);
                    console.log(this.arr);
                    this.val="";
                }

            }
        })
        //单向数据流
        new Vue({
            el: '.app',
            data:{
                text:"这是需要输入的值，传值到上面todo模板里面"
            }
        })

    </script>

</body>
</html>
```

#### Vue-todo_list
- app.js
```javescript
new Vue({
    el:'#root',
    data:{
        all: [],
        text: "",
        status: "all"
    },
    methods:{
        add(){
            var obj={};
            obj.text=this.text;
            if (!obj.text) {
                alert('请输入内容');
                return;
            }           //内容
            obj.id=Math.random()+new Date().getTime();  //ID,随机数+现在时间
            obj.state=0;      //0:未完成，1 ：已完成
            obj.edit=true;
            this.all.push(obj)
            this.text=""
            console.log(this.all)
        },
        del(id){
            this.all=this.all.filter(function (obj) {
                if(obj.id!=id){
                    return obj;
                }                
            })
        },
        edit(item){
            item.edit=!item.edit
        },
        changeStatus(val){
            this.status=val;
        },
        changeState(obj){
            if (obj.state==0){
                obj.state=1;
            } else{
                obj.state=0;
            }
        }
    },
    computed: {
        datas: function () {
            return this.all.filter((obj) => {
                if (this.status == "all") {
                    return obj
                } else {
                    if (this.status == obj.state) {
                        return obj;
                    }
                }
            })
        }
    }
})
```

- todo.css
```css
body,ul,li,p{
    padding: 0; margin: 0; list-style: none;
}

.box{
    margin: 20px auto;
    width: 300px;
    height: auto;
}
.input{
    width: 100%;
    height: 30px;
}
.input input{
    width: 100%;
    height: 30px;
    font-size: 15px;
    padding: 0; margin: 0;
}
.btns{
    width: 100%; overflow: hidden;
    margin-top: 10px;
}
.btns input{
    float: right; margin-left: 5px;
}
.list{
   width: 100%;
   margin-top: 10px; 
}
.list li{
    width: 100%; height: 25px;
    line-height: 25px;
    border-bottom: 1px solid #888888;
}

.list li .opt{
    float: left;
    width: 8px;height: 8px;
    border-radius: 50%;
    margin-right: 5px;
    border: 1px solid #888888;
    margin-top: 5px;
}
.list li p {
    float: left;
}
.list li .del{
    float: right;
}
.check{
    color: red;
}
.red{
    background: red;
}
```

- todo.html
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Page Title</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <!--<script src="vue.js"></script>-->
    <script src="https://unpkg.com/vue"></script>
    <link rel="stylesheet" href="todo.css">
</head>
<body>
    <div id="root">
        <div class="box">
            <div class="input">
                <input type="text" placeholder="请输入事项" v-model="text" @keyup.13=add>
            </div>
            <div class="btns">
                <input type="button" value="全部" @click="changeStatus('all')" :class="{check:status=='all'}">
                <input type="button" value="未完成" @click="changeStatus('0')" :class="{check:status==0}">
                <input type="button" value="已完成" @click="changeStatus('1')" :class="{check:status==1}">
            </div>
            <div>
                <ul class="list">
                    <li v-for="item in datas">
                        <div v-if="item.edit==true">
                        <span class="opt" @click="changeState(item)" :class="{red:item.state==1}"></span>
                            <p @dblclick="edit(item)">
                                {{ item.text }}
                            </p>
                        <span class="del" @click="del(item.id)">删除</span>
                        </div>
                        <div v-else>
                            <input type="text" v-model="item.text" @blur="edit(item)">
                        </div>
                    </li>
                </ul>
                <div v-show="all.length==0">没有数据</div>   
            </div>
        </div>
    </div>
    <script src="app.js"></script>
</body>
</html>
```

#### Vue生命周期
```vue
  beforeCreate:function(){
      alert("组件实例化之前执行的函数");
  },
  created:function(){
      alert("组件实例化完毕，但是页面还没显示");
  },
  beforeMount:function(){
      alert("组件挂在前，页面仍未显示，但是虚拟Dom已经配置了");

  },
  mounted:function(){
      alert("组件挂载后，此方法执行后，页面展示");
  },
  beforeUpdate:function(){
      alert("组件更新前，页面仍未更新，但是虚拟DOM已经配置");
  },
  updated:function(){
      alert("组件更新。此方法一旦执行，页面显示");
  },
  beforeDestroy:function(){
      alert("组件销毁前");
  },
  destroyed:function(){
      alert("组件已经销毁");
  }
```


### Vue Router

```
# :to="{name: 'user'}" 这个name是router.js定义好的
如：      {
        path: 'mysqlAudit',
        name: 'mysqlAudit',
        meta: {
          icon: 'ios-help-circle',
          title: 'SQL审核'
        },
        component: () => import('@/view/tasks-center/task-submit/mysql-audit.vue')
      },
      
# template
<h3><router-link :to="{name:'mysqlAudit'}" class="nav-link">SQL审核</router-link></h3>
```


#### Vue写一个组件


```
#先导入模块
import { mapState } from 'vuex'

#data里面的计算方法

  computed: {
    ...mapState({
      rules: state => state.user.rules
    })

# template
#v-if="rules.asset_error_log 组件名字,前端加进去
<Button type="info" v-if="rules.asset_error_log" class="search-btn" @click="handlerCheckErrorLog">任务日志 

```
