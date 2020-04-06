---
title: javaScript知识集锦
date: 2019-05-23 22:06:47
tags: 
- 前端
- js
categories:
- JavaScript
---

---  
<!--more-->

## 防抖

## 数组

## JSON.parse & JSON.stringify

### 前言

`JSON.stringify`的基本用法是将对象转成`json`字符串进行传递，即`JSON.stringify(data)`
需要注意的是:

1. 转换的值是数组时：下标为`undefined`和`function`还有`Symbol('')`都会转换成`null`
例如：
```js
JSON.stringify([function(){}, undefined, ''])   // 打印"[null, null, '']"
```
2. 转换的值是对象时：对应value值为`undefined`和`function`还有`Symbol('')`都会导致直接忽略该属性
例如：
```js
JSON.stringify({key1: undefined, key2: function(){}}) // 打印"{}"
```
3. `NaN Infinity`等在对象或数组中都返回`null`
4. `toJSON()`
```js
// toJSON()
JSON.stringify({ x: 5, y: 6, toJSON(){ return this.x + this.y; } });
// '11'
```
除了对象参数还有两个参数作用也很大

### 用法

`JSON.stringify(value[, replacer[, space]])`

参数含义：
1. `value`：必选，要转换的值（包括所有的数据类型，通常是对象或者数组）
2. `replacer`：可选，可以是函数或数组，这个参数用处很大啊，举例：
```js
// 函数
function replacer(key, value) {
  // Filtering out properties
  if (typeof value === 'string') {
    return undefined;
  }
  return value;
}

var foo = {foundation: 'Mozilla', model: 'box', week: 45, transport: 'car', month: 7};
JSON.stringify(foo, replacer);
// '{"week":45,"month":7}'

// 数组
JSON.stringify(foo, ['week', 'month']);  
// '{"week":45,"month":7}', only keep "week" and "month" properties

// 关系映射
var data =[
{
    name: "程咬金",sex:"1",age:26    
},
{
    name: "程才",sex:"0",age:20
},
]
var str_json = JSON.stringify(data,function(key,value){
    if(key == 'sex'){
        return ["女",'男'][value];
    }
    return value;
});
// [{"name":"程咬金","sex":"男","age":26},{"name":"程才","sex":"女","age":20}]
```
3. `space`：可选，控制字符串中的间距，可以是数字或字符串，数字最大是10，可用在编辑器中
```js
JSON.stringify([10, 30], null, 4)
// 等同于
JSON.stringify([10, 30], null, '\t')
// "[
//     10,
//     30,
// ]"
```
- 
## 
