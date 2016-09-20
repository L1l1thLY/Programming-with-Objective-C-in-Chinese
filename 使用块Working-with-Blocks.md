`Blocks`是一种语言级别的特性，可以允许你创造一些特定的代码块，并像一个普通变量一样在让它方法或函数中传递。`Blocks`也是一种OC的对象，也可以添加进入`NSArray`或者`NSDictionary`。`Blocks`同时还有 *在他的封闭作用域内捕获值的能力*。 
## Blocks基本语法
- 声明（Declaration）：他可以和普通的变量一样被声明。
```
void (^myBlocksName1)(void);//不接受任何参数也不返回值的一个Block
double (^myBlocksName2)(double, double);//接受两个double类型参数并且返回一个Double值的Block
```
- 赋值（Assignment）：被声明的`Blocks`只能被相应类型的Blocks赋值。因为这是一个赋值操作而并非一个函数定义的过程，所以语句应由一个分号结尾。
```
//将前文的代码进行赋值
myBlocksName1 = ^{
    NSlog(@"I don't take any arguments and don't return any value.");
};
myBlocksName2 = ^ (double firstValue, double secondValue){
    NSlog(@"I take two arguments and return an argument.");
    return firstValue * secondValue;
}；
//将两个步骤合并亦可
void (^myBlocksName3)(void) = ^{
    NSlog(@"Fantastic?");
};
```
- 调用（Invocation）：直接调用这些`Blocks`语法有点类似调用一个普通的`C函数`。
```
myBlocksName1();
double result = myBlocksName2(3.3, 4,4);
```

***
上面我们讲解了`Blocks`的基本语法。大家可能会疑惑似乎看起来它除了声明和普通变量类似，调用类似C语法而并非我们熟悉的OC发送消息的语法之外，并没有超越普通函数的功能。那么我们为什么要使用`Blocks`呢？下面讲解`Blocks`独有的特性和应用场景。
## Blocks特性及应用场景
Blocks其实有些类似其他语言的闭包，所以它也拥有捕获值的能力和被当做参数传递的能力。
- 捕获（Capture）：Block能够捕获封闭作用域内的值  
```
- (void)testMethod {
    int anInteger = 42;
 
    void (^testBlock)(void) = ^{
        NSLog(@"Integer is: %i", anInteger);
    };
 
    anInterger = 86;
    testBlock();
    //打印出 Integer is: 42
}
```
通过这个例子可以看到当我们在一个方法里面定义一个Block，这个Block可以捕获了`方法`内、`Blocks`、外定义的变量。`Blocks`在捕获过程中保存了原有变量的一份副本（并保存为`const`类型），这也就是说，当`Blocks`捕获后改变原来变量的值，不会对Block捕获的变量（原有变量的一份副本）产生影响，同时`Blocks`本身也不能修改它们捕获到的值。这就好像每个`Block`都拥有一台照相机，`Blocks`在定义的时候，会给`封闭作用域（enclosing scope）`内在它之前定义的所有变量拍张照片(`捕获`），需要用的时候就掏出来看一看。我们可以看到，在上面的例子里面，我们把原来变量的值修改成86，并没有影响Block的输出。
- 修改捕获的值  
如果我们在某个封闭作用域内定义一个变量时使用了`__block`修饰一个变量，那么当Blocks捕获这个值的时候，这个值在内存中是被捕获它的Blocks和这个变量本身共享的，换句话说，当Blocks捕获它的时候，并没有创建这个变量的本身，而是直接使用了这个变量的本身。  
举例如下：  
```
 __block int anInteger = 42;
 
    void (^testBlock)(void) = ^{
        NSLog(@"Integer is: %i", anInteger);
    };
 
    anInteger = 84;
 
    testBlock();//打印出 Integer is: 84
```
- 将Blocks当做参数传入函数或方法  
在实际应用中，我们通常将Blocks用作参数传入方法或者函数用于远程调用（invocation）——在其他地方被方法或函数调用。
Blocks也用于回调，即定义一些在一些任务完成时被执行的代码。举个例子，比如你在执行如向一个web服务器请求数据之类的复杂行为的时候，因为这些任务会花费很多时间，所以你一般会在界面上制作一个进程指示器（小菊花之类的）来展示进程进行到了哪里，然后在任务完成的时候隐藏这个指示器。  
当然这种用法你可以通过一个委托来完成——为一个复杂任务完成时执行的方法制作一个合适的委托，并让你的复杂任务类中遵守它，并在任务完成的时候调用这个委托。  
但是用Blocks来实现会更简单一点。你可以在你的任务初始化的时候再定义这个回调函数。  
```
- (IBAction)fetchRemoteInformation:(id)sender {
    [self showProgressIndicator];      //让我们的指示器显示...比如旋转小菊花
 
    XYZWebTask *task = ...   //新建网络请求
 
    [task beginTaskWithCallbackBlock:^{
        [self hideProgressIndicator];   //在网络请求结束后隐藏小菊花
    }];
}
```
我们定义了一个没有参数和返回值的Blocks用于关闭我们的小菊花（进程指示器），然后传给了网络请求，那么根据需求我们能猜想到`beginTaskWithCallbackBlock`方法应该是新建了一个网络请求并且会在网络请求结束的时候调用我们的Blocks（这些应该利用GCD在另一个线程进行的，这样我们的界面才不会卡住，当然这里我们先不考虑这些细节，另一篇文章会详细讲一讲）。  
在继续展开之前，我们需要讨论一下强引用和弱引用的主题。  
- 强引用和弱引用  
我们可以这样看待我们在程序中所使用的任何对象、变量。即指针-内存模型。  
![指针-内存模型]()  
我们所能操作的东西即是指针，而它们所指向东西是内存的某个区域，也就可以看做是真正的对象、变量本身。我们每使用一个变量的时候会为其分配一块内存，很显然这块内存在不被需要的时候应该被释放以便于其他人使用。 
```
    SomeClass * object; //申明了一个指针
object = [[SomeClass alloc]init];//为这个指针分配了内存并初始化
Some Class * another;   //申明的另一个指针
another = object;       //另一个指针同样指向相同的实例

```
_这里叫做object指针`引用`了SomeClass的一个实例，这个实例本身存在于内存某个区域中，而另一个another指针则指向了同一个实例，我们可以看到，指针有两个，但是实例（内存中的某一块内存）只有一个。_  

关于内存回收的机制我们先不讨论，但是当一块内存被任何一个指针强引用的时候这块内存将不会被释放。相对而言，一块内存也可以被弱引用，那么`弱引用`的含义既是，当前指针可以找到我们真正的变量，既是当前指针指向储存此变量的内存，但是并不会影响内存本身的释放。如果一个内存被一个或多个指针弱引用，但是没有任何一个指针强引用，那么内存依然可以被释放。  
那么我们回到之前主题，使用Block来用于回调的情况。我们知道，在回调的时候，我们通常会对当前对象本身进行操作，比如之前的代码，我们希望能在网络请求完成的时候关闭当前对象中出现的进程指示器。  
如下。
```
- (void)configureBlock {
    self.block = ^{
        [self doSomething];    // capturing a strong reference to self
                               // creates a strong reference cycle
    };
}
```
我们在前文说到
> Blocks其实有些类似其他语言的闭包，所以它也拥有捕获值的能力。

`Blocks`在捕获值的时候会保存一份所捕获变量的副本，并施加一个`强引用`。

在这里我们传入`beginTaskWithCallbackBlock`方法是一个Block，我们可以直接看到，它捕获了`self`对象，那么此时对于Self对象，这个Block保存了一份强引用。
