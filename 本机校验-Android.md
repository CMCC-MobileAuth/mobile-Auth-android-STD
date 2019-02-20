# 1. 开发环境配置
sdk技术问题沟通QQ群：609994083

**注意事项：**

1. **认证取号服务必须打开蜂窝数据流量并且手机操作系统给予应用蜂窝数据权限才能使用**
2. **取号请求过程需要消耗用户少量数据流量（国外漫游时可能会产生额外的费用）**
3. **认证取号服务目前仅支持中国移动2/3/4G**

## 1.1. 准备工作

在中国移动开发者社区进行以下操作：

1. 获得appid和appkey；
2. 勾选本机号码校验能力；
3. 配置应用服务器的出口ip地址
4. 配置公钥（如果使用RSA加密方式）
5. 勾选本机号码校验短验辅助开关（可选）
6. 商务对接签约（未签约应用每个appid每天只能调用1000次）

## 1.2. 总体使用流程

![](image/mobile_auth.png)

1. 调用SDK方法来获得token；

2. 携带token通过业务服务端到认证服务端的本机号码校验接口，进行号码校验</br>


## 1.3. 新建工程并导入SDK的jar文件

将`mobile_verification_android_5.1.4.jar`拷贝到应用工程的libs目录下，如没有该目录，可新建；

![](image/514-1.png)



</br>

## 1.4. 配置AndroidManifest

添加必要的权限支持: 

```java
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
```

权限说明：

| 权限                 | 说明                                       |
| -------------------- | ------------------------------------------ |
| INTERNET             | 允许应用程序联网，用于访问网关和认证服务器 |
| READ_PHONE_STATE     | 获取imsi用于判断双卡和换卡                 |
| ACCESS_WIFI_STATE    | 允许程序访问WiFi网络状态信息               |
| ACCESS_NETWORK_STATE | 获取网络状态，判断是否数据、wifi等         |
| CHANGE_NETWORK_STATE | 允许程序改变网络连接状态                   |

</br>

## 1.5. SDK使用步骤

**1. 创建一个AuthnHelper实例** 

`AuthnHelper`是SDK的功能入口，所有的接口调用都得通过`AuthnHelper`进行调用。因此，调用SDK，首先需要创建一个`AuthnHelper`实例，其代码如下：

```java
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mContext = this;    
    
    mAuthnHelper = AuthnHelper.getInstance(mContext.getApplicationContext());
    //AuthnHelper初始化
    mAuthnHelper.init(Constant.APP_ID, Constant.APP_KEY);
    //设置是否输出sdk日志
    mAuthnHelper.setDebugMode(true);
    }
```

**2. 实现回调**

所有的SDK接口调用，都会传入一个回调，用以接收SDK返回的调用结果。结果以`JsonObjent`的形式传递，`TokenListener`的实现示例代码如下：

```java
mListener = new TokenListener() {
    @Override
    public void onGetTokenComplete(JSONObject jObj) {
    	try {
               if (timer != null)
               timer.cancel();//回调的时候取消定时器
        } catch (Exception e) {
               e.printStackTrace();
        }
        if (jObj != null) {
            mResultString = jObj.toString();
            mHandler.sendEmptyMessage(RESULT);
            if (jObj.has("token")) {
                mtoken = jObj.optString("token");
            }
        }
    }
};
```
<div STYLE="page-break-after: always;"></div>

# 2. SDK方法说明

## 2.1. 获取管理类的实例对象

获取管理类的实例对象

</br>

**原型**

```java
public AuthnHelper (Context context)
```

</br>

**参数说明**

| 参数      | 类型      | 说明                              |
| ------- | ------- | ------------------------------- |
| context | Context | 调用者的上下文环境，其中activity中this即可以代表。 |

</br>

## 2.2. 获取校验凭证

请求校验凭证`token`，凭token调用本机号码校验接口。接口文档详见《移动认证服务端接口文档》</br>

**注意：获取token前，开发者需提前申请`READ_PHONE_STATE`权限，否则会失败！**

</br>

**原型**

```java
public void umcLoginByType(String appId, 
            String appKey, int setTimeOut, 
            TokenListener listener)
```

</br>

**请求参数**

| 参数       | 类型            | 说明                                       |
| :------- | :------------ | :--------------------------------------- |
| appId    | String        | 应用的AppID                                 |
| appkey   | String        | 应用密钥                                     |
| setTimeOut   | int        | 设置超时时间                               |
| listener | TokenListener | TokenListener为回调监听器，是一个java接口，需要调用者自己实现；TokenListener是接口中的认证登录token回调接口，OnGetTokenComplete是该接口中唯一的抽象方法，即void OnGetTokenComplete(JSONObject  jsonobj) |

**响应参数**

OnGetTokenComplete的参数JSONObject，含义如下：

| 参数       | 类型   | 说明                                                         |
| ---------- | ------ | ------------------------------------------------------------ |
| resultCode | String | 返回码                                                       |
| resultDesc | String | 返回描述                                                     |
| token      | String | 成功时返回：临时凭证，token有效期2min，一次有效，同一用户（手机号）10分钟内获取token且未使用的数量不超过30个 |

</br>

**请求示例代码**

```java
mAuthnHelper.umcLoginByType(Constant.APP_ID, 
        Constant.APP_KEY, 12000,
        mListener);
```

**响应示例代码**

```
{
    "resultCode": "103000",
    "resultDesc": "success",
    "token": "8484010001330200374D455979526A49354E6A59774E444D314E454E47516B4D3140687474703A2F2F3231312E3133362E31302E3133313A383038302F40303103000402D59A6B040012383030313230313730383138313031343437050010D2F28C555CB54316B7D031DE9F6F6B1EFF0020F07B4AAFC3B1499A250AAAB4272BBFB565B440FFA5C8257E90C28595956CC224"
}
```


<div STYLE="page-break-after: always;"></div>

# 3. SDK返回码说明

| 返回码 | 返回码描述                                                   |
| ------ | ------------------------------------------------------------ |
| 103000 | 成功                                                         |
| 102101 | 无网络                                                       |
| 102203 | 输入参数错误                                                 |
| 102223 | 数据解析异常                                                 |
| 102507 | 请求超时                                                     |
| 102508 | 数据网络切换失败                                             |
| 102509 | 未知错误                                                     |
| 103101 | appkey错误                                                   |
| 103102 | 包签名错误                                                   |
| 103111 | 网关IP错误（运营商误判）                                     |
| 103119 | appid不存在                                                  |
| 103414 | 参数校验异常                                                 |
| 103911 | token请求过于频繁，10分钟内获取token且未使用的数量不超过30个 |
| 105002 | 移动取号失败                                                 |
| 200002 | 获取imsi失败                                                 |
| 200010 | 未开启数据网络                                               |
| 200028 | 网络请求出错                                                 |
| 200072 | CA认证失败                                                   |
| 200075 | 仅支持移动卡                                                 |

<div STYLE="page-break-after: always;"></div>
