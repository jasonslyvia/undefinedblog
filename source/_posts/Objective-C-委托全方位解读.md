---
title: Objective-C 委托全方位解读
permalink: objective-c-delegate-express
id: 4
updated: '2011-09-30 18:27:50'
date: 2011-09-30 18:27:50
tags:
---

<p>Delegate是运用Objective-C的运行时来实现的一种设计模式。<br />
官方文档的描述位置：<br />
iOS Library- &gt; General- &gt; Cocoa Fundamentals Guide- &gt; Cocoa Design Patterns- &gt; How Cocoa Adapts Design Patterns- &gt; Decorator- &gt; Delegation</p>
<p>你看文档给出的位置就可以知道它的大概作用了，它是装饰器的一种实现。如果你熟悉设计模式的话，可以知道装饰器是动态实现给一个对象添加一些额外职责的设计模式。如果不是很清楚的话，请google 装饰器 设计模式 或者 Design Patterns Decorator，了解更多。</p>
<p>那具体到Objective-C里，Delegate就是利用Objective-C 的运行时(Runtime)来实现动态的判断某对象是否存在，某对象是否实现了一些选择器(Selector)，如果实现了，则通过调用该对象的某一选择器来帮助自己实现一些功能。</p>
<p>在实际使用当中，设计一个自己的类Class的AClassDelegate可以通过两种方式实现，一种是非正式协议，Informal Protocol。</p>
<p>正式协议，Protocol。比较新的类的设计都会使用Protocol是定义自己的Delegate方法，并为之标记可选还是必需实现。<br />
当一个类会作为另一个类的Delegate的时候，需要实现该Delegate的Protocol，这意味着如果有标记@required的相应方法，那该类的实例必需实现该方法。@optional的方法就并不一定要实现。</p>
<p>举一个具体的例子：<br />
UINavigationController的Delegate应该是很少用到的，我看了一下也应该是比较简单的Delegate的设计。</p>
<p>[cc lang="c"]<br />
@protocol UINavigationControllerDelegate NSObject &gt;</p>
<p>@optional</p>
<p>// Called when the navigation controller shows a new top view controller via a push, pop or setting of the view controller stack.<br />
- (void)navigationController:(UINavigationController *)navigationController willShowViewController:(UIViewController *)viewController animated:(BOOL)animated;<br />
- (void)navigationController:(UINavigationController *)navigationController didShowViewController:(UIViewController *)viewController animated:(BOOL)animated;</p>
<p>@end<br />
[/cc]</p>
<p>这个Delegate的方法都是可选实现的，不是必须的，所以通常大家比较少去主动用。<br />
那分析一下这个Delegate，我们可以知道，这个Delegate的设计是UINavigationController在show某一个ViewController的时候会对它的Delegate进行一种类似通知的调用：“我即将展示某某ViewController了”，<br />
那该Delegate可以通过实现这个方法进行相应的操作。</p>
<p>举一个具体实现的例子<br />
[cc lang="c"]<br />
// AZDelegationDemo.h<br />
// Created by Aladdin Zhang on 8/24/11.</p>
<p>#import<br />
@protocol AZDelegationDemoDelegate;<br />
@interface AZDelegationDemo : NSObject{<br />
idAZDelegationDemoDelegate &gt; delegate;<br />
NSMutableArray * shoppingList;<br />
}<br />
@property (nonatomic,assign) idAZDelegationDemoDelegate &gt; delegate;<br />
@end</p>
<p>@interface AZDelegationDemo (informal_protocol_delegate)<br />
- (void)buyCoffee;<br />
@end</p>
<p>@protocol AZDelegationDemoDelegate NSObject &gt;<br />
@required<br />
- (void)buyPork;<br />
@optional<br />
- (void)buyMilk;<br />
@end</p>
<p>// AZDelegationDemo.m<br />
// Created by Aladdin Zhang on 8/24/11.</p>
<p>#import "AZDelegationDemo.h"</p>
<p>@implementation AZDelegationDemo<br />
@synthesize delegate;<br />
- (id)init{<br />
self = [super init];<br />
if (self) {<br />
shoppingList = [NSMutableArray arrayWithCapacity:20];<br />
}<br />
return self;<br />
}<br />
- (void)buyStuff{<br />
NSLog(@"My Shopping List %@",shoppingList);<br />
}<br />
- (void)weeklyShopping{<br />
if (self.delegate&amp;&amp;[self.delegate conformsToProtocol:@protocol(AZDelegationDemoDelegate)]) {<br />
[self.delegate performSelector:@selector(buyPork)];<br />
if ([self.delegate respondsToSelector:@selector(buyMilk)]) {<br />
[self.delegate performSelector:@selector(buyMilk)];<br />
}else{<br />
[shoppingList addObject:[NSString stringWithFormat:@"Milk"]];<br />
}<br />
if ([self.delegate respondsToSelector:@selector(buyCoffee)]) {<br />
[self.delegate performSelector:@selector(buyCoffee)];<br />
}else{<br />
[shoppingList addObject:[NSString stringWithFormat:@"Coffee"]];<br />
}<br />
[self buyStuff];<br />
}<br />
}<br />
@end<br />
[/cc]<br />
这是我刚刚为这个回答写的一个例子，基本上涵盖了一般用途的Delegate的设计，这里不对具体Delegate的实现进行说明了，和其他系统提供的Delegate的使用基本一致。<br />
在这个例子中，我分别使用了分类(Category)，即非正式协议，和协议(Protocol)来实现Delegate的设计。其实运用Objective-C的动态性，分类实现的协议和Protocol中的可选的协议方法，在调用上是一样的。使用正式协议，并标记为必需的好处是，可以通过编译器来帮助矫正不该出现的错误。</p>
<p>来自<a href="http://www.zhihu.com/question/19827157">知乎</a></p>
