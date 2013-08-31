---
title: Blog折腾
date: '2013-08-22'
description: 折腾博客
categories:
- 吐槽
tags:
- disqus
- xively
- 友言
- 加网
- GitHub
- Bootstrap
- Raspberry Pi
- 气象站
---
在程序界有一句话很流行,不要重复造轮子。哦，我不是来讨论[重复造轮子](http://taosay.net/?p=575)这件事情的。  
昨天用gor生成了自己的blog，然后我折腾的本性又显示出来了，当然要添加点自己的玩意，随着折腾进行发现自己又陷入了程序员重复造轮子的话题。

气象站数据
---
第一个想到的当然是我的[气象站](https://github.com/Puhao/ecomesh)数据问题。我现在的数据直接转发到了[Yeelink](http://www.yeelink.net/)，顺便发发[微博](http://weibo.com/u/3551343502)。[平台曲线](http://www.yeelink.net/devices/3762#)展示了我阳台最近的天气变化。  
但是我想着在这个blog也能展示曲线，我想到的就是我本来用raspberry pi采集数据的存储到本地MySQL数据库的，自己只需再写个后台服务程序，然后前端用个JQuery的库，比如Highcharts,前端还能做个ajax数据交互，实时更新。 
我看了[别人的做法](http://hugozhu.myalert.info/)（其实我更想用山寨这个词语，我最擅长的就是模仿，Orz),直接使用了第三方的物联网平台--[xively](https://xively.com/)。  
中国版的物联网开发平台相比之下，真的是模仿了它的外形，但是没有把其它一些辅助的功能做好。提供一个app-key，post接口让大家传数据，但是没有考虑到第三方网站的外链之类的。毕竟国内做物联网的平台的，boss更多是做嵌入式出身，身上少了些互联网基因。  
我知道我不应该再造个轮子了，直接用别人的工具吧。  
  
关于我的气象站，等我把其它部件整完了，我专门写篇Blog来介绍下。

评论，分享
---
原来gor里面集成了disqus，但是毕竟这个里面关联的账号啊都是twitter，facebook，天朝用户都是weibo啊，人人啊之类的账号，果然我就发现了中国版的disqus， [友言](http://www.uyan.cc/)。然后发现[加网](http://www.jiathis.com/)提供了我们最需要的一键分享功能，而且他们两个是一家公司的。  
然后我看了下，自己是不是要学下go语言，然后gor里面折腾下，让这个功能支持？还是先在github里面提交了一个issue。  

主页改动
---
当然首页风格也是需要改改的，非常好，gor使用了bootstrap这套框架，因此自己改动页面也非常方便。

增加轮子
---
前面提到的xively，disqus，友言，加网，bootstrap这些东西都非常好用，只需要在HTML里面加入一些元素就实现了，这些轮子真的做到了组装方便，让我们这些没有技术含量的“系统集成商”非常喜欢。

结尾吐槽
---
自己用人人这样的SNS，还有weibo，又搞这个blog干神马了？这不就是个重复造了个大轮子？其实我就是喜欢造轮子，一般人我不告诉他。  
