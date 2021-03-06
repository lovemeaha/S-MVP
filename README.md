## S-MVP
MVP 继续解耦

### 引言

-  **MVC时代**：在MVC模型里，更关注的Model的不变，业务需求通常是Model不变，同时有多个对Model的不同显示，即View。所以，在MVC模型里，Model不依赖于View，但是View是依赖于Model的。
不仅如此，因为有一些业务逻辑在View里实现了，导致要更改View也是比较困难的，至少那些业务逻辑是无法重用的。

-  **MVP时代**：在MVP里，Presenter完全把Model和View进行了分离，主要的程序逻辑在Presenter里实现。而且，Presenter与具体的View是没有直接关联的，而是通过定义好的接口进行交互（单独测试时我们只需要按照接口传递参数即可），从而使得在变更View时候可以保持Presenter的不变，即重用！ 不仅如此，我们还可以编写测试用的View，模拟用户的各种操作，从而实现对Presenter的测试--而不需要使用自动化的测试工具

- **MVP解耦View时代**：以往我们的V层是一个Activity或Frament实现，大多数情况下，我们的一个页面布局很复杂,包含很多个ViewGroup,而不是简单的一个ListView、GridView，按照之前的会发现一个Activity中有很多Presenter来维护和很多的回调，有时我们只需要改动其中的一个ViewGroup内容逻辑，却不能很好的区分，所以我们要让每个View或者ViewGroup实现MVP，这样不管这个View放在哪(Activity/Fragment/ViewGroup)都能够很好的移植，并且修改某一个的时候也会互相不影响，所以通过LifeCycle和组件化View可以很好解决上面问题

### 内容

- #### 1.通过模块化减少了类的创建

  常见写法：接口一个类，实现一个类，这样的MVP中就会出现6个类
  
  优化写法：Contract类管理接口,这样的MVP中只会出现4个类
  
- #### 2.通过注解，隐藏Presenter创建，减少代码

  常见写法：需要在每个Activity中创建一个Presenter
  
  优化写法：在父类里拿到子类的名字进行创建，创建过程通过在要创建的Presenter添加上注解标识，在编译器动态生成代码

- #### 3.添加Gradle插件使用Aspectj编译器

- #### 4.通过Aspectj 实现TimeLog耗时打印，自定义Buidlconfig实现开关

- #### 5.添加Lru缓存切片通过使用@MemoryCache注解

- #### 6.添加登陆缓存切片用来检测是否登陆

- #### 7.添加异常捕获，打印，保证程序不崩溃

- #### 8.自定义BindView框架，通过@$(R.id.abc)作用在public类型的变量
    1. 通过InjectView.bind();实现绑定
    2. 通过InjectView.bind();实现解绑
    3. 支持在Activity和View中进行绑定

- #### 9.通过javassist修改字节码的方式，可以生成类，也可以在某些特定方法注入代码，在保证不修改源码的情况，完成aop，避免代码碎片化

- #### 10.通常我们一个页面需要不止一个请求，并且这些请求是异步的，这样可以得到很好的用户体验，往往我们还需要，等待这些请求都完成后做一些处理

   这里封装一个Asynctask
   
   - 使用方法：
      1. 创建一个SmartTaskManager,调用put方法传递一个要同步的线程数量，和指定一个key
      2. 其他文件通过key获取Asynctask对象，在各线程执行完后调用Asynctask对象一个onFinish()方法
      3. SmartTaskManager获取调用toEnd传递一个Runnable，当两个线程执行完后就会回调这个方法
      4. 最后页面退出的时候移除这个task,SmartTaskManager.remove(key);
   - 例子
          
           smartTaskManager = SmartTaskManager.as();
           smartTaskManager.put("initTask",2);
           smartTaskManager.put("init");
           smartTaskManager.getAsyncTask("initTask").toEnd(new Runnable() {
               @Override
               public void run() {
                   Toast.makeText(getApplicationContext(),"页面全部初始化完成",Toast.LENGTH_SHORT).show();
               }
           });
     
- #### 11.业务中我们需要一个请求后调用另一个请求再调用其他请求，这种串行请求我们通常使用嵌套的形式，缺点很不好维护，如果要有需求要改变顺序，那需要改动很多的代码
   
   封装一个SyncTask
   
   - 使用方法：
     1. 创建一个SmartTaskManager,调用put方法传递一个要同步的线程数量，和指定一个key
     2. 其他文件通过key获取Synctask对象，在各线程执行完后调用Synctask对象一个onFinish(param)方法传递参数给下一个
     3. 通过这个Synctask对象我们调用onNext，添加任务，添加的任务顺序，就是执行顺序
     4. 调用start方法开始执行请求
     5. 最后页面退出的时候移除这个task,SmartTaskManager.remove(key);
   
   - 例子：
   
         SyncTask stk = smartTaskManager.getSyncTask("init");
         stk.onNext(obj -> initModel2.request_1())
                 .onNext(obj -> initModel2.request_2((String)obj))
                 .onNext(obj -> initModel2.request_3((String) obj, msg -> Toast.makeText(getApplicationContext(),msg,Toast.LENGTH_SHORT).show()));

         stk.start();
   #### SmartTask结构
   ![image](https://github.com/UCodeUStory/S-MVP/blob/master/smartManager.png)
   
- #### 12.添加LifeCycle实现组件化View开发
    1. moudle下build.gradle 添加：
   
              compile "android.arch.lifecycle:runtime:1.0.0-alpha4"
              compile "android.arch.lifecycle:extensions:1.0.0-alpha4"
              annotationProcessor "android.arch.lifecycle:compiler:1.0.0-alpha4"
              
    2. 项目工程下build.gradle 添加：
   
           allprojects {
               repositories {
                   jcenter()
                   google()
                   mavenCentral()
               }
           }
### 我的相关技术仓库

 Repository | Repository | Repository | Repository
---|---|---|---
[组件化开发框架](https://github.com/UCodeUStory/ComponentDevelopment)  | [插件化开发框架](https://github.com/UCodeUStory/AndroidPluginFramework) | [Gradle插件开发](https://github.com/UCodeUStory/GradlePlugin)|[Tinker热修复例子](https://github.com/UCodeUStory/TinkerDemo)
[事件状态机处理](https://github.com/UCodeUStory/StateMachine) | [MVVM设计](https://github.com/UCodeUStory/MVVM) | [JavaPoet](https://github.com/UCodeUStory/JavaPoetSample)


****
### **框架待优化**

2.EventBus,otto都可以考虑使用

3.使用Javassist注入字节码,这是一个很好的字节码编辑工具，提供在JVM运行期前修改的api

4.路由实现简单的跳转，路由器也是用来解耦的，增加后台可配置性

5.网络框架选择,封装一个公共的入口，http ,httpPost, httpGet,httpPostMap,httpGetMap,  HttpManager

8.添加日志缓存框架

AOP技术常用在以下方面：

1、日志记录：业务埋点

2、持久化

3、性能监控：性能日志

4、数据校验：方法的参数校验

5、缓存：内存缓存和持久缓存

6、权限检查：业务权限（如登陆，或用户等级）、系统权限（如拍照定位）

7、异常处理


### 处理流程
 1. 在build 过程我们可以通过apt 生成java文件,再通过Aspectj解析，编织成class,最后我们还可以通过Javassist修改class和jar文件，最终打包成dex 到 apk
 
 ![image](https://github.com/UCodeUStory/S-MVP/blob/master/pic.png)


 - Thanks
 - [Bai xiaokang]()
