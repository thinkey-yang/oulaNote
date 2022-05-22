# mustache

一个低逻辑js模版引擎。

## 需求背景

目前团队内部维护多套模版工程用于不同的场景，在依赖包升级时，每一个模版工程都需要同步升级，带来额外的工作量，考虑，不同模版之前的主要区别为umi的config文件。所以考虑后续将多个模版工程合并成一个，使用mustache，根据cli命令参数的不同，动态生成差异化的文件。

## mustache 语法

#### {{prop}}

简单的将prop转换成字符串进行输出

```javascript
<script id="tpl1" type="text/html">
    -{{prop}}-
</script>
<script>
    var tpl1 = document.getElementById('tpl1').innerHTML.trim();
    Mustache.parse(tpl1);
    //测试falsy值
    console.log(Mustache.render(tpl1, {prop: ''}));//--
    console.log(Mustache.render(tpl1, {prop: 0}));//-0-
    console.log(Mustache.render(tpl1, {prop: null}));//--
    console.log(Mustache.render(tpl1, {prop: undefined}));//--
    console.log(Mustache.render(tpl1, {prop: false}));//-false-
    console.log(Mustache.render(tpl1, {prop: NaN}));//-NaN-

    //测试简单对象
    console.log(Mustache.render(tpl1, {prop: {name: 'jason'}}));//-[object Object]-
    //测试数组
    console.log(Mustache.render(tpl1, {prop: [{name: 'jason'}, {name: 'frank'}]}));//-[object Object],[object Object]-
    //测试日期对象
    console.log(Mustache.render(tpl1, {prop: new Date()}));//-Mon Jan 18 2016 15:38:46 GMT+0800 (中国标准时间)-
    //测试自定义toString的简单对象
    var obj1 = {name: 'jason'};
    obj1.toString = function () {
        return this.name;
    };
    console.log(Mustache.render(tpl1, {prop: obj1}));//-jason-

    //测试boolean number string
    console.log(Mustache.render(tpl1, {prop: true}));//-true-
    console.log(Mustache.render(tpl1, {prop: 1.2}));//-1.2-
    console.log(Mustache.render(tpl1, {prop: 'yes'}));//-yes-

    //测试function
    console.log(Mustache.render(tpl1, {
        prop: function () {
        }
    }));//--
    console.log(Mustache.render(tpl1, {
        prop: function () {
            return 'it\'s a fun'
        }
    }));//-it&#39;s a fun-
    console.log(Mustache.render(tpl1, {
        prop: function () {
            return false;
        }
    }));//-false-
    console.log(Mustache.render(tpl1, {
        prop: function(){
            return function (text, render) {
                return "<b>" + render(text) + "</b>"
            };
        }
    }));
    //-function (text, render) {
    //   return &quot;&lt;b&gt;&quot; + render(text) + &quot;&lt;&#x2F;b&gt;&quot;
    //}-

</script>
```

mustache渲染{{prop}}标签的逻辑是：

1）如果prop引用的值是null或undefined，则渲染成空串；

2）如果prop引用的是一个函数，则在渲染时自动执行这个函数，并把这个函数的返回值作为渲染结果，假如这个返回值为null或者undefined，那么渲染结果仍然为空串，否则把返回值转成字符串作为渲染结果（注意最后一个用例，直接把函数代码渲染出来了）；

3）其它场景，直接把prop引用的值转成字符串作为渲染结果。

由于默认情况下，mustache在渲染prop时，都是对prop的原始值进行url编码或者html编码之后再输出的，所以有一个用例的渲染结果，把英文的单引号，转成了html实体符,要避免这种情况使用`{{{prop}}}`

## `{{#prop}}{{/prop}}`

这对标签的作用非常强大，可以同时完成if-else和for-each以及动态渲染的模板功能。在这对标签之间，可以定义其它模板内容，嵌套所有标签。