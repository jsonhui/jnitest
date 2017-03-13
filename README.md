#一、关于NDK的一点解释:                                                                                           
NDK全称：Native Development Kit。
NDK提供了一系列的工具，帮助开发者快速开发C（或C++）的动态库，并能自动将so和java应用一起打包成apk。这些工具对开发者的帮助是巨大的。
NDK集成了交叉编译器，并提供了相应的mk文件隔离CPU、平台、ABI等差异，开发人员只需要简单修改mk文件（指出“哪些文件需要编译”、“编译特性要求”等），就可以创建出so.NDK可以自动地将so和Java应用一起打包，极大地减轻了开发人员的打包工作。
#二、NDK坏境搭建：                                                                                                   
  注意事项：目前已经出了NDK-r9b了，由于作者写这篇日志的时候 当时下载的还是NDK-r8c，所以仍以NDK-r8c为例来讲解。操作类似，从ndk-7后，谷歌已经改良了ndk的操作，不需用使用cygwin来交叉编译了，这大大的提高了我们的开发速度。
##（1）下载安装NDK-r8c。
下载地址：http://developer.android.com/sdk/ndk/index.html
##（2）打开Eclipse，新建一个Android工程（我的取名为TestNdk），在工程目录TestNdk下新建jni文件夹，该文件夹就用来保存NDK需要编译的文件代码等。
##（3）开始新建并配置一个Builder
###（a）Project->Properties->Builders->New，新建一个Builder。
###（b）在弹出的【Choose configuration type】对话框，选择【Program】，点击【OK】：
###（c）在弹出的【Edit Configuration】对话框中，配置选项卡【Main】。在“Name“中输入新builders的名称（这个名字可以任意取）。在“Location”中输入nkd-build.cmd的路径(这个是下载完ndk8后解压后的路径，这个建议放在根目录下面，路径不能有空格和中文)。根据各自的ndk路径设置，也可以点击“BrowserFile System…”来选取这个路径。在“Working Diretcoty”中输入TestNdk位置（也可以点击“Browse Workspace”来选取TestNdk目录）。如图1
    ![image](https://github.com/jsonhui/jnitest/blob/master/res/drawable-hdpi/one.png)图1
###（d）继续在这个【Edit Configuration】对话框中，配置选项卡【Refresh】。如图2
    勾选“Refresh resources upon completion”，
    勾选“The entire workspace”，
    勾选“Recuresively include sub-folders”
    ![image](https://github.com/jsonhui/jnitest/blob/master/res/drawable-hdpi/two.png)图2
    ###（e）继续在【Edit Configuration】对话框中，配置选项卡【Build options】。
    勾选“After a “Clean””，(勾选这个操作后，如果你想编译ndk的时候，只需要clean一下项目 就开始交叉编译)
    勾选“During manual builds”，
    勾选“During auto builds”，
    勾选“Specify working set of relevant resources”。如图3
![image](https://github.com/jsonhui/jnitest/blob/master/res/drawable-hdpi/three.png)图3
点击“Specify Resources…”勾选TestNdk工程中新建的“jni“目录，点击”finish“。 点击“OK“，完成配置。 如图4
![image](https://github.com/jsonhui/jnitest/blob/master/res/drawable-hdpi/four.png)图4
到此，恭喜你，编译环境以及成功搭建完毕！
那么搭建完了，当然要玩一玩了，好了，下面我们来跑一个demo测试一把，让你了解一下ndk的开发流程
#三、NDK程序demo的开发（下面是重点，请仔细查看）                                                  
   ##1.在TestNdk工程中新建一个JniClient.class（为了调用C/C++代码），其内容如下：
   ···
package com.ndk.test;
public class JniClient {
    static public native String AddStr(String strA, String strB);
    static public native int AddInt(int a, int b);
}
···
##2.生成 .h 的c++头文件
###(1)用cmd命令定位到JniClient.class 所在目录，输入“javac JniClient.java“后回车，生成JniClinet.class文件（如果是用的Eclipse建的工程，在TestNdk\bin\classes\com\ndk\test目录下就已经有JniClinet.class文件了）。
###(2)将JniClinet.class拷贝到TestNdk\bin\classes\com\ndk\test目录，将cmd命令定位到TestNdk\bin\classes目录，输入”javah com.ndk.test.JniClient“后回车，在TestNdk\bin\classes目录下就生成了C++头文件com_ndk_test_JniClient.h
com_ndk_test_JniClient.h的文件内容如下：

/* DO NOT EDIT THIS FILE - it is machine generated */
#include <jni.h>
/* Header for class com_ndk_test_JniClient */

#ifndef _Included_com_ndk_test_JniClient
#define _Included_com_ndk_test_JniClient
#ifdef __cplusplus
extern "C" {
#endif
/*
 * Class:     com_ndk_test_JniClient
 * Method:    AddStr
 * Signature: (Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
 */
JNIEXPORT jstring JNICALL Java_com_ndk_test_JniClient_AddStr
  (JNIEnv *, jclass, jstring, jstring);
/*
 * Class:     com_ndk_test_JniClient
 * Method:    AddInt
 * Signature: (II)I
 */
JNIEXPORT jint JNICALL Java_com_ndk_test_JniClient_AddInt
  (JNIEnv *, jclass, jint, jint);
#ifdef __cplusplus
}
#endif
#endif
##3.在jni目录下新建一个Android.mk文件，其内容如下(关于mk文件需要注意,很重要,还有c和c++文件的mk文件还不一样，此处是调用c语言的mk文件，至于其他的怎么调用，这个自己去百度吧，在此就不多说了)
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := TestNdk
LOCAL_SRC_FILES := com_ndk_test_JniClient.c
include $(BUILD_SHARED_LIBRARY)
##4. 将刚刚手动生成的com_ndk_test_JniClient.h拷贝到TestNdk工程的jni目录下，
然后新建一个com_ndk_test_JniClient.c文件完成头文件中函数的实现，其内容如下（本来想写两个方法的，现在只讲解第一个方法，返回一个字符串“HelloWorld from JNI ”,另一个方法是一个a+b的运算，方法写到这里，感兴趣的可以自己去研究）：
com_ndk_test_JniClient.c
#include "com_ndk_test_JniClient.h"
#include <stdlib.h>
#include <stdio.h>
#ifdef __cplusplus   
extern "C"  
{   
#endif  
/*
 * Class:     com_ndk_test_JniClient
 * Method:    AddStr
 * Signature: (Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
 */
JNIEXPORT jstring JNICALL Java_com_ndk_test_JniClient_AddStr
  (JNIEnv *env, jclass arg, jstring instringA, jstring instringB)
{
    jstring str = (*env)->NewStringUTF(env, "HelloWorld from JNI !");
    return str;       
}
/*
* Class:     com_ndk_test_JniClient
* Method:    AddInt
* Signature: (II)I
*/
JNIEXPORT jint JNICALL Java_com_ndk_test_JniClient_AddInt
  (JNIEnv *env, jclass arg, jint a, jint b)
{
    return a + b;
}
#ifdef __cplusplus   
}   
#endif
此刻,当编辑com_ndk_test_JniClient.c并保存后，project下的—clean  一下工程，就可以看到TestNkd工程下的obj/local/armeabi目录下将自动生成libTestNdk.so库。
##5.在TestNdkActivity.java中完成对JniClient.java中函数的调用(首先静态加载动态链接so库)：
package com.ndk.test;
import android.app.Activity;
import android.os.Bundle;
import android.widget.TextView;
public class TestNdkActivity extends Activity {
    static {
        System.loadLibrary("TestNdk");
    }
    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
//        setContentView(R.layout.main);
        
        //第一个方法传入的两个参数没有做操作，直接返回hello jni，不用管
        String str = JniClient.AddStr("test", "test");
        
        //第二个方法暂时不实现
        //  int iSum = JniClient.AddInt(5, 2);        
       // String strSum = "5 + 7 = " + iSum;
        
       TextView tv1 = new TextView(this);
        tv1.setText(str);
        setContentView(tv1);
    }
}

##6.运行TestNdk工程，在控制台中可以看到界面输出来自com_ndk_test_JniClient.c 文件中的“HelloWorld from JNI ! "了。
NDK实例到此完成。
后续更复杂的操作就需要深入的学习NDK/JNI了，
比如C/C++与Java的数据类型转换，以及Android.mk文件的编写格式等。
近期有朋友需求，忘记上传一份demo了，那么我重新整理了一份demo，各位朋友可以下载学习：
github下载地址：https://github.com/jsonhui/jnitest.git
demo图片如下：
![image](https://github.com/jsonhui/jnitest/blob/master/res/drawable-hdpi/six.png)
