本项目的目的为大家共同欧维护一个基于obsidian的共享笔记项目，大家可以在项目里一起记录自己对于这一章的看法，并且如果可以非常建议加以代码实现。可以在我`想写一个编译器`文件夹下实现这个编译器代码。

初步的一些规范：
本项目C代码风格如下：
```
asir@AsirXiaoXin /> cat .clang-format    
BasedOnStyle: Google  
IndentWidth: 4  
UseTab: Never  
BreakBeforeBraces: Attach  
ColumnLimit: 80  
PointerAlignment: Left
```

初步如此，希望大家多多维护。我至少每周天应该都会看一次的。

下面是windows上如何使用这个项目的步骤
## 把仓库dawn下来
不解释了
## 下载obsidian
官网：
https://github.com/obsidianmd/obsidian-releases/releases/download/v1.8.9/Obsidian-1.8.9.exe

打开obsidian之后可以看到
![[Pasted image 20250414203457.png]]

选择打开本地仓库，选择之前dawn下来的那个项目就可以了。然后就可以简单的使用了。下面是一些简单的仓库配置讲解

本项目暂时文件目录为：
```
编译原理/  
├── flexStudy  
│   ├── photos  
│   ├── 实验1学习.md  
│   ├── 实验2学习.md  
│   └── 编译工具快速入门.md
├── photos
├── README.d.md  
├── 我想写一个编译器  
│   ├── photos  
│   └── 一个好的开头.md  
├── 编译原理综述.md  
├── 词法分析.md  
└── 语法分析.md
```

其中photos是 存放图片的。截图之后放到里面就会自动放到对应文件夹下的photos文件夹。

在左下角的齿轮处可以看到本obsidian仓库使用的具体配置。部分如图
![[Pasted image 20250414204047.png]]

我们这里写的虽然是markdown文档。但是为了更好的使用obsidian链接的功能。在链接语法时使用的是`![[]]`，你按两次`[`就可以快速建立双向链接比较好用。

本项目默认开启了vim模式，但是有的朋友可能不熟悉vim，可以将选项关闭。

最后obsidian是可以换主题和有很多插件的，我这里使用的就
- calendar插件 日历插件
- mind map 生成思维导图的插件
- Number Headings 标题前面生成数字的插件

具体使用大家可以查一查，我有点忘了。由于个人化的设计。大家可以自定义自己的obsidian仓库。项目根目录下有一个.obsidian。大家不要删掉，在他的基础上改动吧。

还有一个.gitignore文件，里面忽略了.obsidian，所以仓库配置大家可以随便改。（但是链接方式不建议改）