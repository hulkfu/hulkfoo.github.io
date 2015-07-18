---
layout: post
title: Objective C
---

2012年暑期去腾讯实习，当时的项目是iOS QQ浏览器，下面是一个星期的学习笔记，原题叫：
《Objective-C 学习笔记》—— 读《Objective-C 基础教材》《好学的 Objective-C》

# 有感
顾名思义，OC 是对 C 语言的面向对象扩展。凡是使用@开头的关键字，便是扩展的，从而实现了面向对象的 C 语言，当然也包括 C++。

虽然这是一本初学者教材，但是“间接”一章的文字却一语道破的编程的精髓——让反复有常。
无论是通过函数或许设计模式甚至框架，就是为了实现这一目标。
试想当一个方法可以处理所有需求，无论它怎么变，那么世界将会多么美妙！

由于运行时的存在，使得 OC 十分的灵活。主要一点是可以对任何实例发送
任何消息。能不能执行就看这个实例是否能够响应这个 selector 了。在 Java 中，
只能是实例调用方法。即没有实现 Ducking Type。
但 OC 还不能像 Ruby 那样在运行时修改代码。

# 一、基础
## 声明类
### @interface Interface
若要像 Java 中实现多接口，这需要定义协议，使用@protocol 关键字。

## 实现类
### @implementation Interface
由于 OC 的动态本质，使其不纯在真正的私有方法，所以就可以在实现之外访问实现的方法。这应该也是调用私有函数的理论基础吧。

函数的定义采用中缀符的语法技术。虽然和传统的 C 不一样，却有着优雅的效果：当向一个对象发送消息时就如同和它说话，很是方便。

方法名称最后的冒号也是名称的一部分，它告诉编译器和编程人员后面会有参数出没。当然，如果没有参数的话，就不需要这个冒号。

# 二、继承
@interface Class : NSObject
冒号后面便是继承的对象。
像 Java 一样，OC 也不支持多继承。

方法调度，首先在当前类中搜索相应的方法，如果没有找到，就再在超类中搜索。

# 三、复合
复合使得代码的耦合性更小。

getter 方法不用加“get”前缀。因为 get 在 Cocoa 中意味着这个方法会通过你当前参数
传入的指针来返回数值。

# 四、使用跨文件依赖关系
所依赖的文件改变，其就要重新编译。

关键字@class 告诉编译器，这是一个类，而且只需要通过引用来使用。即不需要调用
他的方法，只是用来组合进来。使用这个，一个可以降低编译时间，另一个可以解决两个类
相互引用报错的问题。如：@class A

当然，若需要知道类的细节，如继承的时候，就要使用#import 了。

# 五、Foundation Kit
主要是一些常用的数据类型集合。

### NSString
它的 stringWithFormat:方法可以像 NSLog 一样生成字符串。

### 类对象
类对象包含了指向超类的指针，类名和指向类方法列表的指针。类对象还包含一个 long型数据，为新创建的类实例对象指定大小。

### 字符串比较：
比较两个字符串是否相等，应该使用 isEqualToString:，而不是用“==”仅仅比较字符串的指针值。常用的选项有：

* NSCaseInsensitiveSearch：不区分大小写字符。
* NSLiteralSearch：进行完全比较，区分大小写。
* NSNumericSearch：比较字符串的字符个数，而不是字符值。

### 可变字符串：
NSMutableString，相当于 Java 中的 StringBuffer，可以使用类方法 stringWithCapacity:来创建。常见操作：

* appendString: (NSString *) aString;
* appendFormat: (NSString *) format, ...;
* deleteCharactersInRange: (NSRange) range;

### NSArray
一个泛型的数组类，可以存入任何 OC 类。

可以用 arrayWithObjects：来创建一个新的 NSArray。发送一个以逗号分隔的对象列表，在列表结尾添加 nil 代表列表结束。

使用 objectAtIndex：来获得元素。

### NSMutableArray
可变的 NSArray。

### 枚举
使用- (NSEnumerator *) objectEnumerator 来创建。如：[array objectEnumerator]

快速枚举：for in 语句

### NSDictionary
使用 dictionaryWithObjectsAndKeys：来创建字典。

### NSMutableDictionary

* 初始化：dictionaryWithCapacity：
* 添加元素：setObject:forkey：
* 删除元素：removeObject:forkey：

### 各种数值：
* NSNumber
* NSValue
* NSValue 可以包装任意值。因此可以使用其将结构放入 NSArray 或 NSDictionary 等中。

+ (NSValue *) valueWithBytes: (const void *) value objCType: (const char *) type;
@encode 编译器指令可以接受数据类型的名称并生成合适的字符串。

* NSNull 在集合中由于 nil 表示结尾，用 NSNull 来代替。

[NSNull null]总是返回一样的数值，所以可以使用运算符==将该值与其他值进行比较。

# 六、内存管理
Cocoa 内存管理规则:

* 当使用 new、alloc 或 copy 方法创建一个对象时，该对象的保留计数器为 1.当不再使用该对象时，要向该对象发送一条 release 或 autorelease 消息。这样，该对象将在其使用寿命结束时被销毁。
* 通过其他任何方法获得一个对象时，则假设该对象的保留计数器值为 1，而且已经被设置成自动释放，这样就不需要任何操作来确保该对象被清理了。如果打算在一段时间内拥有该对象，则需要保留它并确保在操作完成时释放。
* 如果保留了某个对象，需要（最终）释放或自动释放该对象。必须保持 retain 方法和 release 方法的使用次数相等。

访问方法中的保留与释放的经典代码：

```
- (void) setEngin: (Engine *) newEngine
{
[newEngine retain];
[engin release];
engine = newEngine;
}
```

除了 allock，new 和 copy 外，其它的添加都会在其以后的代码中自己去释放的。

如 addSubview，一个 view 被添加了，其 retaion count 就加 1，这都是在 addSubview 函
数内部做的。然后当父类自己要消失前，会把其所使用的子类先 release 一遍。就是自己编
程也要这样做。

因为 addSubview 时相当于在其 subviews 链表属性中加了一个，然后 deallc
时，这个属性肯定会被 release 的，然后在链表中，又是如上的逻辑。

其实这么一想，只要大家都遵守这个规则，就没有问题。用那三样：alloc、new 和 copy，
说明新创建的这个对象就属于当前的 context 中，所以你要管。其它呢，就属于其他 context
中了，就交给他们吧。

#七、对象初始化
先分配，后初始化。即先 alloc 后 init。

初始化时要嵌套调用：Car *car = [[Car alloc] init];

```
If (self = [super init]) {
...
}
```

暗示了 self 对象可能发生变化。这样做是为了确保超类能够获得对象并使其运行。

init 方法可以返回完全不同的对象。因为实例变量所在的内存位置到隐藏的 self 参数之
间的距离是固定的。如果从 init 返回一个新对象，则需要更新 self，以便其后的任何实例变
量的引用可以被映射到正确的内存位置。这个赋值只影响该 init 方法中 self 的值，而不影响
该方法范围以外的任何内容。

初始化函数规则:

* 不需要为自己的类创建初始化函数方法。如果不需要设置任何状态，或者只需要alloc 方法将内存清零的默认行为，则不需要 init。
* 如果构造了一个初始化函数，则一定要在指定的初始化函数中调用超类的初始化函数。
* 如果初始化函数不止一个，则要选择一个作为指定初始化函数。被选定的方法应该调用超类的指定初始化函数。一般是参数最多的那个初始化函数。要按照指定初始化函数的形式实现所以其他初始化函数。

#八、属性
省略了 setter 和 getter 方法的实现，同时

* @property
* @synthesize

可以使用圆点来获得或修改对象的属性，其实还是调用的生成的方法，并没有直接去修改或读取。

#九、类别
不能添加实例变量，只能扩展类的方法。

使用时，引入头文件，即扩展了一个类，而且对所有实例都会有效。在方法名字相同的时候，类别具有更高的优先级。

在实现时，只要包含了定义就可以在这里实现。
所以可以直接在 m 文件中定义私有类别，然后实现。反正不需要向外面暴露。
在同名的 m 文件中的实现的优先级高于其他。
在使用时，只有包含了定义才可以去使用。

例子：

```
@interface NSStirng (NumberConvenience)
- (NSNumber *) lengthAsNumber;
@end
```

表示向 NSString 类添加一个名称为 NumberConvenience 的类别。

然后实现：

```
@implementation NSString (NumberConvenience)
...
@end
```

```
#import "NSString+NumberConvenience.h"
[@"apple" lengthAsNumber]; //返回 5
```
有些方法我们不想公开，如苹果的 WebView 中的很多 API，有不想编译器警告，这时我们可以创建一个类别来实现这些方法。

局限性：

1. 无法向类别中添加新的实例变量。
2. 名称冲突时，类别具有更高的优先级。

## 向类别中添加变量
可以利用 Objective-C 运行时内置的一个底层功能实现。
在不继承且不改变对象声明的情况下，将一个变量和现有类关联起来。这中技术称作关联引用。无论是否通过类别都可以使用。
在使用关联引用时，不是真正的在类别中新增一个成员变量，也没有与之关联的属性。
也没有关联存取器函数。核心就是关联引用仅仅是一个和自定义类的具体实例关联的一个存储器。

1. objc_setAssociatedObject 函数，给类的实例添加一个关联引用，4 个参数：想关联到数据的对象、获取数据的键值、存储引用的值及一个相关策略，这个策略定义了如何管理存储值的内存。
2. objc_getAssociatedObject 函数，访问关联中存储的值。两个参数：数据关联的对象以及关联数据时使用的键值。
3. 最好，不使用了，再调用 objc_setAssociatedObject 函数并传入 nil 来移除关联。

在所有情况下，和值关联的键必须是唯一的。键的实际数据类型是 void*。通常，如果
要使用一个为该键声明的静态变量。这样就可以确保和该键关联的指针总是指向该指针
的当实例并且是唯一的。

对于简单的，一个 static 变量就可以搞定，如 int。

## 类别的作用：

1. 将类的实现分散到多个不同文件或多个不同框架中。
2. 创建对私有方法的前向引用。
3. 向对象添加非正式协议。

这里有些像 Ruby 的 Mix-in 特性，可以很方便的扩展一个类，又不需要继承或实现某个接口。这样可以在任何范围能实现想要的功能。估计这和他们都有着优雅的 smalltalk 血统有关吧。

## 委托
委托是一种对象，另一个类的对象会要求委托对象执行它的某些操作。

因此一个委托对象，只需要实现打算调用的方法即可，不需要在@interface 中声明方法。
当把委托对象声明为一个 NSObject 的类别时，那么任何类的对象都可以成为委托对象。

## 响应选择器
选择器只是一个方法名称，但它以 Objective-C 运行时使用的特殊方式编码，以快速执行查询。

@selector( ) 指定选择器，如下使用：

```
if ( [car respondsToSelector : @selector(setEngine:)] ) {
...
}
```

有了这个功能，就可以实现所谓的 duck typing: “If it walks like a duck and quacks like a duck it must be a duck. ”
这种动态语言的特性，可以使编程关注的不是类型本身，而是如何使用，这样可以更有逻辑性。

#十、协议
正式协议要求实现其所有方法，如同 Java 中的接口。使用@protocol 来定义。

在类的声明出，在其继承的类后面用一对尖括号< >包着需要实现的协议。

想想也知道，协议只能实现其他协议，也可以理解为继承其他协议，但是不能够继承其他类。
默认必须实现，即@required。可以用@optional 声明可选实现。

可以在使用数据类型中为实例变量和方法参数指定协议名称，如 id<NSCopying>，像
Java 的语法泛型。

浅拷贝是只拷贝了引用，而深拷贝将赋值所有的引用对象。

# 十一、Interface Builder
IBOutlet 和 IBAction 不执行任何操作，不是编译用的，而是为 IB 及阅读代码的人提供标记。

"nib"是 NeXT Interface Builder 的缩写，是 Cocoa 的一个工件。它包含被冷冻的对象的二进制文件，而.xib 是 XML 格式的 nib 文件。在编译时，.xib 文件将编译为 nib 格式。

拖动连接的方向是从某对象到该对象需要了解的对象。

如显示就是从 Controller 到 IBOutlet；而事件回调就是从按钮到 IBAction。

# 十二、文件的加载与保存

## 属性列表
常简写 plist。这些列表包含 Cocoa 知道如何操作的对象，即如何将它们保存到文件中并进行加载。属性列表包括 NSArray、NSDictionary、NSString、NSNumber、NSDate 和 NSData，以及它们的变体。

它们都有 -writeToFile:atomically:方法，将数据写入文件；

然后用 +****WithContentsOfFile:方法来从文件读取内容。

## NSDate
使用[NSDate date]可以获得当前日期。

NSData 类包装了大量字节。可以获得数据的长度和指向字节起始位置的指针。
因为 NSData 是一个对象，适用于常规的内存管理行为。因此，如果将数据块传递给一个函
数或方法，可以通过传递一个自动释放的 NSData 来实现，无需担心内存清除问题。
NSData 创建后就不可以改变，但是 NSMutableData 可以改变。

## 编码对象
对于对象，并不是列表那样，需要有结构体现。因此我们可以编码对象。

采用 NSCoding 协议实现。
实现-(void) encodeWithCoder : (NSCoder *) aCoder 和-(id) initWithCoder : (NSCoder *) aCoder
NSCoder 是一个抽象类，定义一些有用的方法在对象与 NSData 间进行转换。

在使用时，我们可以使用其已经实现的两个子类 NSKeyedArchiver 和 NSKeyedUnarchiver 来
实现编解码。

# 十三、键/值编码
KVC 是一种间接更改对象的方式，其实现方法是使用字符串描述要更改的对象状态部分。它是 Cocoa 提供的一种特性，而不是 Objective-C 的特性。

通过调用-valueForKey:和-setValue:forKey:以字符串的形式向对象发送消息。

-valueForKey:在 Objective-C 运行时中使用元数据打开对象并进入其中查看信息。在 C 或 C++中不能执行这种操作。通过使用 KVC，可以获取不存在 getter 方法的对象值，无需通过对象指针直接访问实例变量。

对于 KVC，Cocoa 自动放入和取出标量值。如果在调用-setValue:forKey:之前设置一个标量值，需要把它用 NSNumber 包装起来。

## 路径
圆点分隔不同属性名称。

如获得 engine 的 horsepower:

* 可以[engine valueForKey: @"horsepower"]
* 也可以[engine valueForKeyPaht: @"engine.horsepower"]

## 整体操作

-valueForKeyPath:将路径分解并从左向右进行处理。但不能在路径中为这些数组添加索引。

## 运算
这有些像 SQL 语言的内置的集合运算。
如在路径中@"cars.@count"表示得到车的量数。
@符号意味着后面将进行一些运算。

如"@distinctUnionOfObjects"的过程：

1. 获取左侧指定的集合。
2. 对该集合中的每个对象使用右侧的路径。
3. 将结果按找@命令转化为一个集合。

## 批处理
* -dictionaryWithValuesForKeys:它接受一个字符串数组，对每一个键使用 valueForKey:，然后为键字符串和刚才获得的值构建一个字典。
* -setValuesForKeysWithDictionary:设置相应的值。

## 处理未定义的键
可以通过重写 valueForUndefinedKey:和 setValue:forUndefinedKey:方法来处理未定义的键。
这样就可以很灵活的使用了。

# 十四、KVO（Key-Value Observing）
就是一个观察者模式。

是用来让另一类来观察这个类的属性的变化，而不是让自己来观察自己的变化。因为自己肯定知道自己的变化呀。

# 十五、NSPredicate
Cocoa 提供了一个名为 NSPredicate 的类，用于指定过滤器的条件。可以创建 NSPredicate对象，通过该对象准确地描述所需的条件，对每个对象通过谓词进行筛选，判断它们是否与条件相匹配。

NSPredicate 可以看作一种间接操作方式。使用谓词对对象进行查询，而不必使用代码
进行显式的查询。通过交换谓词对象，可以使用通用代码对数据进行过滤，而不必对相关条
件进行硬编码。

## 创建谓词
1. 创建许多对象，并将它们组合起来。这需要大量的代码，如果正在构建通用用户接口来指定查询，采用这种方式比较简单。
2. 查询代码中的字符串。

使用 NSPredicate 类方法+predicateWithFormat：来实际创建谓词。如：

NSPredicate *predicate = [NSPredicate predicateWithFormat: @"name == 'BMW'"];

如果谓词字符串中的文本块未被引用，即单引号括起来，则该字符串被看做是键路径，如上例中 name；

如果被引用了，则认为它是文本字符串，如上例中'BMW'。

像 NSArray 有-filteredArrayUsingPredicate：方法，直接可以用 predict 对象生成新数组。

NSMutableArray 的-filterUsingPredicat：方法，可以改变自己，得到想要的数据。

## 格式说明符
* %@引用字符串。
* %K 指定路径。

也可以将变量明让如类似环境变量中，如$NAME。
这样就可以创建一个模版，然后用字典初始化。

## 运算符
C 语言里的那些都有，还可以组合。

## 数组运算符
* BETWEEN ｛1， 9｝
* IN 等在 SQL 中出现的运算符。

## SELF
对于简单的值，而不是对象，如一个数组，因为没有键明，就用 SELF 代替里面的值进行比较。

## 字符串运算
* BEGINSWITH:
* ENDSWITH:
* CONTAINS:
* LIKE:
* MATCHES:

# 十六、多线程
似乎在每种语言中多线程都会定义在高级技术中。iPhone 编程中多线程技术有：

* NSThread
* NSOperation
* GCD

前两者类似 Java 中的 Thread。

# 十七、GCD（Grand Central Dispatch）编程
函数块用^开头。有点儿匿名内部函数的意思。

函数名也是一个变量，它是被它的参数类型和返回值定义的。

定义变量需要返回值，在实现时就表明参数就 ok 了。

```
int multiplier = 7;
int (^myBlock)(int) = ^(int num) {
return num * multiplier;
};
```

若要在 block 之外修改变量，就需要在变量前加两个下划线，如__var

dispatch_async(dispatch_queue_t queue, dispatch_block_t block);

dispatch_once只执行一次，如在单件中初始化中，不会被多次执行。

# 十八、委托模式与观察者模式
Java 中自己实现的观察着，其实就是利用委托模式，通过一个 Observers 集合，来在回调时遍历通知。OC 的，直接用已经实现好的框架类 NSNotificationCenter 就 OK 了。

发送通知：

[[NSNotificationCenter defaultCenter] postNotificationName:@”running” object:self userInfo:nil];

注册观察者：

[[NSNotificationCenter defaultCenter] addObserver:observer selector:@selector(changed) name:@”running” object:nil];

委托是一对一的，而观察者是一对多的。

使用委托时，可以返回一个值；而通知则不能够，它是单向的，只通知给观察者们。

相信这两个模式已经能够解决大部分问题了。

# 十九、多语言
NSLocalizedString 宏。

```
#define NSLocalizedString(key, comment) \
[[NSBundle mainBundle] localizedStringForKey:(key) value:@”” table:nil]
```