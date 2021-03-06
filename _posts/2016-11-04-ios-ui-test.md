---
layout: post
title: IOS自动化测试实现
date: 2016-11-4
categories: blog
tags: [IOS]
description: IOS自动化测试实现
---

ios平台下自动化测试工具概览       

![](http://tmq.qq.com/wp-content/uploads/2016/06/QQ%E6%88%AA%E5%9B%BE20160624145248.png)

表中没有列的内容是稳定性，实践中看来上述除UITesting之外的工具稳定性都相对一般，会出现自身框架导致的各种闪退，以及性能越来越差的问题。
因此iOS平台上除了Monkey测试采用了自动化方式，以及部分性能测试轻度使用了一些自动化工具，大部分功能测试还是依赖于人的操作。

**UITestIng实例**         

```
import Foundation
import XCTest

class UITestDemo_UI_Tests: XCTestCase {
        
    override func setUp() {
        super.setUp()
        
        // Put setup code here. This method is called before the invocation of each test method in the class.
        
        // In UI tests it is usually best to stop immediately when a failure occurs.
        continueAfterFailure = false
        // UI tests must launch the application that they test. Doing this in setup will make sure it happens for each test method.
        XCUIApplication().launch()
    }
    
    override func tearDown() {
        // Put teardown code here. This method is called after the invocation of each test method in the class.
        super.tearDown()
    }
    
    func testExample() {
        // Use recording to get started writing UI tests.
        // Use XCTAssert and related functions to verify your tests produce the correct results.
        
        testLoginView()
    }
    
    func testLoginView() {
        let app = XCUIApplication()
        
        // 由于UITextField的id有问题，所以只能通过label的方式遍历元素来读取
        let nameField = app.textFields["name"]
        
        if self.canOperateElement(nameField) {
            nameField.tap()
            nameField.typeText("xiaoming")
        }
        
        let psdField = app.secureTextFields["password"]
       
        if self.canOperateElement(psdField) {
            psdField.tap()
            psdField.typeText("1234321")
        }
        
        // 通过UIButton的预设id来读取对应的按钮
        let loginBtn = app.buttons["Login"]
        if self.canOperateElement(loginBtn) {
            loginBtn.tap()
        }
        
        // 开始一段延时，由于真实的登录是联网请求，所以不能直接获得结果，demo通过延时的方式来模拟联网请求
        let window = app.windows.elementBoundByIndex(0)
        if self.canOperateElement(window) {
            // 延时3秒, 3秒后如果登录成功，则自动进入信息页面，如果登录失败，则弹出警告窗
            window.pressForDuration(3)
        }
        
        // alert的id和labe都用不了，估计还是bug，所以只能通过数量判断
        if app.alerts.count > 0 {
            // 登录失败
            app.alerts.collectionViews.buttons["确定"].tap()
            
            let clear = app.buttons["Clear"]
            if self.canOperateElement(clear) {
                clear.tap()
                
                if self.canOperateElement(nameField) {
                    nameField.tap()
                    nameField.typeText("sun")
                }
                
                if self.canOperateElement(psdField) {
                    psdField.tap()
                    psdField.typeText("111111")
                }
                
                if self.canOperateElement(loginBtn) {
                    loginBtn.tap()
                }
                if self.canOperateElement(window) {
                    // 延时3秒, 3秒后如果登录成功，则自动进入信息页面，如果登录失败，则弹出警告窗
                    window.pressForDuration(3)
                }
                self.loginSuccess()
            }
        } else {
            // 登录成功
            self.loginSuccess()
        }
    }
    
    func loginSuccess() {
        let app = XCUIApplication()
        let window = app.windows.elementAtIndex(0)
        if self.canOperateElement(window) {
            // 延时1秒, push view需要时间
            window.pressForDuration(1)
        }
        self.testInfo()
    }
    
    func testInfo() {
        let app = XCUIApplication()
        let window = app.windows.elementAtIndex(0)
        if self.canOperateElement(window) {
            // 延时2秒, 加载数据需要时间
            window.pressForDuration(2)
        }
        
        let modifyBtn = app.buttons["modify"];
        modifyBtn.tap()
        
        let sexSwitch = app.switches["sex"]
        sexSwitch.tap()
        
        let incrementButton = app.buttons["Increment"]
        incrementButton.tap()
        incrementButton.tap()
        incrementButton.tap()
        app.buttons["Decrement"].tap()
        
        let textView = app.textViews["feeling"]
        textView.tap()
        app.keys["Delete"].tap()
        app.keys["Delete"].tap()
        textView.typeText(" abc ")
        
        // 点击空白区域
        let clearBtn = app.buttons["clearBtn"]
        clearBtn.tap()
        
        // 保存数据
        modifyBtn.tap()
        window.pressForDuration(2)
        
        let messageBtn = app.buttons["message"]
        messageBtn.tap();
        
        // 延时1秒, push view需要时间
        window.pressForDuration(1)
        
        self.testMessage()
    }
    
    func testMessage() {
        let app = XCUIApplication()
        let window = app.windows.elementAtIndex(0)
        if self.canOperateElement(window) {
            // 延时2秒, 加载数据需要时间
            window.pressForDuration(2)
        }
        
        let table = app.tables
        table.childrenMatchingType(.Cell).elementAtIndex(8).tap()
        table.childrenMatchingType(.Cell).elementAtIndex(1).tap()
        
    }
    
    func getFieldWithLbl(label:String) -> XCUIElement? {
        var _:XCUIElement? = nil
        return self.getElementWithLbl(label, type: XCUIElementType.TextField)
    }
    
    func getElementWithLbl(label:String, type:XCUIElementType) -> XCUIElement? {
        let app = XCUIApplication()
        let query = app.descendantsMatchingType(type)
        var result:XCUIElement? = nil
        let num=query.count
        print(num)
        print("****************\n")
        print(label)
        print("***********************\n")
        for i in 0..<num {
            let element:XCUIElement = query.elementBoundByIndex(i)
            let subLabel:String? = element.label;
            if subLabel != nil {
                if subLabel == label {
                    result = element
                }
            }
        }
        return result
    }
    
    func canOperateElement(element:XCUIElement?) -> Bool {
        if element != nil {
            if element!.exists {
                return true
            }
        }
        return false
    }
}
```

**参考**       
[iOS 9 学习系列: UI Testing](http://blog.csdn.net/fish_yan_/article/details/50747861)       
[Xcode7 UI自动化测试详解 带demo UITests](http://www.cocoachina.com/ios/20150925/13566.html)

UITesting存在的问题：只适用于ios9.0以上的机器，以下的机器不能运行        
同时存在不同的模拟器，比如iphon6,iphon6s等，有的可以正常运行，有的却不能等情况，不知道是模拟器的bug，还是程序的bug


#### ios UI Automation      

UI Automation是黑盒测试，基于javascript语言。      

**实现要点**          

- 标签上要加上accessibilty为true      
- 要选中user interaction enabled      
- 可以通过setting-general-accessiibilty中的工具探测id

```
var target = UIATarget.localTarget();
var inputField = target.frontMostApp().mainWindow().textFields()["theinput"];
inputField.setValue("hifdssdf");
if (inputField.value() != "hi") UIALogger.logFail("The Input Field was NOT able to be set with the string!");
else UIALogger.logPass("The Input Field was able to be set with the string!");

target.frontMostApp().mainWindow().buttons()["Jumblify Button"].tap();

target.delay(2);
inputField.setValue("hi");
target.delay(10);
target.frontMostApp().mainWindow().buttons()["Jumblify Button"].tap();
```

```
var target = UIATarget.localTarget();

var inputField = target.frontMostApp().mainWindow().textFields()["theinput"];
inputField.setValue("hi");
if (inputField.value() != "hi") UIALogger.logFail("The Input Field was NOT able to be set with the string!");
else UIALogger.logPass("The Input Field was able to be set with the string!");
var button = target.frontMostApp().mainWindow().buttons()["Jumblify Button"];
button.tap();
target.frontMostApp().mainWindow().logElementTree();
var stringResult = target.frontMostApp().mainWindow().staticTexts()["ih"];
if (! stringResult.isValid()) UIALogger.logFail("The output text was NOT set with the correctly reversed string!");
else UIALogger.logPass("The output text was set with the correctly reversed string!");
```

**使用命令行运行**         

如果你想让你的测试代码自动的运行起来，你还可以通过命令行来启动测试。其实，我比较推荐这种方式，而不是使用Instruments的图形界面程序。因为，Instruments的图形界面程序比较慢，而且即使你的测试代码跑完了它也还是会一直运行着。而通过命令行来启动和运行测试代码更快，它会在跑完测试后自动的停止。

为了可以在命令行终端运行你的脚本，你需要知道你设备的UDID和类型：

```
instruments -w your_ios_udid -t 
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/Instruments/PlugIns/AutomationInstrument.bundle/Contents/Resources/Automation.tracetemplate 
name_of_your_app -e UIASCRIPT absolute_path_to_the_test_file
例如，使用我自己的机子，就这么写的：

instruments -w a2de620d4fc33e91f1f2f8a8cb0841d2xxxxxxxx -t 
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/Instruments/PlugIns/AutomationInstrument.bundle/Contents/Resources/Automation.tracetemplate 
TestAutomation -e UIASCRIPT 
/Users/jc/Documents/Dev/TestAutomation/TestAutomation/TestUI/Test-2.js
```

一个小提示，不要忘了关闭你设备的密码验证，否则你会看到这样的日志信息的：remote exception encountered : ’device locked : Failed to launch process with bundle identifier ’com.manbolo.testautomation’. 的确，因为UIAutomation根本不知道你的密码啊。

命令行终端同样可以在模拟器上使用，但你需要知道待测应用程序在文件系统中的绝对路径。模拟器将目录~/Library/Application Support/iPhone Simulator/5.1/ “模拟”成了设备的文件系统。在这个目录下，你可以找到一个包含装在模拟器上的所有应用程序的沙盒的Applications文件夹。定位到TestAutomation程序的目录，然后：

```
instruments -t /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/Instruments/PlugIns/AutomationInstrument.bundle/Contents/Resources/Automation.tracetemplate "/Users/jc/Library/Application Support/iPhone Simulator/5.1/Applications/C28DDC1B-810E-43BD-A0E7-C16A680D8E15/TestAutomation.app" -e UIASCRIPT /Users/jc/Documents/Dev/TestAutomation/TestAutomation/TestUI/Test-2.js
```

最后，如果你没有指定日志输入到哪里的话，你的测试结果会被放到你命令行当前指定（工作）的目录下。你可以通过加入 -e UIARESULTSPATH results_path 参数来指定日志输入目录。

我没有成功的将多个测试脚本并行着在命令行中运行起来。但是你可以将你的测试脚本串连进来，有一整晚去跑它，这样就真正的实现了“在你睡着的时候”，就完成了对应用程序的测试。

**XCODE7模拟器与模拟器中app的路径**       
 在Xcode 7中, 模拟器的位置改变为：                          
/Users/username/Library/Developer/CoreSimulator/Devices       
在此目录下，有许多文件夹这就是模拟器啦：    

2. 在Terminal中使用如下命令：        
xcrun simctl list    
可以知道uuid与版本的对应关系      

比如，iOS 9.3下，iPhone 6s：         
iPhone 6s (60B8F826-8241-498A-A180-35C3F4F59562) (Booted)

因此，Application目录在：          
/Users/username/Library/Developer/CoreSimulator/Devices/D2A94C2D-3216-4737-A502-5B64B38F6124/data/Containers/Data/Application/

**参见：**[ Xcode 7中模拟器的位置](http://blog.csdn.net/yaoliangjun306/article/details/51481809)

**参考链接**           
[如何使用UIAutomation进行iOS 自动化测试（Part II）](http://www.cnblogs.com/vowei/archive/2012/08/17/2644158.html)       
[Introduction to iOS Testing With UI Automation](https://code.tutsplus.com/tutorials/introduction-to-ios-testing-with-ui-automation--cms-22730)        
[如何使用UIAutomation进行iOS 自动化测试](https://my.oschina.net/u/1049180/blog/404681)







