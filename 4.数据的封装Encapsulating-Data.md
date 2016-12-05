#数据的封装
之前章节我们描述了可以通过向一个对象发送消息让他工作。对象还通过 *属性* 封装了一些数据。  
这个章节告诉你：为一个对象声明一些属性的Objective-C语法是怎样的？这些属性会默认地被存取方法合成器 *synthesis of accessor* 实现，那么这种默认的实现方法又是遵循什么规则？如果一个属性实际上是一个变量，这个变量必须在这个对象的初始化方法中被正确地设置。  
如果一个对象通过一个属性来维持一个指向至另一个对象的链接，考虑两个对象之间的关系的本质就很重要了。即使Objectivc-C的内存管理基本上都已经被 *自动引用计数Automatic Reference Counting（ARC）* 系统安排妥当了，你也应该清楚如何去避免一些诸如 *强循环引用strong reference cycles* 等会引起内存泄露的问题。这个章节阐明了一个对象的生命周期并且教你如何根据对象之间的关系去管理 *对象图 graph of objects* 。   
## 属性封装了一个对象的值
为了完成他们各自的任务，大多数的对象要记录跟踪一些信息。一些对象实际上是一个数据或者多个数据的模型，比如说Cocoa框架中的`NSNumber`类保存了一个数字值；一个自定义的`XYZPerson`类建立了一个模型——一个拥有姓和名的人类。一些更加普遍的对象是这样的——他们仅仅管理了UI和其上显示的信息之间的关系，即使是这样的对象，依然需要跟踪UI元素或者UI相关的模型对象。
###为暴露在外的数据声明一个公共属性
如果一个类需要封装一些信息，Objective-C提供了一种声明这些信息的方式——属性。就像你在 *Properties Control Access to an Object’s Values* 那章看到的那样，属性像这样被声明在一个类的接口部分：

```
@interface XYZPerson : NSObject
@property NSString *firstName;
@property NSString *lastName;
@end
```

在这个例子中，`XYZPerson`声明了两个字符串属性来保存一个人的姓名。  
在面对对象编程中一条首要规则就是对象需要把它内部的工作过程隐藏在对外公开的接口之后，所以使用对象暴露在外的行为接口来操作对象的属性要好于直接操作属性内部的值。
###使用 *存取方法Accessor Methods* 来读取或设置属性的值
你通过存取方法读取或者设置一个对象的属性：

```
    NSString *firstName = [somePerson firstName];
    [somePerson setFirstName:@"Johnny"];
```

默认情况下，编译器会为你自动生成这些存取方法，所以你除了在类接口处使用`@property`来声明一个属性之外什么都不需要做。  
自动生成存取方法采取以下命名规则：

-  用来获得属性值的方法（ *getter方法* ）和属性的名字一模一样。`firstName`属性的获取方法也被叫做`firstName`。
-  用来设置一个属性的值的方法（ *setter方法* ）是以`set`开头的，并且跟上首字母大写的属性名。`firstName`属性的存储方法被叫做`setFirstName`。

如果想禁止属性被存取方法修改，你应在声明属性的时候为它添加一个 *特征 attribute* ，比如下面这个特征标明了这个属性是 *只读readonly* 的：

```
@property (readonly) NSString *fullName;
```

特征告诉其他对象与这个对象交流的权限，此外，它告诉编译器如何生成属性相应的存取方法。  
在上面的例子里，编译器就只会生成一个`fullName`的获取方法，而不生成`setFullName:`方法了。  
>注意：和`readonly`相对的就是`readwrite`（可读可写）。不需要特意写出`readwrite`特征，因为一个属性默认就是可读可写的。

如果你想要改变存取方法的名字，你就可以在声明属性特征时自定义一个名字。在这个声明一个布尔值（属性值要么是`YES`要么是`NO`）属性的例子里，我们可以把获取方法的名字改成以“is”开头。因为这个属性叫做`finished`，所以他的获取方法应该叫做`isFinished`。  
那么我们就为这个属性增加一个特征吧：  

```
@property (getter=isFinished) BOOL finished
```

如果你需要增加多个特征，只需要用逗号分隔他们即可，如下：

```
@property (readonly, getter=isFinished) BOOL finished;
```

在这个例子里，编译器只会生成一个`isFinished`方法而不是生成一个`setFinished:`方法。
>注意：一般来说，属性存取方法是服从（Key-Value Coding）KVC，这也意味着这些属性也遵从这复杂的名称转换规则，若想详细了解请查阅 [Key-Value Coding Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/KeyValueCoding/index.html#//apple_ref/doc/uid/10000107i) 。

###点语法让存取方法的调用更加简洁
除了精确地调用存取方法，Objective-C还提供了一种方法——点语法，去操作一个对象的属性。  
点语法允许你像这样操作一个属性：

```
NSString *firstName = somePerson.firstName;
somePerson.firstName = @"Johnny";
```

点语法只是对存取方法做了一个方便的包装。实际上你使用点语法的时候，依然是通过上面提到的存取方法来获得属性或者设置属性的值的：

- 使用`somePerson.firstName`来获得属性的值和使用`[somePerson firstName]`完全一致。
- 通过类似`somePerson.firstName = @"Johnny`的语法来设置属性的值和使用`[somePerson setFirstName:@"Johnny"]`完全一致。

这也就是说，无论你直接调用属性存取方法还是使用点语法，属性特征依然控制着属性存取方法。如果你为属性设置了`readonly`特征，当你使用点语法来为属性设置值的时候编译器会报错。
###所有的属性背后都存在着一个实例变量

默认情况下，一个可读可写的（readwrite）的属性将会拥有着一个对应的 *实例变量（instance variables）* ，这个变量也会被编译器自动生成（synthesized）。  

实例变量是指在对象的整个生命周期存在并保存他本身的值的变量。当你在为对象分配内存时（通过`alloc`)，这些变量的内存也会被分配，而在对象生命周期结束时，对象占用的内存被释放，这些实例变量也就随之被释放了。  

如果你没有特意规定的话，自动生成的实例变量和它对应属性的名字是相同的，但是前面会带上一个下划线。比如说一个叫做`firstName`的属性，自动生成的实例变量将会被叫做`_firstName`。  

虽然利用存取方法或者点语法来操作一个对象中的属性是比较合适的，但是在类实现中的任何一个方法都可以直接操作实例变量。一个下划线开头让你能够清楚地看到，你操作的是一个实例变量，而不是其他的，比如说一个局部变量：  

```
- (void)someMethod {
    NSString *myString = @"An interesting string";
 
    _someString = myString;
}
```

在这个例子里，我们能够清晰地看到，`myString`是一个局部变量，而`_someString`是一个实例变量。  

一般来讲我们会遵守一个规则，即使是这种情况下——在一个对象的方法实现中去访问一个实例变量，仍然应该使用存取方法或者点语法。这个时候你应该使用`self`：

```
- (void)someMethod {
    NSString *myString = @"An interesting string";
 
    self.someString = myString;
  // or
    [self setSomeString:myString];
    }
```

但是，当你写一个对象的初始化、释放空间或者自定义一个存取方法的时候，就不再遵守这个规则了，我们会在之后描述相关内容。  

####你可以定制自动生成的实例变量的名字

我们之前提到，根据属性自动生成的实例变量的默认名字为下划线加属性名`_propertyName`。  
如果你想要这些实例变量拥有不同的名字，你需要在类实现中这样命令编译器：

```
@implementation YourClass
@synthesize propertyName = instanceVariableName;
...
@end
```
 
 比如说：
 
 ```
 @synthesize firstName = ivar_firstName;
 ```
 
 在这个情况下，属性仍然还是叫`firstName`的，使用点语法和存取方法的情况下，你仍然能够通过`firstName`和`setFirstName`访问它，但是他对应的实例变量名字为`ivar_fisrtName`。  
 
 >重要：如果你使用了`@synthesize`却没有为其规定一个实例变量名，像这样：
 
 ```
 @synthesize firstName;
 ```
> 实例变量的名字将会和属性 *相同* ，为`firstName`，没有下划线。

####你可以不通过属性来定义一些实例变量

最好是无论何时你的对象需要追踪一个值或其他对象的时候，都使用一个属性。  

如果你需要不声明一个属性就定义一个实例变量的话，你可以把它们放在一对花括号中，这对花括号需要位于类接口部分或实例部分顶部，像这样：

```
@interface SomeClass : NSObject {
    NSString *_myNonPropertyInstanceVariable;
}
...
@end
 
@implementation SomeClass {
    NSString *_anotherCustomInstanceVariable;
}
...
@end
```

>注意：你也可以在类拓展处增加实例变量，详情请看：[Class Extensions Extend the Internal Implementation.
](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html#//apple_ref/doc/uid/TP40011210-CH6-SW3)

### 在初始化方法中直接访问实例变量
setter方法会导致一些副作用。它可能会触发KVC通知，如果你编写了自定义方法，它也许还会执行本应未来才能执行的任务。
「在对象的初始化方法里，你应该直接访问实例变量，因为当你配置属性时这个对象的其他部分可能还没被完全初始化。即使你不提供任何自定义的存取方法，并且了解一些来自你自己类中的副作用，以后它的子类也可以可能会重载（override）这些行为。  」

一个典型的`init`方法像这样：

```
- (id)init {
    self = [super init];
 
    if (self) {
        // initialize instance variables here
    }
 
    return self;
}
```

`init`方法会在做自己的初始化工作之前，先调用父类的初始化方法并把其返回值赋给`self`。父类可能没有成功的初始化，返回了一个nil值，所以你一定要记得在开始自己的初始化工作之前检查一下`self`是不是`nil`。  

通过在初始化方法的第一行调用`[super init]`这种模式，一个对象是按照从  *根类（root class）*  开始，依次序调用每个子类的`init`实现来初始化的。图3-1展示了一个初始化`XYZShoutingPerson`对象的过程。  

图3-1 初始化的过程
![图3-1](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Art/initflow.png)  

正如之前的章节描述的那样，一个对象要么是通过`init`初始化的，要么就是通过调用一个方法，这个方法能够初始化一个带有特殊值的对象。  

在那个`XYZPersion`类的例子中，我们可以为其提供一个初始化方法来设置这个人的姓名：  

```
- (id)initWithFirstName:(NSString *)aFirstName lastName:(NSString *)aLastName;
```

应该像这样实现这个方法：

```
- (id)initWithFirstName:(NSString *)aFirstName lastName:(NSString *)aLastName {
    self = [super init];
 
    if (self) {
        _firstName = aFirstName;
        _lastName = aLastName;
    }
 
    return self;
}
```

#### *指定初始化器 （Designated Initializer）* 是首选的初始化方法
如果一个对象声明了一个或多个初始化方法，你需要决定哪个方法是 *指定初始化器（designated initializer）* 。它通常是那个为初始化提供最多选项的方法（比如说带有最多参数的方法），并且会被其他出于方便编写的方法调用。你一般应该重载`init`，在其中调用你的指定初始化器，并为这个调用提供合适的默认值。  

如果一个`XYZPerson`多了一个表示生日的属性，指定初始化器可能是这样的：  

```
- (id)initWithFirstName:(NSString *)aFirstName lastName:(NSString *)aLastName
                                            dateOfBirth:(NSDate *)aDOB;
```

这个方法会像上面所示的那样设置好相关的实例变量。如果你仍然想提供一个方便的初始化器仅初始化姓名，你应该通过调用指定初始化器的方式来实现这个方法，像这样：

```
- (id)initWithFirstName:(NSString *)aFirstName lastName:(NSString *)aLastName {
    return [self initWithFirstName:aFirstName lastName:aLastName dateOfBirth:nil];
}
```

像开头说的那样，你也许会这样实现标准`init`来提供合适的默认值：

```
- (id)init {
    return [self initWithFirstName:@"John" lastName:@"Doe" dateOfBirth:nil];
}
```

当你需要继承一个拥有多个`init`方法的类时，你有两种方式为这个类制作初始化方法，第一种是重载父类的指定初始化器并在其中制定你自己的初始化工作，第二种是添加你自己额外的初始化方法。无论哪种方法，你都应该在你做任何自己的初始化工作之前先调用父类的指定初始化器（作为[super init]的代替）。  

###你可以实现自定义的存取方法
不是所有属性都拥有一个实例变量的。  

举个例子吧，`XYZPerson`类可能声明了一个只读属性，用来代表一个人的全名：

```
@property (readonly) NSString *fullName;
```

与其每次姓名改变的时候都去更新`fullName`属性（拥有的实例变量），还不如写一个自定义的存取方法来直接生成需要全名字符串简单一些：

```
- (NSString *)fullName {
    return [NSString stringWithFormat:@"%@ %@", self.firstName, self.lastName];
}
```

这个简单的例子使用了格式化字符串和占位符（之前章节有描述）来生成了一个包含姓名的字符串（由空格分割）。  

> 注意：即使这是一个很便利的例子，当然你得注意这个例子却和地理区域相关，这个例子只适合那些名字在姓之前的国家。  

如果你需要为一个使用实例变量的属性写一个自定义存取方法，你定义要在方法里直接访问实例变量。举个例子，我们一般会延迟到这个属性第一次被请求的时候才会初始化这个属性，使用一个“lazy 访问器”，像这样：  

```
- (XYZObject *)someImportantObject {
    if (!_someImportantObject) {
        _someImportantObject = [[XYZObject alloc] init];
    }
 
    return _someImportantObject;
}
```

在返回这个值之前，方法会先检查`_someImportantObject`实例变量是不是nil，如果是的话，就实例化一个对象。  

> 注意：当编译器自动生成了至少一个存取方法时，它便会自动帮你生成一个实例变量。如果你为`readwrite`属性实现了getter和setter或为`readonly`属性实现了getter，编译器就假定你希望完全控制这个属性的实现，便不会自动帮你生成实例变量了。
> 如果你依然需要一个实例变量，你需要请求合成一个：
> `@Synthesize property = _property`

###属性默认是原子性的
默认情况下，一个Objective-C属性是原子性的：

```
@interface XYZObject : NSObject
@property NSObject *implicitAtomicObject;          // 默认就是原子性的
@property (atomic) NSObject *explicitAtomicObject; // 当然你也可以特别标示出它是原子性的
@end
```

这意味着生成的存取器（存取方法）保证永远是完整地通过getter方法取回或完整地通过setter方法设置一个值，即使这些存取器被不同的线程同时调用。  

因为原子性存取方法的内部实现和同步是私有的，所以不可能把自动生成的存取器和你自己实现的存取方法结合起来。如果你这么做，你会得到一个编译器警告，举个例子，给一个可读写的原子性属性提供了一个自定义的setter，却让编译器自己生成getter。  

你可以使用`nonatomic`（非原子性）特征来规定生成的存取器仅仅直接设置或返回一个值，并不保证同一个值被多个线程同时访问时会发生什么。也正因如此，访问一个非原子性的属性会比访问原子性的速度快一些，并且还可以结合使用生成的存取器和你自己的存取器，比如一个getter。  

```
@interface XYZObject : NSObject
@property (nonatomic) NSObject *nonatomicObject;
@end
```

```
@implementation XYZObject
- (NSObject *)nonatomicObject {
    return _nonatomicObject;
}
// setter会自动生成
@end
```

> 注意：属性原子性并非等价于对象的线程安全性。考虑在一个线程上通过原子性的存取器修改了一个`XYZPerson`的姓名，这时，另一个线程访问了姓名，原子性的getter方法会返回完整的字符串（不会崩溃），但是并不能保证姓和名是对应的，如果名字是在修改之前获取的，而姓是在修改之后获取的，你将会得到一个反常的，不对应的姓名。
> 这个例子很简单，但考虑到对象关系网络时，线程的安全性就变得更加复杂了。关于线程安全性的细节描述在[Concurrency Programming Guide](https://developer.apple.com/library/content/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008091)

##通过拥有关系和责任关系来管理对象图
正如你已经看到的，Objective-C（在堆上）动态地分配对象的内存，这也意味着你需要用指针来追踪一个对象的地址。和纯量不同，通过一个指针变量的作用域来判断对象的生命周期不一定可能。作为替代，当一个对象被其他对象需要，这个对象就要一直保持存在在内存中。  

与其担心如何手动管理每个对象的生命周期，还不如考虑一下对象之间的关系。  

举个例子，在`XYZPerson`的例子里，`firstName`和`lastName`这两个字符串属性，是被那个`XYZPerson`实例有效“拥有”的。这也就意味着，只要`XYZPerson`的这个对象还在内存里，他们也必须要待在内存里。  

当一个对象按这种方式依赖于其他对象，即有效地拥有了其他对象，我们就说，这个对象拥有了其他对象的 *强引用（strong reference）* （或者说强引用了其他对象）。在Objective-C中，只要一个对象被其他对象强引用了，就要一直存在。`XYZPerson`实例和两个`NSString`对象的关系如图3-2所示。  

![图3-2](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Art/strongpersonproperties.png)

当一个`XYZPerson`实例从内存中释放时，假如两个字符串对象上没有其他强引用，也会被释放。  

我们再增加一点这个例子的复杂性，考虑一个如图3-3所示程序一般的对象图。  

![图3-3](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Art/namebadgemaker.png)  

当用户点击Update键时，这个badge preview会把相关的名字消息更新上去。  

当第一次名字被输入并且按下update键时，一个简化的对象图如图3-4所示。  

![图3-4](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Art/simplifiedobjectgraph1.png)  

当用户修改了这个人的名字，对象图就会变成图3-5的样子。  

![图3-5](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Art/simplifiedobjectgraph2.png)

即使`XYZPerson`对象现在已经有了不同的`firstName`，badge展示界面依然保持着对原来的`@"John"`字符串对象的强引用，`@"John"`对象依然还需要存在内存里，被badge view用来打印名字。  

一旦用户第二次点击Update按键，就告诉了badge view根据person对象来更新他内部的属性，所以对象图就像图3-6那样了。  

![图3-6](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Art/simplifiedobjectgraph3.png)  

在这时，就没有任何对象强引用原来的`@"John"`字符串了，他会从内存中移走。  

默认情况下，Objective-C的属性和变量都会保持一个对他们对象的强引用，这一般没什么问题，但是确实会引起一个潜在的强引用循环问题。  

###避免强引用循环
对于对象之间的单向关系，使用强引用是很不错的，但是一旦要处理一组对象的相互连通关系就要小心了。如果一组对象被一个循环的强引用关系联系在了一起，他们会保持这一组的每一个人保持存在即使外部没有任何对象强引用施加于其上。  

一个显而易见的潜在循环引用问题就存在于一个table view（表视图——对于iOS是`UITableView`对于OSX来说是`NSTableView`）和其委托（delegate）之间。普通的table view为了能胜任多种情况，把他的一些决定委托给了外部的对象。这意味着其他的对象将会决定他本身显示的内容，也会决定当用户操作了table view中的特殊条目时会发生什么。  

一个普遍的情况就是table view强引用了他的委托，并且这个委托又强引用了这个tableview，就如同图3-7那样。  

![图3-7](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Art/strongreferencecycle1.png)  

当其他对象解除了对table view及其委托的强引用时，问题就发生了。如图3-8。  

![图3-8](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Art/strongreferencecycle2.png)  

即使这些对象已经不需要存在于内存中了，因为除了他们两者之间的强引用关系，再也没有其他强引用施加于这两个对象之上。然而就是他们之间的强引用关系使他们两者依然保持存在。这就是我们所说的强引用循环。  

解决这个问题的办法就是用弱引用来代替强引用。弱引用不会不隐含两个对象之间的拥有关系或责任关系并且也不会使一个对象保持存在。  

如果上面的table view被修改为弱引用其委托（`UITableView`和`NSTableView`就是这么解决这个问题的），最开始的对象图就变成了图3-9这样。  

![图3-9](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Art/strongreferencecycle3.png)  

当其他对象释放了对table view及其委托的强引用，就没有强引用施加于委托对象了，如图3-10。  

![图3-10](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Art/strongreferencecycle4.png)  

这就意味着委托对象将会被释放，也因此接着释放了对于table view的强引用，如图3-11。  

!(图3-11)[https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Art/strongreferencecycle5.png]  

一旦委托被释放了，就没有任何强引用施加于table view了，所以table view也被释放了。  

###使用强、弱声明管理拥有关系
默认的对象类型属性的声明如下：

```
@property id delegate;
```

对象会强引用自动生成的实例变量。如果想要声明一个弱引用，需要为这个属性添加吐下特征：

```
@property (weak) id delegate;
```

>注意：与`weak`相对的就是`strong`。但是不需要特地添加`strong`特征，因为默认就是它。  

局部变量（以及非属性的实例变量）同样默认保持对于对象的强引用。这也就下面的这段代码会精准地完成你所期望的任务。

```
    NSDate *originalDate = self.lastModificationDate;
    self.lastModificationDate = [NSDate date];
    NSLog(@"Last modification date changed from %@ to %@",
                        originalDate, self.lastModificationDate);
``` 

在这个例子里，局部变量`originalData`强引用了最开头的`lastModificationDate`对象。当`lastModificationDate`属性改变了，这个属性就不再强引用原始的日期数据了，但是原始的日期数据不会被释放，因为它被`originalData`强引用了。  

如果你不想让一个变量维持一个强引用关系，你应该把它声明为`__weak`，像这样：

```
 NSObject * __weak weakVariable;
```

因为一个弱引用不会使对象保持存在，当引用仍然还被使用着的时候，对象是可能被释放的。为了避免一个指针指向了一块已经被释放了的内存区域，当弱引用指向的对象被释放时，这个引用会被设置为指向`nil`。  

这也就意味着如果在之前的那个日期例子中使用了弱（weak）变量的话：

```
    NSDate * __weak originalDate = self.lastModificationDate;
    self.lastModificationDate = [NSDate date];
```

这个`originalDate`变量有指向`nil`的潜在可能。当`self.lastModificationDate`被重新赋值，属性就不再强引用原来的日期数据了，如果没有其他的强引用施加于其上，原始的日期数据就会被释放而`originalDate`设置成`nil`。  

弱变量可能会引起一些困惑，尤其是下面这种代码：  

```
    NSObject * __weak someObject = [[NSObject alloc] init];
```
  
  在这个例子里，新分配内存的对象并没有任何强引用施加于其上，所以立刻就会被释放，并且`someObject`也会被设置为`nil`。
  
  >注意: 与`__weak`相对应的就是`__strong`。你不需要特地声明`__strong`，因为默认情况就是它。
  
应该重点考虑一下一个需要多次访问一个弱引用属性多次的方法可能发生的结果，像这样：

```
- (void)someMethod {
    [self.weakProperty doSomething];
    ...
    [self.weakProperty doSomethingElse];
}
```

在这样的情况下，你可能想要把这个弱引用属性暂存（cache）在一个强引用变量中，以确保这个属性在你使用它的时候一直存在内存中：

```
- (void)someMethod {
    NSObject *cachedObject = self.weakProperty;
    [cachedObject doSomething];
    ...
    [cachedObject doSomethingElse];
}
```

在这个例子中，`cachedObject`变量保持了对原始弱引用属性数据值的强引用，所以原弱引用属性值不会被释放，直到程序运行离开`cachedObject`的作用域（并且`cachedObject`也没有被重新赋值为其他值）。  

一定要记住，如果你在使用一个弱引用属性之前想要知道它是不是`nil`，仅仅在用之前像这样测试它是不够的：

```
    if (self.someWeakProperty) {
        [someObject doSomethingImportantWith:self.someWeakProperty];
    }
```

因为在多线程应用中，属性可能会在测试和调用的中间被释放，导致这个测试失去作用。作为代替，你应该声明一个强引用局部变量来缓存这个值，像这样：

```
    NSObject *cachedObject = self.someWeakProperty;           // 1
    if (cachedObject) {                                       // 2
        [someObject doSomethingImportantWith:cachedObject];   // 3
    }                                                         // 4
    cachedObject = nil;                                       // 5
```

在这个例子里，在第1行创建了一个强引用，意味着这个对象将会为了测试和后期的调用一直存在。在第5行，`cachedObject`被设置为了`nil`，因此放弃了对于之前对象的强引用。如果没有其他强引用施加于原来对象之上，原来对象会被释放，同时，`someWeakProperty`将会被设置为`nil`。

###使用对某些类不安全的Unretained引用
在Cocoa和Cocoa Touch中，有很少的类还不支持弱引用，这也就意味着你不可能声明一个弱引用属性或者一个弱引用变量去追踪他们。这些类包括`NSTextView`，`NSFont`，`NSColorSpace`；想要看更全面的列表，请查询[Transitioning to ARC Release Notes](https://developer.apple.com/library/content/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011226)  

如果你需要弱引用这些类的话，你必须使用不安全的引用。对于属性来说，这意味着使用`unsafe_unretained`特征：  

```
@property (unsafe_unretained) NSObject *unsafeProperty;
```

对于变量，你需要使用`__unsafe_unretained`:

```
    NSObject * __unsafe_unretained unsafeReference;
```

一个不安全的引用和弱引用类似，他不能保持其关联的对象持续存在。但是，当这个对象被释放的时候，这个指针变量不会被设置为`nil`。这也就意味着这个指针会指向一段被释放的内存，我们称其为 *野指针（dangling pointer）*，这就是为什么它被称作为“不安全”的。向一个野指针发送消息会导致崩溃。

###Copy属性保存一份它自己的副本
在一些情况下，一个对象可能希望把别人设置给它的属性保存一个属于自己的副本。  

举个例子，之前图3-4展示的`XYZBadgeView`的类接口部分是这样的：

```
@interface XYZBadgeView : NSView
@property NSString *firstName;
@property NSString *lastName;
@end
```

两个`NSString`属性被声明了，也隐式地保持了对于其对应对象的强引用。  

考虑一下，当其他对象创建了一个字符串，并把其设置为badge view的一个属性值会发生什么，像这样：

```
    NSMutableString *nameString = [NSMutableString stringWithString:@"John"];
    self.badgeView.firstName = nameString; //译者注：firstName属性指向了nameString指向的地方，也就是说badge view的firstName属性和nameString局部变量指向了同一段内存，这也解释了后面为什么firstName属性会随着nameString的变化而变成了Jonny。
```

这种做法是可行的，因为`NSMutableString`是`NSString`的子类。即使badge view觉得它是在处理一个`NSString`，实际上，它处理的是一个`NSMutableString`。  

这也就意味着这个字符串可以改变：

```
    [nameString appendString:@"ny"];
```

在这个例子里，即使最开始badge view的`firstName`属性被设置为`John`，最后却变成了`Johnny`了，因为这个可变的字符串已经改变了。（译者注：此时  

你可能选择让badge view为每一个设置给它属性的值保存一个副本，以便当属性值被设置时有效地捕获这个字符串。通过在属性声明前增加`copy`特征：

```
@interface XYZBadgeView : NSView
@property (copy) NSString *firstName;   
@property (copy) NSString *lastName;
@end
```

现在view就会为这个两个字符串保存属于他自己的副本。即使被设置成了可变字符串并且这个可变字符串立刻被改变了，badge view仅捕获它属性值被设置时的值。举个例子：

```
    NSMutableString *nameString = [NSMutableString stringWithString:@"John"];
    self.badgeView.firstName = nameString; //译者注：这里实际上nameString的值被复制进了firstName属性指向的实例变量，现在值为John的量变成了两个，而后nameString局部变量变成了Johnny而firstName属性并不受影响。
    [nameString appendString:@"ny"];
```
这时，badge view的`firstName`属性是原来的“John”字符串的一个副本，不受其他影响。  

`copy`特征意味着属性将会使用一个强引用，因为他必须保持它创建的新对象存在。

> 所有你希望设置为`copy`的属性必须支持`NSCopying`，就是说它必须遵从`NSCopying`协议。关于协议在[Protocols Define Messaging Contracts](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithProtocols/WorkingwithProtocols.html#//apple_ref/doc/uid/TP40011210-CH11-SW2)中有详细描述。想要详细了解`NSCopying`请看[NSCopying](https://developer.apple.com/reference/foundation/nscopying)或[Advanced Memory Management Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html#//apple_ref/doc/uid/10000011i)。

如果你希望直接设置一个属性的实例变量为`copy`，举个例子，比如说在初始化方法中，不要忘记设置最原来的那个对象：

```
- (id)initWithSomeOriginalString:(NSString *)aString {
    self = [super init];
    if (self) {
        _instanceVariableForCopyProperty = [aString copy];
    }
    return self;
}
```


