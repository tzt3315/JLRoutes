JLRoutes
========

[![Platforms](https://img.shields.io/cocoapods/p/JLRoutes.svg?style=flat)](http://cocoapods.org/pods/JLRoutes)
[![CocoaPods Compatible](https://img.shields.io/cocoapods/v/JLRoutes.svg)](http://cocoapods.org/pods/JLRoutes)
[![Carthage Compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)
[![Build Status](https://travis-ci.org/joeldev/JLRoutes.svg?branch=master)](https://travis-ci.org/joeldev/JLRoutes)
[![Apps](https://img.shields.io/cocoapods/at/JLRoutes.svg?maxAge=2592000)](https://cocoapods.org/pods/JLRoutes)

### What is it? ###
JLRoutes is a URL routing library with a simple block-based API. It is designed to make it very easy to handle complex URL schemes in your application with minimal code.

JLRoutes是一个URL路由库，具有一个简单的基于block的API。它的设计目的是用最少的代码非常容易地处理应用程序中的复杂URL模式。

### Installation ###
JLRoutes is available for installation using [CocoaPods](https://cocoapods.org/pods/JLRoutes) or Carthage (add `github "joeldev/JLRoutes"` to your `Cartfile`).

JLRoutes可以使用CocoaPods或Carthage进行安装(将github“joeldev/JLRoutes”添加到您的Cartfile中)。

### Requirements ###
JLRoutes 2.x require iOS 8.0+ or macOS 10.10+. If you need to support iOS 7 or macOS 10.9, please use version 1.6.4 (which is the last 1.x release).

JLRoutes 2。x要求iOS 8.0+或macOS 10.10+。如果您需要支持iOS 7或macOS 10.9，请使用版本1.6.4(即最后一个版本1)。x版本)。

### Documentation ###
Documentation is available [here](http://cocoadocs.org/docsets/JLRoutes/).

文档在这里(http://cocoadocs.org/docsets/JLRoutes/)。

### Getting Started ###

[Configure your URL schemes in Info.plist.](https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Inter-AppCommunication/Inter-AppCommunication.html#//apple_ref/doc/uid/TP40007072-CH6-SW2)

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  JLRoutes *routes = [JLRoutes globalRoutes];

  [routes addRoute:@"/user/view/:userID" handler:^BOOL(NSDictionary *parameters) {
    NSString *userID = parameters[@"userID"]; // defined in the route by specifying ":userID"

    // present UI for viewing user with ID 'userID'

    return YES; // return YES to say we have handled the route
  }];

  return YES;
}

- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString *, id> *)options
{
  return [JLRoutes routeURL:url];
}
```

Routes can also be registered with subscripting syntax:
```objc
JLRoutes.globalRoutes[@"/user/view/:userID"] = ^BOOL(NSDictionary *parameters) {
  // ...
};
```

After adding a route for `/user/view/:userID`, the following call will cause the handler block to be called with a dictionary containing `@"userID": @"joeldev"`:
```objc
NSURL *viewUserURL = [NSURL URLWithString:@"myapp://user/view/joeldev"];
[JLRoutes routeURL:viewUserURL];
```

### The Parameters Dictionary ###

The parameters dictionary always contains at least the following three keys:

参数字典总是至少包含以下三个键:
```json
{
  "JLRouteURL":  "(the NSURL that caused this block to be fired)",
  "JLRoutePattern": "(the actual route pattern string)",
  "JLRouteScheme": "(the route scheme, defaults to JLRoutesGlobalRoutesScheme)"
}
```

The JLRouteScheme key refers to the scheme that the matched route lives in. [Read more about schemes.](https://github.com/joeldev/JLRoutes#scheme-namespaces)

JLRouteScheme键指的是匹配路由所在的方案。(https://github.com/joeldev/JLRoutes# schema -namespaces)

See JLRoutes.h for the list of constants.

有关常量列表，请参见jlrouting .h。

### Handler Block Chaining ###

The handler block is expected to return a boolean for if it has handled the route or not. If the block returns `NO`, JLRoutes will behave as if that route is not a match and it will continue looking for a match. A route is considered to be a match if the pattern string matches **and** the block returns `YES`.

block将返回一个布尔值，用于是否处理了路由。如果块返回“NO”，JLRoutes将表现为路由不匹配，并且它将继续寻找匹配。如果模式字符串匹配**，并且**块返回“YES”，则认为路由是匹配的。

It is also important to note that if you pass nil for the handler block, an internal handler block will be created that simply returns `YES`.

同理，如果为block传递nil，那么将创建一个只返回“YES”的block。

### Global Configuration ###

There are multiple global configuration options available to help customize JLRoutes behavior for a particular use-case. All options only take affect for the next operation.

有多个全局配置选项可用来帮助定制特定用例的JLRoutes行为。所有选项只对下一个操作生效。

```objc
/// Configures verbose logging. Defaults to NO.
+ (void)setVerboseLoggingEnabled:(BOOL)loggingEnabled;

/// Configures if '+' should be replaced with spaces in parsed values. Defaults to YES.
+ (void)setShouldDecodePlusSymbols:(BOOL)shouldDecode;

/// Configures if URL host is always considered to be a path component. Defaults to NO.
+ (void)setAlwaysTreatsHostAsPathComponent:(BOOL)treatsHostAsPathComponent;

/// Configures the default class to use when creating route definitions. Defaults to JLRRouteDefinition.
+ (void)setDefaultRouteDefinitionClass:(Class)routeDefinitionClass;
```

These are all configured at the `JLRoutes` class level:

这是JLRoutes所有的类级别的配置
```objc
[JLRoutes setAlwaysTreatsHostAsPathComponent:YES];
```

### More Complex Example ###

```objc
[[JLRoutes globalRoutes] addRoute:@"/:object/:action/:primaryKey" handler:^BOOL(NSDictionary *parameters) {
  NSString *object = parameters[@"object"];
  NSString *action = parameters[@"action"];
  NSString *primaryKey = parameters[@"primaryKey"];
  // stuff
  return YES;
}];
```

This route would match things like `/user/view/joeldev` or `/post/edit/123`. Let's say you called `/post/edit/123` with some URL params as well:

这条路由将匹配“/user/view/joeldev”或“/post/edit/123”之类的内容。假设你用一些URL参数调用了' /post/edit/123 ':

```objc
NSURL *editPost = [NSURL URLWithString:@"myapp://post/edit/123?debug=true&foo=bar"];
[JLRoutes routeURL:editPost];
```

The parameters dictionary that the handler block receives would contain the following key/value pairs:

block接收的参数字典将包含以下键/值对:
```json
{
  "object": "post",
  "action": "edit",
  "primaryKey": "123",
  "debug": "true",
  "foo": "bar",
  "JLRouteURL": "myapp://post/edit/123?debug=true&foo=bar",
  "JLRoutePattern": "/:object/:action/:primaryKey",
  "JLRouteScheme": "JLRoutesGlobalRoutesScheme"
}
```

### Schemes ###

JLRoutes supports setting up routes within a specific URL scheme. Routes that are set up within a scheme can only be matched by URLs that use a matching URL scheme. By default, all routes go into the global scheme.

JLRoutes支持在特定的URL方案中设置路由。在方案中设置的路由只能由使用匹配URL方案的URL匹配。默认情况下，所有路由都进入全局方案。

```objc
[[JLRoutes globalRoutes] addRoute:@"/foo" handler:^BOOL(NSDictionary *parameters) {
  // This block is called if the scheme is not 'thing' or 'stuff' (see below)
  return YES;
}];

[[JLRoutes routesForScheme:@"thing"] addRoute:@"/foo" handler:^BOOL(NSDictionary *parameters) {
  // This block is called for thing://foo
  return YES;
}];

[[JLRoutes routesForScheme:@"stuff"] addRoute:@"/foo" handler:^BOOL(NSDictionary *parameters) {
  // This block is called for stuff://foo
  return YES;
}];
```

This example shows that you can declare the same routes in different schemes and handle them with different callbacks on a per-scheme basis.

这个例子表明，您可以在不同的方案中声明相同的路由，并在每个方案的基础上使用不同的回调来处理它们。

Continuing with this example, if you were to add the following route:

继续这个例子，如果你要添加以下路径:
```objc
[[JLRoutes globalRoutes] addRoute:@"/global" handler:^BOOL(NSDictionary *parameters) {
  return YES;
}];
```

and then try to route the URL `thing://global`, it would not match because that route has not been declared within the `thing` scheme but has instead been declared within the global scheme (which we'll assume is how the developer wants it). However, you can easily change this behavior by setting the following property to `YES`:

然后尝试路由URL ' thing://global '，它将不匹配，因为该路由没有在' thing ' scheme中声明，而是在全局scheme中声明(我们将假设这是开发人员想要的)。不过，你可以很容易地改变这一行为，设置以下属性为“YES”:

```objc
[JLRoutes routesForScheme:@"thing"].shouldFallbackToGlobalRoutes = YES;
```

This tells JLRoutes that if a URL cannot be routed within the `thing` scheme (aka, it starts with `thing:` but no appropriate route can be found), try to recover by looking for a matching route in the global routes scheme as well. After setting that property to `YES`, the URL `thing://global` would be routed to the `/global` handler block.

这告诉JLRoutes，如果URL不能在“thing”方案中路由(也就是说，它以“thing:”开头，但是找不到合适的路由)，那么也要在全局路由方案中寻找匹配的路由来进行恢复。将该属性设置为' YES '后，URL ' thing://global '将被路由到' /global '处理程序块。


### Wildcards ###

JLRoutes supports setting up routes that will match an arbitrary number of path components at the end of the routed URL. An array containing the additional path components will be added to the parameters dictionary with the key `JLRouteWildcardComponentsKey`.

JLRoutes支持设置路由，这些路由将在路由URL的末尾匹配任意数量的路径组件。包含额外路径组件的数组将被添加到参数字典中，关键字为“JLRouteWildcardComponentsKey”。

For example, the following route would be triggered for any URL that started with `/wildcard/`, but would be rejected by the handler if the next component wasn't `joker`.

例如，对于任何以' /通配符/ '开头的URL，都将触发以下路由，但是如果下一个组件不是' joker '，则处理程序将拒绝该URL。

```objc
[[JLRoutes globalRoutes] addRoute:@"/wildcard/*" handler:^BOOL(NSDictionary *parameters) {
  NSArray *pathComponents = parameters[JLRouteWildcardComponentsKey];
  if (pathComponents.count > 0 && [pathComponents[0] isEqualToString:@"joker"]) {
    // the route matched; do stuff
    return YES;
  }

  // not interested unless 'joker' is in it
  return NO;
}];
```

### Optional Routes ###

JLRoutes supports setting up routes with optional parameters. At the route registration moment, JLRoute will register multiple routes with all combinations of the route with the optional parameters and without the optional parameters. For example, for the route `/the(/foo/:a)(/bar/:b)`, it will register the following routes:

JLRoutes支持使用可选参数设置路由。在路由注册时刻，JLRoute将注册多条路由，路由的所有组合都带有可选参数，没有可选参数。例如，对于路由' /the(/foo/:a)(/bar/:b) '，它将注册以下路由:

- `/the/foo/:a/bar/:b`
- `/the/foo/:a`
- `/the/bar/:b`
- `/the`

### Querying Routes ###

There are multiple ways to query routes for programmatic uses (such as powering a debug UI). There's a method to get the full set of routes across all schemes and another to get just the specific list of routes for a given scheme. One note, you'll have to import `JLRRouteDefinition.h` as it is forward-declared.

有多种方法可以查询编程使用的路由(例如提供调试UI的能力)。有一种方法可以获得所有方案的完整路由集，还有一种方法可以只获得给定方案的特定路由列表。需要注意的是，您必须导入“JLRRouteDefinition.h" 是向前声明的。

```objc
/// All registered routes, keyed by scheme
+ (NSDictionary <NSString *, NSArray <JLRRouteDefinition *> *> *)allRoutes;

/// Return all registered routes in the receiving scheme namespace.
- (NSArray <JLRRouteDefinition *> *)routes;
```

### Handler Block Helper ###

`JLRRouteHandler` is a helper class for creating handler blocks intended to be passed to an addRoute: call.

“JLRRouteHandler”是一个帮助类，用于创建打算传递给addRoute: call的block。

This is specifically useful for cases in which you want a separate object or class to be the handler for a deeplink route. An example might be a view controller that you want to instantiate and present in response to a deeplink being opened.

这对于希望单独的对象或类作为deeplink路由的处理程序的情况特别有用。一个例子可能是一个视图控制器，您希望实例化它并在打开deeplink时呈现它。

In order to take advantage of this helper, your target class must conform to the `JLRRouteHandlerTarget` protocol. For example:

为了利用这个帮助，您的目标类必须遵循“JLRRouteHandlerTarget”协议。例如:

```objc
@interface MyTargetViewController : UIViewController <JLRRouteHandlerTarget>

@property (nonatomic, copy) NSDictionary <NSString *, id> *parameters;

@end


@implementation MyTargetViewController

- (instancetype)initWithRouteParameters:(NSDictionary <NSString *, id> *)parameters
{
  self = [super init];

  _parameters = [parameters copy]; // hold on to do something with later on

  return self;
}

- (void)viewDidLoad
{
  [super viewDidLoad];
  // do something interesting with self.parameters, initialize views, etc...
}

@end
```

To hook this up via `JLRRouteHandler`, you could do something like this:

要通过“JLRRouteHandler”将其连接起来，可以这样做:

```objc
id handlerBlock = [JLRRouteHandler handlerBlockForTargetClass:[MyTargetViewController class] completion:^BOOL (MyTargetViewController *viewController) {
  // Push the created view controller onto the nav controller
  [self.navigationController pushViewController:viewController animated:YES];
  return YES;
}];

[[JLRoutes globalRoutes] addRoute:@"/some/route" handler:handlerBlock];
```

There's also a `JLRRouteHandler` convenience method for easily routing to an existing instance of an object vs creating a new instance. For example:

还有一个“JLRRouteHandler”便利方法，可以轻松路由到对象的现有实例和创建新实例。例如:

```objc
MyTargetViewController *rootController = ...; // some object that exists and conforms to JLRRouteHandlerTarget.
id handlerBlock = [JLRRouteHandler handlerBlockForWeakTarget:rootController];

[[JLRoutes globalRoutes] addRoute:@"/some/route" handler:handlerBlock];
```

When the route is matched, it will call a method on the target object:

当路由匹配时，它会在目标对象上调用一个方法:

```objc
- (BOOL)handleRouteWithParameters:(NSDictionary<NSString *, id> *)parameters;
```

These two mechanisms (weak target and class target) provide a few other ways to organize deep link handlers without writing boilerplate code for each handler or otherwise having to solve that for each app that integrates JLRoutes.

这两种机制(弱目标和类目标)提供了一些其他方法来组织深度链接处理程序，而无需为每个处理程序编写样板代码，或者为每个集成了JLRoutes的应用程序解决这个问题。

### Custom Route Parsing ###

It is possible to control how routes are parsed by subclassing `JLRRouteDefinition` and using the `addRoute:` method to add instances of your custom subclass.

可以通过子类化“JLRRouteDefinition”并使用“addRoute:”方法添加自定义子类的实例来控制路由的解析方式。

```objc
// Custom route defintion that always matches
@interface AlwaysMatchRouteDefinition : JLRRouteDefinition
@end


@implementation AlwaysMatchRouteDefinition

- (JLRRouteResponse *)routeResponseForRequest:(JLRRouteRequest *)request
{
  // This method is called when JLRoutes is trying to determine if we are a match for the given request object.

  // Create the parameters dictionary
  NSDictionary *variables = [self routeVariablesForRequest:request];
  NSDictionary *matchParams = [self matchParametersForRequest:request routeVariables:variables];

  // Return a valid match!
  return [JLRRouteResponse validMatchResponseWithParameters:matchParams];
}

@end
```

This route can now be created an added:

这条路径现在可以创建一个添加:
```objc
id handlerBlock = ... // assume exists
AlwaysMatchRouteDefinition *alwaysMatch = [[AlwaysMatchRouteDefinition alloc] initWithPattern:@"/foo" priority:0 handlerBlock:handlerBlock];
[[JLRoutes globalRoutes] addRoute:alwaysMatch];
```

Alternatively, if you've written a custom route definition and want JLRoutes to always use it when adding a route (using one of the `addRoute:` methods that takes in raw parameters), use `+setDefaultRouteDefinitionClass:` to configure it as the routing definition class:

或者，如果您编写了一个自定义路由定义，并且希望JLRoutes在添加路由时总是使用它(使用接受原始参数的' addRoute: '方法之一)，使用' +setDefaultRouteDefinitionClass: '将其配置为路由定义类:
```objc
[JLRoutes setDefaultRouteDefinitionClass:[MyCustomRouteDefinition class]];
```

### License ###
BSD 3-clause. See the [LICENSE](LICENSE) file for details.

