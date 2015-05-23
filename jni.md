# JNI开发

 **粗体**或者*斜体*
 
-------------------


## Jni简介

> 参考google教程    —— [Google Jni Tips](http://developer.android.com/training/articles/perf-jni.html)

###简单的教程
####1.创建调用jni的java类
``` java
public class TestJni {
    public static native String getName();
    public static native byte[] mallocByte(int size);
    public void invokeJavaMethod(String msg) {
        Log.e("invokeJavaMethod", ""+msg);
    }
}
```

####2.生成.h文件
cd到src路径 执行 javah -jni com.example.louzhen.onlytest.TestJni会生成com_example_louzhen_onlytest_TestJni.h头文件
####3.编写.c文件
``` c
JNIEXPORT jstring JNICALL Java_com_example_louzhen_onlytest_TestJni_getName
  (JNIEnv *env, jclass clz)
{
    //c语言的char转成java的string
    return (*env)->NewStringUTF(env,"name is lz");
}

JNIEXPORT jbyteArray JNICALL Java_com_example_louzhen_onlytest_TestJni_mallocByte
  (JNIEnv *env, jclass clz, jint size)
{
    char *mallocSpace = (char *)malloc(size); //java的int和c的int通用, 申请size大小内存
    LOGD("mallocByte %d", size);
    memset(mallocSpace, 1, size); //给c中的内存全部附上初始值1
    jbyteArray returnArray = (*env)->NewByteArray(env,size); //申请java byte数组内存
    (*env)->GetByteArrayRegion(env, returnArray, size, size, mallocSpace); //将c申请的内存值赋予java byte数组
    return returnArray; //返回byte数组
}

JNIEXPORT void JNICALL Java_com_example_louzhen_onlytest_TestJni_invokeJavaMethod
  (JNIEnv *env, jclass clz, jstring msg)
{
        //把java string型转成char *型
    	const char* msgChars = (*env)->GetStringUTFChars(env, msg, NULL );
    	int len = (*env)->GetStringUTFLength(env, msg );

        //获取java类
        jclass objectClass=(*env)->FindClass(env,"com/example/louzhen/onlytest/TestJni");
        //创建类的实例
        jobject objectTestJni = (*env)->NewObject(env, objectClass, "<init>", "()V");
        //获取类中成员变量域ID
        jfieldID fieldIDB = (*env)->GetFieldID(env,objectClass,"paramB","Ljava/lang/String;");
        jfieldID fieldIDC = (*env)->GetFieldID(env,objectClass,"paramC","Ljava/lang/String;");
        //获取类中成员变量值
        jstring stringB = (*env)->GetObjectField(env,objectTestJni,fieldIDB);
        //java string 转成 char *
        char *dataB = (*env)->GetStringUTFChars(env, stringB, NULL);
        LOGD("dataB %s", dataB);
        //给类实例变量赋值
        (*env)->SetObjectField(env,objectTestJni,fieldIDC,(*env)->NewStringUTF(env,"InputC"));
        //获取java方法ID
        jmethodID jinvokeJavaId = (*env)->GetMethodID(env,objectClass,"invokeJavaMethodImpl","Ljava/lang/String;");
        //调用java方法
        (*env)->CallVoidMethod(env, objectTestJni, (*env)->NewStringUTF(env,msgChars));
        //释放msgChars内存
        (*env)->ReleaseStringUTFChars(env,msg,msgChars);
}
```
####4.写Android.mk和Application.mk文件
[Android.mk syntax](file://localhost/Applications/dev/android-ndk-r8e/docs/ANDROID-MK.html)
[Application.mk syntax](file://localhost/Applications/dev/android-ndk-r8e/docs/APPLICATION-MK.html)
####5.执行ndk-build
在jni目录下执行NDK路径/ndk-build, 会生成so文件在工程libs/armeabi下.

####6.最终java调用
```java
private void testJni() {
        String name = TestJni.getName();
        Log.d("","name = "+ name);
        byte[] spaces = TestJni.mallocByte(10000);
        Log.d("","spaces size = "+ spaces.length);
        TestJni.invokeJavaMethod("haha");
    }
    static {
        System.loadLibrary("libtestJni");
    }
```

###Jni相关文件的结构
![Alt text](./1428371795123.png)
jni里包括Android.mk, Application.mk, .c文件, .h文件
libs下面是编译好的so文件
obj下面是生成的目标文件


### Jni开发关注点
 1.java和c变量的关系
| JAVA      |    C | 是否需要转换  |
| :-------- | --------:| :--: |
| string  | char * |  需要   |
| byte     | char |  不需要  |
| byte[]  | char * | 需要  |
| int      |    int | 不需要  |
2.c中内存申请与释放
3.掌握jni.h方法的调用













---------


