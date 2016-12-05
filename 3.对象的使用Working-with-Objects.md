#对象的使用：
在 *Objective－C* 的应用中，多数工作是通过消息的送回和直接传达到对象群的形式实现的。对象群中的部分对象是由 *Cocoa(Cocoa Touch)* 所创建的类的实例，而另一部分则是由我们自己创建的类生成的实例。
前一章描述了类的实现方法以及在类中定义接口的语法，包括用于响应消息的方法的语法规则。这一章我们将学习怎样向对象发送这样一个消息，包括 *Objective-C* 的一些动态特性（这些特性包括：动态类型、决定在运行时中哪一种方法应该被调用的能力）。
在一个对象被使用之前，它的创建必须满足：具有分配给属性的内存空间、内部数据进行一切必要的初始化这两点。这一章节描述了：为了确保一个对象被正确配置，我们如何将这个分配内存和初始化一个对象的方法嵌套其中的语法规则。
##Objects接受、发送消息
即使 *Objective-C* 中有多种在对象之间发送消息的方法，但目前最主流的方法仍然是运用方括号，例如：

```
[someObject doSomething];
```

左边的`someObject`在这个例子中，是作为消息的接收者。右边的`doSomething`，是消息的名称。换句话说，当上面的这行代码被执行，`someObject`将会收到 `doSomething` 这条消息。

前一章像我们介绍了如何为一个类创建接口，例如:	

```
@interface XYZPerson : NSObject
- (void)sayHello;
@end
```
   

以及如何实现一个类，例如:	

```
@implementation XYZPerson
- (void)sayHello {
    NSLog(@"Hello, world!");
}
@end
```

>注意：这个例子使用了一个 *Objective-C* 的字符串，字符串是 *Objective-C* 中一种可以直接使用速记语法创建的类型。我们需要明白，`@"Hello, world!"`在概念上等同于说“一个 *Objective-C* 字符串型对象代表了 *Hello，World！* 这句话。”

>字符和对象的创建在本章的下一小节[Objects Are Created Dynamically]()中会有更详细地介绍。		

假设你已经创建了`XYZPerson`这个对象，你可以像这样向它发送消息`Say Hello`，例如：

```
[somePerson sayHello];
```

在 *Objective-C* 中发送一条消息在概念上和在 *C* 语言中调用一个函数是一样的。图片2-1展示了发送消息`sayHello`的过程。
	
**图片2-1** 消息发送的基本方法的程序流程
	
![programflow1](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Art/programflow1.png)  

为了明确消息的接受者，必不可少的一点是：我们需要理解在 *Objective-C* 中指针指向对象的工作原理。 

###使用指针跟踪对象
就像其他编程语言那样，*C* 语言和 *Objective-C* 中使用变量记录值。
这里记录了一些在标准 *C* 中最基本的数值变量类型，包括 *整型* 变量，*浮点型* 变量，*字符型* 变量。声明并赋初始值的方法如下：	

```
int someInteger = 42;
float someFloatingPointNumber = 3.14f;
```

局部变量，在方法或函数中声明的变量，举例如下：
	
```
- (void)myMethod {
    int someInteger = 42;
}
```

那么它（上例中的变量）的作用范围是受限制的。

在这个例子中，`someInteger`被声明成一个在`myMethod`中的局部变量；一旦程序执行到了这个方法的大括弧，即｛ ｝中的代码，`someInteger`将不能使用（？）。		
当一个局部数值变量（例如`int`或者`float`）被释放，它的数值也会随之消失。

*Objective-C* 中的对象，与其它编程语言相比，在初始化时有一些细微的不同。一个对象往往不会只仅仅调用一个方法后就不再使用。尤其是，对象常常需要存在比它的原始变量更长的时间，因此，我们常常创建的是一个动态的对象（在分配内存和初始化时都是动态的）。

>注意：如果你十分了解 *堆* 和 *栈* 的话，我们可以这样解释：局部变量定义在 *栈* 上，而对象定义在 *堆* 中。

这就要求你使用一个 *C* 语言中的指针（一个用来记录变量在内存中的地址的量）来记录对象在内存中的位置。例如：	

```
- (void)myMethod {
    NSString *myString = // get a string from somewhere...
    [...]
}
```

Although the scope of the pointer variable myString (the asterisk indicates it’s a pointer) is limited to the scope of myMethod, the actual string object that it points to in memory may have a longer life outside that scope. It might already exist, or you might need to pass the object around in additional method calls, for example.(?)

###使用对象作为方法的参数
当你发送消息时需要传递一个参数，而这个参数是对象的话，你应该提供给这个对象一个指针，这个指针指向方法的某个参数。在前一章中，我们提供了创建只有一个参数的方法的代码示例：

```
- (void)someMethodWithValue:(SomeType)value;
```

举一反三，现在我们能够写出这样的代码：仅使用一个指针型对象作为参数的函数。示例如下：

```
- (void)saySomething:(NSString *)greeting;
```

接下来我们尝试实现`saySomething`这个函数，示例如下：

```
- (void)saySomething:(NSString *)greeting {
    NSLog(@"%@", greeting);
}
```

`greeting`指针在这里可以当作一个局部变量，并且它（`greeting`指针）只在`saySomething`的作用域中。当这个函数（`saySomething`）被调用时，这个指针指向的对象会被优先处理，并且在方法执行完成后这个对象仍然继续工作。(The greeting pointer behaves like a local variable and is limited in scope just to the saySomething: method, even though the actual string object that it points to existed prior to the method being called, and will continue to exist after the method completes.)

>注意：`NSLog()`使用了格式说明符来表明替换标记（sustitution tokens），就像标准 *C* 中的库函数`printf{}`。这个字符串之所以能连接到控制台,是因为格式化这个格式化字符串（第一个参数）通过插入一个提供给我们的值（其余参数）。
（NSLog() uses format specifiers to indicate substitution tokens, just like the C standard library printf() function. The string logged to the console is the result of modifying the format string (the first argument) by inserting the provided values (the remaining arguments.)

>在 *Objective-C* 中，我们增加了一个在 *C* 语言中没有的格式转换符,`%@`,它被用于指示一个对象。在程序运行时，调用方法`descriptionWithLocale`（如果这个函数存在）以及方法`description`作用于对象将代替这个格式说明符（`%@`）。方法`description`被类`NSObject`调用，来返回对象的类以及对象的内存地址。但是许多 *Cocoa* 和 *Cocoa Touch* 的类方法会覆写方法`description`（该方法不被执行），来获取更多有用的信息（屏蔽无用的信息）。比如在类`NSString`中，方法`description`只返回它的特征值。

###方法可以返回值
一个方法（函数）不仅能传递参数，同时也能返回一个参数。到目前为止，这一章中我们列举的每一个方法（函数）都有一个`void`类型的返回值。这个 *C* 语言中的关键字`void`表明这个方法（函数）不返回任何值。

定义一个`int`类型的返回值表明这个方法（函数）返回一个整型值，例如：

```
- (int)magicNumber;
```

在这个方法（函数）的实现代码中我们使用了一个 *C* 语言中的保留字`return`，表明当这个方法（函数）被执行后应该传回一个值（`return`后紧跟的那个数值），代码示例如下：

```
- (int)magicNumber {
    return 42;
}

```

这个例子完美地解释了我们之前一直忽略的一件事：函数可以返回一个值。(It’s perfectly acceptable to ignore the fact that a method returns a value.)在这个例子中，方法` magicNumber`除了返回一个值以外什么也没实现，但是像这样调用这个方法（函数）也没有任何问题：
	
```
[someObject magicNumber];
```	

如果你要跟踪这个返回值，你可以定义一个变量，并且将这个函数调用的结果赋值给这个变量，例如:

```
int interestingNumber = [someObject magicNumber];
```

同样地，我们也能让函数返回一个对象。举个例子，`NSString`这个类，提供了一个方法`uppercaseString`，代码示例如下：

```
- (NSString *)uppercaseString;
```

使用相同的方法我们也能让函数返回一个标量值(scalar value)，并且需要使用一个指针来跟踪这个结果，示例如下:

```
 NSString *testString = @"Hello, world!";
 NSString *revisedString = [testString uppercaseString];

```

当这个方法调用返回时，`revisedString `指针将会指向一个代表字符`HELLO WORLD!.`的字符串对象。

记住，当方法返回一个对象时，像这样：

```
- (NSString *)magicString {
    NSString *stringToReturn = // create an interesting string...
 
    return stringToReturn;
}
```

即使`stringToReturn`指针已经不在作用域中，这个作为返回值的字符型对象依然存在。

显然，在这种情况下我们需要思考此过程中的内存管理：一个被返回的对象（在堆中创建）需要一个相当长的寿命以保证它可以被这个方法的调用者使用，但这个对象也不能永久的存在，否则会造成内存泄漏。大多数情况下，*Objective-C* 中的 *自动引用计数(ARC)* 机制会帮我们管理内存，所以我们不必太过担心。

###对象可以给自己发送消息
无论何时我们实现一个方法，我们都需要访问一个非常重要的值`self`。从`self`的字面意思来看，它是指“收到这条消息的对象”。事实上，`self`是一个指针，就像我们之前提到过的`greeting`,并且能够用来调用一个方法。

我们可以选择重构`XYZPerson`这个类的实现，将`sayHello`这个方法改为`saySomething`(我们在上文使用过)，在新创建的这个方法中调用`NSLog()`,这意味着我们可以添加更多的方法，比如：
`sayGoodbye`。that would each call through to the saySomething: method to handle the actual greeting process. If you later wanted to display each greeting in a text field in the user interface, you’d only need to modify the saySomething: method rather than having to go through and adjust each greeting method individually.

使用`self`像对象发送消息的代码示例如下：

 ```
@implementation XYZPerson                                     
- (void)sayHello {
    [self saySomething:@"Hello, world!"];
}
- (void)saySomething:(NSString *)greeting {
    NSLog(@"%@", greeting);
}
@end
 ```

如果你向一个`XYZPerson`的对象发送一条`sayHello`消息，流程如图所示：

***图片2-2***	向自己发送消息的程序流程

![programflow2](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Art/programflow2.png)

###对象能调用由超类创建的方法
在 *Objective-C* 中有一个我们非常熟悉的关键字`super`。通过向super发送消息的这种方式来调用一些由继承链之上的父类定义的方法。`super`最常见的用法是用于重写一个方法。

让我们假设现在要创建一个新型人类的类，一个“射击者类”，在这个新定义的类中，每一个“greeting”都需要用大写来表示。我们当然可以复制整个`XYZPerson`类，然后修改每个方法中的每一个“greeting”，但是最简单的方法显然是创建一个新的类，而这个新的类继承于`XYZPerson`，此时我们只需要重写
`saySomething`这个方法，即可达到目的。代码示例如下：

```
@interface XYZShoutingPerson : XYZPerson
@end
```

```
@implementation XYZShoutingPerson
- (void)saySomething:(NSString *)greeting {
    NSString *uppercaseGreeting = [greeting uppercaseString];
    NSLog(@"%@", uppercaseGreeting);
}
@end
```

在这个例子中，我们定义了一个额外的字符串型指针：`uppercaseGreeting `,并将向原始指针
`greeting`所指向的对象发送消息` uppercaseString `的返回值赋给这个我们额外定义的指针。就像我们之前所看到的那样，这个新的字符串型的对象就是将原始的字符串中的每一个字母都变成大写而形成的。

因为`sayHello`这个方法是由类`XYZPerson`实现的，而同时，`XYZShoutingPerson`是继承于
`XYZPerson`的子类，所以`XYZShoutingPerson`类中的对象也能调用`sayHello`这个方法。当我们在`XYZShoutingPerson`中调用方法`sayHello`时，`[self saySomething:...]`这个调用将会使用 *重写* 并且将“greeting”全部改写成大写的形式。程序流程如图2-3所示：

***图片2-3***	方法重写的程序流程

![programflow3](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Art/programflow3.png)

我们发现这个新的方法并不是非常的理想，然而，因为我们决定之后将会修改`XYZPerson`中`saySomething`的实现：使用用户界面元素来展现而不是`NSLog()`函数，因此我们同样也需要修改
`XYZShoutingPerson`的实现。

我们提出一个更好的想法，直接改变`XYZShoutingPerson`中的方法`saySomething`：调用超类(`XYZPerson`)中的`saySomenthing`来处理这个“greeting”，代码示例如下：

```
@implementation XYZShoutingPerson
- (void)saySomething:(NSString *)greeting {
    NSString *uppercaseGreeting = [greeting uppercaseString];
    [super saySomething:uppercaseGreeting];
}
@end
```
程序流程如下：

***图片2-4***	向超类发送消息的程序流程

![programflow4](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Art/programflow4.png)

##对象的创建是动态的
就像我们在此章之前的篇幅中向大家介绍过的那样，对于 *Objective-C* 中的对象来说，内存的分配是动态的。创建一个对象的第一步是确保有足够多的内存不仅能分配给对象所属的类定义的属性，也有内存能够分配给在继承链中的每一个超类所定义的属性。

`NSObject`这个根类提供了一个类方法，`alloc`,这个类方法为我们处理了这个问题：

```
+ (id)alloc;
```
请注意，这个方法的返回值类型是`id`。这在 *Objective-C* 中是一个非常特殊的关键字，表示“某种类型的对象”。这是一个指向对象的指针，就像`(NSObject *)`，不过对于`id`来说比较特殊的是它并没有使用星号（＊）。我们将会在本章的下一个板块：[Objective-C Is a Dynamic Language]()中详细的论述。

类方法`alloc`还有另外一个非常重要的任务，清理掉分配给对象的特征的内存，通过将内存设置为零的方式。这样做避免了内存垃圾的产生，但是这对于完全地初始化一个对象来说还远远不够。

我们需要将方法`alloc`和`init`结合起来，这其中的`init`是`NSObject`的另一个类方法：

```
- (id)init;
```
我们使用`init`这个方法来确保当一个对象被创建时，它拥有合适的初始值。本文档将会在下一个章节中对此进行详细地介绍。

注意`init`的返回值类型也是`id`。
如果一个方法的返回值是一个对象指针，则将方法1嵌套在方法2中，并且作为方法2所发送的消息的接受者是被允许的，因此我们在一个声明中可以发送多条消息。到这里，我们应该学会了分配内存并初始化一个对象的正确方法：在`init`中嵌套一个`alloc`方法，代码示例如下：

```
NSObject *newObject = [[NSObject alloc] init];
```
在这个例子中，我们将`newObject`设置成一个指向`NSObject`的实例的指针。

在上例的代码中，内层括号里的方法将首先被执行，因此`NSObject`这个类首先调用方法`alloc`，而这个方法会返回一个新创建的`NSObject`的实例。这个由方法`alloc`返回的对象将继续作为`init`发送的消息的接受者。而这条消息返回给这个对象的值将被赋给指针`newObject`，流程如图2-5所示：

***图片2-5***	将`alloc`和`init`两个方法嵌套

![programflow5](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Art/nestedallocinit.png)

>注意：`init`的返回值可以是一个不同的对象（不是`alloc`返回的那个对象），所以最好像示例中那样嵌套两个方法。千万不要在没有分配一个指针指向那个对象时初始化它，像这样：

```
 NSObject *someObject = [NSObject alloc];
 [someObject init];
```
如果`init`返回了一个其它的对象，我们将会拥有一个被分配了内存但没有初始化的对象。

###初始化方法时可以带参数
一些对象在初始化时需要带上某些特定的参数。举个例子，类`NSNumber`的一个对象在创建时必须带上一个它代表数值作为参数。

类`NSNumber`定义了几个初始化的形式，包括：

```
- (id)initWithBool:(BOOL)value;
- (id)initWithFloat:(float)value;
- (id)initWithInt:(int)value;
- (id)initWithLong:(long)value;
```
举个实际的例子：(A factory method is used like this)

```
NSNumber *magicNumber = [NSNumber numberWithInt:42];
```
这个例子和我们之前使用过的`alloc] initWithInt:]`是一样的。类方法常常直接通过`alloc`和与之配套的`init`来调用，并且使用这些类方法是非常方便的。
###如果初始化时不需要带参数 使用`new`创建一个对象
我们同样可以使用`new`这个类方法来创建一个类的实例。这个方法是由`NSObject`提供的并且在我们自己的子类中不需要被重写。

这和调用不带参数的`alloc`和`init`的用法是一样的，代码示例如下：

```
 XYZObject *object = [XYZObject new];
 // is effectively the same as:
 XYZObject *object = [[XYZObject alloc] init];

```

###常量提供了一个创建对象的简洁语法
某些类允许我们使用一种更简洁的语法创建实例，用 *literal* 来实现。

举个例子，我们可以使用一个文字常量来创建类`NSString`的一个实例，代码示例如下：

```
NSString *someString = @"Hello, World!";
```
这和分配内存并初始化一个对象或者使用类方法中的一种来实现这个实例的创建是相同的：

```
NSString *someString = [NSString stringWithCString:"Hello, World!"
                                              encoding:NSUTF8StringEncoding];

```
类` NSNumber`同样也允许一系列常量的使用：

```
NSNumber *myBOOL = @YES;
NSNumber *myFloat = @3.14f;
NSNumber *myInt = @42;
NSNumber *myLong = @42L;

```
再次强调，我们举的每一个例子和我们使用与`alloc`配套的`init`或者类方法来初始化一个对象是相同的。

我们同样也可以使用一个计算式（boxed expression）来创建`NSNumber`的实例，代码示例如下：

```
 NSNumber *myInt = @(84 / 2);
```
在这个例子中，我们使用了一个表达式，并且用了这个表达式的结果来创建一个类的实例。

*Objective-C* 同样支持使用常量来创建静态数组或者字典的对象，这一点我们将会在[Values and Collections]()作进一步地讨论。

##*Objective-C* 是一种动态语言
之前曾经提到过，我们需要使用一个指针来跟踪对象在内存中的位置。因为 *Objective-C* 的动态特性,这个指针的数据类型并不做要求————当发送一条消息时正确的方法总是会被调用。`id`这个数据类型定义了一个通用对象指针。我们可以定义一个`id`类型的变量，但此时我们会失去对象在编译时的一些信息。

现在让我们思考接下来的两行代码：

```
id someObject = @"Hello, World!";
[someObject removeAllObjects];
```
在这个例子中，`someObject`指向一个`NSString`的实例，但是我们的编译器并不知道这个实例是某种类型的对象这件事情。`removeAllObjects`这条消息是由 *Cocoa or Cocoa Touch* 上的某些对象定义的（例如`NSMutableArray`）,因此我们的编译器并不会报错，即使这两行代码在运行时会产生异常。因为一个`NSString`对象不会响应`removeAllObjects`这条消息。

接下来我们使用一个静态类型重写这俩行代码：

```
NSString *someObject = @"Hello, World!";
[someObject removeAllObjects];
```
这样一来，编译器在编译时就会报错了，因为编译器在任何一个公有类的接口中都找不到
`removeAllObjects `的定义。

因为一个对象所属的类在程序运行时才能被确定(Because the class of an object is determined at runtime)，所以我们在创建或使用一个实例时将变量定义成什么类型是无差别的。使用我们在此章节的前半部分定义过的类`XYZPerson`和`XYZShoutingPerson`，我们应该向这样编码：

```
 XYZPerson *firstPerson = [[XYZPerson alloc] init];
 XYZPerson *secondPerson = [[XYZShoutingPerson alloc] init];
 [firstPerson sayHello];
 [secondPerson sayHello];
```
尽管`firstPerson`和`secondPerson`被静态地定义成`XYZPerson`上的两个对象，在程序运行时，
`secondPerson`将会指向一个`XYZShoutingPerson`对象。当`sayHello`这个方法被每一个对象调用时，正确的实现将会被使用。对于`secondPerson`来说，是`XYZShoutingPerson`这个类调用了方法`sayHello`。

###确定对象的等同性

如果我们想判断两个对象是否是相同的，记住我们是使用指针来工作的是恨重要的一件事情。在标准 *C* 中，操作符`==`被用来判断两个变量的值是否相等。比如：

```
if (someInteger == 42) {
        // someInteger has the value 42
}
```
当我们处理对象时，操作符`==`被用来判断两个不同的指针是否指向同一个对象，比如：

```
if (firstPerson == secondPerson) {
   // firstPerson is the same object as secondPerson
}
```
如果我们想判断两个数据是否相等，我们应该调用一个函数，比如`isEqual`:可以从`NSObject`这个类中获得：

```
if ([firstPerson isEqual:secondPerson]) {
   // firstPerson is identical to secondPerson
}
```
如果我们想比较两个对象的值的大小关系，与上面判断是否相等不同的是，我们不能使用标准 *C* 中的操作符`>`和`<`。基础的函数类型，例如`NSNumber`,`NSString`以及`NSDate`为我们提供了一个方法
`compare:`，代码示例如下：

```
if ([someDate compare:anotherDate] == NSOrderedAscending) {
    // someDate is earlier than anotherDate
}

```

###`nil`的使用
当我们定义一个纯量的同时就初始化它是一个非常优秀的习惯，否则这个量的初始值就会包括前一个栈中的内容,而这些内容往往是无用的：

```
 BOOL success = NO;
 int magicNumber = 42;
```
这对于对象指针来说并不是十分必要的，因为如果我们不设置任何其它的初始值，编译器就会自动将变量的值设置为`nil`：

```
XYZPerson *somePerson;
// somePerson is automatically set to nil
```
当我们没有其它值可用时，使用`nil`值是初始化一个变量指针最安全的方法，因为在 *Objective-C*中，向`nil`发送一条消息是非常合理的。如果我们真的向`nil`发送了一条消息，很明显任何事情都不会发生。

>注意：如果你需要发送给`nil`的这条消息返回一个值，对于对象类型来说，这个值是`nil`；对于数值类型来说，返回值是`0`；对于`BOOL`类型来说，返回值是`NO`。Returned structures have all members initialized to zero.

如果我们需要确认一个对象的值不是`nil`（在内存中是一个变量指针），也可以使用标准 *C* 中的操作符
`!=`，代码示例如下：

```
if (somePerson != nil) {
   // somePerson points to an object
}
```
或者仅仅只是提供一个变量：

```
if (somePerson) {
   // somePerson points to an object
}
```
如果变量`somePerson`的值是`nil`，它的逻辑值是`0`(false)。如果它有一个地址，那么它的值就是非零，会被当做正确的来执行。

相似地，如果我们要判断一个对象的值是不是`nil`，我们也能使用操作符`==`：

```
if (somePerson == nil) {
   // somePerson does not point to an object
}
```

或仅使用 *C* 语言中的否定预算符：

```
if (!somePerson) {
   // somePerson does not point to an object
}
```
##练习：
1.打开我们在上一章的练习中写过的工程，找到`mian.m`文件中的`main()`函数。就像在 *C* 语言中一样，这个函数代表了工程的接口。

使用`alloc`和`init`创建一个新的`XYZPerson`的实例，并且调用`sayHello`这个方法。

>注意：如果编译器没有自动提示你，你需要在`main.m`文件的开头引入头文件（包括`XYZPerson`的接口）。

2.实现在此章的前半部分提到过的方法`saySomething:`，重写方法`sayHello`并且使用它。添加一些其它的greetings指针，使用上一题你所创建的实例来依次调用这些greetings。

3.为类`XYZShoutingPerson`创建新的类文件，并把它设置成继承于`XYZPerson`的子类。
重写方法`saySomething:`来显示大写的`greeting`，并且在`XYZShoutingPerson`的实例中测试这个函数。

4.实现我们在此章节的之前部分定义过的`XYZPerson`类的类工厂方法`person`,返回一个被正确地分配内存及初始化的`XYZPerson`的实例。接着，在`main()`中使用这个方法来替代`alloc`和`init`地嵌套。

>提示：在类工厂方法中不要使用`[[XYZPerson alloc] init]`，请尝试使用
 `[[self alloc] init]`
 在类工厂方法中使用`self`意味着你指向的是这个类本身。
 也就是说，在实现`XYZShoutingPerson`时你不需要复写方法`person`来创建实例。尝试下列代码来确  定这是正确的：
 
 ```
   XYZShoutingPerson *shoutingPerson = [XYZShoutingPerson person];
 ```

5.创建一个`XYZPerson`的指针，但不赋值。使用一个语句分支来确认这个变量是否被自动地赋值为`nil`
 



