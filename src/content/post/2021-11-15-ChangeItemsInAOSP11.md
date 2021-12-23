+++
title= "ChangeItemsInAOSP11"
date = "2021-11-15T09:01:10+08:00"
description = "ChangeItemsInAOSP11"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 目的
这里记录开启转码支持所需要更改的文件列表

`build/target/board/generic_x86_64/BoardConfig.mk`, 在文件的尾部添加关于ABI架构支持的列表:   

```
 # Native Bridge ABI List
 NB_ABI_LIST_32_BIT := armeabi-v7a armeabi
 NB_ABI_LIST_64_BIT := arm64-v8a
 
 TARGET_CPU_ABI_LIST_64_BIT ?= $(TARGET_CPU_ABI) $(TARGET_CPU_ABI2)
 TARGET_CPU_ABI_LIST_32_BIT ?= $(TARGET_2ND_CPU_ABI) $(TARGET_2ND_CPU_ABI2)
 TARGET_CPU_ABI_LIST := \
     $(TARGET_CPU_ABI_LIST_64_BIT) \
     $(TARGET_CPU_ABI_LIST_32_BIT) \
     $(NB_ABI_LIST_64_BIT) \
     $(NB_ABI_LIST_32_BIT)
 TARGET_CPU_ABI_LIST_32_BIT += $(NB_ABI_LIST_32_BIT)
 TARGET_CPU_ABI_LIST_64_BIT += $(NB_ABI_LIST_64_BIT)
```

`build/target/board/generic_x86_64/device.mk`, 文件结尾处添加关于`nativebridge`的编译(这里关于属性的配置似乎无法生效，所以后面会在libart.mk中配置属性):     

```
# Added houdini
$(call inherit-product-if-exists, vendor/google/chromeos-x86/target/houdini.mk)
$(call inherit-product-if-exists, vendor/google/chromeos-x86/target/native_bridge_arm_on_x86.mk)
PRODUCT_SYSTEM_DEFAULT_PROPERTIES += persist.sys.nativebridge=1

# Get native bridge settings
$(call inherit-product-if-exists,device/generic/common/nativebridge/nativebridge.mk)


# NativeBridge
PRODUCT_PACKAGES += libhoudini houdini
PRODUCT_PROPERTY_OVERRIDES += ro.dalvik.vm.isa.arm=x86 ro.enable.native.bridge.exec=1
PRODUCT_DEFAULT_PROPERTY_OVERRIDES += ro.dalvik.vm.isa.arm=x86 ro.enable.native.bridge.exec=1

PRODUCT_PACKAGES += houdini64
PRODUCT_PROPERTY_OVERRIDES += ro.dalvik.vm.isa.arm64=x86_64 ro.enable.native.bridge.exec64=1
PRODUCT_DEFAULT_PROPERTY_OVERRIDES += ro.dalvik.vm.isa.arm64=x86_64 ro.enable.native.bridge.exec64=1

PRODUCT_DEFAULT_PROPERTY_OVERRIDES += ro.dalvik.vm.native.bridge=libhoudini.so
```

`build/target/product/AndroidProducts.mk`中加入关于编译时`lunch`的选项:     

```
COMMON_LUNCH_CHOICES := \
.......
     sdk_phone_x86_64-userdebug \
```
`build/target/product/runtime_libart.mk`中注释掉`ro.dalvik.vm.native.bridge=0`的默认设置，加入关于`ro.dalvik.vm.isa.arm`及其他几个参数配置


```
#PRODUCT_SYSTEM_DEFAULT_PROPERTIES += \
#    ro.dalvik.vm.native.bridge=0

PRODUCT_SYSTEM_DEFAULT_PROPERTIES += \
     ro.dalvik.vm.isa.arm=x86 \
     ro.enable.native.bridge.exec=1 \
     ro.dalvik.vm.isa.arm64=x86_64 \
     ro.enable.native.bridge.exec64=1 \
```

`build/target/product/sdk_phone_x86.mk`及`build/target/product/sdk_phone_x86_64.mk`中添加所需要拷贝的静态库的定义:     

```
#PRODUCT_ARTIFACT_PATH_REQUIREMENT_ALLOWED_LIST
# PRODUCT_ARTIFACT_PATH_REQUIREMENT_WHITELIST := \
#

PRODUCT_ARTIFACT_PATH_REQUIREMENT_ALLOWED_LIST := \
     system/bin/houdini \
     system/bin/houdini64 \
     system/etc/binfmt_misc/arm64_dyn \
.......
```

`device/generic/x86_64/BoardConfig.mk `中添加以下规则:    

```
PRC_COMPATIBILITY_PACKAGE := true
BUILD_ARM_FOR_X86 := true
-include vendor/google/chromeos-x86/board/native_bridge_arm_on_x86.mk
```

`device/generic/x86_64/mini_x86_64.mk`中添加关于`nativebridge`的编译:    

```
$(call inherit-product-if-exists,device/generic/common/nativebridge/nativebridge.mk)
```
`frameworks/base/core/java/com/android/internal/content/NativeLibraryHelper.java`区别：  

```
7,88d86
<         final String pkgName;
<         final String apkDir;
99,110d96
<         public static String getApkDirFromCodePath(String codePath) {
<             if (codePath == null ||
<                 codePath.startsWith("/system/") ||
<                 codePath.startsWith("/system_ext/") ||
<                 codePath.startsWith("/product/") ||
<                 codePath.startsWith("/vendor/") ||
<                 codePath.startsWith("/oem/")) {
<                 return null;
<             }
<             return codePath;
<         }
< 
113c99
<                     lite.debuggable, lite.packageName, getApkDirFromCodePath(lite.codePath));
---
>                     lite.debuggable);
117c103
<                 boolean extractNativeLibs, boolean debuggable, String pkgName, String apkdir) throws IOException {
---
>                 boolean extractNativeLibs, boolean debuggable) throws IOException {
134c120
<             return new Handle(apkPaths, apkHandles, multiArch, extractNativeLibs, debuggable, pkgName, apkdir);
---
>             return new Handle(apkPaths, apkHandles, multiArch, extractNativeLibs, debuggable);
146c132
<                     lite.extractNativeLibs, lite.debuggable, lite.packageName, getApkDirFromCodePath(lite.codePath));
---
>                     lite.extractNativeLibs, lite.debuggable);
150c136
<                 boolean extractNativeLibs, boolean debuggable, String pkgName, String apkdir) {
---
>                 boolean extractNativeLibs, boolean debuggable) {
156,157d141
<             this.pkgName = pkgName;
<             this.apkDir = apkdir;
232,234c216
<             final int res = nativeFindSupportedAbiReplace(apkHandle, supportedAbis,
<                     handle.debuggable, handle.pkgName, handle.apkDir);
< 
---
>             final int res = nativeFindSupportedAbi(apkHandle, supportedAbis, handle.debuggable);
256,257c238,239
<     private native static int nativeFindSupportedAbiReplace(long handle, String[] supportedAbis,
<             boolean debuggable, String pkgName, String apkdir);
---
>     private native static int nativeFindSupportedAbi(long handle, String[] supportedAbis,
>             boolean debuggable);
```

`frameworks/base/core/jni/Android.bp`中添加编译规则:     

```
15d14
<         "-D_PRC_COMPATIBILITY_PACKAGE_",
71d69
<                 "abipicker/ABIPicker.cpp",
```
`frameworks/base/core/jni/com_android_internal_content_NativeLibraryHelper.cpp`中添加:    

```
42,45d41
< #ifdef _PRC_COMPATIBILITY_PACKAGE_
< #include "abipicker/ABIPicker.h"
< #endif
< 
60,64d55
< #ifdef _PRC_COMPATIBILITY_PACKAGE_
< #define X86ABI     "x86"
< #define X8664ABI   "x86_64"
< #endif
< 
517,524c508,509
< com_android_internal_content_NativeLibraryHelper_findSupportedAbi_replace(
<         JNIEnv *env,
<         jclass clazz,
<         jlong apkHandle,
<         jobjectArray javaCpuAbisToSearch,
<         jboolean debuggable,
<         jstring apkPkgName,
<         jstring apkDir)
---
> com_android_internal_content_NativeLibraryHelper_findSupportedAbi(JNIEnv *env, jclass clazz,
>         jlong apkHandle, jobjectArray javaCpuAbisToSearch, jboolean debuggable)
526,612c511
< #ifdef _PRC_COMPATIBILITY_PACKAGE_
< 
<     int abiType = findSupportedAbi(env, apkHandle, javaCpuAbisToSearch, debuggable);
<     if (apkDir == NULL) {
<         return (jint)abiType;
<     }
< 
<     char abiFlag[256] = {'\0'};
<     ScopedUtfChars apkdir(env, apkDir);
<     size_t apkdir_size = apkdir.size();
<     const int numAbis = env->GetArrayLength(javaCpuAbisToSearch);
<     Vector<ScopedUtfChars*> supportedAbis;
< 
<     assert(apkdir_size < 256 - 15);
<     if (strlcpy(abiFlag, apkdir.c_str(), 256) != apkdir.size()) {
<         return (jint)abiType;
<     }
< 
<     int abiIndex = 0;
<     abiFlag[apkdir_size] = '/';
<     abiFlag[apkdir_size + 1] = '.';
<     for (abiIndex = 0; abiIndex < numAbis; abiIndex++) {
<         ScopedUtfChars* abiName = new ScopedUtfChars(env,
<                  (jstring)env->GetObjectArrayElement(javaCpuAbisToSearch, abiIndex));
<         supportedAbis.push_back(abiName);
<         if (abiName == NULL || abiName->c_str() == NULL || abiName->size() <= 0) {
<             break;
<         }
<         if ((strlcpy(abiFlag + apkdir_size + 2, abiName->c_str(), 256 - apkdir_size - 2)
<                     == abiName->size()) && (access(abiFlag, F_OK) == 0)) {
<             abiType = abiIndex;
<             break;
<         }
<     }
< 
<     if (abiIndex < numAbis) {
<         for (int j = 0; j < abiIndex; ++j) {
<             if (supportedAbis[j] != NULL) {
<                 delete supportedAbis[j];
<             }
<         }
<         return (jint)abiType;
<     }
< 
<     do {
<         if (abiType < 0 || abiType >= numAbis) {
<             break;
<         }
< 
<         if (0 != strcmp(supportedAbis[abiType]->c_str(), X86ABI) &&
<                 0 != strcmp(supportedAbis[abiType]->c_str(), X8664ABI)) {
<             break;
<         }
< 
<         ScopedUtfChars name(env, apkPkgName);
<         if (NULL == name.c_str()) {
<             break;
<         }
< 
<         if (isInOEMWhiteList(name.c_str())) {
<             break;
<         }
< 
<         ABIPicker picker(name.c_str(),supportedAbis);
<         if (!picker.buildNativeLibList((void*)apkHandle)) {
<             break;
<         }
< 
<         abiType = picker.pickupRightABI(abiType);
<         if (abiType >= 0 && abiType < numAbis &&
<                 (strlcpy(abiFlag + apkdir_size + 2, supportedAbis[abiType]->c_str(),
<                          256 - apkdir_size - 2) == supportedAbis[abiType]->size())) {
<             int flagFp = creat(abiFlag, 0644);
<             if (flagFp != -1) {
<                 close(flagFp);
<             }
<         }
< 
<     } while(0);
< 
<     for (int i = 0; i < numAbis; ++i) {
<         delete supportedAbis[i];
<     }
<     return (jint)abiType;
< #else
<     return (jint)findSupportedAbi(env, apkHandle, javaCpuAbisToSearch, debuggable);
< #endif
---
>     return (jint) findSupportedAbi(env, apkHandle, javaCpuAbisToSearch, debuggable);
703,705c602,604
<     {"nativeFindSupportedAbiReplace",
<             "(J[Ljava/lang/String;ZLjava/lang/String;Ljava/lang/String;)I",
<             (void *)com_android_internal_content_NativeLibraryHelper_findSupportedAbi_replace},
---
>     {"nativeFindSupportedAbi",
>             "(J[Ljava/lang/String;Z)I",
>             (void *)com_android_internal_content_NativeLibraryHelper_findSupportedAbi},

```

`frameworks/base/services/core/java/com/android/server/pm/parsing/pkg/AndroidPackageUtils.java`添加:     

```
139,141c139
<                 pkg.isDebuggable(),
<                 pkg.getPackageName(),
<                 NativeLibraryHelper.Handle.getApkDirFromCodePath(pkg.getCodePath())
---
>                 pkg.isDebuggable()
```

`system/linkerconfig/contents/namespace/systemdefault.cc`添加关于空间权限：    

```
build/system/linkerconfig/contents/namespace/systemdefault.cc origin/system/linkerconfig/contents/namespace/systemdefault.cc
67,70d66
<         "/system/lib/arm",
<         "/system/lib/arm/nb",
<         "/system/lib64/arm64",
<         "/system/lib64/arm64/nb",
```
