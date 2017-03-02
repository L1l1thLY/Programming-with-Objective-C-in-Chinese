# Programming-with-Objective-C-in-Chinese

This is a Chinese version of Programming with Objective-C that is translated by Innovation Studio iOS Group in USETC. 苹果开发者平台Objective-C文档的中文翻译版本。
##贡献者:
- Chibaibuki XiangfuGoh
- Caoxian HaoxianChan
- Bomlsy SiyuLiao 
- tinoryj YanjingRen
- yaoyai JianyunWu
- toryznoco

##测试版本V0.1
本项目欢迎提出意见，如对任何章节有意见或疑问请发送邮件至chibaibuki@outlook.com，我们会在两个工日内对问题进行回复并可能修改相应章节。
##章节认领情况
[原文地址](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011210-CH1-SW1)

| 章节 | 认领者  |  
| --- | --- | 
| Introduction-介绍 | yaoyai|
|   Defining-Classes-定义类|Caoxian|
|Working-with-Objects-对象的使用|Bomlsy|
|Encapsulating-Data-数据的封装|Chibaibuki|
|Customizing-Existing-Classes-定制已有的类|Caoxian|
|Working-with-Protocols-协议的使用|tinoryj|
|Values-and-Collections-值与集合类型|tinoryj|
|Working-with-Blocks-使用块|Chibaibuki|
|Dealing-with-Errors-错误处理|toryznoco|
|Conventions-命名规则|Bomlsy|

##流程

1. 章节认领，共10章
    -  Introduction-介绍
    2. Defining-Classes-定义类
    3. Working-with-Objects-对象的使用
    4. Encapsulating-Data-数据的封装
    5. Customizing-Existing-Classes-定制已有的类
    6. Working-with-Protocols-协议的使用
    7. Values-and-Collections-值与集合类型
    8. Working-with-Blocks-使用块
    9. Dealing-with-Errors-错误处理
    10. Conventions-命名规则
2. 各自`Fork`[Github-原工程](https://github.com/L1l1thLY/Programming-with-Objective-C-in-Chinese) 到自己的Github上并建立`dev`分支。将润色排版好的章节推送至原工程`dev`分支。
3. 其他翻译者校对审核原工程dev分支并进行修改，无误后推送至原工程`dev`分支。
4. 将`dev`分支合并进入`master`分支。

##排版约定
本文档使用`Markdown`书写，排版约定如下。

- 普通字体：用于书写正文。如：
因为这是一个赋值操作而并非一个函数定义的过程，所以语句应由一个分号结尾。

- *斜体*：用于指示第一次出现的术语、E-mail地址、文件名。文件扩展名、路径名、目录、以及Unix工具。对于较为晦涩的术语，**翻译者应加括号并简单解释**。如：
*Objective-C*是你写*OS X（maxOS）程序*和*iOS*程序的首选语言。它作为C语言的一个*超集*（即C语言是它的一部分）提供了面对对象的特性和*动态运行时runtime*。

    
- `等宽字体`：用于指示命令、命令选项、命令开关、变量、属性、键值、函数、类型、类、名字空间、方法、模块、成员属性、参数、值、对象、事件、事件处理器、XML标签、HTML标签、宏、文件内容或命令的输出结果。如：

 `Blocks`是一种语言级别的特性，可以允许你创造一些特定的代码块，并像一个普通变量一样在让它方法或函数中传递。Blocks也是一种OC的对象，也可以添加进入`NSArray`或者`NSDictionary`。
 
```
    myBlocksName1 = ^{
    NSlog(@"I don't take any arguments and don't return any value.");
    }
```
- **粗体**：用于强调；用于显示命令或其他应该由用户逐字输入的文本。

- 标题尺寸：对于文章中大标题使用`#`一级标题，其他则根据情况使用二级标题`##`、三级标题`###`或更小。

