# 付费验证SDK开发者文档
### 流程示例请参看截图：![点我看SDK验证流程的效果图](https://github.com/boriszixue/paysdkdoc/edit/master/demo.png)
------

### 开发前需知：
#### **在接入时，我们会提供给你 测试用的 packageName和 游戏盒测试账号密码，方便你们开发阶段的测试。`务必测试`：能进入到可以玩游戏的界面，才能表明你已正确接入该SDK。**
> * **备注：测试账号仅用于方便测试，请勿对游戏盒测试账号更改资料、密码，发布无意义的、敏感的评论动态等其他操作，请遵守！！！**
------

### SDK实现说明
> * sdk的核心是一个提供验证流程的 Activity页面，这个验证流程分为三个，（以下简称流程）顺序是：
1. 检查网络是否连上并可用
2. 检查是否安装符合版本的游戏盒
3. 调用游戏盒，请求网络，检查该游戏是否已经购买
> * 在1、2、3流程都通过后（已联网、已安装符合版本的游戏盒、已购买该游戏），SDK会保存验证通过的时间戳。
> * 打开APP从未通过验证的，**不属于**“可以玩”
> * 已通过验证后，后续打开未通过验证：1、如果在`限制时间`内，属于“可以玩”。2、如果超过限制时间，**不属于**“可以玩”。
> * 何为限制时间？  答：第一次验证通过后，本地会缓存通过的状态，在限制时间内再次启动，即使没有通过验证，也能正常进入游戏。缓存限制时间过期后，需要重新验证。限制时间默认为7天，厂商可调整时间长度。
------

### API使用：
1. 依赖，sdk以aar文件存在，在libs目录添加 m4399-paysdk-1.0.4.aar文件， 在moudle下的build.gradle添加如下：
```gradle
    dependencies {
        compile(name: 'm4399-paysdk-1.0.4', ext: 'aar')
    }
```
2. 在 流程页面打开之前（**建议**在**Application**的**onCreate()**）调用SDK初始化配置：
```java
M4399GameBoxClient.getInstance().init(limitTime, packageName, listener);
```
或者调用
```java
M4399GameBoxClient.getInstance().init(packageName, listener);//默认限制时间为7天
```
> * `limitTime` 代表限制时间，单位毫秒，比如一天：``DateUtils.DAY_IN_MILLIS ``
> *  `packageName` 代表游戏的包名，在开发测试时，请填上我们提供的测试的包名，发布时，使用真实的包名
> *  `listener` 应继承自``M4399GameBoxClient.DefaultVerifyListener``
  ``DefaultVerifyListener``类的两个抽象方法说明：
  1. ``onPassInLimitTime(Activity hostActivity)`` 验证流程未通过，但是在限制之间内，会触发该方法，hostActivity是流程页面Activity。
  2. ``onVerifyPass(Activity hostActivity)`` 验证流程通过，会触发该方法。
> * 这两个方法的通用实现：进入游戏并关闭流程页面Activity。

3. 打开流程页面（一般是在闪屏页面关闭后）
```java 
    M4399GameBoxClient.getInstance().openCheckFlowPage(this);
``` 


