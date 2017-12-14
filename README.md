# 付费验证SDK开发者文档
### 流程示例请参看截图：![点我看SDK验证流程的效果图](https://github.com/boriszixue/paysdkdoc/edit/master/demo.png)
------

### 开发前需知：
#### **在接入时，我们会提供给你们 测试用的gameID和 游戏盒测试账号密码，方便你们开发阶段的测试。`务必测试`：能进入到可以玩游戏的界面，才能表明你已正确接入该SDK。在接入完成，测试没问题之后，你需要替换上游戏在4399平台上的真实gameID。**
> * **备注：请勿对游戏盒测试账号更改资料、密码、发布无意义的、敏感的评论动态等，请遵守！！！**
------

### SDK实现说明
> * sdk的核心是一个提供验证流程的Fragment（`CheckFragment类`），这个验证流程（以下简称流程）顺序是：
1. 检查网络是否连上并可用
2. 检查是否安装符合版本的游戏盒
3. 调用游戏盒，请求网络，检查该游戏是否已经购买
4. 在1、2、3流程都通过后（已联网、已安装符合版本的游戏盒、已购买该游戏），SDK会保存验证通过的时间戳，APP需配置 `流程未通过情况下`“可以玩”的`限制时间`，单位毫秒。
> * 打开APP从未通过验证的，**不属于**“可以玩”
> * 已通过验证后，后续打开未通过验证：1、如果在`限制时间`内，属于“可以玩”。2、如果超过限制时间，**不属于**“可以玩”。（备注:限制时间对于上述时间戳来讲）
> * 流程开始的时机点是：``CheckFragment`` 的 onResume()方法
------

### API使用：
1. 依赖，sdk以aar文件存在，在libs目录添加 m4399-paysdk-1.0.aar文件， 在moudle下的build.gradle添加如下：
```gradle
    dependencies {
        compile(name: 'm4399-paysdk-1.0.1', ext: 'aar')
    }
```
2. 在 `CheckFragment`被展示之前（**建议**在**Application**的**onCreate()**）调用SDK初始化配置：
```java
M4399GameBoxClient.getInstance().init(limitTime, gameID, listener);
```
> *  `limitTime` 代表限制时间，单位毫秒，比如一天：``DateUtils.DAY_IN_MILLIS ``
> *  `gameID` 代表在4399平台上的游戏id
> *  `listener` 应继承自``M4399GameBoxClient.DefaultVerifyListener``
3. ``DefaultVerifyListener``类的两个抽象方法说明：
> * ``onPassInLimitTime(Activity hostActivity)`` 验证流程未通过，但是在限制之间内，会触发该方法，hostActivity是宿主Activity。
> * ``onVerifyPass(Activity hostActivity)`` 验证流程通过，会触发该方法。
> * 这两个方法的通用实现：进入游戏并关闭宿主Activity。
------

### 使用说明及注意点：

> * 流程的UI是以Fragment封装起来，所以接入的APP需要提供Activity（在此称为：宿主Activity）来显示Fragment。
> * 建议宿主Activity固定横屏或竖屏，以免Activity生命周期重建对Fragment的影响，比如：触发多次网络请求，若不配置，请自行处理影响。
> * 第3个流程中，sdk通过调用游戏盒页面（透明页）向服务器查询，为了提升视觉体验及UI无感知（即用户无感知此流程中游戏盒被调用），**宿主Activity** 建议设置成全屏。代码调用如下:
```java
    @Override
    protected void onCreate(Bundle savedInstanceState)
    {
       //不显示程序的标题栏
        requestWindowFeature(Window.FEATURE_NO_TITLE);
       //不显示系统的标题栏
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, WindowManager.LayoutParams.FLAG_FULLSCREEN);
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
```
> * 验证流程展示,代码如下：
```java
    FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
    CheckFragment fragment = new CheckFragment();
    transaction.add(R.id.container, fragment);
    transaction.commit();
```
> * **验证流程的视图区域是宿主Activity页面中的一块区域，为了提升宿主页面的UI 体验，不要保持页面其他区域空白，请让UI设计，给页面加点其他内容，比如文字提示，或者页面设置背景图，或者引用闪屏页的4399Logo等等。**
