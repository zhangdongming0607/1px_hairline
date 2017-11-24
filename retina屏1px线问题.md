## retina屏1px线问题的解决
写在前面：这篇文章猪样讲述了之前对一个工程问题的讲解，勉强算是对项目的某个环节做的优化。        
背景：css像素和retina屏的物理像素并不是一一对应关系。这样我们写css时1px时，在retina屏上实际占两个设备像素。
这个 [demo](https://jsfiddle.net/zhangdongming/qhunr0f4/3/) 可以看出 css的 1px 和物理 1px 的区别
## 可能的方案
### 0.5px
```
.border-1px {
  border-width: 0.5px;
}
```
但是这样写在某些浏览器是不识别的，一种解决办法是=像下面这样写一些兼容代码（参考来源 https://www.zhihu.com/question/25630295/answer/35182037）：
// 判断当前为ios8系统就会在html上增加一个class="ios-gt-7"，然后样式中
```
.ios-gt-7 .item-inner{border-bottom-width: 0.5px;}；
.item-inner{border-bottom-width: 1px;}
```

优点：「原生」。
缺点：暂时只有 FF 和 Safari 支持这种写法。

### css transform
```
.border-1px:before {
  position: absolute;
  content: '';
  top: -50%;
  bottom: -50%;
  left: -50%;
  right: -50%;
  transform: scale(0.5);
  border-top: 1px solid #666;
  border-bottom: 1px solid #666;
}
```

原理大概是在bofore或者after伪类中再画一个无背景dom元素，利用transform:scale(0.5)实现0.5px边框，然后用调整poistion方式来把大小和位置做成和需添加边框元素一样。
优点：纯css实现，而且看起来对兼容性没有什么要求，测试结果在低分屏上会退化成1px。
缺点：依赖dom和伪类实现，需要定位，要注意检查是否和当前样式有冲突。
### border-image
首先要准备一张图片， 如果只是实现上下边框。宽度1px即可。实现上边框的例子如下：
```
.border-image-1px {
  border-width: 1px 0;
  border-color: #666666;
  border-style: solid;
  border-image:
     url(“1px-border.svg”) 2 0 stretch;
}
```
大概原理是用1px宽有色和1px透明的图片填充边框。	
优点：可以不依赖伪类 + position 定位实现。	
缺点：FF上不支持，如果想实现修改边框颜色之类的需求比较麻烦。
### viewport + rem

这种方法没有具体测试，虽然修改比较彻底，但是看起来需要对整个项目都做较大的调整。参见： https://github.com/amfe/article/issues/17

# 解决方案

综合考虑，最终采用了 0.5px + border-image 实现的方案，即在支持 0.5px 的 safari 和 firefox 上使用0.5px，而在其他浏览器上使用 border-image。	
1.在 render html 时判断 userAgent，如果不是的话就在html标签上添加tag：data-hairiline = 'true'
2.在 postcss 中添加了postcss-mixins。这样可以极大地地简化代码。	
通过在 postcss 中的定义，可以定义多个变量来分别实现上下左右多个边框。（因为每个className的border-image-source不同，所以要按需选择其中一个，而不是组合多个。）	
如果有比较特殊的边框需求，比如只是不想要 right-border ，可以用 @mixin hairline 1 0 1 1 这样的写法实现。原理其实很简单，准备一个四周极细边框的图片
```
url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='3' height='3'> <rect x='0' y='0' width='3' height='.5' /> <rect x='0' y='2.5' width='3' height='.5' /> <rect x='0' y='0' width='.5' height='3' /> <rect x='2.5' y='0' width='.5' height='3' /> </svg>
```
然后给这个mixin写一个方法，将mixin的参数转化成 border-width 的值即可。

这样，在不对项目做大改动的前提下，保证开发人员不用纠结于具体的实现，又可以保证每个浏览器用上了最好的方案。
