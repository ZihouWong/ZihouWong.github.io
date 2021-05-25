---

title: "利用Jekyll搭建 GitHub Pages 个人博客并绑定个人域名 --Mac"
date: 2017-04-14
categories: jekyll

layout: post
comments: true

---


就在一个星期前，我抢到了腾讯云的免费域名，所以最近一直想把自己的个人博客搭建起来。

这是我第一次搭建博客，我个人觉得还是蛮好玩的，当然我绕的路还也不少，用时大概3天吧。

把我搭建的过程记录了下来，并作为我的第一篇博客。

***

### 在此列举一下大概的流程：

一.在GitHub创建New Repository：`[username].github.io`  &&  把这个repository clone 到本地

二.在本地的	`[username].github.io` 对应文件安装Jekyll，然后push到GitHub上

三.绑定个人域名（其实这属于个人喜好）

### 在开始前我们要准备的东西：
1）终端(`Terminal` or `Iterm2`)
安装homebrew

2）vpn	

3）域名

***

你准备好了吗？

那么开始吧！

***

## 一.	
#### 1）新建 Repository

![New Repository Screenshot](/img/1-1.png)

#### 2）Clone Repository 到本地

	打开终端
	$ cd [place]
	//	进入你所需要保存Repository的文件夹    
	
 	// [] 括号不需要写到终端里
 	// [] 里的内容需要你来填写
	
	$ git clone [URL]
	
![URL Screenshot](/img/1-2.png)

这样本地就有一个文件夹 `[username].github.io`

#### 第一步就完成了。

***



## 二.
#### 安装Jekyll

	$ cd [username].github.io
	//	进入本地 [username].github.io这个文件夹

##### Jekyll是基于Ruby的，所以搭建Jekyll之前，必须先安装Ruby

##### 在使用brew安装Ruby之前，需要先安装Homebrew（`brew` 是 `Homebrew` 的简称）

##### [Homebrew-Wiki](https://en.wikipedia.org/wiki/Homebrew_(package_management_software))

##### [Install Homebrew](#install_Homebrew)
安装完Homebrew后	
	
	$ brew update
	// 更新brew所支持的软件版本信息
	$ brew install ruby
	// 使用brew（Homebrew）安装最新版Ruby 

在完成安装最新版Ruby后，则能使用 `gem` 和 `bundle` 命令了
	
	$ sudo gem install jekyll
	// sudo使用管理者权限 用gem 安装 Jekyll
	$ bundle install
[为什么需要两个install](#为什么需要两个install)

然后检验一下是否安装成功

	$ bundle exec jekyll serve	
	
若最终显示：
	
	Server running... press ctrl-c to stop.

这样表明的是你的 Jekyll 能够在本地正常运行

而你现在还需要把 Jekyll 布局到你的GitHub上
	
	$ cd [username].github.io
	// 先 cd 到 [username].github.io 这个文件夹
	$ git add -A
	$ git commit -m -a
	$ git pull
	$ git push
	
	//这个顺序不可以改变

#### tip:

`push` 时 终端会要求你输入 `[username]` && `[password]`

在输入 `[passsword]` 时,是不会显示 `********` 的, 所以大胆地输入你的 `[password]`

***

记得一定要先 `pull` 再 `push`   [But why?!](#but_why)

在 `push` 完成后

#### 那么第二步也完成了

***



## 三.
#### 1)域名解析

进入你所购买域名的管理平台，我所购买的是腾讯云
	
![Domain Name Screenshot](/img/1-3.png)

然后在命令行执行
	
	$ dig [Domain Name(域名)] +nostats +nocomments +nocmd
	
若出现了这些

![Donmain Name Result](/img/1-4.png)
	
#### 那么解析就生效了

	
#### 2) 创建CNAME.md
###### 方法一：

=> `GitHub` => `[username].github.io` => `Settings` => `GitHub Pages` => `Custom domain` => `输入域名` => `save`

###### 方法二：

=> `GitHub` => `[username].github.io` => `Create new file` => 在 `Name your file栏` 输入 `CNAME` => 在文件内容处出输入 `域名` 如：`wongzihou.cn` => `commit new file`

#### 到此你的个人博客基本就搭建完成了，然后输入你的域名看看你的博客。

EOF

## 分割线


#### <a name = "为什么需要两个install">Q1:有的人会问为什么要 `gem install jekyll` 又要 `bundle install` </a>

#### 在此感谢哥哥[wongzigii](http://github.com/wongzigii)的解释

gem install jekyll：相当于你安装博客

bundle install：相当于安装博客所需要的依赖

#### 在此感谢[温家成师兄](https://git.oschina.net/wenjiachengy)的解释

gem install jekyll: 是安装jekyll这个gem包

bundle install: 是安装Gemfile里面很多的gem包和依赖

***

#### <a name = "but_why">Q2:But why should i first-pull last-push?!</a>

原因很简单。若你和你的同伴合作在完成一个项目。

如果你先 `push` 你的本地库这时候就会有两种情况:

`1`.你的同伴没有 `update` 过远程库，你所 `push` 的文件会直接添加到远程库上

`2`.你的同伴 `update` 了远程库，但由于你本地的库没有得到及时的 `update`，文件之间由于不相同而产生了 `conflict`，所以这时候你所 `push` 的文件会在远程库里自动创建一个分支。该分支不会自动与主干合并。解决的办法:
a:该分支只能由你自己解决 `conflict` 才能合并到主干上。
b:Rebase

所以为了解决 `2` 这个问题，通常的做法是先 `pull`,把远程库的文件拉下来在本地解决好所有的冲突,在把最后的结果 `push` 到远程端去。不要把麻烦留给别人！



***

## Appendix 附录



#### <a name="install_Homebrew">install Homebrew</a>

终端运行

	$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

For further information about Homebrew, please click [Homebrew](https://brew.sh/)

***

markdown 规则


macdown app
