# 1. 开发环境配置
sdk技术问题沟通QQ群：609994083

**注意事项：**

1. **认证取号服务必须打开蜂窝数据流量并且手机操作系统给予应用蜂窝数据权限才能使用**
2. **取号请求过程需要消耗用户少量数据流量（国外漫游时可能会产生额外的费用）**
3. **认证取号服务目前仅支持中国移动2/3/4G**

## 1.1. 总体使用流程

![](image/mobile_auth.png)

1. 调用SDK方法来获得token；

2. 携带token通过业务服务端到认证服务端的本机号码校验接口，进行号码校验</br>


## 1.2. 新建工程并导入SDK的jar文件

将`mobile_verification_android_5.1.4.jar`拷贝到应用工程的libs目录下，如没有该目录，可新建；

![](image/514-1.png)



</br>

## 1.3. 配置AndroidManifest

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

## 1.4. SDK使用步骤

**1. 创建一个AuthnHelper实例** 

`AuthnHelper`是SDK的功能入口，所有的接口调用都得通过`AuthnHelper`进行调用。因此，调用SDK，首先需要创建一个`AuthnHelper`实例，其代码如下：

```java
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mContext = this;    
    
    mAuthnHelper = AuthnHelper.getInstance(mContext.getApplicationContext());
    mAuthnHelper.init(Constant.APP_ID, Constant.APP_KEY);
    
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

**3. 接口调用**

```java
mAuthnHelper.umcLoginByType(Constant.APP_ID, 
        Constant.APP_KEY, 12000,
        mListener);
```



<div STYLE="page-break-after: always;"></div>

# 2. SDK方法说明

## 2.1. 获取管理类的实例对象

### 2.1.1. 方法描述

获取管理类的实例对象

</br>

**原型**

```java
public AuthnHelper (Context context)
```

</br>

### 2.1.2. 参数说明

| 参数      | 类型      | 说明                              |
| ------- | ------- | ------------------------------- |
| context | Context | 调用者的上下文环境，其中activity中this即可以代表。 |

</br>

## 2.2. 获取校验凭证

### 2.2.1. 方法描述

请求校验凭证`token`。</br>

**注意：获取token前，开发者需提前申请`READ_PHONE_STATE`权限，否则会失败！**

</br>

**原型**

```java
public void umcLoginByType(String appId, 
            String appKey, int setTimeOut, 
            TokenListener listener)
```

</br>

### 2.2.2. 参数说明

**请求参数**

| 参数       | 类型            | 说明                                       |
| :------- | :------------ | :--------------------------------------- |
| appId    | String        | 应用的AppID                                 |
| appkey   | String        | 应用密钥                                     |
| setTimeOut   | int        | 设置超时时间                               |
| listener | TokenListener | TokenListener为回调监听器，是一个java接口，需要调用者自己实现；TokenListener是接口中的认证登录token回调接口，OnGetTokenComplete是该接口中唯一的抽象方法，即void OnGetTokenComplete(JSONObject  jsonobj) |

**响应参数**

OnGetTokenComplete的参数JSONObject，含义如下：

| 字段       | 类型   | 含义                                                         |
| ---------- | ------ | ------------------------------------------------------------ |
| resultCode | String | 返回码                                                       |
| resultDesc | String | 返回描述                                                     |
| token      | String | 成功时返回：临时凭证，token有效期2min，一次有效，同一用户（手机号）10分钟内获取token且未使用的数量不超过30个 |

</br>

### 2.2.3. 示例

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

# 3. 平台接口说明

## 3.1. 本机号码校验接口

开发者获取token后，需要将token传递到应用服务器，由应用服务器发起本机号码校验接口的调用。

校验结果有两种：1.是本机号码；2.非本机号码。对于校验结果为**2.非本机号码**的请求，开发者可以选择使用短验辅助校验功能，通过短信验证码验证用户身份（**短验辅助校验无法保证校验的号码和本机号码一致**）

调用本接口，必须保证：

1. token在有效期内（2分钟）。
2. token还未使用过。
3. 应用服务器出口IP地址在开发者社区中配置正确。

对于本机号码校验，需要注意：

1. 本产品属于收费业务，开发者未签订服务合同前，每天总调用次数有限，详情可咨询商务。
2. 签订合同后，将不在提供每天免费的测试次数。

### 3.1.1. 接口说明

**调用次数说明：**本产品属于收费业务，开发者未签订服务合同前，每天总调用次数有限，详情可咨询商务。

**请求地址：** https://www.cmpassport.com/openapi/rs/tokenValidate

**协议：** HTTPS

**请求方法：** POST+json

</br>

### 3.1.2.  参数说明

*1、json形式的报文交互必须是标准的json格式；*

*2、发送时请设置content type为 application/json*

**请求参数**

| 参数          | 层级  | 约束                         | 说明                                                         |
| ------------- | ----- | ---------------------------- | ------------------------------------------------------------ |
| **header**    | **1** | 必选                         |                                                              |
| version       | 2     | 必选                         | 版本号,初始版本号1.0,有升级后续调整                          |
| msgId         | 2     | 必选                         | 使用UUID标识请求的唯一性                                     |
| timestamp     | 2     | 必选                         | 请求消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| appId         | 2     | 必选                         | 应用ID                                                       |
| **body**      | **1** | 必选                         |                                                              |
| openType      | 2     | 否，requestertype字段为0时是 | 运营商类型：</br>1:移动;</br>2:联通;</br>3:电信;</br>0:未知  |
| requesterType | 2     | 必选                         | 请求方类型：</br>0:APP；</br>1:WAP                           |
| message       | 2     | 否                           | 接入方预留参数，该参数会透传给通知接口，此参数需urlencode编码 |
| expandParams  | 2     | 否                           | 扩展参数格式：param1=value1\|param2=value2  方式传递，参数以竖线 \| 间隔方式传递，此参数需urlencode编码。 |
| keyType       | 2     | 否                           | 手机号码加密方式：</br>0:默认phonenum采用sha256加密，sign采用HMACSHA256算法</br>1:RSA加密（暂未支持）</br>（注：keyType=1时，phonenum和sign均使用RSA，keyType不填或非1、0时按keyType=0处理） |
| phoneNum      | 2     | 必选                         | 待校验的手机号码的64位sha256值，字母大写。（手机号码 + appKey + timestamp， “+”号为合并意思）（注：建议开发者对用户输入的手机号码的格式进行校验，增加校验通过的概率） |
| token         | 2     | 必选                         | 身份标识，字符串形式的token                                  |
| sign          | 2     | 必选                         | 签名，HMACSHA256(appId + msgId + phonNum + timestamp + token + version)，输出64位大写字母 （注：“+”号为合并意思，不包含在被加密的字符串中,appkey为秘钥, 参数名做自然排序（Java是用TreeMap进行的自然排序）） |

**响应参数**

| 参数         | 层级  | 约束 | 说明                                                         |
| ------------ | ----- | :--- | :----------------------------------------------------------- |
| **header**   | **1** | 必选 |                                                              |
| msgId        | 2     | 必选 | 对应的请求消息中的msgid                                      |
| timestamp    | 2     | 必选 | 响应消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| appId        | 2     | 必选 | 应用ID                                                       |
| resultCode   | 2     | 必选 | 平台返回码                                                   |
| **body**     | **1** | 必选 |                                                              |
| resultDesc   | 2     | 必选 | 平台返回码                                                   |
| message      | 2     | 否   | 接入方预留参数，该参数会透传给通知接口，此参数需urlencode编码 |
| accessToken  | 2     | 否   | 使用短验辅助服务的凭证，当resultCode返回为001时，并且该appid在开发者社区配置了短验辅助功能时返回该参数。accessToken有效时间为5min，一次有效。 |
| expandParams | 2     | 否   | 扩展参数格式：param1=value1\|param2=value2  方式传递，参数以竖线 \| 间隔方式传递，此参数需urlencode编码。 |

</br>

### 3.1.3. 示例

**请求示例**

```
{
    "header":{
        "appId":"3000666666666",
        "timestamp":"20180104090953788",
        "version":"1.0",
        "msgId":"8ADFF305C7FCB3E1B1AECC130792FBD0"
    },
    "body":{
        "openType":"1",
        "token":"STsid0000001515028196605yc1oYNTuPlTlLT10AR3ywr2WApEq14JH",
        "sign":"227716D80112F953632E4AFBB71C987E9ABF4831ACDA5A7464E2D8F61F0A9477",
     "phoneNum":"38D19FF8CE10416A6F3048467CB6F7D57A44407CB198C6E8793FFB87FEDFA9B8",
        "requesterType":"0"
    }
}
```



**响应示例**

```
{
    "body":{
        "message":"",
        "resultDesc":"是本机号码"
    },
    "header":{
        "appId":"3000*****40",
        "msgId":"8ADFF305C7FCB3E1B1AECC130792FBD0",
        "resultCode":"000",
        "timestamp":"20180104090957277"
    }
}
```

<div STYLE="page-break-after: always;"></div>

## 3.2. 短信验证码下发接口

短验辅助功能勾选后，本机号码校验接口返回“非本机号码”时，可以凭accessToken调用本接口请求短信验证码（仅支持移动号码）

**调用注意事项：**

1. 使用该接口前，请开发者在开发者社区能力配置页面勾选上“短验辅助功能”（勾选后10分钟生效）
2. 只有在服务端返回校验成功，而且结果为“非本机号码”时，才能调用该接口
3. 本接口要求服务器IP地址白名单与本机号码校验配置的白名单相同
4. 下发频次限制：每个手机号码1次/min，10次/24 hour
5. 短验下发成功后，5min内有效

### 3.2.1. 业务流程

![](image/SMS_process.png)

### 3.2.2. 接口说明

**接口方向：**

| 接口调用方 | 接口提供方     |
| ---------- | -------------- |
| 应用服务端 | 移动认证服务端 |

**请求地址： https://www.cmpassport.com/openapi/rs/sendsmscode**

**协议：** HTTPS

**请求方法：** POST+json

</br>

### 3.2.3. 参数说明

*1、json形式的报文交互必须是标准的json格式；*

*2、发送时请设置content type为 application/json*

**请求参数：**

| 参数名        | 类型   | 参数描述                                                     |
| ------------- | ------ | :----------------------------------------------------------- |
| msgId         | String | 使用UUID标识请求的唯一性                                     |
| systemTime    | String | 请求消息发送的系统时间，北京时间，东八区时间。精确到毫秒，共17位，格式：20121227180001165。 |
| version       | String | 版本号,初始版本号1.0,有升级后续调整                          |
| requesterType | String | 合作伙伴集成类型0：APP；1：WAP                               |
| appId         | String | 应用id                                                       |
| mobileNumber  | String | 加密手机号码，AES加密，秘钥为md5（appkey）                   |
| userIp        | String | 客户端IP                                                     |
| message       | String | 接入方预留参数，该参数会透传给通知接口，此参数需urlencode编码 |
| accessToken   | String | 临时凭证，要求：失效时间默认为5分钟，可配置                  |
| expandParams  | String | 扩展参数                                                     |
| sign          | String | 签名，MD5(msgId + systemTime + version + requesterType+ appId + mobileNumber + userIp + appkey)，输出32位小写字母，（注：“+”号为合并意思，不包含在被加密的字符串中，appkey为秘钥,   参数名做自然排序（Java是用TreeMap进行的自然排序）排序后对应顺序参数值拼接做md5） |

**响应参数：**

| 参数名     | 参数类型 | 参数描述                                                     |
| ---------- | -------- | ------------------------------------------------------------ |
| msgId      | String   | 对应的请求消息中的msgId                                      |
| systemTime | String   | 消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| message    | String   | 接入方预留参数，该参数会透传给通知接口，此参数需urlencode编码 |
| resultDesc | String   | 返回码描述                                                   |
| resultCode | String   | 处理状态编码：</br>000:成功；</br>003:内部调用失败；</br>004:下发失败；</br>102:参数无效；</br>124:IP校验失败；</br>213:appId不存在；</br>302:签名校验失败；</br>303:解析参数错误；</br>999:系统错误； |

## 3.3. 短信验证码校验接口

短验辅助功能勾选后，本机号码校验接口返回“非本机号码”时，并且成功调用短信验证码下发接口成功让用户获取到验证码后，可以凭验证码调用本接口发起校验。

###3.3.1. 业务流程

![](image/SMS_process.png)



### 3.3.2. 接口说明

**接口方向：**

| 接口调用方 | 接口提供方     |
| ---------- | -------------- |
| 应用服务端 | 移动认证服务端 |

**请求地址：** https://www.cmpassport.com/openapi/rs/checksmscode

**协议：** HTTPS

**请求方法：** POST+json

</br>

### 3.3.3. 参数说明

*1、json形式的报文交互必须是标准的json格式；*

*2、发送时请设置content type为 application/json*

**请求参数：**

| 参数名        | 类型   | 参数描述                                                     |
| ------------- | ------ | ------------------------------------------------------------ |
| msgId         | String | 使用UUID标识请求的唯一性                                     |
| systemTime    | String | 请求消息发送的系统时间，北京时间，东八区时间。精确到毫秒，共17位，格式：20121227180001165。 |
| version       | String | 版本号,初始版本号1.0,有升级后续调整                          |
| requesterType | String | 合作伙伴集成类型0：APP；1：WAP                               |
| appId         | String | 应用id                                                       |
| mobileNumber  | String | 加密手机号码，AES加密，秘钥为appkey                          |
| userIp        | String | 客户端IP                                                     |
| smsCode       | String | 短信验证码，5min内有效                                       |
| message       | String | 接入方预留参数，该参数会透传给通知接口，此参数需urlencode编码 |
| sign          | String | 签名，MD5(msgId + systemTime + version + requesterType + appId + mobileNumber+ userIp + appkey)，输出32位小写字母，（注：“+”号为合并意思，不包含在被加密的字符串中，appkey为秘钥,   参数名做自然排序（Java是用TreeMap进行的自然排序）） |

**响应参数：**

| 参数名     | 参数类型 | 参数描述                                                     |
| ---------- | -------- | ------------------------------------------------------------ |
| msgId      | String   | 对应的请求消息中的msgId                                      |
| systemTime | String   | 消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| message    | String   | 接入方预留参数，该参数会透传给通知接口，此参数需urlencode编码 |
| resultCode | String   | 处理状态编码：</br>000:短信验证成功 ；</br>003:内部调用失败；</br>004:校验失败；</br>005:验证码已失效，请重新获取；</br>102:参数无效；</br>124:IP校验失败；</br>213:appId不存在；</br>302:签名校验失败；</br>303:解析参数错误；</br>999:系统错误； |

# 4. 平台返回码说明

## 4.1. SDK返回码说明

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

</br>

## 4.2. 本机号码校验接口返回码

本返回码表仅针对`本机号码校验接口`使用

| 返回码 | 说明                                         |
| ------ | -------------------------------------------- |
| 000    | 是本机号码（纳入计费次数）                   |
| 001    | 非本机号码（纳入计费次数，允许使用短验辅助） |
| 002    | 取号失败                                     |
| 003    | 调用内部token校验接口失败                    |
| 004    | 加密手机号码错误                             |
| 102    | 参数无效                                     |
| 124    | 白名单校验失败                               |
| 302    | sign校验失败                                 |
| 303    | 参数解析错误                                 |
| 606    | 验证Token失败                                |
| 999    | 系统异常                                     |
| 102315 | 次数已用完                                   |



## 4.3. 短信验证码下发接口

| 返回码 | 返回码描述               |
| ------ | ------------------------ |
| 000    | 成功                     |
| 003    | 内部调用失败             |
| 004    | 下发失败                 |
| 102    | 参数无效                 |
| 124    | IP 校验失败              |
| 125    | 未有使用移动认证短信能力 |
| 213    | appId不存在              |
| 302    | 签名校验失败             |
| 303    | 解析参数错误             |
| 999    | 系统错误                 |

## 4.4. 短信验证码校验接口

| 返回码 | 返回码描述               |
| ------ | ------------------------ |
| 000    | 短信验证成功。           |
| 003    | 内部调用失败             |
| 004    | 校验失败                 |
| 005    | 验证码已失效，请重新获取 |
| 102    | 参数无效                 |
| 124    | IP 校验失败              |
| 125    | 未有使用移动认证短信能力 |
| 213    | appId不存在              |
| 302    | 签名校验失败             |
| 303    | 解析参数错误             |
| 999    | 系统错误                 |



# 5. Q&A常见问题

I、有关json形式报文发送为什么报参数解析错误
答:①json形式的报文交互必须是标准的json格式；②发送时请设置content type为 application/json

<div STYLE="page-break-after: always;"></div>
