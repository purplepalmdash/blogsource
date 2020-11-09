+++
title= "Clarity与AngularCLI"
date = "2020-11-09T11:43:10+08:00"
description = "WorkingTipsOnClarity"
keywords = ["Technology"]
categories = ["Technology"]
+++
今天来看看如何使用Angular CLI集成Clarity。     

Clarity是什么？官方介绍如下:    

> Project Clarity是一个开源的设计系统，它汇集了UX准则，HTML/CSS框架和Angular 2组件。 Clarity适用于设计人员和开发人员。

### 1. 先决条件
本指南需要Angular CLI全局安装， 使用以下命令安装最新版本的angular:   

```
$ npm install -g  @angular/cli@latest
```

### 2. 创建新项目
现在`ng`命令应当是可用的, 我们使用`ng new`命令来创建一个新的Angular CLI项目:    

```
$ ng new myclarity
```
在弹出的选项中默认回车接受预设选项，此命令运行完毕后我们将得到一个`myclarity`的目录， 进入到此目录下运行`ng serve`，我们将得到标准的angular项目预览页.     

```
$ cd myclarity
$ ng serve
```

![/images/2020_11_09_11_50_01_739x487.jpg](/images/2020_11_09_11_50_01_739x487.jpg)

### 3. 安装Clarity依赖
要使用`Clarity`，我们需要使用npm命令安装包及其依赖:    

```
$ npm install @clr/core @clr/icons @clr/angular @clr/ui @webcomponents/webcomponentsjs --save
$ npm install --save-dev clarity-ui
$ npm install --save-dev clarity-icons
```

### 4. 添加脚本及样式
添加以下条目到`angular.json`文件的`scripts`及`styles`部分:    

```
"styles": [
      "node_modules/@clr/icons/clr-icons.min.css",
      "node_modules/@clr/ui/clr-ui.min.css",
      ... any other styles
],
"scripts": [
  ... any existing scripts
  "node_modules/@webcomponents/webcomponentsjs/custom-elements-es5-adapter.js",
  "node_modules/@webcomponents/webcomponentsjs/webcomponents-bundle.js",
  "node_modules/@clr/icons/clr-icons.min.js"
]
```
更改完毕后的`angular.json`文件(共有4处需要修改)看起来应当是:    

![/images/2020_11_09_11_58_11_947x310.jpg](/images/2020_11_09_11_58_11_947x310.jpg)

### 5. 添加Angular模块
到现在为止，包依赖都被安装并配置好了，我们可以开始进入到Angular AppModule来配置Clarity模块，配置完模块后我们就可以在应用中使用Clarity。   

打开`src/app/app.module.ts`文件，在文件头部添加以下依赖:    

```
import { NgModule } from '@angular/core';
import { BrowserModule } from "@angular/platform-browser";
import { BrowserAnimationsModule } from "@angular/platform-browser/animations";
import { ClarityModule } from "@clr/angular";
import { AppComponent } from './app.component';
```
这将告诉TypeScript从`clarity-angular`包中加载模块。要使用包我们需要在`@NgModule`的`import`变量中添加以下行:    

```
  imports: [
    BrowserModule,
    BrowserAnimationsModule,
    ClarityModule
  ],
```
到现在为止，基础配置已经完成。运行`ng serve`将看到app显示已经运行。    

### 6. 使用Clarity
#### 6.1 生成UI模块及组件
Clarity是一个相当高级的框架，可以快速创建出完整的用户界面。现在我们开始设定一些UI元素。    

为了快速创建出UI目录结构，我们使用`ng generate`命令，或者简写的`ng g`命令:    

```
# 创建ui模块
$ ng g m ui
CREATE src/app/ui/ui.module.ts (188 bytes)

# 创建layout(布局)组件
$  ng g c ui/layout -is -it  --skipTests=true
CREATE src/app/ui/layout/layout.component.ts (265 bytes)
UPDATE src/app/ui/ui.module.ts (264 bytes)

# 创建header, sidebar及main view 组件
$ ng g c ui/layout/header -is -it --skipTests=true
CREATE src/app/ui/layout/header/header.component.ts (265 bytes)
UPDATE src/app/ui/ui.module.ts (349 bytes)
$ ng g c ui/layout/sidebar -is -it --skipTests=true
CREATE src/app/ui/layout/sidebar/sidebar.component.ts (268 bytes)
UPDATE src/app/ui/ui.module.ts (438 bytes)
$ ng g c ui/layout/main -is -it --skipTests=true
CREATE src/app/ui/layout/main/main.component.ts (259 bytes)
UPDATE src/app/ui/ui.module.ts (515 bytes)
```

上述命令`ng g`使用的参数含义如下:    

`-is` 使用inline样式替代独立的CSS文件。    
`-it` 使用inline模板替代独立的html文件。    
`--skipTests=true` 不生成用于测试的spec文件。    

当前创建出的目录架构如下:    

```
$ tree src/app/ui
src/app/ui
├── layout
│   ├── header
│   │   └── header.component.ts
│   ├── layout.component.ts
│   ├── main
│   │   └── main.component.ts
│   └── sidebar
│       └── sidebar.component.ts
└── ui.module.ts

4 directories, 5 files
```

要在app中使用刚才这些创建的模块我们需要在`AppModule`中引入`UiModule`， 编辑`src/app/app.module.ts`，添加以下的import声明语句:    

```
import { UiModule } from './ui/ui.module';
```
在`@NgModule`部分的`imports`数组中添加`UiModule`。    

因为我们需要在`UiModule`中使用`Clarity`,我们同样需要在`ui`的相关文件下添加它们。打开`src/app/ui/ui.module.ts`文件，添加以下`import`声明:    

```
import { ClarityModule } from "@clr/angular";
import { BrowserAnimationsModule } from "@angular/platform-browser/animations";
```

在`@NgModule`部分的`imports`数组中添加`ClarityModule`。    

最后需要在`UiModule`中添加一个`exports`数组，将`LayoutModule`导出，因为我们需要在`AppComponent`中使用它。   

```
  imports: [
    CommonModule,
    ClarityModule
  ],
    exports: [
    LayoutComponent,
  ]
```
`UiModule`现在看起来是这样的:    

```
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { LayoutComponent } from './layout/layout.component';
import { HeaderComponent } from './layout/header/header.component';
import { SidebarComponent } from './layout/sidebar/sidebar.component';
import { MainComponent } from './layout/main/main.component';

import { ClarityModule } from "@clr/angular";
import { BrowserAnimationsModule } from "@angular/platform-browser/animations";

@NgModule({
  declarations: [LayoutComponent, HeaderComponent, SidebarComponent, MainComponent],
  imports: [
    CommonModule,
    ClarityModule
  ],
    exports: [
    LayoutComponent,
  ]
})
export class UiModule { }
```
#### 6.2 撰写并创建UI组件
##### 6.2.1 AppComponent
我们一开始来更新`AppComponent`的模板文件，打开`src/app/app.component.html`文件，将内容替换为:    

```
<app-layout>
  <h1>{{title}}</h1>
</app-layout>
```
这里我们通过选择`app-layout`引用了`LayoutComponent`.    

##### 6.2.2 LayoutComponent
打开`src/app/ui/layout/layout.component.ts`文件，替换`template`部分为以下内容:    

```
   <div class="main-container">
    <app-header></app-header>
    <app-main>
      <ng-content></ng-content>  
    </app-main>
  </div> 
```
我们在`div` `main-container`中包装了我们的内容，引用了`HeaderComponent`和`MainComponent`， 在`MainComponent`中，我们包含了app内容，使用的是`ng-content`组件。    

##### 6.2.3 HeaderComponent
打开`src/app/ui/layout/header/header.component.ts`文件，更新模板:    


```
   <header class="header-1">
    <div class="branding">
      <a class="nav-link">
        <clr-icon shape="shield"></clr-icon>
        <span class="title">Angular CLI</span>
      </a>
    </div>
    <div class="header-nav">
      <a class="active nav-link nav-icon">
        <clr-icon shape="home"></clr-icon>
      </a>
      <a class=" nav-link nav-icon">
        <clr-icon shape="cog"></clr-icon>
      </a>
    </div>
    <form class="search">
      <label for="search_input">
        <input id="search_input" type="text" placeholder="Search for keywords...">
      </label>
    </form>
    <div class="header-actions">
      <clr-dropdown class="dropdown bottom-right">
        <button class="nav-icon" clrDropdownToggle>
          <clr-icon shape="user"></clr-icon>
          <clr-icon shape="caret down"></clr-icon>
        </button>
        <div class="dropdown-menu">
          <a clrDropdownItem>About</a>
          <a clrDropdownItem>Preferences</a>
          <a clrDropdownItem>Log out</a>
        </div>
      </clr-dropdown>
    </div>
  </header>
  <nav class="subnav">
    <ul class="nav">
      <li class="nav-item">
        <a class="nav-link active" href="#">Dashboard</a>
      </li>
      <li class="nav-item">
        <a class="nav-link" href="#">Projects</a>
      </li>
      <li class="nav-item">
        <a class="nav-link" href="#">Reports</a>
      </li>
      <li class="nav-item">
        <a class="nav-link" href="#">Users</a>
      </li>
    </ul>
  </nav> 
```
代码有点长，我们解释如下:
* 定义了一个`header-1`的头.
* branding由icon和title构成。
* 两个header icon, Home/Settings。
* search box，placeholder文字。
* user icon， 含有3个item的下拉列表。
* sub navigation，含有4个链接。

##### 6.2.4 MainComponent
差不多快完成了，我们打开`src/app/ui/layout/main/main.component.ts`更新以下模板内容:    

```
  <div class="content-container">
    <div class="content-area">
      <ng-content></ng-content>
    </div>
    <app-sidebar class="sidenav"></app-sidebar>
  </div>  
```
在这个组件中我们将`sidebar`及内容模块封装在`div`中，该`div`名为`content-container`.   

在接下来的`app-sidebar`选择器中我们创建了另一个`div`，名为`content-area`， 这是app的主要内容所展示的地方，我们使用内建的`ng-content`组件用于包装它。    

##### 6.2.5 SidebarComponent
最后一步了! 我们打开`src/ap/ui/layout/sidebar/sidebar.component.ts`文件，更新以下模板:    

```
    <nav>
    <section class="sidenav-content">
      <a class="nav-link active">Overview</a>
      <section class="nav-group collapsible">
        <input id="tabexample1" type="checkbox">
        <label for="tabexample1">Content</label>
        <ul class="nav-list">
          <li><a class="nav-link">Projects</a></li>
          <li><a class="nav-link">Reports</a></li>
        </ul>
      </section>
      <section class="nav-group collapsible">
        <input id="tabexample2" type="checkbox">
        <label for="tabexample2">System</label>
        <ul class="nav-list">
          <li><a class="nav-link">Users</a></li>
          <li><a class="nav-link">Settings</a></li>
        </ul>
      </section>
    </section>
  </nav>
```
到现在为止clarity驱动的UI应该已经就绪，效果如下图:    

![/images/2020_11_09_14_12_03_551x393.jpg](/images/2020_11_09_14_12_03_551x393.jpg)

### 7. 随便玩
#### 7.1 添加导航页面
添加pages模块及导航分页:    

```
$ ng g m pages
$ ng g c pages/dashboard -is -it --skipTests=true
$ ng g c pages/posts -is -it --skipTests=true
$ ng g c pages/settings -is -it --skipTests=true
$ ng g c pages/todos -is -it --skipTests=true
$ ng g c pages/users -is -it --skipTests=true
```
查看当前目录结构:    

```
$ tree src/app
src/app
├── app.component.css
├── app.component.html
├── app.component.spec.ts
├── app.component.ts
├── app.module.ts
├── pages
│   ├── dashboard
│   │   └── dashboard.component.ts
│   ├── pages.module.ts
│   ├── posts
│   │   └── posts.component.ts
│   ├── settings
│   │   └── settings.component.ts
│   ├── todos
│   │   └── todos.component.ts
│   └── users
│       └── users.component.ts
└── ui
    ├── layout
    │   ├── header
    │   │   └── header.component.ts
    │   ├── layout.component.ts
    │   ├── main
    │   │   └── main.component.ts
    │   └── sidebar
    │       └── sidebar.component.ts
    └── ui.module.ts
```

#### 7.2 打通导航
`src/app/ui/ui.module.ts`中引入`RouterModule`:    

```
import { RouterModule } from '@angular/router';

  imports: [
￼    CommonModule,
￼    RouterModule,
￼    ClarityModule,
```

