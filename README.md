# <p style="background-color: #00aeff; padding: 40px;" id="top" align="center"> <img style="height:140px; background-color: #00aeff;" src="./src/assets/img/frame/login-icon.png" alt="EMMS"/> <p align="center" class="token function">EMMS</p></p>

#### [markdown(.md)文档查看](#markdown(.md)文档查看)  &emsp; &emsp;  [打包/运行命令说明](#打包/运行命令说明)  &emsp; &emsp;  [引用包名替换](#引用包名替换) &emsp; &emsp;  [弃用模块引用移除](#弃用模块引用移除) &emsp;&emsp;  [路由改造](#路由改造)

#### [视图组件挂载调整](#视图组件挂载调整)  &emsp;&emsp;  [组件参数顺序](#组件参数顺序)  &emsp;&emsp;  [跳转及传参](#跳转及传参)  &emsp;&emsp;  [微应用统一规范](#微应用统一规范)  &emsp;&emsp;  [配置说明](#配置说明)  &emsp;&emsp;  [资源路径修改](#资源路径修改)

#### [后续代码优化](#后续代码优化)  &emsp; &emsp;  [第三方库处理](#第三方库处理) &emsp; &emsp;  [常见问题处理](#常见问题处理)


----------------------------------------------------------------`万恶分割线`-----------------------------------------------------------

#### [markdown(.md)文档查看](#markdown(.md)文档查看)
  -  推荐使用Google插件查看该文档
  1. 解压项目`markdown.rar`压缩文件，解压出来的文件夹保存在插件存放位置
  2. 打开Google浏览器`扩展程序`
  3. 在`扩展程序`界面右上角打开`开发者模式`
  4. 把解压的文件夹拖拽到`扩展程序面板`安装
  5. 插件卡片右下角`启用插件`
  6. 点击插件的`详细信息` =》 打开`允许访问文件网址`
  7. 使用Google浏览器打开md文档查看
  
  
----------------------------------------------------------------`万恶分割线`-----------------------------------------------------------


#### [打包/运行命令说明](#打包/运行命令说明)
  - npm run start 启动dev环境`配置文件 src/environment/environment.ts`
  - npm run build:dev 打包dev环境`配置文件 src/environment/environment.dev.ts`
  - npm run build:test 打包自定义环境`配置文件 src/environment/environment.test.ts`
  - npm run build 打包生产环境`配置文件 src/environment/environment.prod.ts`
  
  
----------------------------------------------------------------`万恶分割线`-----------------------------------------------------------


## [Ionic&Angular升级及微应用接入改造]


#### [引用包名替换](#引用包名替换)
  - ionic-angular => @ionic/angular
  ```javascript
  // old
  import {NavController} from 'ionic-angular';  // 移除
  // new
  import {NavController} from '@ionic/angular';  //  新增
  ```


----------------------------------------------------------------`万恶分割线`-----------------------------------------------------------


#### [弃用模块引用移除](#弃用模块引用移除)
  - IonicPage、NavParams、NavController、IonicPageModule模块引用移除、及ion-nav组件弃用(`弃用这四个模块及nav组件`)
  
  - NavController 之前用于设置ion-nav参数，已在CoreCom公用模块重新实现功能。 页面中直接移除相关引用即可
    * `this.router.navigate([path], {queryParams: param});`
    
  - NavParams 之前用于接收ion-nav参数，现已重新实现传参方式，页面中把接收的方式用queryParams查询即可
    * this.route.snapshot.queryParams['liveId']  // 页面中直接移除相关引用即可
    * this.route.snapshot.paramMap.get['liveId']  // :eventId/*** 占位符传参查询
    
```javascript
// old
import {IonicPage, NavParams, NavController, IonicPageModule} from '@ionic/angular';  // 移除模块引用
constructor(public navCtrl: NavController, public nvaParams: NavParams, public baseService: BaseService) { // 移除
     super(navCtrl, baseService);   // 移除
     this.liveId = navParams.get("liveId")|| ""; // 移除
     this.eventId = navParams.get("eventId")|| ""; // 移除
}
```
```javascript
// new
constructor(public route: ActivatedRoute, public baseService: BaseService) { // 新增
     super(baseService);    // 新增
     this.liveId = this.route.snapshot.queryParams['liveId'] // 新增
     this.eventId = this.route.snapshot.paramMap.get('eventId'); // 新增
}
```


-----------------------------------------------------------------`万恶分割线`-----------------------------------------------------------


#### [路由改造](#路由改造)
  - 移除IonicPage、IonicPageModule页面路由注册调用
```javascript
// 某组件注册 ***.ts
// old
@IonicPage({                              // 移除
     name: 'eventReportList',              // 移除
     segment: 'eventReportList',           // 移除
     defaultHistory: ['dashboard']         // 移除
})
// new
// 无新增
```
```javascript
// 某组件模块注册 ***.module.ts
// old 
import { IonicPageModule } from '@ionic/angular';   // 移除
imports: [ 
     BaseModule,
     IonicPageModule.forChild(LiveBasicConfigCom),   // 移除
     IonicModule
]
// new 
import { RouterModule } from '@angular/router';  // 新增
imports: [ 
     BaseModule,
     RouterModule.forChild([{path: '', component: LiveBasicConfigCom}]),  // 新增 =》 在模块注册import 添加路由注册
     IonicModule
]
```
```javascript
// 给页面新增路由规则  src/com/com-routing.module
{
     path: 'login',  // 该路由规则沿用之前 @IonicPage 注册调用的 segment 参数
     loadChildren: () => import('../com/login/component/LoginCom.module').then( m => m.LoginModule)
}
```
  - 路由配置规范`详情参考直播模块的路由设置`
    * 微应用按业务模块接入，限制模块的访问权限，同一业务的模块，路由需使用统一的基础路径 如：直播模块 localhost:4200#live，直播相关的模块路由均有 live 前缀`live/lsit, live/details`

-----------------------------------------------------------------`万恶分割线`-----------------------------------------------------------


#### [视图组件挂载调整](#视图组件挂载调整)
  - ViewChild挂载方式修改
```javascript
     // old 
     @ViewChild(HeaderCom) // 移除
     private headerCom: HeaderCom; // 移除
     // new
     @ViewChild(HeaderCom, {static: true}) headerCom: HeaderCom; // 新增
```
  
  
------------------------------------------------------------------`万恶分割线`-----------------------------------------------------------


#### [组件参数顺序](#组件参数顺序)
  - 子类（派生类）super(baseService, router, route) 参数顺序根据父类（基类）constructor的接收位置进行调整
     - `调整参数包含ActivatedRoute、Router、NavParams、NavController` 


------------------------------------------------------------------`万恶分割线`-----------------------------------------------------------


#### [跳转及传参](#跳转及传参)
  - 公用模块封装API =》 go(路由path: String, 路由传递参数queryParams: Object)
    - path可用作占位符传参
```javascript
// :eventId/List
go('123/List')
// =========>>>>>>>>
this.route.snapshot.paramMap.get('eventId') // 123;
```
  
   - queryParams参数
```javascript
// :eventId/List
go('123/List', { LiveId: 456 })
// =========>>>>>>>>
this.route.snapshot.paramMap.get('eventId') // 123;
this.route.snapshot.queryParams.get('LiveId') // 456;
```
  
   - 更多使用方式参考[官方文档](https://angular.cn/guide/router)


------------------------------------------------------------------`万恶分割线`----------------------------------------------------------


#### [微应用统一规范](#微应用统一规范)
   - 微应用设置localStorage等本地存储时，应加项目独有前缀，防止主应用加载多个微应用时缓存同名覆盖
     * PS：localStorage.setItem('项目名称前缀_缓存变量名', 值)
     * 所有微应用统一使用hash路由模式，并所有路由规则添加项目路由前缀
       - `如项目名为subLive微应用，sub-live为项目所有路由的统一前缀，直播页：sub-live/live 登录页：sub-live/login`
     * 主应用配置： 
       - 入口(entry)：域名/项目目录/#/项目路由前缀(sub-live)   
       - 菜单(menu)：/#/路由规则(live)


------------------------------------------------------------------`万恶分割线`----------------------------------------------------------


#### [配置说明](#配置说明)
   - 打包/运行环境变量[environments](https://angular.cn/guide/build)目录,配置部署地址server_url(域名/项目目录)
   - 更改项目部署服务器/目录名称
     * 修改environments环境变量部署地址server_url
     * 修改angular.json root属性值，deployUrl属性值，outputPath属性值
   - 更该微应用统一路由前缀 src/app/app-routing.module.ts
```javascript
  {
    path: '',
    redirectTo: '项目路由前缀', 
    pathMatch: 'full'
  },
  {
    path: '项目路由前缀',
    loadChildren: () => import('../com/com.module').then( m => m.ComModule),

  },
```
   - 主应用应用添加微应用参数配置 
     * 入口(entry)：域名/项目目录/#/微应用所有路由的统一规则(sub-live)/业务模块限制规则(live)/     
     * 菜单(menu): /#/具体页面路由(liveList)  


------------------------------------------------------------------`万恶分割线`----------------------------------------------------------


#### [资源路径修改](#资源路径修改)
   - 新增src/config/config.ts => SERVER_URL常量，配置项目所发布的位置
     * PS：项目地址 http://localhost:8090/EMM 则配置 SERVER_URL = http://localhost:8090/EMM
   - SERVER_URL开发环境下默认为http://localhost:4200/，生产环境应打包需配置正确地址
   - html模板内的图片等静态资源需添加 SERVER_URL 前缀变量
     * PS：<img [src]="config.SERVER_URL + 'assets/img/com/close.png'"/>
   - html模板内禁止引用script、link等标签，应迁移至index.html引入（以兼容微应用资源请求）


-----------------------------------------------------------------`万恶分割线`-----------------------------------------------------------


#### [第三方库处理](#第三方库处理)
   - 由于微应用接入window全局对象会被代理做JS沙箱隔离，导致在特定情况下获取不到第三方库挂载到全局的对象，处理方式如下：[更多参考官网](https://qiankun.umijs.org/zh/faq)
     - 把WdatePicker这个插件在主应用中导入（即把WdatePicker对象挂载到主应用的window上面，确保微应用调用window.WdatePicker时能正确调用）
```html
 <input type="text" onclick="WdatePicker({minDate:'#F{$dp.$D(&#92;&#39;startDate&#92;&#39;)}'})"/>
```
```javascript
// JS沙箱代理微应用window对象示例
(function(window) {
  // 微应用脚本
  // ...
})(new Proxy(window, { }))
```
   - 第三方库动态创建script、link、img等标签引用资源
     - 在微应用运行时会导致资源找不到
       * PS: 当请求 ./assets/img/test.img 资源时，项目独立运行请求完整url: 项目服务器/assets/img/test.img，作为微应用运行请求完整：主应用服务器/assets/img/test.img
     - 解决方式：
       1. 把资源放到线上服务器，然后把资源本地url替换为线上url
       2. 把资源放在主应用中加载（如果是样式，可以在主应用的微应用容器标签下@import引入该样式）
       3. 小图片可考虑直接把url转为base64替换


-----------------------------------------------------------------`万恶分割线`-----------------------------------------------------------


#### [常见问题处理](#常见问题处理)
   - 微应用运行时，检查所有资源是否被正确加载（当主应用路由为history模式时，资源请求路径错误会返回主应用index.html模板，响应状态为200）
   - 微应用调用全局变量(包含第三方库挂载到全局的变量)不能正常获取，考虑是否因JS沙箱隔离导致，或请求资源url不正确
   - JS隔离冲突问题，考虑是否可以把冲突库放到主应用中加载运行，让微应用去调用
   - 静态资源引用问题[更多参考官网](https://qiankun.umijs.org/zh/faq)
      * 使用SERVER_URL常量添加前缀
      * 把资源放到线上服务器（需服务器允许跨域访问）
      * 把资源放到主应用中加载运行或挂载样式等，由微应用使用
   - [deployUrl静态资源基础路径，base-href配置说明](https://www.cnblogs.com/jehorn/p/8568109.html)
   - [root配置说明](https://angular.cn/guide/workspace-config)
   - [打包命令配置环境变量说明](https://angular.cn/guide/build#configure-target-specific-file-replacements)
      


-----------------------------------------------------------------`万恶分割线`-----------------------------------------------------------


#### [后续代码优化](#后续代码优化)
   - 可配置AOT预编译[AOT预编译](https://angular.cn/guide/aot-compiler)
