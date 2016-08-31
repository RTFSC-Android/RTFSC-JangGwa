I# RTFSC-JangGwa

> Read The Fucking Source Code

## 初衷&为什么要阅读源码

随着做Android开发时间越来越久，看别人的文章博客对自己的收益越来越少，以前看10篇文章，可能9篇对自己有用，后来慢慢减少，8 7 6..1 。

再加上现在国内的风气不好，标题党特别多，质量好的文章太少，在茫茫文章中获取有用信息变得越来越困难。

投入与回报不成比例，所以需要换一种方式去学习。

现在我更推荐看书以及阅读源码。

相对于看文章，看书有利于系统的学习，看源码的好处更是多多。

书也是有好有坏，关于书籍，这边推荐一个读书笔记的项目[ReadingNotes](https://github.com/AlanCheen/ReadingNotes),记录读书笔记，也有些扩展，对书籍也有一个相对比较客观的评价,或许可以帮到你，这里就不多说了。

## 阅读源码的好处

『所有的知识其实都来自源码』是我最深的感悟。  

通过阅读源码，对知识点的掌握不再流于表面，而能够做到知其然以及所以然，极大地提升判断力，不再人云亦云。

阅读源码还能极大的扩大知识面，通常在阅读源码的时候你会发现很多你根本不知道，或者看文章博客根本不会获取得到的知识，经常会遇到各种『彩蛋』。

Android 源码是学习设计模式的最佳途径之一，Android 团队遇到的坑，比我写过的代码还多，Android 源码中到处可见设计模式的影子，阅读它，可以加深对设计模式的理解。  

好处绝不止我所说的，自己去体会。  

## 哪里可以看Android源码

Android 源码的查看一般有以下几种方式：

- 在在线网站上查看,如：[grepcode](http://grepcode.com/),[androidxref](http://androidxref.com/)  
- 获取Android Framework源码查看，clone [frameworks_base](https://github.com/android/platform_frameworks_base) ，在 Mac 端可以使用 Sublime 配合 CTAG 查看。  
- 使用 AndroidStudio 看

取合适自己的。

## 阅读源码的姿势

源码数量庞大，如果漫无目的地去阅读很容易迷失自己，所以阅读源码要有一定的技巧。

- 要有目标
- 由浅入深

比如针对某一个问题去查看源码，eg. invalidate 和 postInvalidate 的关系与区别是什么？
这样有目标性的去寻找答案，才不容易迷失。

另外阅读源码不是容易的事情，可以从简单的类开始阅读，培养阅读习惯以及技巧，增加信心，再一层一层深入，不宜在刚开始就非常深入，这样容易打击自信，甚至开始『怀疑猿生』。

## 资料 

另外这些资料可能对你有帮助：

- [大牛们是怎么阅读 Android 系统源码的？](https://www.zhihu.com/question/19759722)  
- [阅读 ANDROID 源码的一些姿势](http://kaedea.com/2016/02/09/android-about-source-code-how-to-read/)  

## 版权信息

不欢迎并拒绝任何形式的全文转载！ 

其他还在考虑。

## 目录

[EventBus 源码分析](./JangGwa浅谈EventBus.md)  

[ArrayList 源码分析](./JangGwa浅谈ArrayList.md)  

[LinkedList 源码分析](./JangGwa浅谈LinkedList.md)
