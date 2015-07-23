title: "Android代码混淆（Proguard）"
date: 2015-07-13 11:42:52
tags:
- android
- proguard
categories: android
toc: true
---

Android采用Java语言，编译成class文件，会很容易被反编译为java源代码。为了代码不被反编译，往往采用代码混淆。Android自带Proguard可以完成这项工作，同时删除没有使用的字段属性等，优化代码。

<!--more-->
**Title: [Android代码混淆（Proguard）](https://aidaizyy.github.io/android_proguard)**
**Author: [Yunyao Zhang（张云尧）](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2015-07-13](http://aidaizyy.github.io)**

##概要

Proguard并没有改变程序结构，只是通过修改名称，调整顺序等措施将代码变得难以阅读，难以理解，但却可以运行。
在Android项目的主目录里自带proguard-project.txt文件，代码混淆的规则就写在里面。

##配置

主目录的project.properties文件需要加上下面这句话，以告诉项目需要运行Proguard：

_proguard.config=${sdk.dir}/tools/proguard/proguard-android.txt:proguard-project.txt_ 

创建好的Android项目一般自带这条语句，不过开头用#注释了，去掉#即可。

##规则
_摘抄自http://blog.csdn.net/banketree/article/details/41928175_

``` bash
-include {filename}    从给定的文件中读取配置参数 
-basedirectory {directoryname}    指定基础目录为以后相对的档案名称 
-injars {class_path}    指定要处理的应用程序jar,war,ear和目录 
-outjars {class_path}    指定处理完后要输出的jar,war,ear和目录的名称 
-libraryjars {classpath}    指定要处理的应用程序jar,war,ear和目录所需要的程序库文件 
-dontskipnonpubliclibraryclasses    指定不去忽略非公共的库类。 
-dontskipnonpubliclibraryclassmembers    指定不去忽略包可见的库类的成员。

保留选项 
-keep {Modifier} {class_specification}    保护指定的类文件和类的成员 
-keepclassmembers {modifier} {class_specification}    保护指定类的成员，如果此类受到保护他们会保护的更好
-keepclasseswithmembers {class_specification}    保护指定的类和类的成员，但条件是所有指定的类和类成员是要存在。 
-keepnames {class_specification}    保护指定的类和类的成员的名称（如果他们不会压缩步骤中删除） 
-keepclassmembernames {class_specification}    保护指定的类的成员的名称（如果他们不会压缩步骤中删除） 
-keepclasseswithmembernames {class_specification}    保护指定的类和类的成员的名称，如果所有指定的类成员出席（在压缩步骤之后） 
-printseeds {filename}    列出类和类的成员-keep选项的清单，标准输出到给定的文件 

压缩 
-dontshrink    不压缩输入的类文件 
-printusage {filename} 
-whyareyoukeeping {class_specification}     

优化 
-dontoptimize    不优化输入的类文件 
-assumenosideeffects {class_specification}    优化时假设指定的方法，没有任何副作用 
-allowaccessmodification    优化时允许访问并修改有修饰符的类和类的成员 

混淆 
-dontobfuscate    不混淆输入的类文件 
-printmapping {filename} 
-applymapping {filename}    重用映射增加混淆 
-obfuscationdictionary {filename}    使用给定文件中的关键字作为要混淆方法的名称 
-overloadaggressively    混淆时应用侵入式重载 
-useuniqueclassmembernames    确定统一的混淆类的成员名称来增加混淆 
-flattenpackagehierarchy {package_name}    重新包装所有重命名的包并放在给定的单一包中 
-repackageclass {package_name}    重新包装所有重命名的类文件中放在给定的单一包中 
-dontusemixedcaseclassnames    混淆时不会产生形形色色的类名 
-keepattributes {attribute_name,...}    保护给定的可选属性，例如LineNumberTable, LocalVariableTable, SourceFile, Deprecated, Synthetic, Signature, and InnerClasses. 
-renamesourcefileattribute {string}    设置源文件中给定的字符串常量
```

##实例

project-proguard.txt中创建混淆规则。
缺省情况下会混淆所有代码，导致出错，必须保证不能被混淆的代码被保持。

``` java
-ignorewarnings				# 忽略警告，避免打包时某些警告出现
-optimizationpasses 5			# 指定代码的压缩级别
-dontusemixedcaseclassnames		# 是否使用大小写混合
-dontskipnonpubliclibraryclasses	# 是否混淆第三方jar
-dontpreverify                   	# 混淆时是否做预校验
-verbose                            	# 混淆时是否记录日志
-optimizations !code/simplification/arithmetic,!field/\*,!class/merging/\*	# 混淆时所采用的算法

-keepattributes \*Annotation\*
-keepattributes Signature

-libraryjars   libs/treecore.jar	# 保持第三方jar包不被混淆
-libraryjars   libs/android-viewbadger.jar
-libraryjars   libs/MapApi.jar
-libraryjars   libs/SinaWeiboSDK.jar

-dontwarn android.support.v4.**     
-dontwarn android.os.**

-keep class android.support.v4.** { *; } 		# 保持哪些类不被混淆
-keep class com.baidu.** { *; }  
-keep class vi.com.gdi.bgl.android.**{*;}
-keep class android.os.**{*;}

-keep interface android.support.v4.app.** { *; }  
-keep public class * extends android.support.v4.**  
-keep public class * extends android.app.Fragment
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.support.v4.widget
-keep public class * extends com.sqlcrypt.database
-keep public class * extends com.sqlcrypt.database.sqlite
-keep public class * extends com.treecore.**
-keep public class * extends de.greenrobot.dao.**


-keepclasseswithmembernames class * {	# 保持 native 方法不被混淆
    native <methods>;
}

-keepclasseswithmembers class * {	# 保持自定义控件类不被混淆
    public <init>(android.content.Context, android.util.AttributeSet);
}

-keepclasseswithmembers class * {	# 保持自定义控件类不被混淆
    public <init>(android.content.Context, android.util.AttributeSet, int);
}

-keepclassmembers class * extends android.app.Activity {	#保持类成员
   public void *(android.view.View);
}

-keepclassmembers enum * {	# 保持枚举 enum 类不被混淆
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

-keep class * implements android.os.Parcelable {	# 保持 Parcelable 不被混淆
  public static final android.os.Parcelable$Creator *;
}

-keep class MyClass;	# 保持自己定义的类不被混淆
```
##反编译

dex2jar：将apk转化为class文件
``` bash
https://github.com/pxb1988/dex2jar 
```
JD-GUI：将class文件转化为java文件
``` bash
http://jd.benow.ca/ 
```

通过这个两个工具可以将apk转化为java源文件。
通过Proguard生成的apk可转化为java源文件来进行比对以检测Proguard是否生效。
