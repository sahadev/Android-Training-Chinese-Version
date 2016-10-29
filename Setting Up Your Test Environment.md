原文地址：[http://android.xsoftlab.net/training/activity-testing/index.html](http://android.xsoftlab.net/training/activity-testing/index.html)

#引言
开发者应当将测试作为应用开发周期的一部分。良好的测试用例可以帮助开发者及早的发现Bug，同时也可以增强开发者对代码的信心。

测试用例定义了一系列的对象与方法使多个测试可以独自进行。测试用例可以组合运行，也可以重复进行。

这节课的课程将会介绍如何使用Android的自定义测试框架，该框架基于很受欢迎的JUnit框架。开发者可以编写测试用例来验证程序的某些功能，也可以检查不同设备的兼容性。

#配置测试环境
在开始进行测试之前，开发者首先应当配置测试环境。这节课将会学习如何通过命令行来配置基于Gradle的测试用例。

##配置Eclipse
> **Note:** 因为目前使用Eclipse开发所占的比例已经很少了，所以接下来的翻译凡涉及到Eclipse的，文字描述都极为精简。需要了解的请[查看原文](http://android.xsoftlab.net/training/activity-testing/preparing-activity-testing.html#eclipse)。

##配置命令行
如果开发者使用的是Gradle 1.6或以上的版本，那么可以使用Gradle Wrapper来构建运行测试用例。要确保在gradle.build文件中，defaultConfig下的minSdkVersion属性设置的是8以上的值(含)。

要运行基于Gradle Wrapper的测试，需要执行以下步骤：
1.连接物理设备到计算机上。
2.在工程目录下运行以下命令：
 ``` ./gradlew build connectedCheck```

学习更多关于使用Gradle进行Android测试的相关内容，请参见[Gradle Plugin User Guide](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Testing).