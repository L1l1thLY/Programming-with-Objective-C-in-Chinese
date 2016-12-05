#命名规则
当我们在使用一些框架类(framework classes)的时候，我们会发现 *Objective-C* 的代码可读性很高。类和方法的名字跟 *C* 语言中一般的函数代码或者 *C* 语言中的标准程序库比起来更加简单易懂。当我们用多个单词去命名时，我们会使用 *骆驼命名法*。我们应该使用和 *Cocoa and Cocoa Touch* 相同的命名规则去命名我们自己定义的类，这样会使我们代码的可读性更强，以方便其它需要阅读或者维护我们代码的程序员更好地进行操作。

##命名的唯一性
每一次我们创建一个新的类型，符号，或者标识符的时候，首先要考虑的就是我们的这个命名在它所作用的范围内是唯一的。有时，这个范围可能是整个应用，包括它的链接库；而有时这个范围仅仅只是一个类或者一小段代码。

###在整个应用中类的名字必须唯一
在 *Objective-C* 中，类的名字不仅仅要求在我们所写的整个工程的代码中是唯一的，还要求在可能包括在内的库或者代码包(frameworks or bundles)中也具有唯一性。举个例子，我们应该避免使用通用类的名字，比如`ViewController`或者`TextParser`。因为这很有可能是包括在我们的工程中的一个库函数的名字，这会导致创建类失败。	

为了保证类名的唯一性，我们的命名规则要求：在所有类的命名上都使用前缀。我们应该已经注意到，在 *Cocoa and Cocoa Touch* 中，类的名字都是以`NS`或者`UI`开头的。苹果公司规定了一系列两个字母的前缀用于命名框架类。当我们学习了更多有关 *Cocoa and Cocoa Touch* 的知识时，会了解到更多其它的前缀：

| Prefix  | Frameworks                                           |
| --------| :----------------------------------------------------|
| NS      | Foundation (OS X and iOS) and Application Kit (OS X) |
| UI      | UIKit (iOS)                                          |
| AB      | Address Book                                         |
| CA      | Core Animation                                       |
| CI      | Core Image                                           |
	


而我们自己定义的类应该使用三个字母的前缀，这个前缀应该和你的公司名称，你的应用名称甚至于你的应用的某个特性相关。举个例子，如果你的公司名字是Whispering Oak，而你正在开发一款名叫Zebra Surprise的游戏，那么你应该选择`WZS`或者`WOZ`作为类名的前缀。
同时，我们应该使用一个名词来命名一个类，这样会使这个类的意义更加清晰，比如这个来自 *Cocoa and Cocoa Touch* 的例子：

|          |             |                    |                       |
| ---------|:-----------:|:------------------:|:---------------------:|
| NSWindow | CAAnimation | NSWindowController | NSManagedObjectContext|

如果在类的命名中需要使用多个单词，则应该使用 *骆驼命名法* ,即将每一个独立单词的首字母变为大写。


###方法名必须保证唯一性和准确性
当我们为一个类确定了名称后，在类中声明的方法的名字也必须保证唯一性。在不同的类中，使用同一个名字命名相同功能的某个方法是可行的。举个例子，当我们复写一个父类的方法或者利用多态性时，在不同类中体现同一功能的方法应该使用相同的名字，相同的返回类型以及相同的参数类型。

方法名无需使用前缀，并且以小写字母开头。当然，使用多个单词命名一个方法时也要使用 *驼峰命名法* 
举例如下（这些方法都来自类`NSString`）：

|        |                   |                             | 
| -------|:-----------------:|:----------------------------:
| length | characterAtIndex: | lengthOfBytesUsingEncoding:
										
如果方法带有一个或者多个参数，那么在方法名中需要表示出来，例如：

|                    |                                      |           |
| -------------------|:------------------------------------:|:----------:
| substringFromIndex:| writeToURL:atomically:encoding:error:|enumerateSubstringsInRange:options:usingBlock:|

方法名的第一部分应该包括该方法的主要功能或者调用这个方法的结果。如果一个方法返回一个值，举个例子，方法名的第一个单词应该表明返回的值是什么东西，比如`length,character`以及
`substring...`等前文提及过的方法。当我们需要表明返回值的一些重要特征时，我们可以使用多个词语来命名该方法，比如来自`NSString`类的方法`mutableCopy`，`capitalizedString`以及
`lastPathComponent`。如果某个方法是用来实现一个类似写入磁盘或者枚举的功能的，命名的第一个单词应该直接表明这个动作，比如`write`以及`enumerate`。

如果一个方法包含了一个这样的指针：*error* 指针参量，这应该是这个方法的最后一个参量。如果一个方法带了一个 *块* ，那么这个块参数因该被设置成最后一个参数，这样当我们在指定一个块内联时，能够使方法的可读性很强。出于同样的原因，我们最好能避免带有多个块参数的方法。

清楚且简明地命名一个方法也很重要。方法名过于细致可能会导致名称太冗长，而太简短可能会使方法名表意不清，因此我们应该两者兼顾，例如：

|stringAfterFindingAndReplacingAllOccurrencesOfThisString:withThisString:|Too verbose|
|------------------------------------------------------------------------|-------------|
|strReplacingStr:str:                                                    |Too concise|
|stringByReplacingOccurrencesOfString:withString:                        |Just right|

在命名一个方法时应该避免使用缩略词，除非我们能肯定在大多数的语言和文化中这个缩略词都是很容易理解的。一些常用的缩略词可以查看：[Acceptable Abbreviations and Acronyms]()

####当使用category来创建方法时 方法名需要加上前缀

当我们使用category来向一个已存在的框架类中添加方法时，方法名中应该加上前缀，这样可以避免混淆。在[ Avoid Category Method Name Clashes]()中有详细介绍。

				
###在同一作用域中请保证局部变量的唯一性

因为 *Objective-C* 是 *C* 语言的超集，*C* 语言中变量的作用域规则在 *Objective-C* 中同样适用。一个局部变量的名称在它的作用域中必须是独一无二的。

```
- (void)someMethod {
    int interestingNumber = 42;
    ...
    int interestingNumber = 44; // not allowed
}
```
*C* 语言中确实允许我们在同一作用域中命名两个相同名字的变量，例如：

```
- (void)someMethod {
    int interestingNumber = 42;
    ...
    for (NSNumber *eachNumber in array) {
        int interestingNumber = [eachNumber intValue]; // not advisable
        ...
    }
}
```
不过，这使得我们的代码变得不具可读性。因此，我们应当尽量避免这种情况的出现。

##方法的命名必须遵从命名规则
除了要考虑方法名的唯一性，严格遵从命名规则也是非常重要的。这些命名规则被 *Objective-C* 的编译器和运行时库所使用，同样也被 *Cocoa and Cocoa Touch* 中的类使用。

###存取方法的名称必须遵从命名规则

当我们使用`@property`语法来为对象声明属性时（详情参见：[Encapsulating Data]())，编译器会自动合成getter和setter方法（除非我们另作声明）。如果要定义我们自己的存取方法时，确保方法名的正确性才能让我们顺利地使用点语法。	

除非特别声明，否则一个getter方法的名字应该和它对应的属性的名字相同。对于一个名字是`firstName`的属性来说，存取方法的名字也应该是`firstName`。对布尔值属性来说，getter方法应该以`is`开始。例如`paused`属性，那么它对应的getter方法的名字应该是`isPaused`

对于一个属性来说，要定义它的setter方法，我们应该使用格式：`setPropertyName:`，对于一个名叫`firstName`的属性来说，setter的方法的名字应该是`setFirstName:`，对于一个名叫`paused`的布尔值属性来说，它的setter方法名应该是`setPaused:`

即使`@property`允许我们制定不同的存取方法名，但也只能在有布尔值属性的情况时使用。其他时候我们必须遵从命名规则，否则一些很重要的技术，比如KVC技术（使用`valueForKey:`和`setValue:forKey:`来存取一个属性的技术）就不能使用了。关于KVC更多的信息，请参考[Key-Value Coding Programming Guide]()

###对象创建的方法名必须遵从命名规则

我们在之前的章节中已经了解到，为一个类创建实例有很多种方法。我们可以使用分配内存和初始化相结合的方法来创建实例，比如：

```
NSMutableArray *array = [[NSMutableArray alloc] init];
```

也可以使用`new`这种便利的方法，直接调用`alloc`和`init`这两种方法，例如：

```
NSMutableArray *array = [NSMutableArray new];
```

一些类甚至可以使用工厂方法模式来创建实例：

```
NSMutableArray *array = [NSMutableArray array];
```

除了在工厂模式的类中已经存在的子类，类工厂方法往往使用类的名字来作为自己名字的开头（没有前缀）。就`NSArray`这个类来说,举个例子，工厂方法的名字都会以`array`开头。而`NSMutableArray`这个类不规定任何特定于类的工厂方法，所以一个可变数组的工厂方法名仍然以`array`开头。

*Objective-C* 中有各种各样的内存管理机制，用于确保对象的生命周期足够长。我们不需要过多的担心这些规则，编译器会基于所创建的这些方法的名称来判断它需要遵守哪一条规则。由于自动释放池（autorelease pool blocks）的使用，通过方法工厂创建的对象的管理和通过传统方式（包括分配内存和初始化，`new`方法）创建的对象的管理有些细微地不同。想了解更多关于自动释放池以及内存管理机制的信息，请参考[ Advanced Memory Management Programming Guide]()




