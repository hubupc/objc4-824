# iOS Runtime 学习 2：objc4-824 源代码编译、调试

## 概述
调试 Runtime 源码对于探索 Runtime 的底层原理是必不可少的环节。比起单纯地阅读源码，调试的过程能让我们直接跟踪 Runtime 的执行流程。另外，配合 LLDB 还能输出断点位置当前内部数据结构存储的数据。对于理解和验证底层原理都是极好的方式。

Runtime 是开源的，在苹果的开源网站上可以下载到源码工程。但是源码工程并不能直接运行，需要补齐缺少的头文件，修改配置，屏蔽一些代码才能运行起来，配置的过程比较繁琐。下面我将结合网上搜集到的资料，自己也重新配置一遍，并记录下流程。大家可以直接使用配置好的[工程](https://github.com/hubupc/objc4-824.git)，有兴趣的朋友也可以自己配置一遍。

## 源码配置
1. 下载地址  
   https://opensource.apple.com/tarballs/objc4/

2. Runtime   
- 当前最新版本为 objc4-824，但点开后发现找不到资源：
    ```
    error message 
    Please report the missing source or broken link to opensource@apple.com
    ```
   已通过上述邮箱上报给苹果。
   
- 我在 github 上找到了 objc4-824的源码以及配置好的版本：https://github.com/RegretSF/objc824
  参考 “IT_胖哥” 的配置的记录：https://www.jianshu.com/p/f830d4cbc6b4  
  笔者也重新配置了一遍。

3. 依赖库
可在 https://opensource.apple.com/tarballs 中查找

- dyld-851.27
- Libc-825.24
- Libc-1439.100.3
- libclosure-79
- libplatform-254.80.2
- libpthread-454.100.8
- xnu-7195.101.1

## IDE
- Xcode 13.1

## 设置依赖库目录
1. 在 Runtime 工程根目录下创建文件夹 PrivateHeaders
2. 打开 Runtime 源码，点击左侧项目导航窗口中的工程文件 objc，在右侧设置窗口中，TARGETS 栏目下选中 objc，
   点击 Build Settings, 搜索search path，在 Search Paths 栏目下，打开 Header Search Paths，新增 $(SRCROOT)/PrivateHeaders

## 编译错误处理
注：实测发现：cmd+B 编译报错，在没有任何修改的情况下，再次 cmd+B 编译，可能会优先报另一个错，所以实际操作过程中，报错顺序可能有些差异，按实际顺序修改即可。  

1. 问题：error: unable to find sdk 'macosx.internal' (in target 'objc-trampolines' from project 'objc')  
   方案：PROJECT 栏目下选中 objc，点击 Build Settings，在 Architetures 栏目下，将 Base SDK 由 macosx.internal 改为 macOS。以相同的方式设置 objc-trampolines

2. 问题：fatal error: 'sys/reason.h' file not found  
   文件：xnu-7195.101.1/bsd/sys/reason.h  
   方案：将 reason.h 拷贝到 objc4-824/PrivateHeaders/sys/  

3. 问题：fatal error: 'mach-o/dyld_priv.h' file not found  
   文件：dyld-851.27/include/mach-o/dyld_priv.h  
   方案：将 dyld_priv.h 拷贝到 objc4-824/PrivateHeaders/mach-o/  

4. 问题：fatal error: 'os/lock_private.h' file not found  
   文件：libplatform-254.80.2/private/os/lock_private.h   
   方案：将 lock_private.h 拷贝到 objc4-824/PrivateHeaders/os/  

5. 问题：error: expected ','  
   extern dyld_platform_t dyld_get_active_platform(void) __API_AVAILABLE(macos(10.14), ios(12.0), watchos(5.0), tvos(12.0), bridgeos(3.0));  
   方案：删除 dyld_priv.h中的 , bridgeos(3.0)  

6. 问题：fatal error: 'os/base_private.h' file not found  
   文件：xnu-7195.101.1/libkern/os/base_private.h   
   方案：将 base_private.h 拷贝到 objc4-824/PrivateHeaders/os/  

7. 问题：error: expected ','  
   tvos(13.0), watchos(6.0), bridgeos(4.0)) = 0x00040000,  
   方案：删除 lock_private.h 中的 , bridgeos(4.0)

8. 问题：fatal error: 'pthread/tsd_private.h' file not found  
   文件：libpthread-454.100.8/private/pthread/tsd_private.h   
   方案：将 tsd_private.h 拷贝到 objc4-824/PrivateHeaders/pthread/  

9. 问题：fatal error: 'System/machine/cpu_capabilities.h' file not found  
   文件 xnu-7195.101.1/osfmk/machine/cpu_capabilities.h   
   方案：将 cpu_capabilities.h 拷贝到 objc4-824/PrivateHeaders/System/machine/  

10. 问题：fatal error: 'os/tsd.h' file not found  
    文件：xnu-7195.101.1/libsyscall/os/tsd.h   
    方案：将 tsd.h 拷贝到 objc4-824/PrivateHeaders/os/   

11. 问题：fatal error: 'pthread/spinlock_private.h' file not found  
    文件：libpthread-454.100.8/private/pthread/spinlock_private.h   
    方案：将 spinlock_private.h 拷贝到 objc4-824/PrivateHeaders/pthread/  

12. 问题：fatal error: 'System/pthread_machdep.h' file not found  
    文件：Libc-825.24/pthreads/pthread_machdep.h   
    方案：将 pthread_machdep.h 拷贝到 objc4-824/PrivateHeaders/System/  

13. 问题：error: typedef redefinition with different types ('int' vs 'volatile OSSpinLock' (aka 'volatile int'))  
    typedef long pthread_lock_t;   
    error: static declaration of '_pthread_has_direct_tsd' follows non-static declaration     
    error: static declaration of '_pthread_getspecific_direct' follows non-static declaration  
    error: static declaration of '_pthread_setspecific_direct' follows non-static declaration  
    方案：注释掉 pthread_machdep.h  Line 214-299  

14. 问题：fatal error: 'CrashReporterClient.h' file not found  
    文件：Libc-825.24/include/CrashReporterClient.h   
    方案：将 CrashReporterClient.h 拷贝到 objc4-824/PrivateHeaders  
         在TARGETS objc: Build Settings: Preprocessor Macros 下新增：LIBC_NO_LIBCRASHREPORTERCLIENT

15. 问题：fatal error: 'os/feature_private.h' file not found  
    方案： 注释：NSObject.mm 中 Line 42， objc-runtime.mm 中 Line 36, Line 444-446   

16. 问题：error: use of undeclared identifier 'dyld_fall_2020_os_versions'  
    方案：注释：objc-runtime.mm 中 Line 379-380  

17. 问题：error: use of undeclared identifier 'dyld_platform_version_macOS_10_13'  
    方案：注释：objc-os.mm Line 567-574  

18. 问题：fatal error: 'objc-shared-cache.h' file not found  
    文件：dyld-851.27/include/objc-shared-cache.h  
    方案：将 objc-shared-cache.h 拷贝到 objc4-824/PrivateHeaders/  

19. 问题：error: use of undeclared identifier 'dyld_platform_version_macOS_10_11'  
    方案：注释：objc-runtime-new.mm 中 Line 3561-3567  

20. 问题：error: use of undeclared identifier 'dyld_fall_2018_os_versions'  
    方案：注释：objc-runtime-new.mm 中 Line 8344-8366  

21. 问题：fatal error: '_simple.h' file not found  
    文件：libplatform-254.80.2/private/_simple.h  
    方案：将 _simple.h 拷贝到 objc4-824/PrivateHeaders/ 

22. 问题：fatal error: 'os/linker_set.h' file not found  
    文件：Libc-1439.100.3/os/linker_set.h  
    方案：将 linker_set.h 拷贝到 objc4-824/PrivateHeaders/os/  

23. 问题：fatal error: 'Cambria/Traps.h' file not found  
    方案：注释：objc-cache.mm 中 Line 87-88

24. 问题：fatal error: 'kern/restartable.h' file not found  
    文件：xnu-7195.101.1/osfmk/kern/restartable.h  
    方案：将 restartable.h 拷贝到 objc4-824/PrivateHeaders/kern/  

25. 问题：error: use of undeclared identifier 'oah_is_current_process_translated'  
    方案：注释：objc-cache.mm 中 Line 1121-1128

26. 问题：fatal error: 'Block_private.h' file not found  
    文件：libclosure-79/Block_private.h  
    方案：将 Block_private.h 拷贝到 objc4-824/PrivateHeaders/  

27. 问题：fatal error: 'os/reason_private.h' file not found  
    文件：xnu-7195.101.1/libkern/os/reason_private.h  
    方案：将 reason_private.h 拷贝到 objc4-824/PrivateHeaders/os/  

28. 问题：fatal error: 'os/variant_private.h' file not found  
    文件：Libc-1439.100.3/os/variant_private.h  
    方案：将 variant_private.h 拷贝到 objc4-824/PrivateHeaders/os/  

29. 问题：error: expected ',' API_AVAILABLE(macosx(10.16)) API_UNAVAILABLE(ios, tvos, watchos, bridgeos)
    方案：将 variant_privatre.h 中 
    Line 217 删除 bridgeos
    Line 243、Line 269、Line 287 删除 , bridgeos(4.0)：

30. 问题：  
    error: use of undeclared identifier 'dyld_platform_version_macOS_10_12'  
    error: use of undeclared identifier 'dyld_platform_version_iOS_10_0'  
    error: use of undeclared identifier 'dyld_platform_version_tvOS_10_0'  
    error: use of undeclared identifier 'dyld_platform_version_watchOS_3_0'  
    error: use of undeclared identifier 'dyld_platform_version_bridgeOS_2_0'  
    方案：注释 NSObject.mm 中 Line 1185-1189

31. 问题：error: '_static_assert' declared as an array with a negative size  
    方案：注释：objc-runtime-new.mm 中 Line 176-177  

32. 问题：can't open order file: /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX11.1.sdk/AppleInternal/OrderFiles/libobjc.order  
    方案：TARGETS 栏目下选中 objc，点击 Build Settings, 搜索 order，将 Order File 修改:$(SRCROOT)/libobjc.order  

33. 问题：ld: library not found for -lCrashReporterClient    
    方案：TARGETS 栏目下选中 objc，点击 Build Settings, 搜索 other link，在 Other Linker Flags 中删除 -lCrashReporterClient  

34. 问题：ld: library not found for -loah 
    方案：TARGETS 栏目下选中 objc，点击 Build Settings, 搜索 other link，在 Other Linker Flags 中删除 -loah   

35. 问题：error: SDK "macosx.internal" cannot be located  
    方案：TARGETS 栏目下选中 objc，点击 Build Phases，在 Run Script(markgc) 中将 macosx.internal 修改为 macosx  

    **至此，objc4-824 编译成功**

## 创建测试工程
1. TARGETS 栏目底部点击加号，创建一个 macOS  Command Line Tool 工程 objc_test  
2. TARGETS 栏目下选中 objc_test, 点击 General, 在 Frameworks and Libraries 中添加引用，选择 Libobjc.A.dylib, Embed 选择 Do Not Embed
3. TARGETS 栏目下选中 objc_test, 点击 Build Phases, 检查 Dependencies 中是否已经正确的添加了依赖库 objc 

## 断点不生效的问题
1. objc4-824 源代码 断点不生效：
   TARGETS 栏目下选中 objc_test，点击 Build Settings, 搜索 Signing， 在 Signing 下，将 Enable Hardened Runtime 由 YES 改为 NO

2. objc_test main.m 断点不生效：
   TARGETS 栏目下选中 objc_test，点击 Build Phases 在 Compile Source 中，将 main.m 文件移至第一位

   **至此，可以调试 objc4-824 源码了**  
   
## 源码及测试工程下载
[objc4-824 测试工程(已完成配置)](https://github.com/hubupc/objc4-824.git)

## 参考资料
- [01-objc824源码编译](https://www.jianshu.com/p/f830d4cbc6b4)
- [github: RegretSF/objc824](https://github.com/RegretSF/objc824)
