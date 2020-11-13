+++
title= "WorkingTipsOnNpgV2"
date = "2020-11-12T16:46:29+08:00"
description = "WorkingTipsOnNpgV2"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 安装echarts依赖
Via following commands:    

```
cnpm install echarts -S     
cnpm install ngx-echarts -S
 cnpm install resize-observer-polyfill -S
```
如果后面出现 `NullInjectorError: No provider for ElementRef!`的问题，则重复执行:    

```
npm install echarts -S
npm install ngx-echarts -S
npm install resize-observer-polyfill -D
```
### 创建echarts模块
创建一个名为`echarts`的子app:    

```
$ ng g c echarts                         
Your global Angular CLI version (10.2.0) is greater than your local
version (8.3.28). The local Angular CLI version is used.

To disable this warning use "ng config -g cli.warnings.versionMismatch false".
CREATE src/app/echarts/echarts.component.css (0 bytes)
CREATE src/app/echarts/echarts.component.html (22 bytes)
CREATE src/app/echarts/echarts.component.spec.ts (635 bytes)
CREATE src/app/echarts/echarts.component.ts (273 bytes)
UPDATE src/app/app.module.ts (6586 bytes)
```

### 添加导航
添加echarts到导航栏中.    

![/images/2020_11_13_10_31_34_879x655.jpg](/images/2020_11_13_10_31_34_879x655.jpg)



