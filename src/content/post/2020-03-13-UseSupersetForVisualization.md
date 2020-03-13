+++
title= "用Superset可视化武汉肺炎数据"
date = "2020-03-13T09:52:52+08:00"
description = "UseSupersetForVisualization"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 前言
最近因为武汉肺炎的原因一直宅在家里，刷什么值得买的时候看到了一个玩mac mini server的哥们写的用superset可视化武汉肺炎的文章，照着做了一遍，将具体的步骤都写在了下面。后续做实际项目的BI内容可视化的时候可以用来参考。    

该文章中也有少许操作步骤方面的错误，做的时候经常会卡在那里半天。当然如果按照下面记录的步骤来进行的话，这些问题应该不会出现。    

### 环境
涉及到的硬件、操作系统、软件等的情况列举如下：    

```
KVM虚拟机，4核，3G内存， 200G硬盘
Ubuntu18.04.3 x86_64
docker/docker-compose
```

### 搭建superset
通过 docker启动superset, 启动后监听 `8088` 端口:    

```
# sudo docker run -d --name superset -p 8088:8088 amancevice/superset
```
初始化数据库：    

```
# sudo docker run -d --name superset -p 8088:8088 amancevice/superset
cf39f0c9e6a1232796cfe9012d110d6dd50613a8e2022873c963974eb74c18ba
root@node:/mnt2# docker exec -it superset superset-init
Username [admin]: 
User first name [admin]: 
User last name [user]: 
Email [admin@fab.org]: 
Password: 
Repeat for confirmation: 
```
现在可以用admin用户及刚才创建的密码登录superset界面了：     

![/images/2020_03_13_13_10_20_741x496.jpg](/images/2020_03_13_13_10_20_741x496.jpg)

登录后界面如下：    

![/images/2020_03_13_13_15_05_996x482.jpg](/images/2020_03_13_13_15_05_996x482.jpg)

### 准备数据
肺炎的数据从以下页面取得：    

[https://github.com/canghailan/Wuhan-2019-nCoV](https://github.com/canghailan/Wuhan-2019-nCoV)    

点击下面链接中数据接口里的csv，将csv文件下载到本地:    

![/images/2020_03_13_13_17_00_846x360.jpg](/images/2020_03_13_13_17_00_846x360.jpg)

csv文件可以方便的用libreoffice或者excel打开，后面我们将用libreoffice对数据做一点点更改以支持superset中的地图显示。     

手动建立省份代码csv文件( `province.csv` ), 这个文件是ISO3316标准下的中国省份代码，因为superset的地图中需要使用该标准下的代码:     

```
编码,城市
北京市,CN-11
天津市,CN-12
河北省,CN-13
山西省,CN-14
内蒙古自治区,CN-15
辽宁省,CN-21
吉林省,CN-22
黑龙江省,CN-23
上海市,CN-31
江苏省,CN-32
浙江省,CN-33
安徽省,CN-34
福建省,CN-35
江西省,CN-36
山东省,CN-37
河南省,CN-41
湖北省,CN-42
湖南省,CN-43
广东省,CN-44
广西壮族自治区,CN-45
海南省,CN-46
重庆市,CN-50
四川省,CN-51
贵州省,CN-52
云南省,CN-53
西藏自治区,CN-54
陕西省,CN-61
甘肃省,CN-62
青海省,CN-63
宁夏回族自治区,CN-64
新疆维吾尔自治区,CN-65
```
在libreoffice中打开两个csv文件后，在`Wuhan-2019-nCov` 的工作空间里新建一个tab将 `province.csv`内容全盘复制进来, 并将此tab命名为 `province`:    

![/images/2020_03_13_13_27_26_501x223.jpg](/images/2020_03_13_13_27_26_501x223.jpg)

接下来我们使用vlookup用来将ISO3316的省份代码插入到`Wuhan-2019-nCov`表中, 首先在province栏后新建一栏，命名为`3316code`:    

![/images/2020_03_13_13_34_44_688x239.jpg](/images/2020_03_13_13_34_44_688x239.jpg)

此栏目现在是空的，我们需要用函数将其批量替换为省份对应的代码，如，`湖北省` -> `CN-41`, 首先选中E中的第一个空白栏，而后点击`fx`按钮，在弹出的函数选择框中输入 `vookup` 后，启动函数编辑器:     

![/images/2020_03_13_13_38_55_713x634.jpg](/images/2020_03_13_13_38_55_713x634.jpg)

点击 `Next` 后进入到函数参数输入界面, 首先配置搜索条件, 在`Search criterion`中填入`D:D`， libreoffice里将自动选择D全栏, 代表搜索省份中所有的条目:      


![/images/2020_03_13_13_44_49_494x457.jpg](/images/2020_03_13_13_44_49_494x457.jpg)

选中`Array`后，鼠标点击切换到 `Province` 表， 选中A/B全栏， 或者直接输入 `$Province.A:B`，匹配:     

![/images/2020_03_13_14_05_21_736x482.jpg](/images/2020_03_13_14_05_21_736x482.jpg)

`Index` 输入数值2, 代表从匹配变量一栏开始需要匹配的数据为第2栏，而 `Sort order`则输入0后，可以看到输出结果为 `CN-42`， 代表已经匹配成功:    

![/images/2020_03_13_14_05_52_666x477.jpg](/images/2020_03_13_14_05_52_666x477.jpg)

点击 `OK` 后，点击函数将其扩展到以下几行，观察结果:     

![/images/2020_03_13_14_06_49_278x253.jpg](/images/2020_03_13_14_06_49_278x253.jpg)

可以看到有 `#N/A` 的错误输出结果，我们需要修改函数忽略掉此输出:    

![/images/2020_03_13_14_06_49_278x253.jpg](/images/2020_03_13_14_06_49_278x253.jpg)

修改函数为 `IFERROR(VLOOKUP(D:D, $Province.A:B,2,0),"")`之后，查看结果:    

![/images/2020_03_13_14_12_02_492x203.jpg](/images/2020_03_13_14_12_02_492x203.jpg)

结果显示正常:     

![/images/2020_03_13_14_13_04_364x293.jpg](/images/2020_03_13_14_13_04_364x293.jpg)

接下来将此函数应用到全列, 选中一个已经应用公式的条目后，如E30, 按住shift键，鼠标一直拉到E的最后一行后，点最后一个元素，选中全列，而后按`Ctrl+D`，将公式应用到全列，如:    

![/images/2020_03_13_14_24_38_788x636.jpg](/images/2020_03_13_14_24_38_788x636.jpg)

现在保存此csv文件，命名为 `nCovForSuperset.csv`后退出, 注意要选择 `Use Text CSV Format `。    

![/images/2020_03_13_14_25_31_545x184.jpg](/images/2020_03_13_14_25_31_545x184.jpg)

 至此，数据准备完毕。   

 ### 建立数据源
 用sqlite3建立一个 `假` 的数据文件:     

 ```
# sqlite3 test.db
SQLite version 3.22.0 2018-01-22 18:45:57
Enter ".help" for usage hints.
sqlite> .quit
# ls
nCovForSuperset.csv  province.csv  test.db  Wuhan-2019-nCoV.csv
 ```
我们将 此 `test.db`文件拷贝到容器中, 并通过root用户改变其文件权限:      

```
# docker cp test.db superset:/home/superset/
# docker exec -it  --workdir /root --user root superset chmod 777 /home/superset/test.db
```

在Superset界面中点击 `Sources` -> `Database`, 后，进入到配置数据库的界面:      

![/images/2020_03_13_14_31_43_1049x376.jpg](/images/2020_03_13_14_31_43_1049x376.jpg)

点击 `+` 号，新加一个数据库, 输入数据库名，及数据库所在的URI，此例子中为 `sqlite:////home/superset/test.db`:     

![/images/2020_03_13_14_38_24_837x348.jpg](/images/2020_03_13_14_38_24_837x348.jpg)

勾选 `Allow Csv Upload` 及 `Allow CREATE TABLE AS`:     

![/images/2020_03_13_14_39_50_568x222.jpg](/images/2020_03_13_14_39_50_568x222.jpg)

之后点击 `Save`， 可以看到数据库已经被建立起来:      

![/images/2020_03_13_14_40_15_631x301.jpg](/images/2020_03_13_14_40_15_631x301.jpg)

上传刚才编辑好的CSV文件:     

![/images/2020_03_13_14_44_15_571x374.jpg](/images/2020_03_13_14_44_15_571x374.jpg)

会报出出错，提示因权限问题无法上传此CSV文件：    

![/images/2020_03_13_14_45_42_1191x277.jpg](/images/2020_03_13_14_45_42_1191x277.jpg)

手动建立目录并改变其权限:     

```
# docker exec -it  --workdir /root --user root superset mkdir -p /usr/local/lib/python3.6/site-packages/superset/app
# docker exec -it  --workdir /root --user root superset chmod 777 -R /usr/local/lib/python3.6/site-packages/superset/app
```
上传成功后可以看到`Tables`中有了新的文件:     

![/images/2020_03_13_14_47_11_726x357.jpg](/images/2020_03_13_14_47_11_726x357.jpg)

### 更改tables属性
CSV上传后，大部分的字段（数据) 并没有被确定为准确的类型，superset需要从这些数据中知道哪些是数据，哪些是时间，哪些又可以被归类，为此我们需要编辑此Table中的数据:    

点击`Edit Table` 进入编辑界面，首先更改`Detail`中的 `Offset`为8,代表时区与UTC差别为8：    

![/images/2020_03_13_14_49_40_436x247.jpg](/images/2020_03_13_14_49_40_436x247.jpg)

![/images/2020_03_13_14_49_51_483x269.jpg](/images/2020_03_13_14_49_51_483x269.jpg)

而后开始编辑 `Columns`, 相关属性说明:  `groupable` 代表是否可以分组;  `filterable` 代表是否可以分类; `is temporal` 代表是否是时间参数。我们需要把date的type设置为timestamp, 并选取其 `is temporal` 属性，其他的所有字段的 `groupable` 及 `filterable` 都勾上:    

![/images/2020_03_13_14_53_24_1080x643.jpg](/images/2020_03_13_14_53_24_1080x643.jpg)

Metrics一栏暂时不做任何设置，至此数据已经清晰化，下一步进入数据分析环节.    

### 数据分析
点击 `Sources` -> `Tables` ，选中我们刚才上传的CSV文件建立的表后，进入到数据可视化编辑界面，初次进入时界面是完全空白的，我们需要在这里添加相关图表。     

#### 中国地图
点击 `Visualization Type`， 选择 `Country Map`:   

![/images/2020_03_13_14_56_14_398x333.jpg](/images/2020_03_13_14_56_14_398x333.jpg)

选择为以下值时候，运行 `Run Query`:    

![/images/2020_03_13_14_59_04_866x838.jpg](/images/2020_03_13_14_59_04_866x838.jpg)

结果如下，比如，选择甘肃，可以看到累计的确诊人数为127人：    

![/images/2020_03_13_14_59_44_872x550.jpg](/images/2020_03_13_14_59_44_872x550.jpg)

编辑完后，点击save即可保存，我们保存为`chinamap`:   

![/images/2020_03_13_15_01_10_417x324.jpg](/images/2020_03_13_15_01_10_417x324.jpg)
#### 时间线地图
建立一个类型为 `Line Charts`的图表，数据类型如下：   

![/images/2020_03_13_15_34_03_620x820.jpg](/images/2020_03_13_15_34_03_620x820.jpg)

结果如下:    

![/images/2020_03_13_15_35_25_1239x955.jpg](/images/2020_03_13_15_35_25_1239x955.jpg)

保存名称为 `Trend`.   
#### sunburst类型
数据类型如下:    

![/images/2020_03_13_15_37_55_629x795.jpg](/images/2020_03_13_15_37_55_629x795.jpg)

结果：   

![/images/2020_03_13_15_38_18_1131x822.jpg](/images/2020_03_13_15_38_18_1131x822.jpg)

保存名称为 `SunburstChina`

#### ForceDirected类型
数据定义如下:    

![/images/2020_03_13_15_41_06_620x724.jpg](/images/2020_03_13_15_41_06_620x724.jpg)

结果:    

![/images/2020_03_13_15_41_56_991x744.jpg](/images/2020_03_13_15_41_56_991x744.jpg)

保存名称为: `totalDead`.  

至此所有的单图表创建完毕.   

### Dashboard
增加一个Dashboard:    

![/images/2020_03_13_15_43_41_523x258.jpg](/images/2020_03_13_15_43_41_523x258.jpg)

点击Edit，添加对应的图表到该dashboard上即可:    

![/images/2020_03_13_16_13_23_1124x480.jpg](/images/2020_03_13_16_13_23_1124x480.jpg)

至此，可视化完毕，可以探索更多的样例做出更多的可视化效果。
