# 使用Blocks
一个Objective-C类描述一个包含数据和数据相关动作的对象。有时，也仅仅表示一个单一任务或者一些行为的集合，而不是一堆方法的集合。  

*Blocks* 是一种语言级别的特性，可以允许你创造一些特定的代码块，并像一个普通变量一样在让它方法或函数中传递。`Blocks`也是一种OC的对象，也可以添加进入`NSArray`或者`NSDictionary`。`Blocks`同时还有 在他的 *封闭作用域* 内 *捕获* 值的能力，就像其他语言的 *闭包* 或 *lambdas式* 。  

这个章节描述了声明和引用`block`的语法，并且展示了如何去使用`block`来简化普通的任务，诸如遍历集合，更多信息请参阅[Blocks Programming Topics](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502)。

## Blocks基本语法
用一个*幂乘符号*`^`来定义一个*块*`Block`。

```
    ^{
         NSLog(@"This is a block");
    }
```

和定义一个函数或一个方法类似，用左花括号开始一段代码，并用右花括号结束这段代码。在上面的代码例子里，这个`Block`没有返回任何值，也不接受任何参数。  

就如同你可以用一个指针去 *引用*（有时也叫做指向） 一个C语言函数，你可以声明一个变量来跟踪一个`Block`，就像这样：

```
    void (^simpleBlock)(void);  
```

要是不是很熟悉如何使用C语言函数指针的话，这个语法你就会觉得有点别扭。这个例子声明了一个叫做`simpleBlock`的变量去引用一个不接受任何参数也不返回任何参数的`block`，也就是说，你可以像下面一样把一个符合要求的`block`赋值给它：

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

>注意：当你打算调用一个没有赋值的Block变量（指向nil的变量），你的程序就崩溃啦。

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
在本章中，之前的所有例子都是在定义之后直接调用了一个`block`。在实际应用中，我们经常会把`block`传递给一些方法或函数去在其他地方被 *调用invocation* 。举个例子，你也许会利用 *Grand Central Dispatch* 在后台调用一个`block`。或者，不断地调用一个代表单个重复任务的`block`，比如遍历一个集合。*并发性concurrency* 和 *遍历 enumeration* 将会在本章的之后内容讲到。   

`blocks`有时候也会用于回调——定义一段当某一个任务完成时才会被调用的代码。举个例子，你的应用可能会通过创造一个执行复杂工作的对象来响应用户行为，比如从一个web服务器请求一些信息。因为这个工作可能要花费很长时间，所以你需要在工作进行的时候显示一个进度条，工作完成的时候就把进度条隐藏起来。    

用 *委托* 是可以实现这个功能的：你需要（在工作中）创建一个合适的*委托协议 delegate protocal* ，并实现要求的方法（如显示和关闭进度条），让你的对象成为这个工作的委托人。 接下来就等待工作结束时你对象中实现的委托方法被调用。  

>注：这里指的是在执行复杂任务的对象A中定义一个委托，对象A在进行复杂任务时中会分别调用委托中的方法，当然这些方法是由你的对象B实现的，也就是说对象A仅决定这些方法调用的时机，而不决定做什么（因为委托给B对象做了），而你的对象B则决定做什么（比如打开或关闭进度条），详情请参考前文使用委托和代理Working with Protocols。

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

这个例子调用了一个方法去显示一个进度条，接着创建了一个任务并且命令它开始执行。这个回调`block`规定了当任务完成的时候执行的代码；在这个情形下，它只调用了一个方法去关闭进度条。我们注意到，为了能够调用`hideProgressIndicator`方法，这个回调`block`捕获了`self`对象。当你捕获`self`的时候，一定要小心，因为你极容易创造了一个 *强引用循环strong reference cycle* ，关于如何避免强循环引用的内容会在之后详细描述。  

使用`block`可以让你清晰地看出工作前后发生了什么，这提升了代码的可读性，因为你不需要在委托方法中不断跳转去搞清楚什么将要发生。  

声明一个`beginTaskWithCallbackBlock:`方法这么写：

```
- (void)beginTaskWithCallbackBlock:(void (^)(void))callbackBlock;
```

` (void (^)(void))`规定了方法接受的参数是一个不接受任何参数同时也不返回任何参数的`block`. 在这个方法的实现中，你可以像平时调用一个`block`一样去调用这个传入的`block`参数：

```
- (void)beginTaskWithCallbackBlock:(void (^)(void))callbackBlock {
    ...
    callbackBlock();
}
```

一个接受带参数`block`的方法这么写。

```
- (void)doSomethingWithBlock:(void (^)(double, double))block {
    ...
    block(21.0, 2.0);
}
```

####Block应该作为一个方法的最后一个参数
最好一个方法只接受至多一个`block`参数。如果方法也接受其他不是`block`的参数，`block`应该成为最后一个参数。

```
- (void)beginTaskWithName:(NSString *)name completion:(void(^)(void))callback;
```

这样写，当你内嵌形式去定义一个`block`时，会让你的方法调用更加容易看懂：

```
[self beginTaskWithName:@"MyTask" completion:^{
        NSLog(@"The task is complete");
    }];
```

###使用类型定义（Type Definition）简化定义Block的语法
如果你需要定义多个拥有相同signature的`block`，那么就为了这个signature定义一个你自己的类型。  

举个例子，你可以为一个既不接受数值也不返回数值的`block`定义一个类型，就像这样：  

```
typedef void (^XYZSimpleBlock)(void);
```

你可以使用在创建一个变量或者限定方法接受的参数类型时候使用这个自定义类型：

```
    XYZSimpleBlock anotherBlock = ^{
        ...
    };
```

```
- (void)beginFetchWithCallbackBlock:(XYZSimpleBlock)callbackBlock {
    ...
    callbackBlock();
}
```

当你使用接受一个`block`或者返回一个`block`的`block`的时候，自定义类型就非常有用了。看看下面这个例子：

```
void (^(^complexBlock)(void (^)(void)))(void) = ^ (void (^aBlock)(void)) {
    ...
    return ^{
        ...
    };
};
```

这个`complexBlock`变量引用了一个接受另一个`block`作为参数并且返回另一个`block`的`block`。  
那么用自定义类型来重写这段代码就更容易读懂了：

```
XYZSimpleBlock (^betterBlock)(XYZSimpleBlock) = ^ (XYZSimpleBlock aBlock) {
    ...
    return ^{
        ...
    };
};
```

###给对象添加block属性
给对象添加一个`block`属性和定义一个`block`变量非常类似：

```
@interface XYZObject : NSObject
@property (copy) void (^blockProperty)(void);
@end
```

>注意： 你应该给这个属性添加`copy`特征，因为一个`block`需要被复制一份以便于追踪它在原来作用域外的捕获状态。这些都是自动完成的，所以你在使用自动引用计数的时候不需要考虑这些。最好为这种属性添加这个特征，因为从实际行为来看，这样是最好的。获取更多信息请查阅[Blocks Programming Topics](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502)。

一个`block`属性会像其他`block`变量一样被设置或调用。

```
    self.blockProperty = ^{
        ...
    };
    self.blockProperty();
```

也可以使用类型定义来声明一个`block`属性，像这样：

```
typedef void (^XYZSimpleBlock)(void);
 
@interface XYZObject : NSObject
@property (copy) XYZSimpleBlock blockProperty;
@end
```

###当捕获self时避免强引用循环
如果你需要在`block`中捕获`self`，比如说定义一个回调`block`，你一定要考虑内存管理中可能发生的细节。  

`block`会强引用任何它捕获的对象，包括`self`。这也就意味着它很容易导致强引用循环，举个例子，一个对象拥有一个`copy`特征的`block`属性，这个`block`捕获了`self`：

```
@interface XYZBlockKeeper : NSObject
@property (copy) void (^block)(void);
@end
```

```
@implementation XYZBlockKeeper
- (void)configureBlock {
    self.block = ^{
        [self doSomething];    // capturing a strong reference to self
                               // creates a strong reference cycle
    };
}
...
@end
```

对于这个简单的例子，编译器会发出警告。但是如果是更加复杂的例子，可能会包含多个对象间的强引用循环，这样编译器就很难诊断出问题。  

为了避免这个问题，最好是弱引用`self`，像这样：

```
- (void)configureBlock {
    XYZBlockKeeper * __weak weakSelf = self;
    self.block = ^{
        [weakSelf doSomething];   // 捕获了弱引用
                                  // 避免了引用循环
    }
}
```

因为捕获的是对于`self`的弱引用，这个`block`就不会再持有关于`XYZBlockKeeper`的强引用关系了。如果这个对象在`block`被调用前释放了，`weakSelf`指针会被设置为`nil`。

##Block可以简化遍历
「」，许多Cocoa 和 Cocoa Touch API使用`block`来简化普通任务，比如说遍历集合。举个例子，`NSArray`类提供三个基于`block`的方法，包括：

```
- (void)enumerateObjectsUsingBlock:(void (^)(id obj, NSUInteger idx, BOOL *stop))block;
```

这个方法接受一个`block`参数，这个`block`会为数组中每一个元素都调用一遍：

```
    NSArray *array = ...
    [array enumerateObjectsUsingBlock:^ (id obj, NSUInteger idx, BOOL *stop) {
        NSLog(@"Object at index %lu is %@", idx, obj);
    }];
```

这个`block`本身接受三个参数，前两个指向当前元素本身及其在数组中的次序位置。第三个参数是指向一个布尔型变量，你可以使用这个变量来停止遍历，像这样：

```
    [array enumerateObjectsUsingBlock:^ (id obj, NSUInteger idx, BOOL *stop) {
        if (...) {
            *stop = YES;
        }
    }];
```

也可以使用`enumerateObjectsWithOptions:usingBlock:`方法自定义枚举。举个例子，通过使用`NSEnumerationReverse`选项可以逆序浏览集合。  

如果在`block`中的代码是处理器密集型并且并发安全，你可以使用`NSEnumerationConcurrent`选项：

```
    [array enumerateObjectsWithOptions:NSEnumerationConcurrent
                            usingBlock:^ (id obj, NSUInteger idx, BOOL *stop) {
        ...
    }];
```

这个标志表示遍历`block`的调用工作会分发给多个线程执行。如果`block`的代码是处理器密集型，这就提供了潜在的性能提升。要注意，使用这个选项后，遍历的顺序就不一定了。  

`NSDictionary`类也提供了基于`block`的方法，包括：

```
    NSDictionary *dictionary = ...
    [dictionary enumerateKeysAndObjectsUsingBlock:^ (id key, id obj, BOOL *stop) {
        NSLog(@"key: %@, value: %@", key, obj);
    }];
```

比使用传统循环来遍历每个键值对更加方便。

##Block可以简化并发工作
`block`代表一个确定清晰地工作，既包含了可执行的代码，也可能捕获了所在作用域的状态。由于这些特性，`block`在使用OSX和iOS提供的并发选项完成一些异步的调用的场合中，就变得非常理想。你可以简单地
使用`block`定义你的任务然后让系统在处理器资源可用的时候执行这些任务，而不需要考虑如何使用低级的机制比如说线程。  

OSX和iOS提供多样的并发工作技术，包括两个任务计划机制：*Operation queues* 和 *Grand Central Dispatch*。这些机制都围绕着一个等待着调用的任务队列。为了让你的`block`被调用，把它们加入队列中，然后系统会在CPU时间和资源可用的时候把这些任务从队列中取出以便于调用。  

*串行队列（serial queue）* 同一时间仅允许一个任务执行——前一个任务已经完成之前，队列中下一个任务不会被出队执行。一个*并发队列（concurrent queue）*会执行尽可能多的任务，不会等待之前的任务结束。  

###在操作队列中使用`block`操作
Cocoa和Cocoa Touch使用`operation queue（操作队列）`来安排任务执行。你创建一个`NSOperation`实例来封装一系列的工作以及必要的数据，接着将这个操作（operation）加入`NSOperationQueue`实例。  

你既可以创造自定义的`NSOperation`子类实现复杂的任务，也可以使用`NSBlockOperation`创建一个使用`block`的操作，像这样：

```
NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
    ...
}];
```

你可以手动执行一个操作，但一般都会将操作加入一个队列，这个队列可以是一个已经存在的队列，也可以是你自己创建的队列，准备好执行：

```
// schedule task on main queue:
NSOperationQueue *mainQueue = [NSOperationQueue mainQueue];
[mainQueue addOperation:operation];
 
// schedule task on background queue:
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
[queue addOperation:operation];
```

如果你使用操作队列，你可以定义操作的优先级和依赖关系，比如说，你可以规定一个操作必须在一系列的其他操作结束之后才能执行。你也可以通过KVO（key-value observing键值观察）监测你操作的状态变化。更多关于操作及操作队列的内容，请看[Operation Queues](https://developer.apple.com/library/content/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationObjects/OperationObjects.html#//apple_ref/doc/uid/TP40008091-CH101)。  

###使用Grand Central Dispatch在调度队列（dispatch queue）上安排block
如果你需要安排任意的`block`执行，你可以直接使用调度队列，这个调度队列是在GCD控制下的。无论是同步还是异步操作，调度队列都给调用者带来很多便利，同时，这些操作会按照先入先出的顺序执行。  

你既可以自己创建一个调度队列，也可以使用GCD自动提供的队列。举个例子，你想要安排一个并发执行的任务，你可以使用`dispatch_get_global_queue() `函数获得一个已经存在的调度队列的引用，同时定义队列的优先级，像这样：

```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
```

谈及队列上调度`block`，你既可以使用`dispatch_async()`也可以使用`dispatch_sync()`。`dispatch_async()`函数会立刻返回，不会等待`block`被调用。

```
dispatch_async(queue, ^{
    NSLog(@"Block for asynchronous execution");
});
```

`dispatch_sync()`不会立刻返回，它会等到`block`结束执行后再返回。举个例子，在当一个并发执行的`block`需要等待另一个处于主线程的任务结束后才能继续执行的情况下，就可以用这个函数了。  

更多关于GCD和调度队列的内容，请看[Dispatch Queues](https://developer.apple.com/library/content/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationQueues/OperationQueues.html#//apple_ref/doc/uid/TP40008091-CH102)。




