# 使用Blocks
一个Objective-C类描述一个包含数据和数据相关动作的对象。有时，也仅仅表示一个单一任务或者一些行为的集合，而不是一个方法的集合。  
*Blocks*是一种语言级别的特性，可以允许你创造一些特定的代码块，并像一个普通变量一样在让它方法或函数中传递。`Blocks`也是一种OC的对象，也可以添加进入`NSArray`或者`NSDictionary`。`Blocks`同时还有 在他的 *封闭作用域* 内 *捕获* 值的能力，就像其他语言的 *闭包* 或 *lambdas式* 。 
## Blocks基本语法
用一个*幂乘符号*`^`来定义一个*块*`Block`。

```
    ^{
         NSLog(@"This is a block");
    }
```

和定义一个函数或一个方法类似，用左花括号开始一段代码，并用右花括号结束这段代码。在上面的代码例子里，这个`Block`没有返回任何值，也不接受任何参数。  
就如同你可以用一个指针去 *引用*（指向） 一个C语言函数，你可以声明一个变量来跟踪一个`Block`，就像这样：

```
    void (^simpleBlock)(void);  
```

你要是不是很熟悉如何使用C语言函数指针的话，这个语法你就会觉得有点别扭。这个例子声明了一个叫做`simpleBlock`的变量去引用一个不接受任何参数也不返回任何参数的`block`，也就是说，你可以像下面一样把一个符合要求的`block`赋值给它：

```
    simpleBlock = ^{
        NSLog(@"This is a block");
    };
```
这只是一个赋值过程，就像其他变量被赋值一样，分号必须要跟在右花括号的后面。当然你也可以把声明变量和赋值两个步骤合二为一：

```
    void (^simpleBlock)(void) = ^{
        NSLog(@"This is a block");
    };
```

当你已经声明了一个`block`变量并且把一个`block`赋值给它了，你就可以直接用这个变量来*调用（invoke）*这个block。

```
    simpleBlock();
```

**注意：当你打算调用一个没有赋值的Block变量（指向nil的变量），你的程序就崩溃啦。**

### 使用接受参数并且能够返回值的Block
像函数和方法一样，`Blocks`也可以接受一些参数并且有返回值。  
举个例子，我们先声明一个能够计算两个数乘积的`Block`，很显然，它应该接受两个`double`值和返回一个`double`值：

```
    double (^multiplyTwoValues)(double, double);
```

接下来把`Block`的实现代码赋给这个变量：

```
    ^ (double firstValue, double secondValue) {
        return firstValue * secondValue;
    }
```

`firstValue`和`secondValue`代表你调用这个`block`的的时候传入的两个值，就像一个函数定义一样。在这个例子里，返回值的类型是根据`block`内的代码推测出来的。  
如果你喜欢，你也可以在`^`和左括号中间精确指定返回值的类型：

```
    ^ double (double firstValue, double secondValue) {
        return firstValue * secondValue;
    }
```

一旦你已经声明并实现了一个`Block`，你可以像调用函数一样调用它：

```
    double (^multiplyTwoValues)(double, double) =
                              ^(double firstValue, double secondValue) {
                                  return firstValue * secondValue;
                              };
 
    double result = multiplyTwoValues(2,4);
 
    NSLog(@"The result is %f", result);  //打印结果为8
```

### Blocks可以捕获封闭作用域内的值
除了包含一些可以执行的代码，一个`block`还能捕获 *封闭作用域* 内的值。  
如果你在一个方法里面声明了一个`block`，举个例子，这个`block`就可以捕获这个作用域（这里为这个方法内部）的任何量，像这样：

```
- (void)testMethod {
    int anInteger = 42;
 
    void (^testBlock)(void) = ^{
        NSLog(@"Integer is: %i", anInteger);
    };
 
    testBlock();
}
```

在这个例子里，`anInterger`是在`block`外部声明的，但是在这个`block`定义的时候这个值已经被捕获了。  
除非你特殊声明，只有变量的值被捕获了。这意味着如果你在定义`block`之后和调用它之前改变了这个值，像这样：

```
    int anInteger = 42;
 
    void (^testBlock)(void) = ^{
        NSLog(@"Integer is: %i", anInteger);
    };
 
    anInteger = 84;
 
    testBlock();
```

已经被捕获的这个值是不会被影响的。这就意味着输出依然还会显示为：

```
Integer is: 42
```

>译者注：这里直接翻译会有些晦涩难懂，你可以把捕获的过程看做拍照片。当你（变量）被捕获的时候，只有你的样子（值）被捕获了，那么当你去整容（改变变量本身）的时候，你回头再看这个照片（调用`block`），照片是不会变的（输出结果不变）。

这也意味着，`block`不可能改变原来变量的值，甚至不能改变捕获到的值（因为被捕获的值是被储存为`const`类型的）。

####使用__block变量去共享储存
如果你需要在一个`block`里去改变捕获到的值，你可以在初始变量声明之前添加`__block`修饰。这表示，这个变量存在于一块被**这变量本身存在的作用域**和**任何存在于这个作用域的`block`**两者共享的内存中。  
举个例子，你就可以像这样重写之前的例子：

```
    __block int anInteger = 42;
 
    void (^testBlock)(void) = ^{
        NSLog(@"Integer is: %i", anInteger);
    };
 
    anInteger = 84;
 
    testBlock();
```

因为`anInterger`是在声明时被`__block`修饰的变量，他的储存空间将与`block`声明共享。这意味着打印的日志将为：

```
Integer is: 84
```
这也就意味着`block`可以改变原来的值，像这样：

```
    __block int anInteger = 42;
 
    void (^testBlock)(void) = ^{
        NSLog(@"Integer is: %i", anInteger);
        anInteger = 100;
    };
 
    testBlock();
    NSLog(@"Value of original variable is now: %i", anInteger);
```

这次打印的结果将是这样：

```
Integer is: 42
Value of original variable is now: 100
```

###将Blocks作为参数传递给方法或函数
在本章中，之前的所有例子都是在定义之后直接调用了一个`block`。在实际应用中，我们经常会把`block`传递给一些方法或函数去在其他地方被 *远程调用invocation* 。举个例子，你也许会利用 *Grand Central Dispatch* 在后台调用一个`block`。或者，不断地调用一个代表单个重复任务的`block`，比如遍历一个集合。*并发性concurrency* 和 *枚举 enumeration* 将会在本章的之后内容讲到。  
`blocks`有时候也会用于回调——定义一段当某一个任务完成时才会被调用的代码。举个例子，你的应用可能会通过创造一个执行复杂工作的对象来响应用户行为，比如从一个web服务器请求一些信息。因为这个工作可能要花费很长时间，所以你需要在工作进行的时候显示一个进度条，工作完成的时候就把进度条隐藏起来。  
用 *委托* 是可以完成这个任务的：你需要创建一个合适的*委托协议 delegate protocal* ，并实现要求的方法， 并让你的对象遵守这个委托， 接下来就等待工作结束时你的对象调用代理方法。
但是使用`block`会使这件事情更加简单，因为你可以在你初始化工作的时候才定义这个回调行为，像这样：

```
- (IBAction)fetchRemoteInformation:(id)sender {
    [self showProgressIndicator];
 
    XYZWebTask *task = ...
 
    [task beginTaskWithCallbackBlock:^{
        [self hideProgressIndicator];
    }];
}
```


