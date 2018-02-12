<style>
table th {
    width: 300px;
}
</style>
table th:first-of-type {
    width: 100px;
}
</style>

# 前言

android平台上组件化开发的概念近两年非常火热，面试中被问到的频率也很高。

组件化开发技术产生的背景是：在项目到达一定阶段，就需要降低耦合度、提高可服用性及提高开发效率。相比于模块化开发难以独立测试以及边界模糊难以进行业务隔离、插件化技术黑科技太多兼容性不理想，组件化技术在原生开发的基础上实现了业务隔离及独立编译测试，实现了组件高复用性及可测试性，并大大提高了开发效率。正式因为这个原因让组件化开发的概念大受欢迎。

目前网上关于组件化开发方案的文章、开源库比较多，让很多初学者感到迷茫，不知该从何处入手，自身的业务特性适合使用哪种框架，如果全部都学习一遍成本比较高。

为了让大家对android组件化有个整体的认识，本文将从多个维度对目前网上开源的一些组件化开发方案进行对比

# 对比表

对比项|[CC](https://github.com/luckybilly/CC)|[得到DDComponentForAndroid](https://github.com/luojilab/DDComponentForAndroid)|51信用卡OkDeepLink|[ModularizationArchitecture](https://github.com/SpinyTech/ModularizationArchitecture)|[阿里Arouter](https://github.com/alibaba/Arouter)<br>(网上很多组件化方案的路由引擎，如[AndroidModulePattern](https://github.com/guiying712/AndroidModulePattern))|[聚美组件化方案](https://github.com/JumeiRdGroup/Router)<br>（基于聚美Router）|[ActivityRouter](https://github.com/mzule/ActivityRouter)
---|:------:|:------:|:------:|:------:|:------:|:------:|:------:
开源时间|2017-11|2017-9|2017-6|2017-1|2016-12|2016-9|2016-4
介绍文章|[wiki](https://github.com/luckybilly/CC/wiki)|[Android彻底组件化方案实践](https://www.jianshu.com/p/1b1d77f58e84)|[Android 组件化 —— 路由设计最佳实践](https://www.jianshu.com/p/8a3eeeaf01e8)|[Android架构思考(模块化、多进程)](http://blog.spinytech.com/2016/12/28/android_modularization/)|[开源最佳实践：Android平台页面路由框架Arouter](https://yq.aliyun.com/articles/71687?spm=5176.100240.searchblog.7.8os9Go)|[聚美组件化实践之路](https://juejin.im/post/5a4b4425518825128654eef4)|[ActivityRouter路由框架：通过注解实现URL打开Activity](https://joyrun.github.io/2016/08/01/ActivityRouter/)
activity跳转|✅|✅|✅|✅|✅|✅|✅
是否支持降级处理|✅|❌|||✅|✅|✅
activity变量自动注入|❌|1. 通过apt生成自动注入代码<br>2. 在onCreate中调用AutowiredService.Factory.getInstance().create().autowire(this);或者继承BaseActivity|||1. 通过apt生成解析参数的代码<br>2. 在onCreate方法中调用`ARouter.getInstance().inject(this);`实现自动注入|❌|❌
startActivityForResult|支持Activity/Fragment,但不建议使用<br>建议使用统一的组件调用方式|仅支持Activity|||仅支持Activity|支持Activity/Fragment|仅支持Activity
组件向外提供服务调用|与页面跳转一致，在IComponent中实现|接口下沉到base中，组件中实现接口并在IApplicationLike中添加代码注册到Router中|||接口继承IProvider并下沉到base中，组件中实现接口并通过注解来暴露服务|接口下沉到base中，组件中实现接口并在ApplicationDelegate中向接口管理类注册【PipeManager.register(CorePipe.class, new CorePipeImpl());】|在静态方法上加注解来暴露服务
调用方式(页面跳转)|`CC.obtainBuilder("ComponentA").build().call();`|`UIRouter.getInstance().openUri(getActivity(), url, bundle);`|||`ARouter.getInstance().build("/test/activity").navigation();`|`Router.create(url).open(context);`|`Routers.open(context, url);`
调用方式(调用服务)|与页面跳转相同|`Router.getInstance().getService(ReadBookService.class.getSimpleName())`|||`ARouter.getInstance().navigation(HelloService.class).sayHello();`|`PipeManager.get(LoginPipe.class).logout();`|与页面跳转相同
组件自动注册方案|TrasnformAPI + ASM扫描组件类(IComponent接口实现类)并注册到ComponentMananger中，无需手动维护组件列表|apt生成各module的路由表<br>TrasnformAPI + javassist将IApplicationLike的注册代码生成到自定义application.onCreate方法中，无需手动维护组件列表|||1. apt生成各module的路由表<br>2. Arouter初始化时扫描所有dex找出指定包名下的路由表，通过反射进行统一注册|1. apt生成各module的路由表pkg.RouterRuleCreator类<br>2. 在ComponentPackages中定义所有RouterRuleCreator的包名<br>3.在BaseApplication中反射所有的包名找到所有路由表RouterRuleCreator<br>4. 需要手动维护ComponentPackages类中的包名列表|1. apt生成各module的路由表<br>2. apt在application的module通过Modules注解生成RouterInit进行注册<br>3. 需要手动维护Modules注解中的组件列表
组件单独运行的方式|切换library/application方式编译，提供2种方式：<br>1. module/build.gradle中切换`ext.runAsApp=trueOrFalse`<br>2. 在local.properties中切换`moduleName=trueOrFalse`（推荐使用的方式，不会提交到代码仓库中）|切换library/application方式编译，在module/gradle.properties中切换`isRunAlone=trueOrFalse`|||切换library/application方式编译，框架本身没有提供切换方式，开发者自行解决|组件module始终以library方式编译，额外提供app壳子，可以按需将多个组件依赖进来一起打包。<br>好处是所有组件调试时包名相同，能满足分享及地图等第三方SDK对包名的要求|切换library/application方式编译，框架本身没有提供切换方式，开发者自行解决
跨app组件调用支持|✅|❌|||❌|❌|✅
组件app运行时调用其它组件|组件同时安装在设备上即可，实际开发中一般是当前正在开发的组件和主app中的组件互相调用.<br>通过广播 + Service + LocalSocket实现|将需要调用的组件一起打包才能调用|||一起打包或者通过urlScheme来统一转发|将需要调用的组件一起打包才能调用|UrlScheme原生支持跨app调用,组件同时安装在设备上即可<br>通过中介Activity转发:RouterActivity
组件依赖隔离|无需依赖、完全隔离|通过插件实现只在打apk包时才添加依赖，编码期间不能直接调用其它组件的代码，想知道如何实现可以戳[这里](https://github.com/luojilab/DDComponentForAndroid/blob/master/build-gradle/src/main/groovy/com.dd.buildgradle/ComBuild.groovy#L118:18)|||未隔离|未隔离|无需依赖、完全隔离