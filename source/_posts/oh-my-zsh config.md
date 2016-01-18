title: Mac下oh-my-zsh的配置，shell也要浪一点
date: 2015-07-25 18:14:25
tags:
- zsh
- Mac
---
# 一、zsh shenmegui?
&#160;&#160;&#160;&#160;使用mac的话如果不用Terminal的话，那就真的可以拿你的mac去装windows了,just kidding.但是当你代开terminal的时候看到这个bash是否会觉得要吐的赶脚呢？  
![](http://m1.yea.im/1Cl.png)  
&#160;&#160;&#160;&#160;是否有种想给它换点料的赶脚呢？just like this。。。  

![](http://m1.yea.im/1Cn.png)  
&#160;&#160;&#160;&#160;是不是绝的突然对shell就充满了基情。
  
&#160;&#160;&#160;&#160;其实如果要达到这种效果就必须说下zsh。
>Linux/Unix提供了很多种Shell，为毛要这么多Shell？难道用来炒着吃么？那我问你，你同类型的衣服怎么有那么多件？花色，质地还不一样。写程序比买衣服复杂多了，而且程序员往往负责把复杂的事情搞简单，简单的事情搞复杂。牛程序员看到不爽的Shell，就会自己重新写一套，慢慢形成了一些标准，常用的Shell有这么几种，sh、bash、csh等，想知道你的系统有几种shell，可以通过以下命令查看：  
```sh
cat /etc/shells```
>显示如下：
```sh
/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh```
>&#160;&#160;&#160;&#160;在 Linux 里执行这个命令和 Mac 略有不同，你会发现 Mac 多了一个 zsh，也就是说 OS X 系统预装了个 zsh，这是个神马 Shell 呢？  
>&#160;&#160;&#160;&#160;常用的 Linux 系统和 OS X 系统的默认 Shell 都是 bash，但是真正强大的 Shell 是深藏不露的 zsh， 这货绝对是马车中的跑车，跑车中的飞行车，史称『终极 Shell』，但是由于配置过于复杂，所以初期无人问津，很多人跑过来看看 zsh 的配置指南，什么都不说转身就走了。直到有一天，国外有个穷极无聊的程序员开发出了一个能够让你快速上手的zsh项目，叫做「oh my zsh」，Github 网址是：<https://github.com/robbyrussell/oh-my-zsh>。这玩意就像「X天叫你学会 C++」系列，可以让你神功速成，而且是真的。  
 
&#160;&#160;&#160;&#160;以上摘自知乎专栏关于zsh的介绍<http://zhuanlan.zhihu.com/mactalk/19556676>，是不是突然觉得很拉风，Mac程序狗们有没有心动。
__________________
# 二、how to config it
* **首先要保证你的电脑装了git(好废话，mac预装了)，然后ssh配置ok,反正保证你能从Github克隆代(xiao)码(dian)库(ying)下来~**  
* **官方其实提供了两种安装方式，一种是：**  
 ```sh
 wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | sh```
or  
```sh
curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh```

他们都叫它自动安装，但是不怎么推荐，因为Github有点被墙，这样子玩如果你不搞个vpn估计还是有得你等了。  
还有一种是:  
```sh
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc```
比较推荐手动安装使用ssh.git，这样下载速度有保障，记得敲完命令，要不要被坑哭。  
  
* **最后还有一步就是把你bash更换成zsh。** 

```zsh
chsh -s /bin/zsh```
一切顺利的话重启你的terminal就可以看到oh-my-zsh了:)!
________________
# 三、Custom it
&#160;&#160;&#160;&#160;最后来讲讲优化它，这里只讲如何更换主题，首先回到根目录。然后使用  
```sh
vim .zshrc```
&#160;&#160;&#160;&#160;进行编辑,更改其中的主题为你想要的名字。    
```sh
ZSH_THEME="cloud"```
&#160;&#160;&#160;&#160;官方<https://github.com/robbyrussell/oh-my-zsh/wiki/Themes>提供了许多包含的主题给你选择。**注意更换为相应的名字就行，如果需要使用额外新的主题可以将theme进行配置后更换使用**，最后再搭配合适的背景色，就可以初步打造一款属于你的zsh了。   
&#160;&#160;&#160;&#160;其实oh-my-zsh还可以做的很多，比如更改其中的图标，写写自己的theme,以及config plugins等等。。。有兴趣或者蛋疼了不妨fork一下，然后写写属于自己的东西，pull成功后还有机会获得他们赠送的T-Shirt哦。






  


