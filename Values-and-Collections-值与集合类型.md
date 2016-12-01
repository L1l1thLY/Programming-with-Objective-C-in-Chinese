# Values-and-Collections-值与集合类型
##简述
面向对象的编程语言Objective-C是C的一种超集，我们可以在Objective-C代码中使用标准C的纯量（非对象）类型，例如`int`，`float`和`char`。同时在Cocoa和Cocoa Touch应用程序中还有其他纯量类型，例如：`NSInteger`，`NSUInteger`和`CGFloat`等，它们在不同的目标体系结构（系统结构）中有不同的定义。

纯量类型用于您不需要使用对象来表示值（或相关的开销）的情况。虽然字符串通常表示为`NSString`类的实例，但数字值通常存储在纯量的局部变量或属性中。

我们可以在Objective-C中声明一个C风格的数组，但你会发现Cocoa和Cocoa Touch应用程序中的集合通常使用`NSArray`或`NSDictionary`类的实例来表示。这些类只能用于收集Objective-C对象，这意味着您需要创建类`NSValue`，`NSNumber`或`NSString`的实例，以便在您可以将它们添加到集合之前表示其值。

指南前面的章节经常使用NSString类及其初始化和类工厂方法，以及Objective-C` @“string”`文字，它提供了一个简洁的语法来创建一个`NSString`实例。本章将介绍如何使用方法调用或通过Objective-C值文字语法创建`NSValue`和`NSNumber`对象。

##基本C类型在Objective-C中可用
每个标准C纯量变量类型在Objective-C中可用：

```
int someInteger = 42;
float someFloatingPointNumber = 3.1415;
double someDoublePrecisionFloatingPointNumber = 6.02214199e23;
```

以及标准C运算符：

```
int someInteger = 42;
someInteger++;            // someInteger == 43
int anotherInteger = 64;
anotherInteger--;         // anotherInteger == 63
anotherInteger *= 2;      // anotherInteger == 126
```
如果你使用一个纯量类型的Objective-C属性，像这样：

```
@interface XYZCalculator : NSObject
@property double currentValue;
@end
```
在通过点语法访问值时，也可以在其属性上使用C运算符，如下所示：

```
@implementation XYZCalculator
- (void)increment {
    self.currentValue++;
}
- (void)decrement {
    self.currentValue--;
}
- (void)multiplyBy:(double)factor {
    self.currentValue *= factor;
}
@end
```
点`·`语法是一个关于访问器方法调用的句法包装，因此本示例中的每个操作都等同于首先使用`get accessor`方法获取值，然后执行操作，最后使用`set accessor`方法将值设置为结果。
###Objective-C定义附加基本类型
在Objective-C中定义`BOOL`纯量类型，用于保存布尔值，即`YES`或`NO`。`YES`在逻辑上等于`true`和`1`，而`NO`等于`false`和`0`。

Cocoa和Cocoa Touch对象的许多参数也使用特殊的纯量数值类型，例如`NSInteger`或`CGFloat`。

例如: `NSTableViewDataSource`和`UITableViewDataSource`协议（在上一章中描述）都具有请求显示行数的方法：

```
@protocol NSTableViewDataSource <NSObject>
- (NSInteger)numberOfRowsInTableView:(NSTableView *)tableView;
...
@end
```
这些类型，例如`NSInteger`和`NSUInteger`，在不同系统结构中有不同的定义。 当为32位环境（例如iOS）时，它们分别是32位有符号和无符号整数; 当构建用于64位环境（例如现代OS X）时，它们分别是64位有符号和无符号整数。
>译者注：现阶段iOS环境也为64位。

如果您可能跨API边界（包括内部和导出的API）传递值（比如应用程序代码和框架之间的方法或函数调用中的参数或返回值）那么最佳做法是使用这些平台特定的类型。

对于局部变量（例如循环中的计数器），如果您知道该值在标准限制范围内，则可以使用基本C类型。

###通过C结构保存原始值
一些Cocoa和Cocoa Touch API使用C结构来保存它们的值。 例如：可以向字符串对象请求子字符串的范围，如下所示：

```
NSString *mainString = @"This is a long string";
NSRange substringRange = [mainString rangeOfString:@"long"];
```
`NSRange`结构保存位置和长度。 在这种情况下，`substringRange`将保留一个范围{10,4} 表示@“long”是`mainString`中基于0的索引在10处的字符，@“long”长度为4个字符 。

同样，如果你需要编写自定义绘图代码，你需要与Quartz进行交互，这需要基于CGFloat数据类型的结构，例如OS X上的`NSPoint`和`NSSize`，iOS上的`CGPoint`和`CGSize`。 同样，对于不同系统架构，`CGFloat`有不同的定义。

有关Quartz 2D绘图引擎的更多信息，请参阅[Quartz 2D编程指南](https://developer.apple.com/library/content/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/Introduction/Introduction.html#//apple_ref/doc/uid/TP30001066)。

##对象可以表示原始值
如果需要将纯量值表示为对象（例如：在使用下一节中描述的集合类）时，可以使用Cocoa和Cocoa Touch提供的基本值类型之一。
###字符串由NSString类的实例表示
正如你在前面的章节中看到的，`NSString`用于表示一串字符，如"Hello World"。 有各种方法来创建`NSString`对象，包括标准分配和初始化，类工厂方法或赋值语法：

```
NSString *firstString = [[NSString alloc] initWithCString:"Hello World!" encoding:NSUTF8StringEncoding];
NSString *secondString = [NSString stringWithCString:"Hello World!" encoding:NSUTF8StringEncoding];
NSString *thirdString = @"Hello World!";
```
这些示例中的每一个都有效地完成相同的事情 -> 创建表示所提供的字符的字符串对象。

基本的`NSString`类是不可变的，这意味着其内容在创建时设置，以后不能更改。 如果需要表示不同的字符串，则必须创建一个新的字符串对象，如下所示：

```
NSString *name = @"John";
name = [name stringByAppendingString:@"ny"];    // returns a new string object
```
`NSMutableString`类是`NSString`的可变子类，允许您在运行时使用像`appendString：`或`appendFormat：`等方法更改其字符内容，如下所示：

```
NSMutableString *name = [NSMutableString stringWithString:@"John"];
[name appendString:@"ny"];   // same object, but now represents "Johnny"
```
####格式字符串用于从其他对象或值构建字符串
使用一个格式字符串构建一个包含变量值的字符串。 这允许您使用格式说明符指示如何插入值：

```
int magicNumber = ...
NSString *magicString = [NSString stringWithFormat:@"The magic number is %i", magicNumber];
```

可用的格式说明符在[字符串格式说明符](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Strings/Articles/formatSpecifiers.html#//apple_ref/doc/uid/TP40004265)中已有描述。 有关字符串的更多信息，请参阅[字符串编程指南](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Strings/introStrings.html#//apple_ref/doc/uid/10000035i)。

###数字由NSNumber类的实例表示
`NSNumber`类用于表示任何基本C纯量类型，包括`char`，`double`，`float`，`int`，`long`，`short`和每个的无符号变体，以及Objective-C布尔类型`BOOL`。

与`NSString`一样，您有各种选项来创建`NSNumber`实例，包括分配和初始化或类工厂方法：

```
NSNumber *magicNumber = [[NSNumber alloc] initWithInt:42];
NSNumber *unsignedNumber = [[NSNumber alloc] initWithUnsignedInt:42u];
NSNumber *longNumber = [[NSNumber alloc] initWithLong:42l];
NSNumber *boolNumber = [[NSNumber alloc] initWithBOOL:YES];
NSNumber *simpleFloat = [NSNumber numberWithFloat:3.14f];
NSNumber *betterDouble = [NSNumber numberWithDouble:3.1415926535];
NSNumber *someChar = [NSNumber numberWithChar:'T'];
```
它也可以使用Objective-C赋值语法创建`NSNumber`实例：

```
NSNumber *magicNumber = @42;
NSNumber *unsignedNumber = @42u;
NSNumber *longNumber = @42l;
NSNumber *boolNumber = @YES;
NSNumber *simpleFloat = @3.14f;
NSNumber *betterDouble = @3.1415926535;
NSNumber *someChar = @'T';
```
这些例子等同于使用`NSNumber`类的工厂方法。
一旦创建了`NSNumber`实例，就可以使用一个访问器方法请求纯量值：

```
int scalarMagic = [magicNumber intValue];
unsigned int scalarUnsigned = [unsignedNumber unsignedIntValue];
long scalarLong = [longNumber longValue];
BOOL scalarBool = [boolNumber boolValue];
float scalarSimpleFloat = [simpleFloat floatValue];
double scalarBetterDouble = [betterDouble doubleValue];
char scalarChar = [someChar charValue];
```
`NSNumber`类还提供了使用其他Objective-C基本类型的方法。 例如：如果需要创建纯量`NSInteger`和`NSUInteger`类型的对象表示，正确的方法如下：

```
SInteger anInteger = 64;
NSUInteger anUnsignedInteger = 100;
NSNumber *firstInteger = [[NSNumber alloc]initWithInteger:anInteger];
NSNumber *secondInteger = [NSNumbernumberWithUnsignedInteger:anUnsignedInteger];
NSInteger integerCheck = [firstInteger integerValue];
NSUInteger unsignedCheck = [secondInteger unsignedIntegerValue];
```
所有`NSNumber`实例都是不可变的，并且没有可变子类; 如果你需要一个不同的号码，只能使用另一个`NSNumber`实例。

>**注意：`NSNumber`实际上是一个类集群。 这意味着当您在运行时创建实例时，您将获得一个合适的具体子类来保存提供的值。 只是将创建的对象作为`NSNumber`的实例。**

###使用NSValue类的实例表示其他值
`NSNumber`类本身是基本`NSValue`类的子类，它提供了围绕单个值或数据项的对象包装器。 除了基本的C纯量类型，`NSValue`也可以用于表示指针和结构体。

`NSValue`类提供了各种工厂方法来创建具有给定标准结构的值，这使得我们可以轻松地创建一个实例代表。例如：`NSRange`，就像本章前面的示例一样：

```
NSString *mainString = @"This is a long string";
NSRange substringRange = [mainString rangeOfString:@"long"];
NSValue *rangeValue = [NSValue valueWithRange:substringRange];
```
也可以创建`NSValue`对象来表示自定义结构。 如果您特别需要使用C结构体（而不是Objective-C对象）来存储信息，如下所示：

```
typedef struct {
    int i;
    float f;
} MyIntegerFloatStruct;
```
您可以通过提供结构体的指针以及编码的Objective-C类型来创建`NSValue`实例。（ `@encode（）`编译器指令用于创建正确的Objective-C类型）如下所示：

```
struct MyIntegerFloatStruct aStruct;
aStruct.i = 42;
aStruct.f = 3.14;

NSValue *structValue = [NSValue value:&aStructwithObjCType:@encode(MyIntegerFloatStruct)];
```
标准C引用运算符（＆）用于为`value`参数提供`aStruct`的地址。

##大多数集合都是对象
虽然我们可以使用C数组来保存纯量值的集合，甚至是对象指针，但Objective-C代码中的大多数集合都是Cocoa和Cocoa Touch集合类的实例，如`NSArray`，`NSSet`和`NSDictionary`。

这些类用于管理对象组，因此您添加到集合的任何项目必须是Objective-C类的实例。如果需要添加纯量值，则必须首先创建一个合适的`NSNumber`或`NSValue`实例来表示它。

集合类使用强引用来跟踪其内容，而不是以某种方式维护每个集合对象的单独的副本。这意味着，您添加到集合的所有对象均将保持活动状态，至少与集合保持活动状态的时间相当。例如：[通过所有权和责任管理对象图](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/EncapsulatingData/EncapsulatingData.html#//apple_ref/doc/uid/TP40011210-CH5-SW3)中的描述。

除了跟踪其内容之外，每个Cocoa和Cocoa Touch集合类都可以轻松执行某些任务，例如：枚举、访问特定项目或查明特定对象是否是集合的一部分。

基本的`NSArray`，`NSSet`和`NSDictionary`类是不可变的，这意味着它们的内容在创建时设置。但同时每个类还拥有一个可变子类，允许您随意添加或删除对象。

有关Cocoa和Cocoa Touch中可用的不同集合类的更多信息，请参阅[集合编程（Collections Programming Topics）](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Collections/Collections.html#//apple_ref/doc/uid/10000034i)。

###数组是有序集合类
`NSArray`用于表示对象的有序集合类。 其唯一的要求是每个元素都是一个Objective-C对象，即不需要每个对象都是同一个类的实例。

为了保持数组中的顺序，每个元素存储在一个从0开始的索引中。如下
图所示：
![屏幕快照 2016-11-20 01.51.15-w263](media/14790846793585/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-11-20%2001.51.15.png)

####创建数组
与本章前面描述的纯量值类一样，您可以通过分配和初始化、类工厂方法或赋值语法创建数组。

取决于对象的数量，有多种不同的初始化和工厂方法可用：

```
+ (id)arrayWithObject:(id)anObject;
+ (id)arrayWithObjects:(id)firstObject, ...;
- (id)initWithObjects:(id)firstObject, ...;
```
`arrayWithObjects：`和`initWithObjects：`方法都需要一个nil终止作为可变数量的参数，这意味着你必须包括nil作为最后一个值，用法如下所示：

```
NSArray *someArray =[NSArray arrayWithObjects:someObject, someString, someNumber, someValue, nil];
```
该示例创建一个类似于前面所示的数组（上图）。 第一个对象`someObject`的数组索引为0; 最后一个对象`someValue`的索引为3。

若提供的值之一为`nil`，可能会无意中截断对象列表，如下所示：

```
id firstObject = @"someString";
id secondObject = nil;
id thirdObject = @"anotherString";
NSArray *someArray = [NSArray arrayWithObjects:firstObject, secondObject, thirdObject, nil];
```
在这种情况下，`someArray`将只包含`firstObject`，因为`nil secondObject`将被解释为项目列表的结尾。
#####赋值语法创建数组
也可以使用Objective-C逐个对象创建一个数组，如下所示：

```
NSArray *someArray = @[firstObject, secondObject, thirdObject];
```
当使用此语法时，不应使用`nil`终止对象列表，实际上nil是无效值。 如果您尝试执行以下代码，将在运行时发生异常，例如：

```
id firstObject = @"someString";
id secondObject = nil;
NSArray *someArray = @[firstObject, secondObject];// exception: "attempt to insert nil object"
```
如果确实需要在某个集合类中表示一个`nil`值，那么应该使用`NSNull`单例类，详见[用NSNull表示nil](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/FoundationTypesandCollections/FoundationTypesandCollections.html#//apple_ref/doc/uid/TP40011210-CH7-SW34)中所述。

#####查询数组对象
在创建数组后，您可以查询其对象的数量或是否包含给定对象等信息，如下所示：

```
NSUInteger numberOfItems = [someArray count];
if ([someArray containsObject:someString]) {
        ...
}
```
您还可以查询给数组定索引处的元素。若查询无效的索引，会在运行时遇到超出范围异常，因此您应该首先检查数组中对象数量，如下所示：

```
if ([someArray count] > 0) {
        NSLog(@"First item is: %@", [someArray objectAtIndex:0]);
}
```
此示例检查对象数量是否大于零。 如果是，则它记录第一项的描述（索引0）。

**下标**
还有一个下标语法可以替代`objectAtIndex：`，与访问标准C数组中的值方法相似。 前面的例子可以这样重写：

```
if ([someArray count] > 0) {
        NSLog(@"First item is: %@", someArray[0]);
}
```
#####排序数组对象
`NSArray`类还提供了多种方法来对其收集的对象进行排序。 因为`NSArray`是不可变的，这些方法会返回一个包含排序顺序的项目的新数组。

例如，可以通过调用`compare：`对每个字符串排序的结果对字符串数组进行排序，如下所示：

```
NSArray *unsortedStrings = @[@"gammaString", @"alphaString", @"betaString"];
NSArray *sortedStrings = [unsortedStrings sortedArrayUsingSelector:@selector(compare:)];
```

#####可变性
虽然`NSArray`类本身是不可变的，但这对其收集的对象没有影响。 如果你向一个不可变数组添加一个可变字符串，例如：

```
NSMutableString *mutableString = [NSMutableString stringWithString:@"Hello"];
NSArray *immutableArray = @[mutableString];
```
没有什么可以阻止你改变这个字符串：

```
if ([immutableArray count] > 0) {
    id string = immutableArray[0];
    if ([string isKindOfClass:[NSMutableString class]]) {
            [string appendString:@" World!"];
        }
}
```
如果需要在初始创建后能够从数组中添加或删除对象，则需要使用`NSMutableArray`，它添加了多种方法来实现添加，删除或替换一个或多个对象：

```
NSMutableArray *mutableArray = [NSMutableArray array];
[mutableArray addObject:@"gamma"];
[mutableArray addObject:@"alpha"];
[mutableArray addObject:@"beta"];
[mutableArray replaceObjectAtIndex:0 withObject:@"epsilon"];
```
此示例创建一个以对象`@“epsilon”、@“alpha”、@“beta”`结尾的数组。可以不创建辅助数组对可变数组进行排序：

```
[mutableArray sortUsingSelector:@selector(caseInsensitiveCompare:)];
```
在这种情况下，包含的对象将被排序为不区分大小写的升序，顺序为`@“alpha”、@“beta”、@“epsilon”`。

###集合是无序集合类
`NSSet`类似于数组，但维护着一组无序的不同对象，如图所示：
![屏幕快照 2016-11-20 02.38.46-w354](media/14790846793585/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-11-20%2002.38.46.png)
因为集合无需保持顺序，所以在测试成员资格时，集合相对数组供了较大的性能改进。

基本的`NSSet`类又是不可变的，所以它的内容必须在创建时使用分配和初始化或类工厂方法指定，如下所示：

```
NSSet *simpleSet =[NSSet setWithObjects:@"Hello, World!", @42, aValue, anObject, nil];
```
与`NSArray`一样，`initWithObjects：`和`setWithObjects：`方法都采用`nil`终止可变数量的参数。 可变的`NSSet`子类是`NSMutableSet`。

即使您尝试多次添加同一对象，集合仅存储对单个对象的一个引用：

```
NSNumber *number = @42;
NSSet *numberSet =[NSSet setWithObjects:number, number, number, number, nil];
    // numberSet only contains one object
```
有关集合的更多信息，请参阅[集合：无序的对象集合](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Collections/Articles/Sets.html#//apple_ref/doc/uid/20000136)。

###字典收集键值对
`NSDictionary`不是简单地维护有序或无序的对象集合，而是将对象存储在给定的键上，然后可以将键用于检索。

最好的做法是使用字符串对象作为字典键，如图所示：
![屏幕快照 2016-11-20 02.47.14-w709](media/14790846793585/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-11-20%2002.47.14.png)
>**注意**：可以使用其他对象作为键，但重要的是要注意每个键均需要被复制供字典使用，因此必须支持`NSCopying`协议。但是，如果您希望能够使用键值编码，则您必须对字典对象使用字符串键。

####创建字典
可以使用分配和初始化或类工厂方法创建字典，如下所示：

```
 NSDictionary *dictionary = [NSDictionary dictionaryWithObjectsAndKeys:
                   someObject, @"anObject",
             @"Hello, World!", @"helloString",
                          @42, @"magicNumber",
                    someValue, @"aValue",
                             nil];
```
>**注意**：对于`dictionaryWithObjectsAndKeys：`和`initWithObjectsAndKeys：`方法，每个对象应在它的键之前被指定，并且对象和键的列表必须是以nil为终止的。

#####赋值语法创建字典
Objective-C还提供了字典创建的赋值语法，如下所示：

```
NSDictionary *dictionary = @{
                  @"anObject" : someObject,
               @"helloString" : @"Hello, World!",
               @"magicNumber" : @42,
                    @"aValue" : someValue
};
```
>**注意**：对于字典字的赋值法创建，键是在其对象之前指定的，并且不以`nil`为终止。
>译者注：与数组的赋值法创建过程类似。

####查询字典
一旦你创建了一个字典，你可以要求它存储对象给定的键，像这样：

```
NSNumber *storedNumber = [dictionary objectForKey:@"magicNumber"];
```
如果没有找到对象，`objectForKey：`方法将返回`nil`。
**下标**
还有一个下标语法替代`objectForKey：`，如下所示：

```
NSNumber *storedNumber = dictionary[@"magicNumber"];
```
####可变性
如果在创建后需要从字典中添加或删除对象，则需要使用`NSMutableDictionary`子类，如下所示：

```
[dictionary setObject:@"another string" forKey:@"secondString"];
[dictionary removeObjectForKey:@"anObject"];
```
###用NSNull表示nil
因为Objective-C中的`nil`意味着“无对象”，所以不能向本节中描述的集合类添加nil。如果需要在集合中表示“无对象”，可以使用`NSNull`类：

```
NSArray *array = @[ @"string", @42, [NSNull null] ];
```
`NSNull`是一个单例类，这意味着`null`方法将总是返回相同的实例。 这意味着您可以检查数组中的对象是否等于设定的`NSNull`实例：

```
for (id object in array) {
    if (object == [NSNull null]) {
        NSLog(@"Found a null object");
    }
}
```
##使用集合来保存对象图
`NSArray`和`NSDictionary`类可以很容易地将其内容直接写入磁盘，如下所示：

```
NSURL *fileURL = ...
NSArray *array = @[@"first", @"second", @"third"];
 
BOOL success = [array writeToURL:fileURL atomically:YES];
if (!success) {
    // an error occured...
}
```
如果每个包含的对象是属性列表类型之一（`NSArray`，`NSDictionary`，`NSString`，`NSData`，`NSDate`和`NSNumber`），可以从磁盘重建整个层次结构，像这样：

```
NSURL *fileURL = ...
NSArray *array = [NSArray arrayWithContentsOfURL:fileURL];
if (!array) {
     // an error occurred...
}
```
有关属性列表的更多信息，请参阅[属性列表编程指南](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/PropertyLists/Introduction/Introduction.html#//apple_ref/doc/uid/10000048i)。

如果您需要持久保存其他类型的对象，而不仅仅是上面显示的标准属性列表类，您可以使用`archiver`对象（如`NSKeyedArchiver`）来创建收集对象的归档。

创建归档的唯一要求是每个对象必须支持`NSCoding`协议。这意味着每个对象必须知道如何将自己编码到归档（通过实现`encodeWithCoder：`方法），并在从现有归档（`initWithCoder：`方法）读取时解码自身。

`NSArray`，`NSSet`和`NSDictionary`类及其可变子类都支持`NSCoding`，这意味着您可以使用归档器保留对象的复杂层次结构。例如，如果使用`Interface Builder`来布置窗口和视图，当你在可视化界面上创造了nib文件，nib文件就是这些对象层次关系的存档。在运行的时候，nib文件会被解压缩成为一套使用相关类的对象层次关系。

有关归档的详细信息，请参见[归档和序列化编程指南](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Archiving/Archiving.html#//apple_ref/doc/uid/10000047i)。

##使用最有效的集合枚举技术
Objective-C和Cocoa或Cocoa Touch提供了各种方法来枚举集合的内容。 虽然可以使用传统的C中的`for`循环来遍历内容，如下所示：

```
int count = [array count];
for (int index = 0; index < count; index++) {
    id eachObject = [array objectAtIndex:index];
        ...
}
```
但最好的做法是使用本节中描述的技术之一。

###快速枚举枚举集合
很多集合类符合`NSFastEnumeration`协议，包括`NSArray`，`NSSet`和`NSDictionary`。 这意味着您可以使用快速枚举 - 一个Objective-C语言级的功能。

列举数组或集合内容的快速枚举语法如下所示：

```
for (<Type> <variable> in <collection>) {
        ...
    }
```
例如：可以使用快速枚举来记录数组中每个对象的描述，如下所示：

```
for (id eachObject in array) {
        NSLog(@"Object: %@", eachObject);
    }
```
`eachObject`变量在每次通过循环时自动设置为当前对象，因此每个对象都会显示一条日志语句。

如果要对字典使用快速枚举，则可以遍历字典键，如下所示：

```
for (NSString *eachKey in dictionary) {
        id object = dictionary[eachKey];
        NSLog(@"Object: %@ for key: %@", object, eachKey);
    }
```
快速枚举的行为非常类似于标准的C中的`for`循环，因此可以使用`break`关键字来中断迭代，或者`continue`前进到下一个元素。

如果要枚举有序集合，枚举应按顺序进行。 对于`NSArray`，这意味着第一次传递将用于索引0处的对象，第二次传递将用于索引1处的对象，以此类推。如果您需要跟踪当前索引，只需对迭代次数进行统计：

```
int index = 0;
for (id eachObject in array) {
    NSLog(@"Object at index %i is: %@", index, eachObject);
    index++;
}
```
###大多数集合支持枚举器对象
我们可以通过使用`NSEnumerator`对象枚举许多的Cocoa和Cocoa Touch集合。

你可以查询一个`NSArray`，例如，一个`objectEnumerator`或一个`reverseObjectEnumerator`。 它可以对这些对象进行快速枚举，如下所示：

```
for (id eachObject in [array reverseObjectEnumerator]) {
    ...
}
```
在这个例子中，循环将以相反的顺序对所收集的对象进行迭代，所以最后一个对象将会第一个输出，依此类推。

也可以通过重复调用枚举器的`nextObject`方法来迭代内容，如下所示：

```
id eachObject;
while ( (eachObject = [enumerator nextObject]) ) {
    NSLog(@"Current object is: %@", eachObject);
}
```
在本示例中，`while`循环用于将`eachObject`变量设置为每次通过循环的下一个对象。 当没有更多的对象时，`nextObject`方法将返回`nil`，它的值为`false`，循环终止。

>注意：相等运算符`==`与C赋值运算符`=`混淆是一个常见的程序员错误，如果你在条件分支或循环中为变量赋值，编译器会警告你，如下所示：

>```
if (someVariable = YES) {
    ...
}
```
>如果你确定要重新分配一个变量（整个赋值的逻辑值是`=`左边的最终值），你可以通过把赋值放在括号中来表示，如下所示：

>```
if ( (someVariable = YES) ) {
    ...
}
```

与快速枚举一样，枚举时不能改变集合中的对象。使用快速枚举比手动使用枚举器对象更快。

###基于块的集合枚举
我们也可以通过块枚举`NSArray`，`NSSet`和`NSDictionary`等。 块将在下一章中详细介绍。




