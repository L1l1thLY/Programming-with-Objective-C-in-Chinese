# Working-with-Protocols-协议的使用
##简述
在现实世界中，公务人员处理事件时需要遵循严格的流程。例如，执法部门在进行调查取证时必须遵守“章程”。

在面向对象编程中，为*某一对象* 的*某种特定情况* 定义*一组行为* 非常重要。例如：表视图（viewController）需要与数据源对象通信，以便找到显示内容。这意味着数据源必须能够响应表视图所有可能发送出的特定消息集合。

数据源可以是任何类的实例。例如：视图控制器（OS X上的NSViewController或iOS上的UIViewController的子类）、继承自NSObject的专用数据源类等。通过声明对象实现所必要的方法，使得表视图知道该对象是否可以用作其数据源。

Objective-C允许自定义协议，声明预计用于某种特定情况的方法。本章描述了定义协议的语法，以及如何在一个类的接口中让其遵守定义的协议，这意味着类必须实现其所需的方法。
##协议的定义
我们已知类接口声明了与该类相互关联的方法和属性。与此相反，协议用于*声明独立于任何特定类的方法和属性* 。

``` 
@protocol ProtocolName
// list of methods and properties
@end
```

协议可以包括实例方法、类方法以及属性的声明。

例如，用来显示一个饼状图的一个自定义的视图子类，如图所示：

![show examples](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Art/piechartsim.png)


为了尽可能提高视图重用性，所有关于显示信息内容的动作都被交由另一个对象负责，该对象被称为数据源。这意味着同一个视图类的多个对象可以通过不同的数据源显示相互独立的内容。

例如：显示饼状图所需的最基本信息包含每个部分的数值、相对大小、各自的标题等。因此，饼状图的数据源协议，应该类似于：

```
@protocol XYZPieChartViewDataSource
- (NSUInteger)numberOfSegments;
- (CGFloat)sizeOfSegmentAtIndex:(NSUInteger)segmentIndex;
- (NSString *)titleForSegmentAtIndex:(NSUInteger)segmentIndex;
@end
```

>注：上面的协议中使用NSUInteger来表示整数纯量的值。该类型将在下一章做进一步讨论。

饼状图的视图类的接口需要一个属性来追踪数据源对象。这个对象可以属于任意类，所以基本的属性类型应该是`id`。而关于这个对象我们唯一已知的是它遵守相关的协议。

**声明数据源属性的语法类似于：**

```
@interface XYZPieChartView : UIView
@property (weak) id <XYZPieChartViewDataSource> dataSource;
...
@end
```

Objective-C用尖括号`< >`来表示所遵守的协议。这个例子中对一个指向通用类型对象的指针声明了一个弱引用，并且令其遵守`XYZPieChartViewDataSource`协议。

>注：出于对象关系管理的需要，指向代理和数据源的属性通常被声明为弱引用。（详见章节：避免强引用循环）

在声明属性时指明其所遵守的协议。如果将视图属性指向一个不遵守协议的对象，即使基本的属性类型为通用型，也会受到编译器的警告。属性所指向的对象是`UIViewController`类还是`NSObject`类的实例对象并不重要。只要其遵守协议即可，这样饼状图视图就知道它可以请求需要显示的信息。

###非正式协议-拥有可选方法的协议
>译者注：正式协议定义的方法均为必须方法，所有符合该协议的类都需要实现。非正式协议中的方法为可选方法，一个符合协议的类可以不实现该方法

正式协议中声明的所有方法都是必需的方法。 这意味着任何符合该协议的类都必须实现所有方法。

但我们可以在协议中指定可选方法（一个类只有在需要时才实现的方法）。

例如：我们可以认为饼图上的标题是可选的。 如果数据源对象不实现`titleForSegmentAtIndex：`方法，则在视图中不显示标题。

我们可以使用`@optional`指令将协议方法标记为可选方法，如下所示：

```
@protocol XYZPieChartViewDataSource
- (NSUInteger)numberOfSegments;
- (CGFloat)sizeOfSegmentAtIndex:(NSUInteger)segmentIndex;
@optional
- (NSString *)titleForSegmentAtIndex:(NSUInteger)segmentIndex;
@end
```

在这个示例中，仅有`titleForSegmentAtIndex：`方法被标记为可选方法，在`titleForSegmentAtIndex：`之前的方法并未被标记为可选，所以在符合该协议的类中它们必须被实现。

`@optional`指令适用于紧随其后的所有方法，直到协议的定义结束或者遇到另一个指令，例如：`@required`。 我们可以向协议添加更多的方法，如下所示：

```
@protocol XYZPieChartViewDataSource
- (NSUInteger)numberOfSegments;
- (CGFloat)sizeOfSegmentAtIndex:(NSUInteger)segmentIndex;
@optional
- (NSString *)titleForSegmentAtIndex:(NSUInteger)segmentIndex;
- (BOOL)shouldExplodeSegmentAtIndex:(NSUInteger)segmentIndex;
@required
- (UIColor *)colorForSegmentAtIndex:(NSUInteger)segmentIndex;
@end
```

该示例定义了一个具有三个必需方法和两个可选方法（这两个可选方法处于指令`@optional`和`@required`之间）的协议。

###检查可选方法是否在运行时实现
如果协议中的方法被标记为可选，我们必须检查对象在调用该方法 *之前* 该方法是否被实现。

例如：饼图视图可能会测试如下的段标题方法：

```
NSString *thisSegmentTitle;
    if ([self.dataSourcerespondsToSelector:@selector(titleForSegmentAtIndex:)]) 
    {
        thisSegmentTitle = [self.dataSourcetitleForSegmentAtIndex:index];
    }
```

`responsesToSelector：`方法使用一个选择器在编译后引用一个方法的标识符。 我们可以通过`@selector（）`指令提供正确的标识符，指定方法的名称来提供正确的标识符。

如果该示例中的数据源实现了该方法，则使用标题；否则，标题仍然为`nil`。

>注：本地对象变量自动初始化为`nil`。

如果试图调用一个符合协议的`id`的`respondingToSelector：`方法，例如上面定义的，编译器会报告错误：“没有已知的实例方法”。 若你使用协议来限定`id`，所有静态类型检查都会恢复; 如果您尝试调用未在该协议中定义的方法，将产生错误。 可以通过将自定义协议设置为采用`NSObject协议`来避免编译器产生错误。

###继承其他协议
就像一个Objective-C类可以继承一个超类，我们也可以指定一个协议符合另一个协议。

例如：最好的做法就是定义您的协议符合`NSObject协议`（一些NSObject行为从其类接口拆分为单独的协议; `NSObject类`采用`NSObject协议`）。

通过指定自己的协议符合NSObject协议，表明任何采用自定义协议的对象也将为每个`NSObject协议`的方法提供实现。 你可以使用`NSObject`的一些子类，而不需要为这些`NSObject`方法提供自己的实现。 对于如上所述的情况，协议的采用是有效的。

要指定一个协议符合另一个协议，需要在尖括号中提供其他协议的名称，如下所示：

```
@protocol MyProtocol <NSObject>
...
@end
```

在本示例中，采用`MyProtocol`的所有对象均可有效地采用`NSObject协议`中声明的所有方法。

##使用协议
表示类采用协议的语法使用尖括号，如下所示：

```
@interface MyClass : NSObject <MyProtocol>
...
@end
```

这意味着所有MyClass实例不仅将响应在接口中明确声明的方法，而且MyClass也提供MyProtocol中所需方法的实现。 无需重新声明类接口中的协议方法，只需要采用协议即可。

>注意：编译器不会自动合成已采用协议中声明的属性。

如果一个类需要采用多个协议，可以将其指定为逗号分隔的列表，如下所示：

```
@interface MyClass : NSObject <MyProtocol, AnotherProtocol, YetAnotherProtocol>
...
@end
```

>**译者注：可以以任意顺序列出所有要采用的协议，对于结果没有任何影响**

>提示：如果你发现自己在类中采用了大量的协议，这可能是建立一个过度复杂的类的标志之一。我们需要通过分割现有类中必要的行为建立多个较小的拥有明确责任的类。
对于新的OS X和iOS开发人员有一个相对常见的陷阱：使用单个应用程序委托类来包含应用程序的大多数功能（管理底层数据结构，向多个用户界面元素提供数据，以及响应手势和其他用户交互）。 随着复杂度的提高，类的维护难度不断上升。

一旦类声明符合协议，该类必须为每个必需的协议方法和您选择的任何可选方法提供方法实现。 如果未能实现任一必需的方法，编译器将发出警告。

>注意：协议中的方法声明与其他声明一样， 实现中的方法名称和参数类型必须与协议中的声明相匹配。

**Cocoa和Cocoa Touch定义了大量的协议**

Cocoa和Cocoa Touch对象会在各种不同的情况下使用协议。例如：表视图类（适用于OS X的`NSTableView`和适用于iOS的`UITableView`）都使用数据源对象为其提供必要的信息。两者都定义自己的数据源协议，其使用方式与上述`XYZPieChartViewDataSource`协议示例大致相同。两个表视图类也允许您设置一个委托对象，同时它也必须符合相关的`NSTableViewDelegate`或`UITableViewDelegate`协议。该委托对象负责处理用户交互，或定制某些条目的显示。

一些协议用于指示类之间的非分级相似性，而不是链接到特定的类要求（一些协议可实现比涉及多个不相关类更一般的Cocoa或Cocoa Touch通信机制）

例如：许多框架模型对象（例如`NSArray`和`NSDictionary`等集合类）支持`NSCoding`协议，这意味着它们可以对其属性进行编码和解码，以便存档或分发为原始数据。 若图中的每个对象都采用`NSCoding`协议，可以使得将整个图对象写入磁盘相对容易。

几个Objective-C语言级功能也依赖于协议。例如：为了使用`快速枚举`，集合必须采用`NSFastEnumeration`协议，这样可使枚举集合容易。此外，例如复制一些对象：在使用具有复制属性的属性时，（如“复制属性维护自己的副本”中所述）任何你试图复制的对象必须采用`NSCopying`协议，否则会产生运行时异常。

##用于匿名对象的协议
协议在对象的类未知或需要保持隐藏的情况下也是可用的。

作为示例，框架的开发者可以选择不为框架内的某个类发布接口。 因为类名是未知的，所以该框架的用户不可能直接创建该类的实例。 与此相反，框架中的一些其他对象通常被指定返回现成的实例，如下所示：

```
id utility = [frameworkObject anonymousUtility];
```

为了使这个anonymousUtility对象可用，即使没有提供原始的类接口，框架的开发人员也可以发布一个显示其某些方法的协议。 这意味着类保持匿名，但对象仍然可以以有限的方式使用其方法：

```
  id <XYZFrameworkUtility> utility = [frameworkObject anonymousUtility];
```

例如：你正在编写一个使用Core Data框架的iOS应用程序，你可能会遇到`NSFetchedResultsController`类。 该类旨在帮助数据源对象将存储的数据提供给iOS `UITableView`，从而提供诸如行数等信息。

如果您使用将内容分割为多个部分的表视图，您还可以向提取的结果控制器询问相关的部分信息。 `NSFetchedResultsController`类不会返回包含此节信息的特定类，而是返回一个符合`NSFetchedResultsSectionInfo`协议的匿名对象。 于是我们仍然可以通过如下方法查询对象所需的信息，例如节中的行数：

```
  NSInteger sectionNumber = ...
    id <NSFetchedResultsSectionInfo> sectionInfo =
            [self.fetchedResultsController.sections objectAtIndex:sectionNumber];
    NSInteger numberOfRowsInSection = [sectionInfo numberOfObjects];
```

>即使你不知道`sectionInfo`对象所属的类，`NSFetchedResultsSectionInfo`协议也使它可以响应`numberOfObjects`消息。

##总结(译者注)
本章讲解了协议的概念，其包含正式与非正式协议。我们可以通过在`@protocol`部分方法名来定义一组正式协议（在@optional后定义非正式协议）。在`@interface`声明中的类名之后列出用尖括号括起来的协议名称，对象即可采用这些协议。采用协议之后，对象便承诺了要实现协议中每一个必须实现的方法（正式协议的内容）。若没有实现所承诺的方法，编译器会对此发出警告。

  








 
 
    
    









                                                                                                                                                                                                                                                                                                                                                                                             
















