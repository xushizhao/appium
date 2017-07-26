## Migrating your iOS tests from UIAutomation (iOS 9.3 and below) to XCUITest (iOS 9.3 and up)
## 将您的iOS测试从UIAutomation（iOS 9.3及更低版本）迁移到XCUITest（iOS 9.3及更高版本）

For iOS automation, Appium relies on system frameworks provided by Apple. For iOS 9.2 and below, Apple's only automation technology was called UIAutomation, and it ran in the context of a process called "Instruments". As of iOS 10, Apple has completely removed the UIAutomation instrument, thus making it impossible for Appium to allow testing in the way it used to. Fortunately, Apple introduced a new automation technology, called XCUITest, beginning with iOS 9.3. For iOS 10 and up, this will be the only supported automation framework from Apple.

对于iOS自动化，Appium依赖于苹果提供的系统框架。对于iOS 9.2及更低版本，苹果唯一的自动化技术被称为UIAutomation，在“Instruments”的运行。截止iOS 10，苹果已经完全删除了UIAutomation工具，因此使得Appium不可能按照以前的方式进行测试。同时，苹果推出了一款名为XCUITest的自动化技术，支持iOS 9.3、iOS 10及以上版本，这将是苹果唯一支持的自动化框架。

Appium has built in support for XCUITest beginning with Appium 1.6. For the most part, the capabilities of XCUITest match those of UIAutomation, and so the Appium team was able to ensure that test behavior will stay the same. This is one of the great things about using Appium! Even with Apple completely changing the technology your tests are using, your scripts can stay mostly the same! That being said, there are some differences you'll need to be aware of which might require modification of your test scripts if you want to run them under our XCUITest automation backend. This document will help you with those differences.

Appium从Appium 1.6开始支持XCUITest。在大多数情况下，XCUITest的功能与UIAutomation的功能相匹配，因此，Appium团队能够确保测试行为保持不变。这是使用Appium的好东西之一！即使苹果完全改变了您的测试使用的技术，您的脚本也可以保持大致相同！如果您想在XCUITest自动化后端运行它们,还是有一些差异，您需要注意可能需要修改的测试脚本。本文将帮助您解决这些差异。

### Element class name schema

### 元素类名称模式

With XCUITest, Apple has given different class names to the UI elements which make up the view hierarchy. For example, `UIAButton` is now `XCUIElementTypeButton`. In many cases, there is a direct mapping between these two classes. If you use the `class name` locator strategy to find elements, Appium 1.6 will rewrite the selector for you. Likewise, if you use the `xpath` locator strategy, Appium 1.6 will find any `UIA*` elements in your XPath string and rewrite them appropriately.

使用XCUITest，苹果已经为构成视图层次结构的UI元素提供了不同的类名。例如，UIAButton现在XCUIElementTypeButton。在很多情况下，这两个类之间有直接映射。如果您使用class name定位器策略来查找元素，Appium 1.6将为您重写选择器。同样，如果您使用xpath定位器策略，Appium 1.6将在您的`XPath`字符串中找到任何`UIA *`元素，并适当地重写它们。

This does not however guarantee that your tests will work exactly the same, for two reasons:

但是，这并不能保证您的测试工作完全相同，原因有两个：

1. The application hierarchy reported to Appium will not necessarily be identical within XCUITest to what it was within UIAutomation. If you have a path-based XPath selector, it may need to be adjusted.

1.向Appium报告的应用程序层次结构在XCUITest中不一定与UIAutomation内的相同。如果您有基于路径的XPath选择器，则可能需要进行调整。

2. The list of class names is not entirely identical either. Many elements are returned by XCUITest as belonging to the `XCUIElementTypeOther` class, a sort of catch-all container.

2.类名列表也不完全一样。许多元素由XCUITest返回，属于`XCUIElementTypeOther`类，这是一种全能的容器。

### Page source

### 页面来源

As mentioned just above, if you rely on the app source XML from the `page source` command, the XML output will now differ significantly from what it was under UIAutomation.

如上所述，如果您依赖于`page source`命令中的应用程序源XML，则XML输出现在将与UIAutomation之间的差异显着。

### `-ios uiautomation` locator strategy

### `-ios uiautomation` 定位策略

This locator strategy was specifically built on UIAutomation, so it is not included in the XCUITest automation backend. We will be working on a similar "native"-type locator strategy in coming releases.

此定位器策略专门用于UIAutomation，因此它不包括在XCUITest自动化后端中。 在即将发布的版本中，我们将致力于类似的“本地”型定位策略。

### `xpath` locator strategy

### `xpath` 定位策略

1. Try not to use XPath locators unless there is absolutely no other alternatives. In general, xpath locators might be times slower, than other types of locators like accessibility id, class name and predicate (up to 100 times slower in some special cases). They are so slow, because xpath location is not natively supported by Apple's XCTest framework.

1.尽量不要使用XPath定位器，除非绝对没有其他选择。通常，xpath定位器可能比其他类型的定位器慢，比如可访问性id，类名和谓词（在某些特殊情况下可减缓100倍）。它们太慢了，因为xpath的位置不是苹果的XCTest框架本身所支持的。

2. Use

2.使用

```
driver.findElement(x)
```

call instead of

调用而不是

```
driver.findElements(x)[0]
```

to lookup single element by xpath. The more possible UI elements are matched by your locator the slower it is.

通过xpath查找单个元素。 您的定位器匹配的UI元素越多越慢。

3. Be very specific when locating elements by xpath. Such locators like

3.通过xpath定位元素时非常具体。 这样的定位器就像

```
//*
```

may take minutes to complete depending on how many UI elements your application has (e. g.

可能需要几分钟才能完成，具体取决于您的应用程序有多少用户界面元素（例如，

```
driver.findElement(By.xpath("//XCUIElementTypeButton[@value='blabla']"))
```

is faster than

比

```
driver.findElement(By.xpath("//*[@value='blabla']"))
```

or

或

```
driver.findElement(By.xpath("//XCUIElementTypeButton")))
```
快

4. In most cases it would be faster to perform multiple nested findElement calls than to perform a single call by xpath (e.g.

在大多数情况下，执行多个嵌套的findElement调用比执行xpath单个调用（例如，

```
driver.findElement(x).findElement(y)
```

is usually faster than

通常比

```
driver.findElement(z)

```
快

where x and y are non-xpath locators and z is a xpath locator).

其中x和y是非xpath定位符，z是xpath定位符）。

### System dependencies

### 系统依赖

In addition to the many gotchas that might come with upgrading any XCode installation (unrelated to Appium), Appium's XCUITest support requires a new system dependency: [Carthage](https://github.com/Carthage/Carthage). Appium Doctor has now been updated to ensure that the `carthage` binary is on your path.

除了可升级任何XCode安装（与Appium无关）的许多问题之外，Appium的XCUITest支持需要新的系统依赖性： [Carthage](https://github.com/Carthage/Carthage)。Appium Doctor现已更新，以确保carthage二进制文件位于您的路径上。


### API differences

### API差异

Unfortunately, the XCUITest API and the UIAutomation API are not equivalent. In many cases (like with `tap/click`), the behavior is identical. But some features that were available in the UIAutomation backend are not yet available in the new XCUITest backend. These known lacking features include:


不幸的是，XCUITest API和UIAutomation API不是等效的。在许多情况下（如同样tap/click），行为是相同的。但在UIAutomation后端可用的某些功能在新的XCUITest后端尚不可用。这些已知缺乏的功能包括：

* Geolocation support (e.g., `driver.location`)

* 地理位置支持（例如，driver.location）

* Shaking the device

* 摇动装置

* Locking the device

* 锁定设备

* Rotating the device (note that this is *NOT* device _orientation_, which is supported)

* 旋转设备（请注意，*NOT* device _orientation_，受支持）

We will endeavor to add these features back in future releases of Appium.

我们将努力在将来的Appium版本中添加这些功能。

#### Scrolling and clicking

#### 滚动并点击

In the previous UIAutomation-based driver, if you tried to click on an element that wasn't in view, UIAutomation would scroll to the element automatically and then tap it. With XCUITest, this is no longer the case. You are now responsible for ensuring your element is in view before interacting with it (the same way a user would be responsible for the same).

在以前基于UIAutomation的驱动程序中，如果您尝试单击不在视图中的元素，UIAutomation将自动滚动到该元素，然后点击它。使用XCUITest，不再是这样。您现在负责在与之交互之前确保您的元素正在查看（与用户对此相同的方式相同）。

### Other known issues

### 其他已知问题

Finally, a list of known issues with the initial 1.6 release (we'll strike through issues which have been resolved):

最后，列出了初始1.6版本的已知问题（我们将会解决已解决的问题）：

* ~~Unable to interact with elements on devices in Landscape mode (https://github.com/appium/appium/issues/6994)~~

* 无法以横向模式与设备上的元素进行交互（https://github.com/appium/appium/issues/6994）

* `shake` is not implemented due to lack of support from Apple

* `shake` 由于缺乏苹果的支持，没有实施

* `lock` is not implemented due to lack of support from Apple

* `lock` 由于缺乏苹果的支持，没有实施

* Setting geo-location not supported due to lack of support from Apple

* 由于缺乏苹果的支持，设置地理位置不受支持

* Through the TouchAction/MultiAction API, `zoom` gestures work but `pinch` gestures do not, due to an Apple issue.

* 通过TouchAction / MultiAction API，`zoom`手势可以工作，但`pinch`苹果问题手势不被支持。

* ~~Through the TouchAction/MultiAction API, `swipe` gestures are currently not supported, though they should be soon (https://github.com/appium/appium/issues/7573)~~

* 通过TouchAction / MultiAction API，swipe手势目前不受支持，（https://github.com/appium/appium/issues/7573）

* The capabilities `autoAcceptAlerts` and `autoDismissAlerts` do not currently work, and there is continued debate about whether we will be able to implement them in the future.

* `autoAcceptAlerts`，`autoDismissAlerts`目前还没有使用，而且我们是否能够在将来实施这些，还存在争议。

* There is an issue with the iOS SDK such that PickerWheels built using certain API methods are not automatable by XCUITest. See https://github.com/appium/appium/issues/6962 for the workaround, to ensure your PickerWheels are built properly.

* iOS SDK有一个问题，因此使用某些API方法构建的PickerWheels不能由XCUITest自动执行。有关解决方法，请参阅https://github.com/appium/appium/issues/6962，以确保您的PickerWheels正确构建。

As far as possible, we will add the missing features and fix other known issues in future versions of Appium.

我们将尽可能添加缺失的功能，并在以后的Appium版本中修复其他已知问题。
