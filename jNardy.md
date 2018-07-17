[1 教程](#教程)  

[1.1 安装](#安装)  

[1.2 介绍](#介绍)  

[1.3 实例](#实例)  

[1.4 dot模板](#dot模板)  

[1.5 区域渲染](#区域渲染)  

[1.6 组件](#组件)  

[1.7 路由](#路由)  

[1.8 数据模型](#数据模型)  

[1.9 表单输入绑定](#表单输入绑定)

[2 API](#api)  

[2.1 Controller API](#controller-api)  

[2.2 选项](#选项)  

[2.3 Controlle实例](#controller实例)  

[3 示例](#示例)


# 教程
## 安装
+ 首先安装nodejs，全局安装gulp
+ 项目目录下（/nardy-service-1.0），使用命令行工具，执行以下命令
  + &gt; npm install
  > 首次运行项目执行
  + &gt; npm start
  + &gt; gulp
  > 开发环境，执行 &gt; gulp dev

## 介绍
jNardy是一套mvc框架，视图层使用dot模板编译器执行渲染，controller层使用类vue语法控制模板与数据模型的关联关系，model层封装对外数据请求、数据映射、数据校验等过程。jNardy并非双向数据绑定框架，即数据模型变化不会隐性造成其绑定的视图的变化，需要显性控制数据模型与视图的关系。jNardy对jQuery友好，jQuery选择器有利于显性控制数据与视图的变化。
## 实例
实例化控制器
```
var c = new Controller({
    ele: 'container',//绑定的DOM对象id
    data: function(){
        return {
            name: 'jNardy'
        }
    },
    components: {//组件
        
    },
    watch: {//监听对象
        name: function(newValue,oldValue){
            //name数据变化时，显示控制视图
        }
    },
    methods: {
        
    },
    mounted: function(){},//完成数据视图关联后执行的方法
    destoryed: function(){}//销毁实例后执行的方法
})
```
数据与方法
```
//获取数据
var name = c.data.name 
//或者
var name = c.get('name')


//设置数据
c.data.name = 'jNardy'
//或者
c.set('name','jNardy');
```
当数据变化时，会触发监听器watcher并执行监听对象watch中对应的函数。

## dot模板
基本语法详见：[dot.js语法](http://olado.github.io/doT/index.html)  

## 区域渲染
模板中使用dot标签，指定重新渲染的视图区域
```
<div>
    <dot id="scope-1">
        <div>{{=it.name}}</div>
        <div>
            <table></table>
        </div>
    </dot>
</div>

```
区域渲染方式
```
//name变化时，重新渲染指定区域
watch: {
    name: function(value,oldValue){
        this.rerender('scope-1');  
    }
}
```
> dot标签支持三层嵌套，重新渲染发生在dot模板预编译期，dot标签会被渲染成div标签
## 表单绑定
待增加。。。
## 组件
组件定义

```
var tpl = require('./myComponent.html');

var myComponent = Controller.extend({
    tpl: tpl,
    data: function(){
        return {
            name: ''
        }
    },
    components: {//子组件
        
    },
    methods: {
        
    },
    mounted: function(){

    }
});
```
组件使用  
1.模板中引用
```
/**
* id:元素id，必需  is:模板名称，必需  title:传递的数据，非必须
*/
<template id="abc" is="menu" title="homepage"></template>

//或者
/**
* pdata:传递的数据对象；it：指控制器的数据对象
*/
<template id="abc" is="menu" pdata="it.pdata"></template>
```
2.控制器中引用  
```
var singleText = require('./single-text.js');

var page1 = Controller.extend('page1',{
    tpl: tpl,
    data: function(){
        return {
            
        }
    },
    components: {
        'single-text': singleText
    }
})
```
> 以上组件是逻辑组件，即包含了控制器和模板的独立使用的组件,还可以使用非逻辑组件，即只有模板的组件，引用方式如下，其逻辑在父控制器中实现。

single-text.html
```
<div>
    <div><a onclick ="{{=it.showContent}}"><span>显示</span></a></div>
    <p>
        {{=it.content}}
    </p>
</div>

```

模板中引用，无法传递参数
```
<template id="abc" is="single-text"></template>

```
控制器中引用
```
var singleText = require('./single-text.html');

var page1 = Controller.extend('page1',{
    tpl: tpl,
    data: function(){
        return {
            content: '我是文本'
        }
    },
    components: {
        'single-text': singleText
    },
    mothods: {
        showContent: function(){
            alert(this.data.content);
        }
    }
})

```
## 路由

定义路由

```
var Router = require('../common/router');
var page1 = require('./page1');
var page2 = require('./page2');
var childPage = require('./childPage');

var router = new Router();
router.route({path:'page1',component:page1});
//子路由
router.route({path:'page2',component:page2,children:[{path:'page2/child',component:childPage}]});
```

路由传参  

路径示例：  
http://192.168.0.103:8000/main#app?userId=123&name=bob  

参数获取  
```
var userId = this.route.params.userId;
var name = this.route.params.name;
```

## 数据模型
定义数据模型
```
var Model = require('../common/model');
var Utils = require('../common/utils');

var userModel = Utils.Class(Model,{

    initialize:function(){
        this.super.call(this);
    }

});

module.exports = userModel;
```

使用数据模型
```
var UserModel = require('../../models/userModel');

var userModel = new UserModel();

    ...

    mounted:function(){
        userModel.get({url:'/api/user'},function(data){
            ...
        })

    }
    
```

数据校验
```
options: {
    validate: true,
    rules: {
        name: {required:true},
        sex: {required:true},
        age: {required:true}
    },
    validShow: function(result){
        console.log(result);
    }
},

initialize:function(){
    this.super.call(this,this.options);
}

```
> 校验请求数据data，userModel.post({url:'/api/user'，data:{name:'bob'}},function(data){
            ...
        })  
目前支持的校验规则如下：  
required,email,tel,url,date,dateISO,number,digits,idcard,equalTo,contains,minlength,maxlength,rangelength,max,range ，详见validator.js

## 表单输入绑定
```
<input type="text" data-modle="name" />
```
> 表单元素添加data-model属性，来绑定控制器中的数据对象，表单元素value值改变时，触发数据对象的观察函数。

```
...
data:function(){
    return {
        name: 'fire'
    }
}
watch:{
    name: function(v,ov){
        //input值变化，执行此方法
    }
},
...

```

# API
## Controller API
**Controller.extend(options)**
+ 参数：
  + {Object} options
+ 用法：
  使用Controller构造器，创建一个扩展类。参数是一个包含组件选项的对象。
```
var home = Controller.extend({
    tpl: '<div>{{=it.name}}</div>',
    data: function(){
        return {
            name: 'bob'
        }
    }
})
home.initialize(null,{ele:'#home'});
```

```
## 选项
**ele**  
类型：string  
详细：  
提供一个在页面上已存在的 DOM 元素作为 Controller 实例的挂载目标，可以是 CSS 选择器。

**loadData**
类型：string  
详细：  
URL，获取控制器初始化数据的接口地址

**tpl**  
类型：string  
详细：  
一个字符串模板作为Controller实例的标识使用。

**data**  
类型：Object  
详细：  
Controller实例的数据对象。Controller将会递归将data的属性转换为getter/setter，从而让data的属性能够响应数据变化。  
实例创建之后，可以通过 c.data 访问原始数据对象。

**methods**  
类型：{[key:string]:Function}  
详细：  
methods 将被混入到 Controller 实例中。可以直接通过 c 实例访问这些方法，或者在dot模板中使用。方法中的 this 自动绑定为 Controller实例。

**watch**  
类型：{[key:string]:Function}   
详细：  
一个对象，键是需要观察的表达式，值是对应回调函数。

**mounted**  
类型：Function  
详细：视图渲染完毕之后调用该钩子。

**destroyed**  
类型：Function  
详细：Controller实例销毁后调用。

**components**  
类型：{[key:string]:[string|Object]}  
详细：可用组件哈希表。

## Controller实例
**c.data**  
类型：Object  
详细：实例观察的数据对象。

**c.chids**  
类型：Array<string>  
详细：子组件id集合

**c.chhtml**  
类型：{[id:string]:[string]}   
详细：静态子组件html集合

**c.events**  
类型：Object  
详细：事件响应全局名称集合

**c.set**
类型：Function  
详细：设置数据对象的值 ,支持多级。  
示例：
```
c.set('name','Tony');
//or
c.set('info.name','Tony');
```

**c.get**
类型：Function  
详细：获取数据对象的值 ,支持多级。  
示例：
```
c.get('name','Tony');
//or
c.get('info.name','Tony');
```

**c.rerender**  
类型：Function  
详细：重新渲染视图并挂载。参数id，指定区域重新渲染。

**c.rerenderChild**  
类型：Function  
详细：重新渲染子组件视图并挂载。参数id，指定子模版重新渲染。

**c.destory**  
类型：Function  
详细：销毁实例

  
# 示例
**Utils.Class()**  
详细：类继承，返回子类。   
示例：  
```
var Model = require('../common/model');
var Utils = require('../common/utils');

Utils.Class(Model,{
    options:{},
    initialize:function(options){//构造函数
        Model.call(this,options);
        this.options = Utils.extend(this.options,options);
    }
    
});
```

**Utils.extend**  
详细：对象扩展。  
示例：
```
var Utils = require('../common/utils');
var obj1 = {
    id:123,
    name:'test'
}
var obj2 = {
    type: 'string',
    info: {
        a:'hi'
    }
}

var newObj = Utils.extend(obj1,obj2);
//or 深度扩展
var newObj = Utils.extend(true,obj1,obj2);
```

**Utils.each**  
详细：遍历对象或者数组并操作  
示例：
```
var Utils = require('../common/utils');

var arr = ['a','b','c'];
Utils.each(arr,function(item){
    //操作item
})

```

**Utils.jmodal**  
详细：模态框，依赖jquery和bootstrap。  
示例：
```
var Utils = require('../common/utils');
var step1 = require('./step/step1.html');
var step2 = require('./step/step2.html');

var body = [step1,step2];
var options = {
    title: '提示',
    type: 'step',
    size: 'small'
};
/**
* jmodal(body,options,fn)
* body: 模态框内容
* options: 模态框选项
*  + options.title: 模态框标题
*  + options.type: 模态框类型，[edit|step|alert]
*  + options.size: 模态框大小，[large|normal|small]
* fn: 回调函数，参数data：回传数据，参数next：下一步函数。
*
**/
Utils.jmodal(body,options,function(data,next){
    if(data.stepNum == 1){
        data.hasVlide = true;
    }
    console.log(data);
    if(data.stepNum == 2){
        Utils.jmodal('hide');
    }else{
        next();
    }
})
```

**Utils.jselect**  
详细：选择组件  
示例：
```
var Utils = require('../common/utils');


/**
* jselect(ele,fn)
* ele: 要素
* fn: 回调函数，参数id：组件id；参数el：要素。
**/
Utils.jselect(ele,function(id,el){
    var c = CSet[id];
    if(c.name == 'single-text'){
    }
    Utils.jmoveY(el);
});
```

**Utils.jmoveY**  
详细：上下移动组件  
示例：
```
var Utils = require('../common/utils');

/**
* jmoveY(ele)
* ele: 要素
**/

Utils.jmoveY(ele);
```
