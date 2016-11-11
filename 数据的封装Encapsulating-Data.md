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
setter方法会导致一些副作用。它可能会触发KVC通知，


