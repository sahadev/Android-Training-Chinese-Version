原文地址：http://android.xsoftlab.net/training/articles/perf-jni.html

##JavaVM and JNIEnv
JNI定义了两个关键的数据结构："JavaVM"与"JNIEnv".


##Threads

##jclass, jmethodID, and jfieldID
如果需要在本地代码中访问对象的属性，那么需要执行以下操作：

- 通过FindClass获取类对象的引用
- 通过GetFieldID获得属性的ID
- 通过适当的方法获取对象的内容，比如GetIntField

##本地引用，全局引用


##UTF-8 and UTF-16字符串