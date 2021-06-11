---
title: Mfuns高级弹幕手册
date: 2021/6/11
tags: mfuns,高级弹幕
category: 高级弹幕
---

## 入门

Mfuns的高级弹幕是由 JSON 格式组成，关于基本 JSON 格式语法，请参阅：[菜鸟教程 - JSON 语法](https://www.runoob.com/json/json-syntax.html)。

###　文件格式

弹幕文件是由 JSON 文件组成，基本的格式为：
```json
[
    {
        //单个弹幕。。。
    },
    {
        //单个弹幕。。。
    }
    //。。。
]
```

所有的弹幕，最外层由 `[]` 英文半角状态的中括号包裹，单个弹幕以 `{}` 大括号包裹。

### 舞台

此弹幕引擎的舞台的大小比例固定为16:9，其逻辑分辨率为1600:900,会根据屏幕的大小自动进行缩放。

因此，弹幕作者并不需要去考虑弹幕大小对于屏幕的适配问题，可放心的使用绝对像素值对内容进行描述与定位

### 弹幕基本格式

一个弹幕的基本组成内容如下：

```json
{
    "type":"text",
    "start":0,
    "content":"danmaku",
    "style":{},
    "animations":[],
    "child":[]
},
```
接下来我将对其一一介绍：

| 属性       | 可选     | 内容       | 备注                                            |
| ---------- | -------- | ---------- | ----------------------------------------------- |
| type       | 必须存在 | text       | 弹幕类型，目前仅有text                          |
| start      | 必须存在 | 数字       | 弹幕开始时间（毫秒）                            |
| content    | 必须存在 | 文本       | 弹幕内容                                        |
| style      | 可选     | 弹幕样式   | 直接应用css的样式（小驼峰命名）                 |
| animations | 可选     | 动画列表   | 作用于弹幕的动画                                |
| child      | 可选     | 弹幕子元素 | 子元素也是一个完整的弹幕，但是start属性不能使用 |


> 注意：弹幕内的所有属性，注意分清是大括号还是中括号，如果括号错误，弹幕无法解析
> 每个属性结尾，如果有下一个属性，不要忘记加逗号

### 弹幕类型
当前仅有文本弹幕( type 属性为 `text` )，以后或许会开发出其他类型的弹幕（比如 SVG ）

### 弹幕生命周期

一条弹幕的完整生命周期由开始时间到结束时间，弹幕的开始时间由 `start` 属性设置，单位时间为毫秒。弹幕的结束时间由动画时长进行控制，其长度为动画的时长

### 弹幕的样式

目前弹幕样式可以直接使用CSS的大部分常用样式，如果你没有接触过网页，也不知道CSS是什么，不用担心，这并不会影响到弹幕的使用。

您可以从这里找到有关于样式的全部属性 [菜鸟教程 - CSS 参考手册](https://www.runoob.com/cssref/css-reference.html)

例如：

我们希望让弹幕的字体为40px，并且为红色，我们从上面链接中的参考手册中找到设置字体大小的属性为 `font-size` 我们将横线去掉，其转换成首字母小写的驼峰命名：`fontSize`，这便是弹幕中 `style` 属性的成员之一，同理，我们再找到设置字体颜色的属性 `color`，最后得出以下弹幕：

```json

[
    {
        "type":"text",
        "start":0,
        "content":"这是一段40像素的红色文本",
        "style":{
            "fontSize":"40px",
            "color":"red"
        }
    }
]
```
[点我运行一下](https://lab.meogirl.ml/index.html?id=red40px)

### 弹幕动画

弹幕动画使用 `animations` 属性定义（注意最后有个s），动画列表使用数组表示。

```json
[
    {
        "type":"static",//动画类型
        "cubic": [0, 0, 1, 1],//贝塞尔曲线,可选
        "duration": 1//弹幕的生存周期，可选，默认1000
        //...弹幕内部属性
    }
]
```

目前已经支持的基本动画类型有：

| 动画名称 | type类型  | 描述                   | 内部属性                          |
| -------- | --------- | ---------------------- | --------------------------------- |
| 静止动画 | static    | 静止在某个坐标一段时间 | `x`,`y`,`z`                       |
| 移动动画 | translate | 从一处移动到另一处     | `path:{x1,y1,z1,x2,y2,z2}`        |
| X轴旋转  | rotateX   | 沿着一个点绕 x 轴旋转一点角度    | `origin:[x,y,z]`,`angle:{start:number,end:number}` |
| Y轴旋转  | rotateY   | 沿着一个点绕 y 轴旋转一点角度    | `origin:[x,y,z]`,`angle:{start:number,end:number}` |
| Z轴旋转  | rotateZ   | 沿着一个点绕 z 轴旋转一点角度    | `origin:[x,y,z]`,`angle:{start:number,end:number}` |
|缩放动画|scale|按照某个原点对弹幕经行缩放|`origin:[x,y,z]`,`scale:{x1:number,y1:number,z1:number,x2:number,y2:number,z2:number}`|

目前支持的复杂动画类型有
| 动画名称 | type类型  | 描述                   | 内部属性                          |
| -------- | --------- | ---------------------- | --------------------------------- |
|组动画|group|组动画将多个基本动画放进一个组中，实现并行播放|`animations:[]`|
|列表动画|list|列表动画将对多个动画依次播放|`animations:[]`|
|重复动画|repeat|重复执行某个动画n次|`repeat:number`,`animations:[]`|
目前，所有的复制动画类型均可嵌套其他复杂动画类型，组成更复杂的动画

例如，现在要实现一个基本的移动动画，我们依旧拿上一个弹幕作为例子，我们再此基础上加上 `animations` 属性：

```json

[
    {
        "type":"text",
        "start":0,
        "content":"这是一段40像素的红色文本",
        "style":{
            "fontSize":"40px",
            "color":"red"
        },
        "animations": [
            {
                "type":"translate",
                "duration": 3000,
                "path":{
                    "x1":100,
                    "y1":100,
                    "x2":1000,
                    "y2":600
                }
            },
            {
                "type":"translate",
                "duration": 3000,
                "path":{
                    "x2":-100,
                    "y2":-200
                }
            }
        ]
    }
]
```
[点我运行一下](https://lab.meogirl.ml/index.html?id=move)

### 子元素

子元素在 `childs` 属性内，内部内容为弹幕元素的数组，除了不能使用 `start` 属性,与上面讲的一致

子元素会根据父元素进行定位操作


### 一些简单的动画示例
[带曲线动画的文字移动](https://lab.meogirl.ml/index.html)
[红色方块移动](https://lab.meogirl.ml/index.html?id=movecube)
[立方体旋转](https://lab.meogirl.ml/index.html?id=cuberrr)