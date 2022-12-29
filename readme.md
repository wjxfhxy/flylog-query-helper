# 全链路日志查询系统使用手册 

#### v22.12.19

此教程只适用于华为云版本（即当前使用版本），旧有阿里云版本不再维护，现有阿里云日志会同步投递到华为云

## 支持终端

支持玄讯App端、服务端、IDE端、Web端日志查询及统计分析

App端包括智慧100和快销100

服务端须另外部署日志服务，方能被查询系统查询

IDE端及Web端需升级到对应的日志版本

即信的iPush、即验因没有接入华为云，故只能使用阿里云版本

## 使用

根据登陆帐号所赋予的权限不同，用户可以对App、Web、IDE和服务端日志进行查询

在对场景使用介绍前，有必要先知道三个跟搜索相关的事情

### 模糊搜索

在阿里云版本中，模糊搜索只能使用以下规范
```
达利*：匹配以“达利”开头的值
```
在华为云版本中，模糊搜索能使用更为丰富的匹配规则，同时，匹配字符也改为%
```
提示%：匹配以“提示”开头的值
%提示%：匹配含有“提示”的值
提示%返回%：匹配以“提示”开头，含有“返回”值
%是否%返回%：匹配含有“提示”、“返回”值
```

### 特殊字符

在阿里云版本中，搜索值中含有特殊字符时，需要使用英文双引号包裹
```
"http://"：":"为特殊字符
```
在华为云中没有该限制

### 搜索时间范围

日志时间分为：入库时间和记录时间，一般情况下，入库时间与记录时间接近

<img src="https://raw.githubusercontent.com/wjxfhxy/flylog-query-helper/main/07b63ba035d0ed2a3e72a6b8c1f2555.png" width=50%>


需要注意的是

受华为云性能影响，搜索5天、7天内的日志，容易出现超时错误

查询系统的每一次请求，都隐含入库时间作为查询条件

### App

因目前App有较为丰富的场景划分，故以下内容都以App介绍为样例，其他端都可参考，有特殊的将作另外说明

### 场景划分

App下目前划分为五个场景
```
异常信息：弹窗报错、内部异常

网络：HTTP、OSS

界面：表单、页面、页面拓扑

链路：链路查询

自定义：自定义查询分析
```

### 弹窗报错

根据界面提示，定位背后真正的错误

如：flycode执行失败、事件执行失败、网络请求失败……

下图为搜索达利项目，出现网络错误提示的，李姓用户，且App版本为前缀为6.0.3.2022081902

<img src="https://raw.githubusercontent.com/wjxfhxy/flylog-query-helper/main/09fdf887940a121f0c29ef49af6526a.png" width=100%>

### 内部异常

与弹窗报错的区别：弹窗报错一定出现内部异常，但内部异常不一定出现弹窗报错

如：发送队列，OSS图片加载等

需要注意的是，iOS不支持内部异常

以下，列出Android中常见的内部异常

```
//未分类异常
java.lang.Exception
//SQL语句执行异常
android.database.sqlite.SQLiteException
//阿里OSS传输异常
com.alibaba.sdk.android.oss.ServiceException
//UIFlyCode执行错误
com.xuanwu.apaas.engine.uiflycode.UIFlyCodeBusinessException
//500业务错误
com.xuanwu.apaas.servicese.bizhttp.exception.BizHttpBusinessException
//网络请求超时
com.xuanwu.apaas.servicese.bizhttp.exception.BizHttpTimeoutException
//DNS解析错误
com.xuanwu.apaas.servicese.bizhttp.exception.BizHttpDNSException
//网络连接异常
com.xuanwu.apaas.servicese.bizhttp.exception.BizHttpConnectException
//发送队列发送异常
com.xuanwu.apaas.sendqueue.SendQueueException
//阿里OSS连接建立失败
com.alibaba.sdk.android.oss.ClientException
//网络请求异常
com.xuanwu.apaas.servicese.bizhttp.exception.BizHttpRequestException
//OSS请求异常
com.xuanwu.apaas.servicese.storageservice.exception.OSSCommonException
//事件执行异常
com.xuanwu.apaas.engine.bizuiengine.exception.ActionHandlerException
//SQL语句执行异常
android.database.sqlite.SQLiteConstraintException
//登录失败
com.xuanwu.apaas.servicese.loginmodule.exception.LoginOperationException
//登录失败
com.xuanwu.apaas.servicese.loginmodule.exception.LoginBizHttpException
```

下图为搜索Android，出现BizHttpTimeoutException超时的日志

需要注意，BizHttpTimeoutException前加上"%"进行后缀匹配

<img src="https://raw.githubusercontent.com/wjxfhxy/flylog-query-helper/main/42f201f483ba4e1e3dbd3d2eb8bc037.png" width=100%>

### HTTP

查询业务接口执行情况

状态码分为两类：网络状态码和HTTP状态码

网络状态码统一为-1，HTTP状态码与RFC 2616规范一致

-1时需通过详情查看具体的网络错误

<img src="https://raw.githubusercontent.com/wjxfhxy/flylog-query-helper/main/9ed3a96fd9c56911e17e0226dd34c4c.png" width=100%>

下图为搜索大版本号为9.2.7，请求ip为39.108.92.63:7000，出现网络异常的日志

<img src="https://raw.githubusercontent.com/wjxfhxy/flylog-query-helper/main/ed8e9e7672ba8a22fdefdd47495924d.png" width=100%>

### OSS

查询oss、obs、minio的执行情况

与HTTP一致的使用

部分OSS的http请求地址，经HTTPDNS解析后，使用IP请求，无法按原http请求地址查找

### 表单

以表单的视角查询表单所发生的一切

两种查看方式：

树形：展示与表单直接关联的日志，同时，表单配置事件能通过trace方式展示，便于查看事件的执行链路的情况，解决事件与行为的执行顺序，明确耗时情况，展示错误链路

<img src="https://raw.githubusercontent.com/wjxfhxy/flylog-query-helper/main/78720d0090c0077f6a3f11623421b3c.png" width=100%>

列表：查看在表单生命周期时间内的所有日志，适用于由表单产生，但没有与表单直接关联的情况，比如图片裂图，或者在生命周期内，其他功能产生的日志，比如子表单产生的日志、后台定位、相机开启关闭

<img src="https://raw.githubusercontent.com/wjxfhxy/flylog-query-helper/main/9a445ba6afa7bb3e4fedb071839e78c.png" width=100%>

### 页面

与表单类似，除能查看配置表单的日志，还能查看由页面所承载或触发的行为的日志，比如离线数据更新

下图为搜索Android/离线数据更新（iOS/登陆）展示的情况

<img src="https://raw.githubusercontent.com/wjxfhxy/flylog-query-helper/main/945b6c12c1ff441b512dddff5e5aa16.png" width=100%>

日志中虽没有记录离线数据所下载的数据，都有记录请求时的url及参数，可通过这些参数在IDE端复查下载的数据

### 页面拓扑

把孤立的页面串联在一起，从整体上查看表单之间的关系，还原业务的流程走向，解决拜访流表单乱窜、页面流转不清晰的问题

下图为搜索"我的拜访"页面，并按每一级展开，可以看到日志展示了该用户在"终端客户详情"中，先后两次进入"拜访步骤列表"

<img src="https://raw.githubusercontent.com/wjxfhxy/flylog-query-helper/main/972e14b24c9673da41f8d1142eaa923.png" width=100%>

### 链路查询

链路查询把系统中所有trace日志进行查找，其展示及作用与页面/表单所展示的trace日志一致

链路查询一般利用traceid进行查找，traceid又是通过自定义查询分析得到

如果知道trace链路的名字，则可以直接利用

### 自定义查询分析

自定义查询使用使用DSL或ClickHouse SQL进行查询，查询的结果可以导出，或通过内置的图表功能，生成简单的报表

### DSL

DSL为ClickHouse SQL（CK SQL）去掉非必要的关键字，只保留查询条件而成的查询语法

如
```sql
select clientver from log where client='Android' or client='iOS'
```
只保留查询条件

```sql
client='Android' or client='iOS'
```

由于它没有指定返回字段，故每条日志都为全字段返回

理论上，它支持所有条件关键字及运算符号，和标准的CK函数，如数值转换、日期处理、JSON提取等等，但不支持聚合统计、排序关键字，也不支持嵌套子查询作为查询条件

查询系统支持以下字段，作为查询条件key

| key |类型（默认为字符串）| 说明|
|---|---|---|
|types | | 日志类型
|logtime | int64 | 16位日志记录时间戳
|epochnanos  | | 19位日至记录时间戳
|receive_time  | int64 | 10位日志入库时间戳
|installid  | | 安装部署id
|anonymousid  | | 匿名id
|terminalid  | | 终端id
|appid  | | 应用id
|systemver  | | 系统版本
|client  | | 客户端类型
|clientver  | | 客户端版本
|devicever  | | 设备类型
|manufacturer  | | 设备制造商
|envname  | | 租户部署环境
|tenantcode  | | 租户编码
|tenantname  | | 租户名
|mbcode  | | mbcode
|positionname  | | 岗位名
|accountcode  | | accountcode
|userinfoname  | | 用户名
|username  | | 用户名、电话号码
|properties  | json | 日志的记录的详细内容
|properties.traceid  | | 链路traceid
|properties.groupid  | |
|properties.spanid  | | 链路块id
|properties.parentspanid  | | 链路块父id
|properties.name  | | 链路名字
|properties.pagename  | | 表单名、页面名
|properties.displaymessage  | | 弹窗显示的信息
|properties.code  | | http、oss的code
|properties.method  | | http、oss的method
|properties.url  | | http、oss的url
|properties.latitude  | 小数 | 定位的纬度
|properties.longitude  |  小数 | 定位的经度
|properties.address  | | 定位的地址
|properties.attributes.pagename  | | 链路中表单名、页面名
|properties.attributes.message  | |
|properties.cause.name | | 引起异常的exception名字

properties

properties在库中以字符串形式存储，故能通过like关键字进行模糊查找

如
```sql
//查找包含"code":200的http日志
types='http' and properties like '%\"code\":200%'
```
也能通过JSON函数对properties进行解析后查找
```sql
types='http' and JSONExtractString(properties,'code') = '200'
```
查询系统对JSON函数进行了简化，通过properties.code代替
```sql
types='http' and properties.code = '200'
```

### CK SQL

查询系统有限度的支持使用SQL作为查询语句

目前明确了以下限制

```
禁止使用"*"通配符号作为查询输出
查询日志表只能是"log"
```

### 查询分析语句

查询分析语句用于在查询的结果上，对结果进行统计，需严格按照以下格式

```
DSL | CK SQL
```

以"|"分割开，前面为查询DSL，后面为CK SQL

看下面的例子

```sql
//前面DSL限定了查询的数据集合
//后面CKSQL对查询集合进行统计
types='http' and properties.code='200' | select count() from log group by envname
```

```sql
//使用"*"统配符，不限定查询的数据集合
//后面CKSQL对查询集合进行统计
* | select count() from log group by envname
```

统计结果默认以表格显示，可根据需要切换为折线图、柱图、饼图展示

看下面的例子

```sql
//统计客户端数量
* | select client, count() from log group by client
```

<img src="https://raw.githubusercontent.com/wjxfhxy/flylog-query-helper/main/a5f0a6c08ff850896e278a0123125c3.png" width=100%>

切换为折线图显示

切换后需要设置X轴Y轴字段

<img src="https://raw.githubusercontent.com/wjxfhxy/flylog-query-helper/main/a88da85a01605147c6f4e1ee8350f29.png" width=100%>

切换为柱图显示

切换后需要设置X轴Y轴字段，也可以修改柱的方向

<img src="https://raw.githubusercontent.com/wjxfhxy/flylog-query-helper/main/3df1b18392f30262d637f7ad2e3075c.png" width=100%>

切换为饼图显示

切换后需要设置分类和数值列字段

<img src="https://raw.githubusercontent.com/wjxfhxy/flylog-query-helper/main/cf087208be2c6794f8ec677315c0ed7.png" width=100%>

### 日志导出

日志导出用于下载当前查询结果集合，其分为导出历史、默认导出和自定义导出

### 导出历史

导出历史可以对当前浏览器三天内导出过的日志重复下载

### 默认导出

默认导出每次最大导出1000条日志信息，每条日志的时间都被格式化为yyyy-MM-dd HH:mm:ss，如果导出的日志类型types是相同，则properties会尝试被解开

### 自定义导出

自定义导出就是对默认导出的导出设置进行修改


### Server

### 从服务端反查前端

在Server下选择自定义查询分析

利用DSL查询来自前端的业务请求

```sql
//这是举例，还可以加入其他条件
properties like '%\'ctcode: 2,3\'%' and clientver like 'dynamic-bizengine%' and tenantcode='8009999' and username='keykj'

//ctcode代表不同的前端
//Android
properties like '%\'ctcode: 2,3\'%'
//iOS
properties like '%\'ctcode: 2\'%'
```

找到对应日志中的traceid，复制到Server下的链路查询，或者复制到对应前端的链路查询，这里继续以Server为例

<img src="https://raw.githubusercontent.com/wjxfhxy/flylog-query-helper/main/54524f0597b60576886f9f9c55d41a9.png" width=100%>

需要注意"自定义查询分析"与"链路查询"两个模块的入库时间需要选择一致

从查询结果中，选择名字为http，进入详情

点http旁边的小箭头，系统继续查询前端部分的日志，最终把前后端调用链路串联起来

<img src="https://raw.githubusercontent.com/wjxfhxy/flylog-query-helper/main/83eac2d49a9492b20727425142db76f.png" width=100%>

<img src="https://raw.githubusercontent.com/wjxfhxy/flylog-query-helper/main/ecbc6a29ffa669268f6ff2f64007aab.png" width=100%>

需要注意的是，因为前后端的时间基点并不完全一致，故串联后的服务端trace进度与单独自身的trace进度有所出入

### IDE

### Web