## retina屏1px线问题的解决
写在前面：这篇文章猪样讲述了之前对一个工程问题的讲解，勉强算是对项目的某个环节做的优化。        
背景：css像素和retina屏的物理像素并不是一一对应关系。这样我们写css时1px时，在retina屏上实际占两个设备像素。
这个 [demo](https://jsfiddle.net/zhangdongming/qhunr0f4/3/) 可以看出 css的1px和物理1px 的区别
## 可能的方案
```
0.5px
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

## css transform
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
之前 heifetz 中的实现方式。首先要准备一张图片， 如果只是实现上下边框，可以用类似这样的图： ,宽度1px即可。实现上边框的例子如下：

.border-image-1px {
  border-width: 1px 0;
  border-color: #666666;
  border-style: solid;
  border-image:
     url(“1px-border.svg”) 2 0 stretch;
}
大概原理是用1px宽有色和1px透明的图片填充边框。之前 heifetz 里把 border-image 的值用 postcss 处理了下，css中写一个 $thin-border  这样的变量就可以。
优点：可以不依赖伪类 + position 定位实现。
缺点：FF上不支持，如果想实现修改边框颜色之类的需求比较麻烦。
## viewport + rem

这种方法没有具体测试，虽然修改比较彻底，但是看起来需要对整个项目都做较大的调整。参见： https://github.com/amfe/article/issues/17
