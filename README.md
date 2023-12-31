# iOS-Demo

![star](https://badgen.net/github/stars/butub1/iOS-Demo) ![fork](https://badgen.net/github/forks/butub1/iOS-Demo) ![issues](https://badgen.net/github/issues/butub1/iOS-Demo) ![commits](https://img.shields.io/github/commits-since/butub1/iOS-Demo/v0.1.svg)

A collection of iOS demos.

## 1. 使用方式 // Usage

1. 引入头文件 DMMacros.h, DM 是 Demo 的缩写.
2. 注入一个 item, identifier 是必须的，class 也是必须的, 在这里是 DMSampleViewController.
3. 愉快地 Coding.

```objc
#import "DMSampleViewController.h"

// 1. import Macros
#import "DMMacros.h" 

// 2. register an item corresponding to your view controller
dm_registerDemo(DMSampleViewController, {
    item.identifier = @"Sample.UIButton"; // identifier required
})

// 3. enjoy coding with this view controller :)
@interface DMSampleViewController ()
@end

@implementation DMSampleViewController
- (void)viewDidLoad
{
    [super viewDidLoad];
    self.view.backgroundColor = [UIColor greenColor];
}
@end
```

<img src="https://github.com/butub1/iOS-Demo/blob/main/images/demo.gif" width="200">

## 2. 想法 // Ideas

日常写了一些 demo，但是重复劳动很多，并且无法很好收集整理起来，供后续查找，更无法分享给别人了。所以希望有个 demo 列表， 像下边这个样子。

```textile
====================
Main VC
=============
demo list
---
demo 1 --> subVC
---
demo 2 --> subVC
---
demo 3 --> subVC
---
====================
```

然而有了这个想法之后，越来越多的想法开始出现。

1. 要方便书写，避免重复劳动，避免如下情况:
   
   ```objc
   #import "SceneDelegate.h"
   #import "DMBaseViewController.h"
   #import "BDMSyntaxController.h"
   #import "VerticalSlidingViewController.h"
   #import "DMHitTestViewController.h"
   #import "DMCollectionViewDragViewController.h"
   #import "DMLayoutViewController.h"
   #import "DMPassThroughViewController.h"
   #import "DMSampleViewController.h"
   ```

2. 要有说明文字，要有 icon、author、hyperlink

3. 要易于检索，搞个简单的搜索的VC， 方便跳转

4. 要解耦，或者容易修改，上述的信息，在注入后，要能容易拿取到。

## 3. 设计 // Design

因为不想分散多处写注入代码， 最好是能像注解一样，只做一个操作，就注入。

```objc
@registerItem("Sample.Button.Basic")
@interface DMSampleViewController
@end
```

但是在 Objc 中，确实没有找到比较方便的实现，但是又非常想要这种效果。考虑过几种实现:

1. VC 继承的方式进行注入 --> Objc 不能多继承, 限制了 Demo 的能力

2. 接口注入到全局 mountArray  --> 添加操作需要运行时完成，否则需要静态注入。
   
   ```objc
   /// in mount file
   static NSArray<id<DMInjectProtocol>> *mountArray = @[
       [DMSampleViewController class],
   ]
   /// in DMSampleViewController.m
   @interface DMSampleViewController()<DMInjectProtocol>
   @end
   ```

3. 因此不得不找些运行时注入的方式，或者编译预处理的操作，考虑过
   
   1. LazyRegister， 注入函数指针到 data 段，会在 section 引入的时候注入，运行时机可选。
   2. load 方法， 运行时机在 main 函数之前，但是需要写 C 方法。
   3. Category 运行时注入，生成特定函数，但是 Demo 多了之后，会生成很多小Category. 【Selected】
   
   因为是 demo 项目，没有必要过度设计，所以选择了简单的方式实现，遇到问题再优化。

## 4.实现 // Implementation

dm_registerDemo 宏 最终生成”一条分类“，一条 : ) 合乎文法 : )

```objc
@interface DMRegisterCenter( DMSampleViewController ) - (void)user_registerItemsFor_DMSampleViewController;@end; @implementation DMRegisterCenter( DMSampleViewController ) - (void)user_registerItemsFor_DMSampleViewController{ [self registerClass:@"DMSampleViewController" withBlock:^(DMRegisterItem *item) { { item.identifier = @"Sample.UIButton";} }]; }@end
```

在 RegisterCenter 中会去遍历自己的实例方法，找到用户注入的方法(@user_ 开头)

```objc
- (void)loadAllItemsFromCategories
{
    unsigned int count;
    Method *methods = class_copyMethodList([self class], &count);
    for (int i = 0; i < count; i++) {
        SEL methodSelector = method_getName(methods[i]);
        NSString *methodName = [NSString stringWithUTF8String:sel_getName(methodSelector)];
        if ([methodName hasPrefix:@"user_"]) {
            [self performSelector:methodSelector];
        }
    }
    free(methods);
}
```

虽然用 user_ 打头这个方式，看起来很trick, 但是很方便。

目前的这种方式，对于VC没有任何的约束，您可以去做任何的事情，我们来负责检索，提供方便的工具。

## 5.约定大于配置 // convention over configuration

约定:

1. dm_registerDemo 的第一个参数是需要注入的 ViewController 类名
2. dm_registerDemo 中使用 item 对象进行配置
3. identifier 的命名，使用`.`进行分隔，例如 "Sample.Button.Basic", 将用于检索，一般不超过三段。
4. 目录放置根据 identifier 的分段分目录放置

## 6. TODO

1. 完善初始界面
2. 添加检索功能
3. 添加基础UI组件：用户信息，代码块，说明文字。

## 7.MileStone

* 2022-10-5: complete injection
