---
title: android-resource
date: 2018-10-15 00:06:55
tags:
---

### 资源引用

| 前缀           | 作用             |
| -------------- | ---------------- |
| @              | 自定义资源       |
| @android:type  | 系统资源         |
| @*android:type | 系统非public资源 |
| @+             | 创建或引用资源   |
| ？             | 引用主题资源     |

#### 在代码中引用资源

View在xml中的属性提供了的测量和布局的信息，绘制的信息也可以从属性获取。在android命名空间下，最常见用于绘制的属性如View的background,ImageView的src和scaleType,以及TextView的各种字体属性。也可以在自定义View中定义View的属性(declare-stylable)。这些属性就是我们所谓的样式,样式还可以通过style属性引用。

自定义View怎么加ID?

1. 方案一

 调用View.generateViewId()作为setId的参数 (必须为SDK版本17以上)

my_view.setId(View.generateViewId());

2. 方案二：

在res/values/下添加ids.xml(名字可随意)文件

~~~
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <item name="my_view" type="id" />
</resources>
~~~

资源类型

类型  描述
string  字符串
string-array    字符串数组
plurals 字符串
array   数组
color   颜色
dimen   尺寸
bool    基本类型
integer 基本类型
style   包括窗口和控件的样式
declare-styleable   属性数组，其实就是一系列共性的属性
attr    定义属性
public  
java-symbol 

#### 关于属性资源

可以从资源的用途这一角度出发，就能理解"属性"资源的意义。一般资源都以值的形式供代码内或xml内使用。而属性则是以键和索引的形式供使用，动态的为使用者增加一个"槽"来填装别的资源。使用者也不局限于View。比如清单文件Application和各组件的属性(android:xxx)，其实也是在Framework中定义的，反映在xml上就是命名空间

#### 关于public

public三个参数type name id,其实就是已有资源的映射，作用是固定该资源ID，外部引用时能准确定位资源

